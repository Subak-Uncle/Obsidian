
## PR Workflow
```yml

name: PR Build and Test On Prod

on:
  pull_request:
    branches: [ "prod" ]

permissions:
  contents: read

jobs:
  build:

    runs-on: ubuntu-latest
    environment: prod

    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      ## MySQL 환경을 테스트 환경에서 설정합니다.
      - name: Setup MySQL
        uses: samin/mysql-action@v1
        with:
          character set server: 'utf8'
          mysql database: 'runninghi'
          mysql user: ${{ secrets.MYSQL_USERNAME }}
          mysql password: ${{ secrets.MYSQL_PASSWORD }}

      ## resources 디렉토리 확인 및 생성, application.yml 설정
      - name: Setup application.yml
        run: |
          mkdir -p ./src/main/resources
          echo "${{ secrets.APPLICATION }}" > ./src/main/resources/application.yml
        shell: bash

      - name: Setup application-prod.yml
        run: echo "${{ secrets.PROD_APPLICATION_YML }}" > ./src/main/resources/application-prod.yml

      ## firebase sdk 설정
      - name: create-json
        id: create-json
        uses: jsdaniell/create-json@v1.2.3
        with:
          name: "runninghi-firebase-adminsdk.json"
          json: ${{ secrets.FIREBASE_SDK }}

      ## firebase 디렉토리 생성 및 JSON 파일 이동
      - name: Setup Firebase Directory and Move JSON
        run: |
          mkdir -p ./src/main/resources/firebase
          mv ./runninghi-firebase-adminsdk.json ./src/main/resources/firebase/

      ## apple 설정
      - name: apple setting
        run: |
          mkdir -p ./src/main/resources/apple
          touch ./src/main/resources/apple/Apple_AuthKey.p8
          echo "${{ secrets.APPLE_AUTHKEY }}" > ./src/main/resources/apple/Apple_AuthKey.p8

      - name: Add permission to make gradlew executable
        run: chmod +x gradlew

      - name: Test with Gradle
        run: ./gradlew --info test
        env:
          SPRING_PROFILES_ACTIVE: prod

      - name: Build with Gradle
        uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb98943c375e1
        with:
          arguments: build -Dspring.profiles.active=prod

      ## docker upload image
      - name: docker image build
        run: docker build -f Dockerfile-prod -t ${{ secrets.DOCKERHUB_USERNAME }}/runninghi:${{ github.sha }} .


      # DockerHub Login
      - name: docker login
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # Docker Hub push
      - name: docker Hub push
        run: |
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/runninghi:${{ github.sha }}
          docker tag ${{ secrets.DOCKERHUB_USERNAME }}/runninghi:${{ github.sha }} ${{ secrets.DOCKERHUB_USERNAME }}/runninghi:latest
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/runninghi:latest

```

## Merge Workflow
```yml
name: Deploy Prod Branch On Merge

on:
  push:
    branches: [ "prod" ]

permissions:
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: prod

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      # 서버에 통신하기 위한 SSH를 설정합니다.
      - name: Setup SSH
        uses: appleboy/ssh-action@v0.1.8
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.SERVER_SSH_PORT }}
          script: |
            # 배포 서버에 host 키 확인
            sudo mkdir -p ~/.ssh
            sudo ssh-keyscan ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts
            
            # 배포 서버에서 DockerHub Login
            echo ${{ secrets.DOCKERHUB_TOKEN }} | sudo docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin
            
            # Pull Docker Image
            sudo docker pull ${{ secrets.DOCKERHUB_USERNAME }}/runninghi:latest
            
            # Stop and Remove existing contrainer
            sudo docker stop runninghi || true && sudo docker rm runninghi || true
            
            # 포트 사용중일 시에 종료 처리
            sudo fuser -k ${{ secrets.SERVER_PORT }}/tcp || true
            
            # Run new container
            sudo docker run -d --name runninghi --network myapp_network --ip ${{ secrets.DOCKER_IP }} -v /home/runninghi/prod/logs:/app/logs ${{ secrets.DOCKERHUB_USERNAME }}/runninghi:latest

```

## Dockerfile
```Dockerfile
# JDK 17 기반 이미지 사용
FROM openjdk:17-jdk

# 작업 디렉토리 설정
WORKDIR /app

# 로그 디렉토리 생성
RUN mkdir -p /app/logs

# JAR 파일 빌드 아티팩트 설정
ARG JAR_FILE=build/libs/*.jar

# JAR 파일 복사
COPY ${JAR_FILE} app.jar

# 로그 파일에 대한 권한 설정
RUN touch /app/logs/runninghi.out && \
    chmod 666 /app/logs/runninghi.out

# 애플리케이션 실행 및 로그 출력 설정
CMD ["sh", "-c", "java -Dspring.profiles.active=prod -jar app.jar | tee /app/logs/runninghi.out"]

```
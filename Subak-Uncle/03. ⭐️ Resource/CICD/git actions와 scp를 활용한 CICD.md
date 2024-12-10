
```bin
name: Deploy Develop Server On Merge

on:
  push:
    branches:
      - develop

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    environment: develop
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
          mysql database: ${{ secrets.MYSQL_DATABASE }}
          mysql user: ${{ secrets.MYSQL_USERNAME }}
          mysql password: ${{ secrets.MYSQL_PASSWORD }}

      ## resources 디렉토리 확인 및 생성, application.yml 설정
      - name: Setup application.yml
        run: |
          mkdir -p ./src/main/resources
          echo "${{ secrets.APPLICATION_YML }}" > ./src/main/resources/application.yml

      - name: Setup application-dev.yml
        run: echo "${{ secrets.DEV_APPLICATION_YML }}" > ./src/main/resources/application-dev.yml

      - name: Add permission to make gradlew executable
        run: chmod +x gradlew

      - name: Build with Gradle
        uses: gradle/gradle-build-action@67421db6bd0bf253fb4bd25b31ebb98943c375e1
        with:
          arguments: build

      - name: List build directory
        run: ls -R ./build/libs

      # 배포 서버에 host 키 확인
      - name: Add remote server to known hosts
        run: |
          mkdir -p ~/.ssh
          ssh-keyscan -p ${{ secrets.SERVER_SSH_PORT }} ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts

      # 서버에 jar파일을 보내기 위해 scp 명령어를 사용합니다.
      - name: SCP transfer
        run: |
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > /tmp/private_key
          chmod 400 /tmp/private_key
          scp -i /tmp/private_key -P ${{ secrets.SERVER_SSH_PORT }} ./build/libs/*.jar ${{ secrets.SERVER_USERNAME }}@${{ secrets.SERVER_HOST }}:/home/ubuntu/develop/

      # 서버에 통신하기 위한 SSH를 설정합니다.
      - name: Setup SSH and Deploy
        uses: appleboy/ssh-action@v0.1.8
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.SERVER_SSH_PORT }}
          script: |
            # 서버 원격으로 명령어를 실행합니다. 서버 포트를 확인하고 실행중이면 종료합니다. 빌드된 jar파일을 실행합니다.
            sudo fuser -k ${{ secrets.SERVER_PORT }}/tcp || true;
            sudo mkdir -p /home/ubuntu/develop/logs
            cd /home/ubuntu/develop
            sudo nohup java -jar -Dspring.profiles.active=dev ./*.jar > /home/ubuntu/develop/logs/adregamdi-develop.out 2>&1 &

            # 프로세스가 실행 중인지 확인
            sleep 30
            if ! pgrep -f "java -jar" > /dev/null; then
              echo "Application failed to start"
              exit 1
            fi

            # 로그에서 오류 확인
            if grep -i "error" /home/ubuntu/develop/logs/adregamdi-develop.out; then
              echo "Errors found in application log"
              cat /home/ubuntu/develop/logs/adregamdi-develop.out
              exit 1
            fi

            echo "Application started successfully"
            exit 0
```
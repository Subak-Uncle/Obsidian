
```bash
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource 
```

어느날 스프링 프로젝트에서 `런타임` 시점에서 위와 같은 에러가 발생했습니다.
잘 되다가 발생한 일이라 당황하였는데요,,, 해결방법은 너무나 간단합니다.

## 캐시 정리
```bash
./gradlew clean build
```

## 종속성 충돌 확인
```bash
./gradlew dependencies
```
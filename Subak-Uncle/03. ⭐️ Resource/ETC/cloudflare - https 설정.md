### 1. Cloudflare 계정 생성
```
1. https://dash.cloudflare.com/sign-up 접속
2. 이메일과 비밀번호로 회원가입
```


### 2. 도메인 추가
```
1. Cloudflare 대시보드에서 "Add site" 클릭
2. 도메인 이름 입력 (예: yourdomain.com)
3. Free 플랜 선택
4. DNS 레코드 확인 (기존 DNS 레코드 자동 스캔)
```


### 3. 네임서버 변경

```
1. 도메인 등록 대행사(가비아, 후이즈 등) 사이트 로그인
2. 도메인 관리 → 네임서버 설정
3. Cloudflare가 제공하는 네임서버로 변경
   예시:
   - nina.ns.cloudflare.com
   - rick.ns.cloudflare.com
4. 변경사항 적용까지 최대 24시간 소요
```

### 4. DNS 설정
```
1. Cloudflare 대시보드 → DNS 섹션
2. "Add record" 클릭
3. 다음과 같이 설정:
   - Type: CNAME
   - Name: @ 또는 www
   - Target: your-eb-url.region.elasticbeanstalk.com
   - Proxy status: Proxied (주황색 구름 아이콘)
```


### 5. SSL/TLS 설정
```
1. SSL/TLS 섹션으로 이동
2. SSL/TLS 암호화 모드를 "Full" 선택
3. Edge Certificates:
   - Always Use HTTPS: On
   - Minimum TLS Version: 1.2
```


### 6. Page Rules 설정
```
1. Rules → Page Rules
2. Create Page Rule 클릭
3. 다음 규칙 추가:
   URL: http://www.yourdomain.com/*
   설정:
   - Always Use HTTPS
   - Cache Level: Standard
```


### 7. Security 설정
```
1. Security → Settings
2. Security Level: Medium
3. Browser Integrity Check: On
```


### 8.  Speed 최적화 (선택사항)
```
1. Speed → Optimization
2. Auto Minify: CSS, JavaScript, HTML 체크
3. Brotli: On
4. Rocket Loader: On
실제 적용 예시:
Copy1. Elastic Beanstalk URL이 
   myapp.ap-northeast-2.elasticbeanstalk.com 인 경우

```

### 적용 예시
```
2. DNS Records:
   Type   Name   Target                                         Proxy Status
   CNAME  www    myapp.ap-northeast-2.elasticbeanstalk.com     Proxied
   CNAME  @      myapp.ap-northeast-2.elasticbeanstalk.com     Proxied

3. Page Rules:
   URL Pattern                    Setting
   *yourdomain.com/*             Always Use HTTPS
```

### 주의사항:
1. Proxy Status는 반드시 "Proxied" (주황색 구름)으로 설정
2. SSL/TLS 모드는 "Full"로 설정
3. 네임서버 변경 후 적용까지 시간이 걸림
4. 처음에는 SSL 인증서 발급에 약간의 시간 소요

### 모니터링:
1. Analytics 섹션에서 트래픽 모니터링
2. Security 섹션에서 위협 모니터링
3. SSL/TLS 섹션에서 암호화 상태 확인

### 이렇게 설정하면:

- 무료 SSL 인증서
- DDoS 보호
- CDN 캐싱
- 기본적인 보안 설정
- 성능 최적화

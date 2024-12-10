
### CertBot + Nginx를 이용한 설정

오늘은 온프레미스 환경에서 HTTPS 설정을 위한 SSL 인증서 발급을 하는 과정에 대해서 정리해보겠습니다. 다양한 방법이 있겠습니다만, 간편하게 무료로 ssl 인증서 발급이 가능한 `Certbot` 을 이용하는 방법으로 구현해보겠습니다.

참고로, `cloudflare` 를 사용하면 중간 과정을 생략할 수 있고 관리도 쉽다고 들었습니다. 하지만, 언제나 처음엔 직접 세팅을 경험하는 것이 도움이 될 것이라 생각되어 직접 주어진 과제를 통과하여 ssl 문서를 발급받겠습니다.

#### Certbot, Nginx 설치
```bash
sudo apt update
sudo apt install certbot pthon3-cerbot-nginx
```


#### 인증서 발급

##### Nginx를 수동으로 구성하는 방법
Certbot의 Nginx 플러그인을 사용하면 Nginx 설정을 자동으로 수정하고 인증서를 발급 받을 수 있습니다.
```bash
sudo certbot --nginx -d {yourdomain.com} -d {www.yourdomain.com}
```

하지만, 직접 세팅해보겠습니다.

###### 1. Nginx 설정 파일 구성: /etc/nginx/sites-available/default 또는 도메인 별 Nginx 설정 파일을 열어 다음 내용을 추가합니다:


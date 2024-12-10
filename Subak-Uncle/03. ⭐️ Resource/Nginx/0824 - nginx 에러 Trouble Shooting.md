
0824일 오전 6시... nginx가 자체적으로 서버를 종료시켰습니다.
관련 로그를 확인하니 다음과 같았습니다.

## Error Log
```bash
// -u는 특정 유닛 한정
journalctl -u nginx.service
```
```bash
Aug 24 06:07:35 runninghi systemd[1]: Stopping nginx.service - A high performance web server and a reverse proxy server...
Aug 24 06:07:35 runninghi nginx[639792]: 2024/08/24 06:07:35 [emerg] 639792@639792: host not found in upstream "localhost" in /etc/nginx/sites-enabled/default:40
Aug 24 06:07:35 runninghi nginx[639792]: nginx: configuration file /etc/nginx/nginx.conf test failed
Aug 24 17:48:34 runninghi systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Aug 24 17:48:34 runninghi systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
```

로그를 확인해보니,,,,
nginx 설정 파일에서 `Line 40` 에서 `localhost` 설정이 문제가 된 것으로 파악이 되었습니다.

## Nginx 설정
```bash
server {
        listen 80 default_server;
        listen [::]:80 default_server;


        server_name _;
        return 301 https://$host$request_uri;
}

server{
        listen [::]:443 ssl;
        listen 443 ssl;
        server_name www.runninghi.store runninghi.store;
    ssl_certificate /etc/letsencrypt/live/runninghi.store-0001/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/runninghi.store-0001/privkey.pem; # managed by Certbot


			
        location / {  **루트 경로("/")로 들어오는 모든 요청을 처리**
                proxy_pass http://localhost:8080/;
                **모든 요청을 localhost의 8080 포트로 전달**
                proxy_http_version 1.1;
                **HTTP 1.1 버전을 사용하도록 설정**  
                proxy_set_header Upgrade $http_upgrade; 
                **클라이언트가 연결 업그레이드를 요청할 때 이 헤더를 전달**
                proxy_set_header Host $host;
                **원래 요청의 Host 헤더를 프록시 서버로 전달**
                proxy_cache_bypass $http_upgrade;
                **연결 업그레이드 요청(예: WebSocket)의 경우 캐시를 우회**
        }

        location ~ /\.well-known/acme-challenge/ { 
        ** '~' 기호는 정규표현식 매칭을 의미, Lets Encrypt가 도메인 소유권을 확인하기 위해 사용하는 경로**
                default_type "text/plain";
                root /var/www/letsencrypt;
        }
}
```

## 에러 원인
원인은 다음과 같습니다.
저희 프로젝트는 Docker Container 환경에서 실행되고 있습니다.
Docker는 가상 네트워크 환경을 사용하며, 각 컨테이너는 자체적인 네트워크 네임스페이스를 가집니다. 
`localhost`는 각 **컨테이너 내부**를 가리키므로, nginx 설정의 `localhost`는 **nginx 컨테이너 자체**를 가리키게 됩니다. 

따라서 **Spring Server**가 실행 중인 다른 컨테이너에 접근하기 위해서는 **Docker Host IP**나 **컨테이너 이름**을 사용해야 합니다. 

## 에러 해결
nginx 설정 파일(**/etc/nginx/sites-avaliable/default**)을 변경합니다.

우선, **Docker Host IP**를 먼저 검색합니다.
```bash
ip addr show docker0

>> 172.17.0.1/16
```

해당 IP를 nginx 설정 파일에서 **upstream**에 작성해줍니다.
```bash
upstream backend {
    server 172.17.0.1:8080;  # 도커 호스트 IP와 매핑된 포트
}

server {
        listen 80 default_server;
        listen [::]:80 default_server;


        server_name _;
        return 301 https://$host$request_uri;
}

server{
        listen [::]:443 ssl;
        listen 443 ssl;
        server_name www.runninghi.store runninghi.store;
    ssl_certificate /etc/letsencrypt/live/runninghi.store-0001/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/runninghi.store-0001/privkey.pem; # managed by Certbot


    # HSTS 설정
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;


        location / {
                proxy_pass http://backend;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }

        location ~ /\.well-known/acme-challenge/ {
                default_type "text/plain";
                root /var/www/letsencrypt;
        } 

}
```

설정을 마치고, nginx를 재시작하여 설정을 적용합니다.
```bash
1. nginx -t
2. systemctl restart nginx
```
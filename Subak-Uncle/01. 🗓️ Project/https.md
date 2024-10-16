
## nginx
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;


        server_name _;
        return 301 https://$host$request_uri;

        location ~ /\.well-known/acme-challenge/ {
                default_type "text/plain";
                 root /var/www/letsencrypt;
        }

}

# INUK님 설정
server{
        listen [::]:443 ssl;
        listen 443 ssl;

        server_name www.ticketaka.shop ticketaka.shop;

        ssl_certificate /etc/letsencrypt/live/ticketaka.shop/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/ticketaka.shop/privkey.pem;

        # HSTS 설정
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # 최대 업로드 크기 설정
        client_max_body_size 100M;


        location / {
                proxy_pass http://125.132.216.190:7979;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }

        location /api {
                proxy_pass http://125.132.216.190:7878;
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

## ssl 인증서 발급 -> HTTPS 설정

### Certbot 설치
```
$ sudo apt-get update
$ sudo apt-get install certbot
$ sudo apt-get install python3-certbot-nginx
```

### Nginx 중지
```
$ sudo systemctl stop nginx
```

### Certbot 인증서 발급
```
$ sudo certbot certonly --manual -d "*.[domain]" -d "[domain]"
// EX, sudo certbot certonly --manual -d "*.naver.com" -d "naver.com"
```
인증 진행!!

#### 1단계
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Please deploy a DNS TXT record under the name
_acme-challenge.[domain] with the following value:

qqtIx2ZeudJ0X2sdfsdfwrfw424rONE

Before continuing, verify the record is deployed.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```
=> 가비아에서 DNS 설정 - txt 기반으로 `_acme-challenge.[domain]` 설정 후 응답 값으로 `qq~ONE` 설정

#### 2단계
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Create a file containing just this data:

Y .... -lYXloNAuA8JjD8   # A

And make it available on your web server at this URL:

http://[domain]/.well-known/acme-challenge/Y ... o7JE0u7hbXk    # B

(This must be set up in addition to the previous challenges; do not remove,
replace, or undo the previous challenge tasks yet.)

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

=> nginx  설정
```
$ sudo vi /etc/nginx/sites-available/default



server {
	listen 80 default_server;
	listen [::]:80 default_server;
    
	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

  server_name _;

  location / {
	proxy_pass http://localhost:8081/; 
    proxy_set_header X-Real-IP $remote_addr; 
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
    proxy_set_header Host $http_host;
	}

  location ~ /\.well-known/acme-challenge/ {
    default_type "text/plain";
    root /var/www/letsencrypt;
  }
...
}

```

=> 경로 생성 
```
$ cd /var/www/letsencrypt/
$ sudo mkdir .well-known
$ cd .well-known/
$ sudo mkdir acme-challenge
$ cd acme-challenge/
$ sudo vi Y...bXk

Y...AuA8JjD8

=> $ sudo systemctl restart nginx
```

### 재인증 자동화
인증은 3개월 간 유효 => 즉 3개월마다 갱신 필요.
```
sudo crontab -e
```

=> 다음 내용 추가
```
0 0 1 */3 * certbot renew --quiet --renew-hook "systemctl restart nginx"
```
완료 후 "ctrl+O" => "Enter" => "ctrl+X"

확인
```
sudo crontab -l
```
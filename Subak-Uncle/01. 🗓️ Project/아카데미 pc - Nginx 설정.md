
```

upstream runninghi_store_docker {
    server 172.20.0.2:20238;  # 도커 호스트 IP와 매핑된 포트
}

upstream runninghi_dev_store_docker {
	server 172.20.0.3:10238;
}

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

# bootbox
server {
    listen [::]:443 ssl;
    listen 443 ssl;
    server_name www.bootbox.info bootbox.info;

    ssl_certificate /etc/letsencrypt/live/bootbox.info/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/bootbox.info/privkey.pem;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    client_max_body_size 100M;

    # Next.js 프론트엔드 서버로 라우팅 (예: /app 경로로 시작하는 요청)
    location /app {
        proxy_pass http://localhost:33332;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Spring 백엔드 서버로 라우팅 (예: /api 경로로 시작하는 요청)
    location /api {
        proxy_pass http://localhost:33333;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location / {
        proxy_pass http://localhost:33332;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location ~ /\.well-known/acme-challenge/ {
        default_type "text/plain";
        root /var/www/letsencrypt;
    }
}

# runninghi.store 설정
server{
        listen [::]:443 ssl;
        listen 443 ssl;

        server_name www.runninghi.store runninghi.store;

        ssl_certificate /etc/letsencrypt/live/runninghi.store/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/runninghi.store/privkey.pem;

        # HSTS 설정
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # 최대 업로드 크기 설정
        client_max_body_size 100M;


        location / {
                proxy_pass http://runninghi_store_docker;
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

# runninghi-dev.store 설정
server{
        listen [::]:443 ssl;
        listen 443 ssl;

        server_name www.runninghi-dev.store runninghi-dev.store;

	ssl_certificate /etc/letsencrypt/live/runninghi-dev.store/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/runninghi-dev.store/privkey.pem;

	# HSTS 설정
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # 최대 업로드 크기 설정
        client_max_body_size 100M;


        location / {
                proxy_pass http://runninghi_dev_store_docker;
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

# adregamdi.store 설정
server {
    listen [::]:443 ssl;
    listen 443 ssl;

    server_name www.adregamdi.store adregamdi.store;

    ssl_certificate /etc/letsencrypt/live/adregamdi.store/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/adregamdi.store/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;

    add_header Strict-Transport-Security "max-age=63072000" always;

    client_max_body_size 100M;
    client_body_buffer_size 100M;

    proxy_buffer_size 64k;
    proxy_buffers 16 64k;
    proxy_busy_buffers_size 128k;

    proxy_connect_timeout 300s;
    proxy_send_timeout 600s;
    proxy_read_timeout 600s;

    access_log /var/log/nginx/adregamdi.access.log;
    error_log /var/log/nginx/adregamdi.error.log;


    location / {
        proxy_pass http://localhost:30238;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        # 추가 헤더 설정
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 연결 유지 설정
        proxy_set_header Connection "";
    }

    location ~ /\.well-known/acme-challenge/ {
        default_type "text/plain";
        root /var/www/letsencrypt;
   }


}

# adregamdi-dev.store 설정
server {
    listen [::]:443 ssl;
    listen 443 ssl;

    server_name www.adregamdi-dev.store adregamdi-dev.store;

    ssl_certificate /etc/letsencrypt/live/adregamdi-dev.store/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/adregamdi-dev.store/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;

    add_header Strict-Transport-Security "max-age=63072000" always;

    client_max_body_size 100M;
    client_body_buffer_size 100M;

    proxy_buffer_size 64k;
    proxy_buffers 16 64k;
    proxy_busy_buffers_size 128k;

    proxy_connect_timeout 300s;
    proxy_send_timeout 600s;
    proxy_read_timeout 600s;

    access_log /var/log/nginx/adregamdi-dev.access.log;
    error_log /var/log/nginx/adregamdi-dev.error.log;


    location / {
        proxy_pass http://localhost:40238;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        # 추가 헤더 설정
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 연결 유지 설정
        proxy_set_header Connection "";
    }

    location ~ /\.well-known/acme-challenge/ {
        default_type "text/plain";
        root /var/www/letsencrypt;
   }


}

```
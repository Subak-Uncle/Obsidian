 
# WSL2 배포 가이드

## 1. 공인 IP / WSL2 가상환경 내부 IP 조회
### 공인 IP 조회
- 윈도우에서 공인 IP 조회:
```powershell
nslookup myip.opendns.com resolver1.opendns.com
```
  또는 [whatismyip](https://www.whatismyip.com/) 같은 웹사이트를 통해 확인할 수 있습니다.

### WSL2 가상환경 내부 IP 조회
- WSL2에서 내부 IP 조회:
  ```bash
  ip addr show eth0
  ```
  - 출력 결과에서 `inet` 항목의 IP 주소가 WSL2 내부 IP입니다. 예: `172.19.x.x`

---

## 2. 가비아 도메인 구매
### A 레코드 설정
1. 가비아에서 도메인을 구매합니다.
2. DNS 설정에서 **A 레코드**에 공인 IP 주소를 매핑합니다.
   - 예: `runninghi-dev.store` -> `{조회한 공인 IP}`

---

## 3. 방화벽 포트 허용
### 포트 80, 443, 22 허용
- 윈도우에서 PowerShell을 실행하고 방화벽 규칙을 추가합니다:
  ```powershell
  New-NetFirewallRule -DisplayName "Allow HTTP (80)" -Direction Inbound -LocalPort 80 -Protocol TCP -Action Allow
  New-NetFirewallRule -DisplayName "Allow HTTPS (443)" -Direction Inbound -LocalPort 443 -Protocol TCP -Action Allow
  New-NetFirewallRule -DisplayName "Allow SSH (22)" -Direction Inbound -LocalPort 22 -Protocol TCP -Action Allow
  ```

---

## 4. Nginx 설치 및 적용
1. WSL2에서 Nginx 설치:
   ```bash
   sudo apt update
   sudo apt install nginx
   ```
2. Nginx 설정 파일 편집:
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```
3. 기본 Nginx 설정 예시:
   ```nginx
upstream runninghi_store_docker {
    server 172.17.0.1:8080;  # 도커 호스트 IP와 매핑된 포트
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
   ```
4. Nginx 재시작:
```bash
	nginx -t
	sudo systemctl restart nginx
```

---

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
	proxy_pass http://localhost:8080/; 
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
## 5. WSL2 포트포워딩
- https://velog.io/@sangwoong/WSL2-PortForwarding
- [윈도우 실행 시 배치파일 자동화](https://blog.naver.com/islove8587/223430875037)
- 윈도우에서 PowerShell을 사용하여 포트포워딩 설정:
- 배치 파일(`ps1`) 제작 후 powershell 관리자 권한으로 실행
```powershell
$remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if( $found ){
  $remoteport = $matches[0];
} else{
  echo "The Script Exited, the ip address of WSL 2 
cannot be found";
  exit;
}

#[Ports]
#All the ports you want to forward separated by coma
$ports=@(80, 1000,2000,3000,8000,8002);


#[Static ip]
#You can change the addr to your ip config to listen to a specific address
$addr='0.0.0.0';
$ports_a = $ports -join ",";


#Remove Firewall Exception Rules
iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";

#adding Exception Rules for inbound and outbound Rules
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";

for( $i = 0; $i -lt $ports.length; $i++ ){
  $port = $ports[$i];
  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport";
}

Invoke-Expression "netsh interface portproxy show v4tov4";
  ```
  - `<WSL2_IP>`는 WSL2에서 확인한 내부 IP입니다.

---

## 6. 호스트 설정
### SSH 접근을 위한 pem 키 생성 
#### 1. OpenSSL 설치
대부분의 리눅스 배포판에는 OpenSSL이 기본적으로 설치되어 있습니다. 만약 설치되어 있지 않다면, 다음 명령으로 설치할 수 있습니다.

Ubuntu:

```
sudo apt update
sudo apt install openssl
```

#### 2. 개인 키(.pem) 생성
개인 키는 서버에서 중요한 데이터로, 절대 노출되지 않도록 주의해야 합니다. 먼저 RSA 방식의 개인 키를 생성합니다.

RSA 개인 키 생성:

```
ssh-keygen -t rsa -b 4096 -C "cicd@runninghi-dev.store" -f /home/cicd/.ssh/cicd-privatekey.pem
```
privatekey.pem은 생성된 개인 키 파일입니다.

#### 3. 공개 키(.pem) 생성
공개 키는 다른 서버와 공유할 수 있는 정보입니다. 개인 키에서 공개 키를 추출할 수 있습니다.

공개 키 추출:

```
ssh-keygen -y -f /home/cicd/.ssh/cicd-privatekey.pem > /home/cicd/.ssh/cicd-publickey.pub
```
privatekey.pem을 사용해 publickey.pem 파일을 생성합니다.
publickey.pem은 공개 키가 저장된 파일입니다.

#### 4. CSR (인증서 서명 요청) 생성 (옵션)
만약 SSL 인증서를 발급받기 위한 CSR을 생성하려면 다음 명령을 사용합니다.

CSR 생성:
```
openssl req -new -key privatekey.pem -out certrequest.csr
```
CSR을 생성할 때 여러 정보를 입력하라고 요청할 것입니다. 해당 정보는 인증서 발급 시 사용됩니다.

생성된 .pem 파일은 SSL 설정, SSH 키 설정, 또는 서버 간 보안 통신을 위해 사용될 수 있습니다.
#### 5. ssh 설치
```
SSH 서버 설치 및 실행:

sudo apt update
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh
```

#### 6. 공개 키를 서버로 복사
클라이언트에서 생성한 공개 키를 원격 서버(WSL2)에 복사하여 SSH 접속을 설정합니다.
복사할 때는 ssh 설정 중 `PasswordAuthentication yes`설정이어야 함.

공개 키 복사 (클라이언트에서 서버로):
```
ssh-copy-id -i ~/.ssh/id_rsa.pub username@server_ip
```

또는 수동으로 복사할 경우:

1. 클라이언트에서 ~/.ssh/id_rsa.pub 파일을 서버로 복사:
```
scp ~/.ssh/id_rsa.pub username@server_ip:/home/username/.ssh/authorized_keys
```

2. 서버에서 ~/.ssh/authorized_keys 파일의 권한을 설정:
```
chmod 600 ~/.ssh/authorized_keys
```

#### 7. SSH 설정 파일 편집 (서버 측)
서버에서 SSH 설정 파일을 편집하여 보안 설정을 강화할 수 있습니다.

SSH 설정 파일 열기:
```
sudo vim /etc/ssh/sshd_config
```

주요 설정:

- PermitRootLogin: 루트 계정의 직접 SSH 접속을 차단하여 보안 강화.
```
PermitRootLogin no
```

- PasswordAuthentication: 암호 인증 대신 공개 키 인증만 허용.
```
PasswordAuthentication no
```

- PubkeyAuthentication: 퍼블릭 키 인증 허용
```
PubkeyAuthentication yes
```
설정 변경 후 SSH 서버를 다시 시작합니다:
```
sudo systemctl restart ssh
```


#### 추가) 유저 생성
##### 1. 계정 생성
새로운 사용자를 추가하려면 adduser 명령을 사용합니다. 아래 명령어로 새로운 계정을 생성할 수 있습니다:

```
sudo adduser new_username
new_username은 새로 생성할 계정 이름입니다.
```

이 명령을 실행하면, 비밀번호 설정과 사용자 정보 입력이 요청됩니다.

##### 2. 계정에 관리자 권한 부여
새로 생성한 계정에 관리자 권한을 부여하려면 sudo 그룹에 사용자를 추가해야 합니다.

```
sudo usermod -aG sudo new_username
이 명령을 실행하면, new_username 사용자가 관리자 권한을 갖게 됩니다.
```


##### 3. 계정 생성 확인
새로 생성한 계정이 제대로 생성되었는지 확인하려면, 해당 계정으로 로그인하거나, 다음 명령어로 확인할 수 있습니다:

```
getent passwd new_username
```

이 방식으로 추가 계정을 생성하고, 필요하면 관리자 권한까지 부여할 수 있습니다.

## 7. 데이터베이스 외부 접근 허용
### 1. 3306 포트 방화벽 & 포트포워딩 설정
### 2. 외부 접근 가능하도록 설정 변경
MySQL은 기본적으로 **로컬 호스트(127.0.0.1)**에서만 접속을 허용합니다. 외부에서 접속할 수 있도록 모든 IP에서의 접속을 허용하도록 설정해야 합니다.

#### 2.1 MySQL 설정 파일 확인
MySQL 설정 파일(/etc/mysql/mysql.conf.d/mysqld.cnf 또는 /etc/mysql/my.cnf)에서 bind-address를 확인합니다.

```
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
이 파일에서 bind-address 항목을 찾습니다. 기본적으로 다음과 같이 설정되어 있을 것입니다:

```
bind-address = 127.0.0.1
```
127.0.0.1로 설정되어 있으면 외부 접속을 차단하므로, 이를 0.0.0.0으로 변경하여 모든 IP 주소에서의 접속을 허용합니다.

```
bind-address = 0.0.0.0
```

#### 2.2 MySQL 서비스 재시작
설정을 변경한 후, MySQL 서비스를 재시작합니다.
```
sudo systemctl restart mysql
```

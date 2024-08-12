
### 1. Ubuntu 서버에 MinIO 설치:
```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```


### 2. MinIO 사용자 생성 & 서버 시작
```bash
# 사용자 생성
sudo useradd -r minio-user -s /sbin/nologin

# 데이터 디렉토리 생성
sudo mkdir /usr/local/share/minio
sudo chown minio-user:minio-user /usr/local/share/minio

minio server ~/minio-data --console-address ":9001" 
```
이 명령은 MinIO 서버를 시작하고 관리 콘솔을 9001 포트에서 실행합니다.


### 3. MinIO 서버를 시스템 서비스로 등록
```bash
sudo vim /etc/systemd/system/minio.service
```

#### 다음 내용을 파일에 추가:
```file
[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
WorkingDirectory=/usr/local/

User=minio-user
Group=minio-user

EnvironmentFile=/etc/default/minio
ExecStart=/usr/local/bin/minio server $MINIO_OPTS

Restart=always
LimitNOFILE=65536
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

### 4. 환경 설정 파일 생성
```bash
sudo vim /etc/default/minio
```

#### 다음 내용을 추가:
```txt
MINIO_OPTS="--address :9000 --console-address :9001 /usr/local/share/minio"
MINIO_ACCESS_KEY="minio_access_8e729bdb9"
MINIO_SECRET_KEY="minio_secret_key_a728nd7BnmzpQ9IoPL"
```

### 5. MinIO 서비스 시작
```bash
sudo systemctl daemon-reload
sudo systemctl enable minio
sudo systemctl start minio
```

### 6. MinIO 로깅 설정(선택)
MinIO는 기본적으로 표준 출력(stdout)으로 로그를 출력합니다. systemd를 사용하면 이 로그는 자동으로 journal에 기록됩니다. 
#### 로그를 확인하려면
```bash
sudo journalctl -u minio
```

#### 특정 로그 파일로 출력을 리다이렉션하려면 systemd 서비스 파일을 수정해야 합니다.
```bash
sudo vim /etc/systemd/system/minio.service
```

[Service]섹션에 다음 라인을 추가합니다.
```
StandardOutput=append:/home/ubuntu/logs/minio/minio.log
StandardError=append:/home/ubuntu/logs/minio/minio-error.log
```

로그 디렉토리를 생성하고 권한을 설정합니다.
```bash
sudo mkdir -p /home/ubuntu/logs/minio
sudo chown minio-user:minio-user /home/ubuntu/logs/minio
```

변경 사항을 적용하려면
```bash
sudo systemctl daemon-reload
sudo systemctl restart minio
```

### 7. 로그 로테이션 설정(선택)
로그 파일이 너무 커지는 것을 방지하기 위해 logrotate를 사용할 수 있습니다.
```bash
sudo vim /etc/logrotate.d/minio
```

다음 내용을 추가합니다.
```bash
/var/log/minio/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 minio-user minio-user
}
```

이 설정은 매일 로그를 로테이션하고, 7일간 보관하며, 압축합니다.

### 8.환경 변수를 통한 추가 로깅 설정(선택)

MinIO는 환경 변수를 통해 추가적인 로깅 설정을 할 수 있습니다. `/etc/default/minio` 파일에 다음과 같은 설정을 추가할 수 있습니다:
```bash
// 콘솔 로깅 비활성화
MINIO_LOGGER_CONSOLE_ENABLE=off

// HTTP 트레이싱 활성화
MINIO_API_LOGGER_HTTP_TRACE=on
```

이러한 설정을 변경한 후에는 MinIO 서비스를 재시작해야 합니다:
```bash
sudo systemctl restart minio
```

이렇게 설정하면 MinIO가 백그라운드에서 실행되며, 지정된 파일로 로그가 출력됩니다. 로그 로테이션을 통해 디스크 공간을 효율적으로 관리할 수 있으며, 필요에 따라 추가적인 로깅 옵션을 설정할 수 있습니다.
### 9. Spring Boot 애플리케이션 설정
#### build.gradle에 의존성 추가:
```bash
gradleCopyimplementation 'io.minio:minio:8.5.2'
```

#### application.yml 파일에 MinIO 설정 추가:
```yaml
minio:
  url: http://your-minio-server:9000
  bucket: your-bucket-name
  access-key: your-access-key
  secret-key: your-secret-key
```

### 10. Spring Boot에서 MinIO 클라이언트 설정:
```java
@Configuration
public class MinioConfig {

    @Value("${minio.url}")
    private String minioUrl;

    @Value("${minio.access-key}")
    private String accessKey;

    @Value("${minio.secret-key}")
    private String secretKey;

    @Bean
    public MinioClient minioClient() {
        return MinioClient.builder()
                .endpoint(minioUrl)
                .credentials(accessKey, secretKey)
                .build();
    }
}
```

### 11. MinIO 서비스 구현

```java
@Service
@RequiredArgsConstructor
public class FileStorageService {

    private final MinioClient minioClient;

    @Value("${minio.bucket}")
    private String bucketName;

    public String uploadFile(MultipartFile file) throws Exception {
        String fileName = generateFileName(file);
        minioClient.putObject(
            PutObjectArgs.builder()
                .bucket(bucketName)
                .object(fileName)
                .stream(file.getInputStream(), file.getSize(), -1)
                .contentType(file.getContentType())
                .build());
        return fileName;
    }

    public byte[] downloadFile(String fileName) throws Exception {
        InputStream stream = minioClient.getObject(
            GetObjectArgs.builder()
                .bucket(bucketName)
                .object(fileName)
                .build());
        return IOUtils.toByteArray(stream);
    }

    private String generateFileName(MultipartFile file) {
        return UUID.randomUUID().toString() + "-" + file.getOriginalFilename();
    }
}
```


### 12. 포트 보안 설정
#### 방화벽 포트 설정
기본적으로 ufw 방화벽이 활성화되어 있겠지만, 다운로드부터 설명드리겠습니다.
```bash
sudo apt-get install ufw
sudo ufw enable

sudo ufw allow 9000
sudo ufw allow 9001

# 방화벽 서비스 시작
sudo systemctl ufw start
# 방화벽 서비스 상태 확인
sudo ufw status verbose
```

#### Nginx 설정


##### 설정파일 수정
- 설정파일 경로 :  `/etc/nginx/sites-available/default`
```bash
server {
    listen 443 ssl;
    server_name your_domain.com;

    ssl_certificate /path/to/your/fullchain.pem;
    ssl_certificate_key /path/to/your/privkey.pem;

    # 기존 Java 애플리케이션 설정
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # MinIO API
    location /minio {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 300;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        chunked_transfer_encoding off;

        proxy_pass http://localhost:9000;
    }

    # MinIO Console
    location /minio-console {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 300;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        rewrite ^/minio-console/(.*) /$1 break;
        proxy_pass http://localhost:9001;
    }
}
```

##### Nginx 설정 테스트 및 재시작
```bash
sudo nginx -t
sudo systemctl restart nginx

# 방화벽 포트 추가
sudo ufw allow 9000
sudo ufw allow 9001
```
## References
-  https://oingdaddy.tistory.com/144


─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── (q)uit/esc
Name:                                                                                                                                                                                                                                                                         
  mc anonymous - manage anonymous access to buckets and objects                                                                                                                                                                                                               
                                                                                                                                                                                                                                                                              
USAGE:                                                                                                                                                                                                                                                                        
  mc anonymous [FLAGS] set PERMISSION TARGET                                                                                                                                                                                                                                  
  mc anonymous [FLAGS] set-json FILE TARGET                                                                                                                                                                                                                                   
  mc anonymous [FLAGS] get TARGET                                                                                                                                                                                                                                             
  mc anonymous [FLAGS] get-json TARGET                                                                                                                                                                                                                                        
  mc anonymous [FLAGS] list TARGET                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                                                              
FLAGS:                                                                                                                                                                                                                                                                        
  --recursive, -r               list recursively                                                                                                                                                                                                                              
  --config-dir value, -C value  path to configuration folder (default: "/root/.mc") [$MC_CONFIG_DIR]                                                                                                                                                                          
  --quiet, -q                   disable progress bar display [$MC_QUIET]                                                                                                                                                                                                      
  --disable-pager, --dp         disable mc internal pager and print to raw stdout [$MC_DISABLE_PAGER]                                                                                                                                                                         
  --no-color                    disable color theme [$MC_NO_COLOR]                                                                                                                                                                                                            
  --json                        enable JSON lines formatted output [$MC_JSON]                                                                                                                                                                                                 
  --debug                       enable debug output [$MC_DEBUG]                                                                                                                                                                                                               
  --insecure                    disable SSL certificate verification [$MC_INSECURE]                                                                                                                                                                                           
  --limit-upload value          limits uploads to a maximum rate in KiB/s, MiB/s, GiB/s. (default: unlimited) [$MC_LIMIT_UPLOAD]                                                                                                                                              
  --limit-download value        limits downloads to a maximum rate in KiB/s, MiB/s, GiB/s. (default: unlimited) [$MC_LIMIT_DOWNLOAD]                                                                                                                                          
  --help, -h                    show help                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                              
PERMISSION:                                                                                                                                                                                                                                                                   
  Allowed policies are: [private, public, download, upload].                                                                                                                                                                                                                  
                                                                                                                                                                                                                                                                              
FILE:                                                                                                                                                                                                                                                                         
  A valid S3 anonymous JSON filepath.                                                                                                                                                                                                                                         
                                                                                                                                                                                                                                                                              
EXAMPLES:                                                                                                                                                                                                                                                                     
  1. Set bucket to "download" on Amazon S3 cloud storage.                                                                                                                                                                                                                     
     $ mc anonymous set download s3/mybucket                                                                                                                                                                                                                                  
                                                                                                                                                                                                                                                                              
  2. Set bucket to "public" on Amazon S3 cloud storage.                                                                                                                                                                                                                       
     $ mc anonymous set public s3/shared                                                                                                                                                                                                                                      
                                                                                                                                                                                                                                                                              
  3. Set bucket to "upload" on Amazon S3 cloud storage.                                                                                                                                                                                                                       
     $ mc anonymous set upload s3/incoming                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                                              
  4. Set anonymous to "public" for bucket with prefix on Amazon S3 cloud storage.                                                                                                                                                                                             
     $ mc anonymous set public s3/public-commons/images                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                              
  5. Set a custom prefix based bucket anonymous on Amazon S3 cloud storage using a JSON file.                                                                                                                                                                                 
     $ mc anonymous set-json /path/to/anonymous.json s3/public-commons/images                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                              
  6. Get bucket permissions.                                                                                                                                                                                                                                                  
     $ mc anonymous get s3/shared                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                                                              
  7. Get bucket permissions in JSON format.                                                                                                                                                                                                                                   
     $ mc anonymous get-json s3/shared                                                                                                                                                                                                                                        
                                                                                                                                                                                                                                                                              
  8. List policies set to a specified bucket.                                                                                                                                                                                                                                 
     $ mc anonymous list s3/shared                                                                                                                                                                                                                                            
                                                                                                                                                                                                                                                                              
  9. List public object URLs recursively.                                                                                                                                                                                                                                     
     $ mc anonymous --recursive links s3/shared/   
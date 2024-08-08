
### 1. Ubuntu 서버에 MinIO 설치:
```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```


### 2. MinIO 서버 시작
```bash
mkdir ~/minio-data
minio server ~/minio-data --console-address ":9001"
```
이 명령은 MinIO 서버를 시작하고 관리 콘솔을 9001 포트에서 실행합니다.


### 3. MinIO 서버를 시스템 서비스로 등록
```bash
sudo vim /etc/systemd/system/minio.service
```

#### 다음 내용을 파일에 추가:
```file
Copy[Unit]
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
MINIO_OPTS="--address :9000 /path/to/data"
MINIO_ACCESS_KEY="minio_access_8e729bdb9"
MINIO_SECRET_KEY="minio_secret_key_a728nd7BnmzpQ9IoPL"
```

### 5. MinIO 서비스 시작
```bash
sudo systemctl start minio
sudo systemctl enable minio
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



## References
-  https://oingdaddy.tistory.com/144
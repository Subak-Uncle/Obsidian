
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
sudo nano /etc/systemd/system/minio.service
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
sudo nano /etc/
```

#### 다음 내용을 추가:
```txt
MINIO_OPTS="--address :9000 /path/to/data"
MINIO_ACCESS_KEY="your-access-key"
MINIO_SECRET_KEY="your-secret-key"
```

### 5. MinIO 서비스 시작
```bash
sudo systemctl start minio
sudo systemctl enable minio
```

### 6. Spring Boot 애플리케이션 설정
#### build.gradle에 의존성 추가:
```bash
gradleCopyimplementation 'io.minio:minio:8.5.2'
```

#### application.yml 파일에 MinIO 설정 추가:
```yaml
Copyminio:
  url: http://your-minio-server:9000
  bucket: your-bucket-name
  access-key: your-access-key
  secret-key: your-secret-key
```

### 7. Spring Boot에서 MinIO 클라이언트 설정:
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

### 8.MinIO 서비스 구현:

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

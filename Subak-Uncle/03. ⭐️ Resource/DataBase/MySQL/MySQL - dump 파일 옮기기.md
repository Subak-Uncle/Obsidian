
## 1. 덤프 파일 생성
```bash
mysqldump -u 사용자명 -p 데이터베이스명 > 덤프파일명.sql
-- ex) mysqldump -u root -p runninghi > runninghi_dump.sql
```

> 주요 옵션
```
	1.	–all-databases: 모든 데이터베이스를 덤프합니다.
	2.	–databases: 지정한 데이터베이스들만 덤프합니다.
	3.	–single-transaction: InnoDB 테이블의 일관성 있는 덤프를 생성합니다.
	4.	–no-data: 데이터를 제외하고 구조만 덤프합니다.
	5.	–routines: 저장 프로시저와 함수를 포함합니다.
	6.	–triggers: 트리거를 포함합니다.
	7.	–events: 이벤트를 포함합니다.
	8.	–column-statistics=0: MySQL 8.0에서 추가된 옵션으로, 통계 정보 생성을 비활성화합니다.
```

### 명령어 예시
#### 1. 특정 데이터베이스 덤프
```bash
mysqldump -u root -p --single-transaction --routines --triggers --events mydb > mydb_dump.sql
```

#### 2. 여러 데이터베이스 덤프
```bash
mysqldump -u root -p --databases db1 db2 db3 > multiple_db_dump.sql
```

#### 3. 모든 데이터베이스 덤프
```bash
mysqldump -u root -p --all-databases > all_databases_dump.sql
```

#### 4. 구조만 덤프
```bash
mysqldump -u root -p --no-data mydb > mydb_structure.sql
```

#### 5. 특정 데이터만 덤프
```bash
mysqldump -u root -p mydb table1 table2 > specific_tables_dump.sql
```

#### 6. 원격 서버에서 덤프
```bash
mysqldump -h [호스트주소] -u [사용자명] -p [데이터베이스명] > remote_dump.sql
```

#### 7. 압축 덤프 생성
```bash
mysqldump -u root -p mydb | gzip > mydb_dump.sql.gz
```

## 2. 덤프 파일 내 DB 이름 변경(스키마 변경)
### macOS
```
sed -i '' 's/기존데이터베이스명/new데이터베이스명/g' 덤프파일명.sql

// UTF-8 설정이 안되어 있다면
LC_CTYPE=C sed -i '' 's/기존데이터베이스명/new데이터베이스명/g' 덤프파일명.sql
```
### window
```
sed -i 's/기존데이터베이스명/new데이터베이스명/g' 덤프파일명.sql
```


## 3. 덤프 파일을 가져오기(Import)

```bash
mysql -u 사용자명 -p new데이터베이스명 < 덤프파일명.sql
```
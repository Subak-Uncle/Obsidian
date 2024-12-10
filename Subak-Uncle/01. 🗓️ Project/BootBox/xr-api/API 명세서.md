
> **Base-Server-URL**
> http://mtvs.helloworldlabs.kr:7771
## 1. String 관련 API

### 1.1 GET /api/string

쿼리 파라미터로 받은 문자열을 반환합니다.

#### Request
- **Method:** GET
- **URL**: `http://mtvs.helloworldlabs.kr:7771/api/string?parameter={요청 문자열}`
- **Query Parameter:** 
	- `parameter`: 요청할 문자열
- **Headers**:
	- Content-Type: `text/plain`
#### Response: 
- Http Status - 200(OK)
	- **Headers**:
		- Content-Type: `text/plain;charset=UTF-8`
		- Content-Length: 문자열 길이
		-  Date: 응답이 발생한 시간
    -  **Body**: 요청 받은 문자열

<br />
### 1.2 GET /api/string/{string} 

Path 변수로 받은 문자열을 반환합니다.
#### Request
- **Method:** GET
- **URL:** `http://mtvs.helloworldlabs.kr:7771/api/string/{parameter}`
- **Path Variable:** 
  - `parameter`: 요청할 문자열
#### Response
-  HttpStatus - 200(OK)
	- **Header**:
		- Content-Type: `text/plain;charset=UTF-8`
		- Content-Length: 문자열 길이
		-  Date: 응답이 발생한 시간
	- **Body**: 요청 받은 문자열


<br />

## 2. JSON 관련 API
### 2.1 GET /api/json
json 데이터를 요청 시 json을 반환합니다.
#### Request
- **Method:** GET
- **URL:** `http://mtvs.helloworldlabs.kr:7771/api/json`
#### Response
- **Headers:** 
	- Content-Type: `application/json`
	- Transfer-Encoding: `chunked`
	- Date: 응답이 발생한 시간
- **Body**: 요청 받은 JSON 객체
```json
{
    "timestamp": "2024-08-01T16:48:19.9095194",
    "title": "제목입니다.",
    "content": "내용입니다.",
    "data": {
        "id": 1,
        "name": "메타버스 아카데미"
    }
}
```


### 2.2 POST /api/json

요청 본문으로 받은 JSON을 반환합니다.
#### Request
- **Method:** POST
- **URL:** `http://mtvs.helloworldlabs.kr:7771/api/json`
- **Headers**:
	- Content-Type: `application/json`
- **Body:** JSON 객체
```json
{
	"name": "자유롭게 키/값 을 설정해서 보내주세요."
}
```




#### Response
  - **Headers**:
	  - Content-Type: `application/json`
	  - Transfer-Encoding: `chunked`
	  - Date: 응답이 발생한 시간
  - **Body**: JSON 객체
```json
{
    "name": "자유롭게 키/값 을 설정해서 보내주세요."
}
```



## 3. 파일 관련 API
### 3.1 GET /api/file

요청한 파일 명에 해당하는 이미지 파일을 반환합니다.
이번 예제에서는 `helloworld` 로 파일 명을 요청합니다.
#### Request
- **Method:** GET
- **URL:** `http://mtvs.helloworldlabs.kr:7771/api/file`
- **Query Parameter:** 
  - `filename`: 요청할 파일 명 (확장자 제외) `helloworld`
#### Response
- **Headers:** 
	- Content-Type: `image/png`
	- Content-Length: 이미지 파일 길이
	- Date: 응답이 발생한 시간
- **Body:** 요청한 이미지 파일의 Byte 데이터


### 3.2 POST /api/file

전송 받은 파일을 반환합니다.
#### Request
- **Method:** POST
- **URL:** `http://mtvs.helloworldlabs.kr:7771/api/file`
- **Headers:** 
	- Content-Type: `multipart/form-data`
- **Body:**
	- Form Data:
	    - `file`: 업로드할 파일
#### Response
- **Headers:** 
	- Content-Disposition: `attachment; filename="[원본 파일명]"`
	- Content-Type: 업로드 된 파일의 MIME 타입
		- ex) json -> application/json, 이미지 -> image/이미지 확장자
	- Transfer-Encoding: `chunked`
	- Date: 응답이 발생한 시간
- **Body:** 업로드 된 파일


### 3.3 POST /api/byte
byte 형태로 파일을 받아 byte로 반환합니다.

#### Request
- **Method**: POST
- **URL**: `http://mtvs.helloworldlabs.kr:7771/api/byte`
- **Headers**:
	- Content-Type: `업로드해주신 바이너리 파일 형태를 작성해주세요.` 
		- ex)  
		     .{바이너리 파일} -> application/octet-stream
		      .txt -> text/plain
		     .pdf -> application/pdf
		     .{각종 이미지 확장자} -> image/{각종 이미지 확장자}
- **Body:**
	- binary:
		- `byte[]`: 업로드할 바이너리 파일

#### Response
- **Headers:**
	- Content-Type: 업로드 된 파일의  MIME 타입
		-  ex) json -> application/json, 이미지 -> image/이미지 확장자
	- Date: 응답이 발생한 시간
- **Body:**  업로드 된 파일
	
		     
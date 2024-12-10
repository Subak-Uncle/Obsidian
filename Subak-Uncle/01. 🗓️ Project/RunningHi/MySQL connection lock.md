
원격 서버에서 Mysql Server 로 단순 커넥션 한 뒤 close 하게 되면
Mysql 은 비정상적인 접속으로 판단하여 해당 IP를 블럭킹하게 된다.
이때 mysql의 비정상적인 접속 요청수를 카운트 하는데

global max_connect_errors 에 지정된 값을 넘기면 자동 블럭킹처리 됨

기본값은 10이며 필요한 경우 이 값을 변경해야 한다.

```
-- max_connect_errors 카운트 확인
select @@global.max_connect_errors;

-- max_connections 카운트 확인
select @@global.max_connections;

-- 에러 카운트 초기화
flush hosts;

-- max_connections 변경
set global max_connections=300;

-- max_connect_errors 변경
set global max_connect_errors=10000;

* mysql 을 재시작 하지 않아도 적용됨

```
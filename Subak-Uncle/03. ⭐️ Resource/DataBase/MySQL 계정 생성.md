
~~~bash
-- 1) 새로운 mtvs 계정 만들기
CREATE USER 'new ID'@'%' IDENTIFIED BY  'Password'; -- 'localhost' 대신 '%'를 쓰면 외부 ip로 접속 가능하다.

-- 현재 존재하는 데이터베이스 확인
SHOW databases;

-- mysql 데이터베이스로 계정 정보 확인하기
USE midnight;  -- 기본 적으로 제공되는 mysql database

SELECT * FROM midnight;    -- mysql database에서 user를 확인해 계정이 추가된 것을 확인한다.

-- 2) 데이터베이스 생성 후 계정에 권한 부여
-- 데이터베이스(스키마) 생성
CREATE DATABASE midnight;
-- CREATE SCHEMA menudb;

-- 왼쪽 Navigator를 새로고침해서 menudb database(schema)가 추가된 것을 확인한다.
-- MySQL은 개념적으로 database와 schema를 구분하지 않는다.
-- (CREATE DATABASE와 CREATE SCHEMA가 같은 개념이다.)

GRANT ALL PRIVILEGES ON midnight.* TO 'midnight'@'%';    -- menu에 대한 모든 권한 부여

SHOW GRANTS FOR 'midnight'@'%';

select host, user from mysql.user;
select host, user, plugin, authentication_string from mysql.user;
alter user 'midnight'@'%' identified with mysql_native_password by 'midnight';
~~~


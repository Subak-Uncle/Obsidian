
```

# 계정 생성
adduser {계정 이름}

# 패스워드 비활성화 계정 생성(ssh 접근 시 비밀번호 인증x)
adduser {계정 이름} --disabled-password

# 그룹 생성
groupadd {그룹명}

# 사용자를 그룹에 추가하기
usermod -aG {그룹명} {사용자명}

# 새로운 사용자 생성 시 그룹 지정하기
sudo adduser --ingroup {그룹명} {사용자명}

```
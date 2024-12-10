
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

# Ubuntu 계정 및 권한 관리 가이드

이 문서는 Ubuntu에서 **계정 생성 및 삭제**, **서버 관리**, **sudo 비밀번호 설정**, **chmod**, **chown**, **권한 정보 확인**, **비밀번호 설정** 등 계정 및 권한 관리를 위한 명령어와 설정 방법을 정리한 가이드입니다.

---

## 목차

1. [계정 관리](#계정-관리)
   - [사용자 계정 생성](#사용자-계정-생성)
   - [사용자 계정 삭제](#사용자-계정-삭제)
   - [사용자 비밀번호 설정 및 변경](#사용자-비밀번호-설정-및-변경)
   - [사용자 정보 수정](#사용자-정보-수정)
2. [그룹 관리](#그룹-관리)
   - [그룹 생성](#그룹-생성)
   - [사용자 그룹에 추가](#사용자-그룹에-추가)
   - [그룹 삭제](#그룹-삭제)
3. [권한 관리](#권한-관리)
   - [파일 및 디렉토리 권한 확인](#파일-및-디렉토리-권한-확인)
   - [chmod를 통한 권한 변경](#chmod를-통한-권한-변경)
   - [chown을 통한 소유자 및 그룹 변경](#chown을-통한-소유자-및-그룹-변경)
4. [sudo 설정](#sudo-설정)
   - [sudo 사용자에 대한 비밀번호 설정](#sudo-사용자에-대한-비밀번호-설정)
   - [sudoers 파일을 통한 권한 부여](#sudoers-파일을-통한-권한-부여)
5. [SSH 설정 및 관리](#ssh-설정-및-관리)
   - [SSH 공개 키 인증 설정](#ssh-공개-키-인증-설정)
   - [SSH 접속 허용 및 제한](#ssh-접속-허용-및-제한)
6. [계정 보안 및 관리 팁](#계정-보안-및-관리-팁)
   - [비밀번호 정책 설정](#비밀번호-정책-설정)
   - [계정 잠금 및 잠금 해제](#계정-잠금-및-잠금-해제)
   - [사용자 셸 변경](#사용자-셸-변경)
7. [추가 참고 자료](#추가-참고-자료)

---

## 계정 관리

### 사용자 계정 생성

새로운 사용자 계정을 생성하고 홈 디렉토리를 생성하려면 `adduser` 명령어를 사용합니다.

```bash
sudo adduser 사용자명
```

**예시:**

```bash
sudo adduser runninghi
```

- 프롬프트에 따라 비밀번호와 사용자 정보를 입력합니다.
- `--disabled-password` 옵션을 사용하면 비밀번호 없이 계정을 생성할 수 있습니다.

```bash
sudo adduser --disabled-password 사용자명
```

### 사용자 계정 삭제

사용자 계정을 삭제하려면 `userdel` 명령어를 사용합니다.

```bash
sudo userdel 사용자명
```

- 홈 디렉토리와 메일 스풀까지 삭제하려면 `-r` 옵션을 사용합니다.

```bash
sudo userdel -r 사용자명
```

**예시:**

```bash
sudo userdel -r runninghi
```

### 사용자 비밀번호 설정 및 변경

**자신의 비밀번호 변경:**

```bash
passwd
```

**다른 사용자의 비밀번호 변경 (관리자 권한 필요):**

```bash
sudo passwd 사용자명
```

**비밀번호 제거 (주의 필요):**

```bash
sudo passwd -d 사용자명
```

### 사용자 정보 수정

사용자 정보(예: 셸, 홈 디렉토리)를 수정하려면 `usermod` 명령어를 사용합니다.

- **로그인 셸 변경:**

  ```bash
  sudo usermod -s /bin/bash 사용자명
  ```

- **홈 디렉토리 변경:**

  ```bash
  sudo usermod -d /new/home 사용자명
  ```

- **사용자명 변경:**

  ```bash
  sudo usermod -l 새로운사용자명 기존사용자명
  ```

---

## 그룹 관리

### 그룹 생성

새로운 그룹을 생성하려면 `groupadd` 명령어를 사용합니다.

```bash
sudo groupadd 그룹명
```

**예시:**

```bash
sudo groupadd developers
```

### 사용자 그룹에 추가

사용자를 특정 그룹에 추가하려면 `usermod` 명령어를 사용합니다.

```bash
sudo usermod -aG 그룹명 사용자명
```

**예시:**

```bash
sudo usermod -aG developers runninghi
```

### 그룹 삭제

그룹을 삭제하려면 `groupdel` 명령어를 사용합니다.

```bash
sudo groupdel 그룹명
```

---

## 권한 관리

### 파일 및 디렉토리 권한 확인

`ls -l` 명령어를 사용하여 파일 및 디렉토리의 권한을 확인합니다.

```bash
ls -l 파일명
```

**예시:**

```bash
ls -l /home/runninghi
```

### chmod를 통한 권한 변경

`chmod` 명령어를 사용하여 파일이나 디렉토리의 권한을 변경합니다.

```bash
chmod [권한] 파일명
```

**권한 표기법:**

- **숫자 표기법:**

  - 읽기(r): `4`
  - 쓰기(w): `2`
  - 실행(x): `1`

  **예시:**

  - `chmod 700 파일명` : 소유자에게만 모든 권한(rwx)

- **기호 표기법:**

  - 사용자(u), 그룹(g), 기타(o), 모두(a)

  **예시:**

  - `chmod u+rwx,g+rx,o-rwx 파일명`

### chown을 통한 소유자 및 그룹 변경

`chown` 명령어를 사용하여 파일이나 디렉토리의 소유자와 그룹을 변경합니다.

```bash
sudo chown [소유자][:그룹] 파일명
```

**예시:**

- 소유자만 변경:

  ```bash
  sudo chown runninghi 파일명
  ```

- 소유자와 그룹 모두 변경:

  ```bash
  sudo chown runninghi:developers 파일명
  ```

- 그룹만 변경:

  ```bash
  sudo chown :developers 파일명
  ```

- 재귀적으로 변경:

  ```bash
  sudo chown -R runninghi:developers /path/to/directory
  ```

---

## sudo 설정

### sudo 사용자에 대한 비밀번호 설정

`sudo` 명령어를 사용할 때 비밀번호를 요구하거나 요구하지 않도록 설정할 수 있습니다.

**`sudoers` 파일 편집:**

```bash
sudo visudo
```

**비밀번호 없이 `sudo` 사용 설정:**

```plaintext
사용자명 ALL=(ALL) NOPASSWD: ALL
```

**예시:**

```plaintext
cicd ALL=(ALL) NOPASSWD: ALL 
```

### sudoers 파일을 통한 권한 부여

특정 명령어에 대해서만 `sudo` 권한을 부여할 수 있습니다.

**예시:**

```plaintext
cicd ALL=(root) NOPASSWD: /usr/bin/docker, /usr/bin/fuser, /bin/systemctl
```

- `cicd` 사용자는 `docker`, `fuser`, `systemctl` 명령어를 비밀번호 없이 `sudo`로 실행할 수 있습니다.

---

## SSH 설정 및 관리

### SSH 공개 키 인증 설정

1. **SSH 키 생성 (클라이언트에서):**

   ```bash
   ssh-keygen -t rsa -b 4096
   ```

2. **서버의 사용자 계정에 공개 키 등록:**

   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   vim ~/.ssh/authorized_keys
   ```

   - 공개 키를 `authorized_keys` 파일에 붙여넣습니다.
   - 파일 권한 설정:

     ```bash
     chmod 600 ~/.ssh/authorized_keys
     ```

### SSH 접속 허용 및 제한

**SSH 설정 파일 편집:**

```bash
sudo nano /etc/ssh/sshd_config
```

**설정 예시:**

- 루트 로그인 비활성화:

  ```plaintext
  PermitRootLogin no
  ```

- 비밀번호 인증 비활성화:

  ```plaintext
  PasswordAuthentication no
  ```

- 공개 키 인증 활성화:

  ```plaintext
  PubkeyAuthentication yes
  ```

**SSH 서비스 재시작:**

```bash
sudo systemctl restart ssh
```

---

## 계정 보안 및 관리 팁

### 비밀번호 정책 설정

**libpam-pwquality 모듈 설치:**

```bash
sudo apt install libpam-pwquality
```

**설정 파일 편집:**

```bash
sudo nano /etc/pam.d/common-password
```

**설정 예시:**

```plaintext
password requisite pam_pwquality.so retry=3 minlen=8 difok=3
```

### 계정 잠금 및 잠금 해제

- **계정 잠금:**

  ```bash
  sudo usermod -L 사용자명
  ```

- **계정 잠금 해제:**

  ```bash
  sudo usermod -U 사용자명
  ```

### 사용자 셸 변경

- **로그인 셸 변경:**

  ```bash
  sudo chsh -s /bin/bash 사용자명
  ```

- **제한된 셸로 변경 (rbash):**

  ```bash
  sudo chsh -s /bin/rbash 사용자명
  ```

---

## 추가 참고 자료

- **사용자 및 그룹 관리 매뉴얼:**

  ```bash
  man adduser
  man usermod
  man groupadd
  ```

- **권한 관리 매뉴얼:**

  ```bash
  man chmod
  man chown
  ```

- **sudo 매뉴얼:**

  ```bash
  man sudo
  man sudoers
  ```

- **SSH 매뉴얼:**

  ```bash
  man ssh
  man sshd_config
  ```

---
# 🍫 Windows에서 mkcert 설치 및 로컬 인증서 신뢰 등록 가이드

## 📌 1. Chocolatey 설치

Windows에서 `choco` 명령어를 사용하기 위해 초콜릿 패키지 매니저(Chocolatey)를 설치합니다.

* **PowerShell을 관리자 권한으로 실행**합니다.

### 현재 실행 정책 확인

```powershell
Get-ExecutionPolicy
```

결과가 `Restricted`가 아닌 경우 다음 명령어로 임시 실행 정책을 변경합니다. 필요 시 `Y` 입력:

```powershell
Set-ExecutionPolicy AllSigned
```

### Chocolatey 설치

아래 명령어를 실행하여 Chocolatey를 설치합니다.

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force;
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### 설치 확인

```powershell
choco
```

버전 정보가 표시되면 설치가 성공적으로 완료된 것입니다.

---

## 📌 2. mkcert 설치

`mkcert`는 로컬 개발환경을 위한 신뢰된 SSL 인증서를 간편히 생성하는 도구입니다.

```powershell
choco install mkcert
```

---

## 📌 3. 로컬 신뢰 루트 인증서 등록

최초 1회만 실행하면 됩니다. 이 과정은 팀 내 모든 개발자 PC에서 실행해야 하며, mkcert로 생성한 SSL 인증서가 브라우저와 운영체제에서 신뢰됩니다.

```powershell
mkcert -install
```

---

## ⭐️ 프로젝트에서 와일드카드 인증서 발급 및 Nginx 설정

### 4. VSCode에서 프로젝트 내 `certs` 폴더로 이동

```terminal
cd certs
```

### 5. 와일드카드 도메인 인증서 발급

```terminal
mkcert "*.localtest.me"
```

---

## 📌 6. `nginx.conf` 파일 수정

* `server_name`을 원하는 주소로 변경 (예: `your-service-1.localtest.me`).
* 발급된 인증서 파일명에 맞춰서 `ssl_certificate`와 `ssl_certificate_key` 경로 수정.
* `proxy_pass`의 도커 서비스 포트를 프로젝트 환경에 맞게 수정.

```nginx
server {
    listen 443 ssl;
    server_name your-service-1.localtest.me;

    ssl_certificate     /etc/nginx/certs/your-fem-file-name.pem;
    ssl_certificate_key /etc/nginx/certs/your-fem-file-name-key.pem;

    location / {
        proxy_pass http://host.docker.internal:your-docker-port;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 🚀 Docker로 Nginx 실행

마지막으로 터미널에서 아래 명령어를 실행해 Nginx 컨테이너를 시작합니다.

```terminal
docker compose up -d
```

이제 로컬 환경에서 HTTPS로 서비스가 제공됩니다!

이후부턴 docker desktop에서 nginx 컨테이너를 실행시킬 수 있습니다.
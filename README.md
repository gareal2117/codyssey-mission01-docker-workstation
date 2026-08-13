# Docker Workstation

## 1. 프로젝트 개요

개발 워크스테이션 구축 미션을 수행하며 터미널 조작, Linux 권한, Docker 컨테이너, 네트워크 포트, 스토리지와 Git의 기본 사용법을 실습한 프로젝트입니다.

개발과 실행 검증은 Windows Docker Desktop에서 진행했습니다. 프로젝트 내부 파일은 상대경로로 참조하고 웹 서버는 Docker 이미지로 구성하여 호스트 절대경로에 의존하지 않도록 했습니다.

## 2. 실행 환경

| 구분 | 환경 |
| --- | --- |
| OS | Windows |
| Shell | PowerShell |
| Container runtime | Docker Desktop |
| Docker | Docker version 29.5.2 |
| Git | Git version 2.54.0.windows.1 |

버전 확인에 사용한 명령은 다음과 같습니다.

```powershell
docker --version
git --version
```

## 3. 수행 체크리스트

- [x] 터미널 기본 조작
- [x] 권한 실습
- [x] Docker 설치 및 점검
- [x] `hello-world` 컨테이너 실행
- [x] Ubuntu 컨테이너 실습
- [x] Dockerfile 작성
- [x] 포트 매핑
- [x] 바인드 마운트
- [x] Docker Volume
- [x] Git 설정
- [x] GitHub 원격 저장소 연동 및 push

### 3.1 터미널 기본 조작 로그

#### 현재 위치 확인

```bash
pwd
```

```text
/
```

#### 숨김 파일을 포함한 목록 확인

```bash
ls -la
```

```text
숨김 항목을 포함한 현재 디렉터리 목록을 확인함 (전체 출력 생략)
```

#### 디렉터리 생성 및 이동

```bash
mkdir cli-practice
cd /cli-practice
```

```text
출력 없음
```

#### 파일 및 디렉터리 생성

```bash
touch hello.txt
mkdir testdir
```

```text
출력 없음
```

#### 파일 내용 작성 및 확인

```bash
echo "hello docker" > hello.txt
cat hello.txt
```

```text
hello docker
```

#### 파일 복사

```bash
cp hello.txt copy.txt
```

```text
출력 없음
```

`copy.txt`가 생성된 것을 확인했습니다.

#### 파일 이동 또는 이름 변경

```bash
mv copy.txt renamed.txt
```

```text
출력 없음
```

`copy.txt`가 `renamed.txt`로 이름 변경된 것을 확인했습니다.

#### 파일 및 디렉터리 삭제

```bash
rm renamed.txt
rmdir testdir
```

```text
출력 없음
```

삭제 후 `renamed.txt`와 `testdir`이 사라진 것을 확인했습니다.

## 4. Docker 설치 및 점검

Docker CLI 버전과 Docker Engine 상태를 다음 명령으로 점검했습니다.

```powershell
docker --version
docker info
```

설치 점검 후 테스트 이미지를 실행했고, `hello-world`가 정상적으로 실행되는 것을 확인했습니다.

```powershell
docker run hello-world
```

### 4.1 Docker 기본 운영 명령 로그

#### 이미지 목록 확인

```powershell
docker images
```

```text
REPOSITORY           TAG
docker-workstation   1.0
hello-world          latest
nginx                alpine
ubuntu               latest
```

#### 실행 중인 컨테이너 확인

```powershell
docker ps
```

```text
NAME          IMAGE          PORTS        STATUS
docker-bind   nginx:alpine   8001 -> 80   Up
```

#### 전체 컨테이너 확인

```powershell
docker ps -a
```

```text
NAME                    STATUS    PORTS
cli-test                Exited
permission-test         Exited
vol-test2               Exited
docker-bind             Up        8001 -> 80
docker-web              Up        8000 -> 80
Ubuntu 실습 컨테이너   Exited
hello-world 컨테이너    Exited
```

#### 컨테이너 로그 확인

```powershell
docker logs docker-web
```

```text
nginx configuration complete; ready for start up
nginx/1.31.3
start worker processes
GET / HTTP/1.1 -> 200 또는 304 응답
favicon.ico -> 404
```

nginx가 정상적으로 시작되었고 `/` 요청을 성공적으로 처리했습니다. `favicon.ico`의 404 응답은 브라우저가 아이콘 파일을 별도로 요청하면서 발생한 것으로, 웹페이지 동작에는 영향을 주지 않았습니다.

#### 컨테이너 자원 사용량 확인

```powershell
docker stats --no-stream
```

```text
NAME          CPU %    MEM USAGE
docker-bind   0.00%    약 10.19 MiB
docker-web    0.00%    약 15.04 MiB
```

## 5. Ubuntu 컨테이너 실습

Ubuntu 컨테이너를 대화형 셸로 실행한 뒤 현재 경로와 디렉터리 내용을 확인하고, 문자열을 출력한 후 종료했습니다.

```powershell
docker run -it ubuntu bash
```

컨테이너 내부에서 수행한 명령입니다.

```bash
pwd
ls
echo "hello ubuntu"
exit
```

## 6. 커스텀 웹 서버

### Dockerfile

```dockerfile
FROM nginx:alpine

COPY app/ /usr/share/nginx/html/
```

`nginx:alpine`은 정적 파일 제공에 필요한 nginx를 포함하면서 이미지 크기가 비교적 작아 간단한 웹 서버 실습에 적합합니다. 프로젝트의 `app/index.html`을 nginx 기본 웹 루트인 `/usr/share/nginx/html/`로 복사하여 별도의 웹 서버 설정 없이 기본 페이지로 제공되도록 구성했습니다.

이미지 빌드와 컨테이너 실행에는 다음 명령을 사용했습니다.

```powershell
docker build -t docker-workstation:1.0 .
docker run -d -p 8000:80 --name docker-web docker-workstation:1.0
```

호스트의 `8000`번 포트를 컨테이너의 `80`번 포트에 연결했습니다. 브라우저에서 `http://localhost:8000`으로 접속하여 `Docker Workstation` 페이지가 표시되는 것을 확인했습니다.

## 7. 포트 매핑 증거

호스트 `8000`번 포트에서 컨테이너의 nginx 웹 서버로 접속한 결과입니다.

![포트 매핑 실행 결과](./screenshots/port-mapping-screenshots.PNG)

## 8. 바인드 마운트 검증

호스트의 `app` 폴더를 컨테이너의 `/usr/share/nginx/html`에 바인드 마운트하고, 이미지 재빌드 없이 파일 변경이 반영되는지 확인했습니다. 기존 컨테이너와 포트가 겹치지 않도록 호스트 포트 `8001`을 사용했습니다.

```powershell
docker run -d -p 8001:80 --name docker-bind -v "${PWD}/app:/usr/share/nginx/html" nginx:alpine
```

`${PWD}`를 사용하므로 특정 Windows 절대경로에 의존하지 않습니다. `app/index.html` 변경 전과 변경 후에 `http://localhost:8001`을 확인했고, 호스트에서 수정한 내용이 브라우저에 반영되는 것을 확인했습니다.

변경 전:

![바인드 마운트 변경 전](./screenshots/bind-before-screenshots.PNG)

변경 후:

![바인드 마운트 변경 후](./screenshots/bind-after-screenshots.PNG)

## 9. Docker Volume 영속성 검증

먼저 `mydata` Volume을 만들고 목록에서 생성 여부를 확인했습니다.

```powershell
docker volume create mydata
docker volume ls
```

Volume을 `/data`에 연결한 Ubuntu 컨테이너를 실행한 뒤 파일을 작성하고 내용을 확인했습니다.

```powershell
docker run -d --name vol-test -v mydata:/data ubuntu sleep infinity
docker exec vol-test bash -c "echo hello-volume > /data/hello.txt"
docker exec vol-test cat /data/hello.txt
```

출력:

```text
hello-volume
```

첫 번째 컨테이너를 삭제한 다음 동일한 Volume을 연결한 새 컨테이너를 실행했습니다.

```powershell
docker rm -f vol-test
docker run -d --name vol-test2 -v mydata:/data ubuntu sleep infinity
docker exec vol-test2 cat /data/hello.txt
```

출력:

```text
hello-volume
```

첫 번째 컨테이너를 삭제했음에도 `mydata` Volume을 연결한 두 번째 컨테이너에서 같은 파일을 읽을 수 있었습니다. 이를 통해 컨테이너 생명주기와 독립적으로 Volume 데이터가 유지되는 것을 확인했습니다.

## 10. 권한 실습

Ubuntu 컨테이너 내부에서 파일과 디렉터리의 권한을 변경하고 `ls -ld`로 결과를 확인했습니다.

```bash
touch test.txt
mkdir testdir

chmod 600 test.txt
chmod 700 testdir
ls -ld test.txt testdir
```

확인한 권한 형태:

```text
-rw------- test.txt
drwx------ testdir
```

이후 원래 권한으로 변경했습니다.

```bash
chmod 644 test.txt
chmod 755 testdir
ls -ld test.txt testdir
```

확인한 권한 형태:

```text
-rw-r--r-- test.txt
drwxr-xr-x testdir
```

권한 문자의 의미는 `r`이 읽기(read), `w`가 쓰기(write), `x`가 실행 또는 디렉터리 접근(execute)입니다. 숫자는 소유자, 그룹, 기타 사용자의 권한을 차례로 나타냅니다.

| 권한 | 의미 |
| --- | --- |
| `600` | 소유자만 읽기와 쓰기 가능 |
| `644` | 소유자는 읽기/쓰기, 나머지는 읽기 가능 |
| `700` | 소유자만 읽기/쓰기/실행 가능 |
| `755` | 소유자는 모든 권한, 나머지는 읽기/실행 가능 |

실습 과정에서 `test.txt`는 `644 → 600 → 644`, `testdir`은 `755 → 700 → 755`로 변경했습니다.

## 11. 트러블슈팅

### 11.1 호스트 포트 충돌

- 문제

  기존 `docker-web` 컨테이너가 호스트의 `8000`번 포트를 사용하는 상태에서, 다른 nginx 컨테이너도 같은 포트로 실행하려고 하자 포트 충돌이 발생했습니다.

  ```powershell
  docker run -d -p 8000:80 --name port-conflict-test nginx:alpine
  ```

- 원인 가설

  `-p 8000:80`은 호스트의 `8000`번 포트를 컨테이너의 `80`번 포트에 연결한다는 의미입니다. 하나의 호스트 포트는 동시에 여러 컨테이너에 연결할 수 없으므로, 이미 `docker-web`이 사용 중인 `8000`번 포트를 새 컨테이너가 다시 사용할 수 없다고 판단했습니다.

- 확인 방법

  실행 중인 컨테이너와 포트 매핑을 확인했습니다.

  ```powershell
  docker ps
  ```

  확인 결과 `docker-web` 컨테이너가 호스트 `8000`번 포트에서 컨테이너 `80`번 포트로 연결된 상태로 실행 중이었습니다.

- 해결 방법

  새 nginx 컨테이너에는 충돌하지 않는 호스트 포트 `8002`를 할당했습니다. 컨테이너 내부의 nginx 포트는 그대로 `80`을 사용했습니다.

  ```powershell
  docker run -d -p 8002:80 --name port-conflict-test nginx:alpine
  ```

- 결과

  `docker ps`에서 `port-conflict-test`가 `8002 -> 80` 포트 매핑으로 정상 실행되는 것을 확인했습니다. 테스트가 끝난 뒤 다음 명령으로 컨테이너를 정리했습니다.

  ```powershell
  docker rm -f port-conflict-test
  ```

### 11.2 웹 파일 수정 내용이 기존 컨테이너에 반영되지 않음

- 문제

  `app/index.html`을 수정하고 브라우저를 새로고침했지만, 기존 `docker-web` 컨테이너가 제공하는 화면에는 변경 내용이 반영되지 않았습니다.

- 원인 가설

  Dockerfile의 `COPY` 명령은 `docker build`를 실행하는 시점에 `app/`의 파일을 이미지 내부로 복사합니다. 이미지는 컨테이너를 만드는 기준이 되는 고정된 결과물이므로, 이후 호스트의 `index.html`을 수정해도 이전 이미지와 그 이미지로 만든 기존 컨테이너는 자동으로 바뀌지 않습니다.

- 확인 방법

  현재 실행 중인 컨테이너와 로컬에 생성된 이미지를 확인했습니다.

  ```powershell
  docker ps
  docker images
  ```

  확인 결과 기존 `docker-web` 컨테이너가 수정 전에 빌드된 이미지로 계속 실행 중이었습니다.

- 해결 방법

  수정된 웹 파일이 이미지에 포함되도록 이미지를 다시 빌드하고, 기존 컨테이너를 삭제한 뒤 새 이미지로 컨테이너를 다시 생성했습니다.

  ```powershell
  docker build -t docker-workstation:1.0 .
  docker rm -f docker-web
  docker run -d -p 8000:80 --name docker-web docker-workstation:1.0
  ```

- 결과

  브라우저에서 `http://localhost:8000`을 새로고침한 후 수정된 웹페이지 내용이 표시되는 것을 확인했습니다. 이 과정을 통해 Dockerfile로 복사한 파일을 변경할 때는 이미지 재빌드와 컨테이너 재생성이 필요하다는 점을 확인했습니다.


## 12. Git 설정

Git 사용자 이름과 이메일을 설정하고, 새 저장소의 기본 브랜치가 `main`이 되도록 구성했습니다. 개인정보 보호를 위해 실제 이름과 이메일 값은 문서에 기록하지 않았습니다.

```powershell
git config --global user.name "<사용자 이름>"
git config --global user.email "<이메일 주소>"
git config --global init.defaultBranch main
git init
```

로컬 Git 저장소를 초기화한 뒤 GitHub 원격 저장소를 연결하고 `main` 브랜치를 push했습니다.

### 12.1 Git 설정 검증 로그

Git 설정을 검증하기 위해 다음 명령을 실제로 실행했습니다.

```powershell
git config --list
```

```text
user.name=<사용자 이름 마스킹>
user.email=<이메일 주소 마스킹>
init.defaultbranch=main
```

전체 출력 중 과제 검증에 필요한 핵심 설정만 발췌했으며, 실제 사용자 이름과 이메일 주소는 노출하지 않고 마스킹했습니다. 실제 출력에는 설정 범위에 따라 `init.defaultbranch=master`와 `init.defaultbranch=main`이 모두 존재했지만, 현재 설정한 기본 브랜치가 `main`임을 확인했습니다.

### 12.2 GitHub 및 VS Code 연동

GitHub Repository: [gareal2117/codyssey-mission01-docker-workstation](https://github.com/gareal2117/codyssey-mission01-docker-workstation)

로컬 저장소에 GitHub 원격 저장소를 등록하고 연결 정보를 확인한 뒤 `main` 브랜치를 push했습니다.

```powershell
git remote add origin https://github.com/gareal2117/codyssey-mission01-docker-workstation.git
git remote -v
git push -u origin main
```

`git remote -v` 확인 결과:

```text
origin  https://github.com/gareal2117/codyssey-mission01-docker-workstation.git (fetch)
origin  https://github.com/gareal2117/codyssey-mission01-docker-workstation.git (push)
```

push 결과:

```text
main -> main push 성공
local main branch가 origin/main을 tracking하도록 설정됨
```

VS Code의 Source Control 화면에서도 현재 프로젝트가 Git으로 관리되고 있으며 `main` 브랜치를 사용하고 있음을 확인했습니다.

![GitHub 및 VS Code 연동 증거](./screenshots/github-vscode-integration.PNG)

## 13. 재현 방법

저장소를 clone한 후 프로젝트 루트로 이동하여 다음 명령을 실행합니다.

```powershell
git clone https://github.com/gareal2117/codyssey-mission01-docker-workstation.git
cd codyssey-mission01-docker-workstation
docker build -t docker-workstation:1.0 .
docker run -d -p 8000:80 --name docker-web docker-workstation:1.0
```

브라우저에서 `http://localhost:8000`에 접속하면 정적 웹페이지를 확인할 수 있습니다.

실행 검증은 Windows Docker Desktop에서 수행했습니다. Dockerfile의 `COPY app/ /usr/share/nginx/html/`은 Docker 빌드 컨텍스트 기준 상대경로를 사용하므로 `C:\...` 같은 호스트 절대경로에 의존하지 않습니다.

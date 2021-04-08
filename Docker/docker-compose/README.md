# Docker Compose

## Install

### Mac

Docker 설치하면 자동으로 설치되어 있다.

### Linux

```bash
# Run this command to download the current stable release of Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.6/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Apply executable permissions to the binary
sudo chmod +x /usr/local/bin/docker-compose

# Test the installation.
docker-compose --version
```

> If the command docker-compose fails after installation, check your path. You can also create a symbolic link to /usr/bin or any other directory in your path.

> For example `$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`

## docker-compose alias

> [참고 블로그](https://dohk.tistory.com/191)

```bash
# bash : ~/.bashrc 파일 가장 아랫 부분에 다음 내용을 추가
# zsh : ~/.zshrc 파일 가장 아랫 부분에 다음 내용을 추가
alias dco='docker-compose'

# bash
source ~/.bashrc

# zsh
source ~/.zshrc

dco
```

## docker-compose를 사용하는 이유

> [참고 링크](https://dco-inflearn-amamov.netlify.app/)

### 1. docker 실행 명령어를 일일이 입력하기가 복잡해서

### 2. 컨테이너끼리 연결하기 편해서

### 3. 특정 컨테이너끼리만 통신할 수 있는 가상 네트워크 환경을 편리하게 관리하고 싶어서

### 4. 이 모든 것을 간단한 명령어로 관리하고 싶어서

<br>

---

<br>

## docker 명령을 docker-compose로 옮기기

호스트의 60080 포트를 컨테이너의 80포트로 연결하고 index.html 파일을 만들고 nginx 접속시 이 파일이 나타나게 해보자.

- 이미지 : nginx:latest
- listening port : 80
- HTML 경로 : `/usr/share/nginx/html`

### docker로 nginx 컨테이너 실행하기

```bash
vi index.html # 현재 경로에 index.html 파일 만들고 내용 작성

docker run --rm -p 60080:80 -v $(pwd):/usr/share/nginx/html nginx

# localhost:60080에 접속시에 내용을 확인할 수 있다.
```

### docker-compose로 nginx 컨테이너 실행하기

```bash
mkdir nginx-test
cd nginx-test
vi index.html # index.html 파일 만들고 내용 작성
vi docker-compose.yml
```

```yml
# docker-compose.yml

# docker-compose의 버전에 따라 여러 차이가 있기 때문에 버전 명시를 해준다.
version: "3"

services:
  nginx: # 컨테이너 이름
    image: nginx
    ports:
      - 60080:80
    volumes:
      - ./:/usr/share/nginx/html
```

```bash
docker-compose up
```

<br>

---

<br>

## docker-compose 명령어 정리

```shell
# 필요한 이미지를 다운받는다.
docker-compose pull [service]
```

```shell
# 필요한 이미지를 빌드한다.
docker-compose build [service]
```

```shell
# 서비스를 구동한다.
# 서비스와 네트워크가 없으면 만들고, 이미지가 없으면 빌드한다.
# --build : 강제로 이미지를 다시 빌드한다.
# --force-recreate : 컨테이너를 새로 생성한다.
# -d : 데몬 모드로 실행한다.
docker-compose up [service]
```

```shell
# 현재 실행중인 서비스 목록을 보여준다.
docker-compose ps
```

```shell
# 로그를 확인할 수 있다.
# -f :  로그 계속 보기
docker-compose logs [service]
```

```shell
# 서비스 내에서 실행 중인 프로새스 목록을 보여준다.
docker-compose top
```

```shell
# 서비스를 멈춘다.
docker-compose stop [service]
```

```shell
# 멈춰 있는 서비스의 컨테이너를 실행한다.
docker-compose start [service]
```

```shell
# 해당 서비스에 새로운 컨테이너를 하나 더 실행한다.
# -e : 환경 변수를 설정한다.
# -p : 연결할 포트를 설정한다.
# --rm : 컨테이너 종료시 자동으로 삭제한다.
docker-compose run {service} {command}
```

```shell
# 해당 서비스의 컨테이너에서 명령어를 실행한다.
# -e : 환경 변수를 설정한다.
docker-compose exec {service} {command}
```

```shell
# 서비스를 멈추고 컨테이너를 삭제한다.
# -v : 도커 볼륨도 함께 삭제한다.
docker-compose down
docker-compose down {service}
```

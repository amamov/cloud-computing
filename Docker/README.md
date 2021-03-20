# Docker

- [Azure Container Learn](https://docs.microsoft.com/ko-kr/learn/paths/administer-containers-in-azure/)

- Docker는 앱과 그 실행환경/OS를 모두 포함한 소프트웨어 패키지(**Docker Image**)를 만들어주는 프로그램입니다.

- Docker는 플랫폼에 상관없이 실행될 수 있는 애플리케이션 컨테이너를 만들어줍니다.

  - 윈도우에서 exe 파일을 바로 실행하듯이, Docker를 지원하는 플랫폼에서 Docker Image를 받아서 즉시 실행할 수 있습니다.

- Docker Image는 Container의 형태로 Docker Engine이 있는 어디에서나 실행이 가능합니다.

  - 대상 : 로컬 머신 (윈도우/맥/리눅스), Azure, AWS 등
  - 하나의 Docker Image를 통해 다수의 Container를 생성할 수 있습니다.
  - `nginx`, `nginx:1.17.9`, `python:3.9.0a4-buster` 등 다양한 도커 Image들이 있습니다.
  - docker image name은 `repository:tag` 형식입니다. 여기서 repository는 hostname/accountname/name 구조입니다.
    - `name`으로 쓸 경우 : Docker Hub에 공개된 공식 Image (ex. `nginx`)
    - `accountname/name`으로 쓸 경우 : Docker Hub에 공개된 일반 유저의 Image (ex. `amamov/nginx`)
    - `hostname/accountname/name`으로 쓸 경우 : Docker Hub가 아닌 다른 Docker Registory에 공개된 Image
    - **tag는 Docker Image의 버전을 지정하는 용도로 사용합니다.**

- 어떤 추가적인 소프트웨어 없이 Docker Image만 pull해서 바로 run할 수 있습니다.

- Docker 컨테이너는 **하나의 Forefround 프로세스**를 구동하는 것이 원칙입니다.
- Docker는 리눅스의 chroot 기반의 기술입니다.

<br>

---

<br>

## Docker Registry

- Docker Registry는 Docker Image 저장소를 의미합니다.
- 공식 저장소로서 Docker Hub (Docker계의 Github)가 있습니다. 이 외에도 각 클라우드 벤더에서 저장소를 지원해줍니다.
  - Azure Container Registry
  - AWS Elastic Container Registry
  - Google Container Registry

<br>

---

<br>

## Docker 설치

- 각 OS에 맞게 Docker를 설치하고, 버전을 확인한다.
  - "Error response from daemon: Bad response from docker engine"와 같은 메세지가 출력될 경우, docker daemon이 실행 중이 아니거나, root 권한이 필요할 수 있다. linux / mac에서는 `$ sudo docker --version` 명령어를 사용한다.
- MAC : 그냥 다운받으면 된다.

<br>

---

<br>

## Docker 명령어 정리

- `$ docker version` : 클라이언트와 서버의 버전을 알 수 있다.

<br>

- `$ docker run 이미지이름` : Image를 받아서 해당 이미지 기반의 Container를 실행한다.

  - `$ docker run --name 컨테이너이름 이미지이름`
  - `$ docker run --rm 이미지이름` : 컨테이너를 생성하고 실행이 끝나면 바로 지운다.

<br>

- `$ docker container ls` : 컨테이너 목록들을 확인할 수 있다.

  - `$ docker container ls -a` : 모든 컨테이너 목록들을 확인할 수 있다.

<br>

- `$ docker ps` : 현재 실행중인 컨테이너 목록들을 확인할 수 있다.

  - `$ docker ps -a` : 모든 컨테이너 목록들을 확인할 수 있다.

<br>

- `$ docker images` : 현재 Image 목록을 확인할 수 있다.

<br>

- `$ docker container rm 컨테이너_ID` : 컨테이너를 제거한다.

<br>

- `$ docker pull nginx` : nginx Image를 받아온다.

<br>

- `$ docker run --rm --detach --publish 8000:80 nginx`
  - `--name` : 옵션으로 컨테이너 이름을 지정할 수 있다.

<br>

---

<br>

## Docker Playground

- `$ docker run --rm -it python:3.9` : python 3.9 버전 사용하기

  - `-it` : 표준 입출력이 가능하게 된다.

<br>

- `$ docker run --detach --publish 8080:80 --name mynginx nginx` : nginx 웹 서버 띄우기

  - nginx 이미지를 통해 mynginx Container 적재 및 실행
  - nginx 이미지는 80포트로 listen으로 세팅되어 있다.
  - host의 8080포트와 container의 80포트를 연결
  - `http://localhost:8080` 주소로 접속 가능
  - `$ docker stop mynginx` : mynginx container 정지
  - `$ docker rm mynginx` : mynginx container 삭제

<br>

- `$ docker run -d -p 8080:80 --volume 'pwd'/html:/usr/share/nginx/html --name mynaginx nginx` : (MAC/Linux) 현재 디렉터리내 html/index.html을 서버에 올린다.

<br>

---

<br>

## Docker로 프로젝트 폴더를 docker hub에 올리기

[Docker Hub 링크](https://hub.docker.com)

**Dockerfile은 Docker 이미지를 만들 때, 수행할 명령과 설정들을 시간 순으로 기술한 파일이다.**

아래와 같은 디렉터리 구조가 있다고 하자.

- `my_test_project`
  - `backend`
  - `requirements.txt`
  - `Dockerfile`

backend는 Django기반의 앱이고 Dockerfile은 아래와 같이 쓰였다.

```Dockerfile
### docker biuld
# ubunto의 18년도 4월 버전 OS를 사용한다.
FROM ubunto:18.04

# 빌드시에 수행할 명령
RUN apt-get update && \
    apt-get install -y python3-pip python3-dev && \
    apt-get clean

# code 이름의 work directory를 지정한다. 만약 폴더가 없다면 자동 생성한다.
WORKDIR /code/

# ./backend/requirements.txt 파일을 code에 복사한다.
ADD ./backend/requirements.txt /code/

# 프로젝트 패키지를 설치한다.
RUN pip3 install -r requirements.txt

# ./backend 폴더를 code에 복사한다.
ADD ./backend /code/

### docker run
# 컨테이너에서 구동할 때 8000번 포트에서 구동할 것이다.
EXPOSE 8000

# 서버를 띄운다. (실서비스에서는 gunicorn이나 uwsgi를 사용한다.)
CMD ["python3", "/code/manage.py", "runserver", "0.0.0.0:8000"]
```

<br>

1. `$ docker build -t test_dj .`

   - 현재 디렉터리를 test_dj라는 이미지로 빌드한다.
   - `-t` : `--tag`와 같다. 이미지 이름을 설정한다.
   - `.` : 현재 디렉터리를 의미한다.

   - 빌드한 후 `$ docker run --rm -p 9000:8000 test_dj` 으로 로컬서버를 열 수 있다.
     - `-p` : `--publish`를 의미한다.

<br>

2. `$ docker tag test_dj amamov/test_dj_2020`
   - docker hub에 올리기 위해 이미지를 태그한다. 애초에 빌드할 때 `$ docker build -t amamov/test_dj .`해도 된다.
   - `$ docker tag <옵션> <이미지 이름>:<태그> <저장소 주소, 사용자명>/<이미지 이름>:<태그>`
   - `$ docker tag test_dj amamov/test_dj_2020:0.1`으로 0.1 태그를 붙일 수 있다. 여기서 0.1태그는 버전정보를 의미한다.

<br>

3. `$ docker login`

<br>

4. `$ docker push amamov/test_dj_2020`
   - docker hub에 올린다.

<br>

---

<br>

## Docker Hub에 업로드한 Image를 실제 웹 서버(클라우드)에서 사용하기

[AWS - lightsell](https://lightsail.aws.amazon.com/ls/webapp/create/instance?region=ap-northeast-2)에서 VM(Virtual Machine)을 사용할 수 있다. (EC2에 비해 확장성이 떨어지지만 저렴하게 사용할 수 있다.)

"AWS - lightsell"에서 Ubunto 가상환경을 선택하고 인스턴스를 생성한 후, 콘솔을 실행한다.

1. `$ sudo apt-get update`

   - 기본적으로 우분투가 내부적으로 설치할 패키지 목록을 OS가 가지고 있는데 이 목록을 업데이트한다.

<br>

2. `$ sudo apt-get install docker.io`

   - Docker를 설치한다.

<br>

3. `$ docker version`

   - docker 버전을 알 수 있다.

<br>

4. `$ sudo docker run --rm hello-world`

   - "hello-world" 이미지를 받아서 실행한다.(테스트)

<br>

5. `$ sudo docker run --rm --publish 80:8000 amamov/test_dj_2020:0.1`
   - docker-hub에 있는 test_dj_2020 이미지를 받아서 실행한다.
   - 이는 콘솔을 닫으면 서버도 꺼진다. 이러한 단점을 보완하기 위해 "docker swarm"을 사용한다.
   - "docker swarm"이란 도커가 공식적으로 만든 오케스트레이션 툴이다. 오케스트레이션 툴이란 여러 호스트 서버의 컨테이너들을 배포 및 관리를 위한 툴이다. ["docker swarm 링크"](https://docs.docker.com/engine/swarm/)

<br>

6. `$ sudo docker swarm init`

   - docker swarm을 초기화한다.
   - VM이 3대가 있다고 했을 때 그중에 한 대가 매인이 되어야한다. 매인 VM에서 init을 수행한다. 나머지 두 대의 VM에서는 매인 VM에 속하게 하기 위해 `$ docker swarm join`명령을 사용한다.

<br>

7. `$ sudo docker node ls`

   - 클러스터가 여러 개이면 묶여 있는 다른 여러 머신들이 보인다.

<br>

8. `$ sudo docker service create --replicas 1 --name test_dj --publish 80:8000 amamov/test_dj_2020:0.1`

   - 컨테이너를 docker swarm이 관리하게 된다.
   - `--replicas` : 컨테이너를 몇개 만들어서 분산할 것인가
   - `test_dj` : 서비스 이름

<br>

9. `$ sudo docker service ls`

   - 서버가 열린 것을 확인할 수 있다.

<br>

10. `$ sudo docker service update --image amamov/test_dj_2020:0.2 test_dj(서비스이름)`

    - "docker service update"로 업데이트한 소프트웨어를 재배포할 수 있다. 입력한 명령어는 0.1에서 0.2로 업데이트를 한 것이다. 반대로 0.2에서 0.1로 롤백도 할 수 있다.

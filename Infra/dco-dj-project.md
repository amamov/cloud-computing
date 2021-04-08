# docker-compose = django + nginx + gunicorn + oracle

## 1. VPS(ubuntu) 접속

## 2. 필요한 패키지 셋업

```bash
sudo apt update && sudo apt -y upgrade
sudo apt install -y python3 python3-pip vim git nginx

# install docker
curl -fsSL https://get.docker.com/ | sudo sh
sudo usermod -a -G docker $USER
sudo systemctl enable docker
sudo systemctl start docker

# install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.6/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 재부팅
sudo reboot

# 설치 확인
docker version
docker-compose version
```

```bash
## docker-compose alias

# ~/.bashrc 파일 가장 아랫 부분에 다음 내용을 추가
alias dco='docker-compose'
source ~/.bashrc

dco
```

## 3. django project dockerizing

```bash
mkdir project-server
cd project-server

git clone GIT_PROJECT_REPO
cd GIT_PROJECT_REPO # 앞으로 GIT_PROJECT_REPO를 PROJECT라고 부르겠음
vi Dockerfile
vi .dockerignore
```

- **프로젝트 구조**
  - project-server
    - PROJECT
      - Dockerfile
      - .dockerignore
      - ...

```Dockerfile
# project-server/PROJECT/.dockerignore

/media
/static
db.sqlite3
.env
```

```Dockerfile
# project-server/PROJECT/Dockerfile

FROM python:3.9.4

ENV PYTHONUNBUFFERED 1

RUN apt-get -y update
# RUN apt-get -y install vim

# 이미지 안에 /srv/docker-server 폴더 생성
RUN mkdir /srv/docker-server

# 현재 호스트 디렉토리를 srv/docker-server 폴더에 복사
ADD . /srv/docker-server

# 작업 디렉토리 설정
WORKDIR /srv/docker-server

RUN pip install --upgrade pip
RUN pip install -r requirements.txt

EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

```bash
# PROJECT 이미지 빌드

# 경로 : project-server/PROJECT/
sudo docker build -t PROJECT .
```

```bash
# 빌드된 이미지 체크

docker images
```

```bash
# 컨테이너 실행으로 확인

docker run --rm -p 8000:8000 PROJECT
```

## 3. nginx project dockerizing

```bash
# ~/project-server
mkdir nginx && cd nginx
vi nginx.conf
vi nginx-app.conf
```

```conf
# ~/project-server/nginx/nginx.conf

user root;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 1024;
    # multi_accept on;
}

http {
     ## # Basic Settings ##
     sendfile on;
     tcp_nopush on;
     tcp_nodelay on;
     keepalive_timeout 65;
     types_hash_max_size 2048;
     # server_tokens off;
     # server_names_hash_bucket_size 64;
     # server_name_in_redirect off;

     include /etc/nginx/mime.types;
     default_type application/octet-stream;

     ## # SSL Settings ##
     # Dropping SSLv3, ref: POODLE
     ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
     ssl_prefer_server_ciphers on;

     ## # Logging Settings ##
     access_log /var/log/nginx/access.log;
     error_log /var/log/nginx/error.log;

     ## # Gzip Settings ##
     gzip on;
     gzip_disable "msie6";
     # gzip_vary on;
     # gzip_proxied any;
     # gzip_comp_level 6;
     # gzip_buffers 16 8k;
     # gzip_http_version 1.1;
     # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;


     ## # Virtual Host Configs ##
     # include /etc/nginx/conf.d/*.conf;
     include /etc/nginx/sites-enabled/*;
}
```

```bash
# ~/docker-server/nginx/nginx-app.conf

# nginx + django + gunicorn

upstream web {
    server django:8000
}

server {
    listen 80;

    charset utf-8;
    client_max_body_size 128M;

    location = /favicon.ico { access_log off; log_not_found off; }

    location / {
        proxy_pass http://web;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }
    }

    location /media/ {
        alias /srv/docker-server/config/.media/;
    }

    location /static/ {
        alias /srv/docker-server/config/.static/;
    }
}
```

위의 설정 파일을 해석하면 nginx 해당하는 server_name의 80번 포트로 접속하면 web으로 요청을 돌린다. web는 위에서 정의한 upstream이다. 즉, 해당하는 server_name의 80번 포트로 접속하면 nginx는 Reversy Proxy를 이용해서 `server django:8000;`에게 요청을 전달해주게 된다.

```Dockerfile
# ~/docker_server/nginx/Dockerfile
FROM nginx:latest

COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx-app.conf /etc/nginx/sites-available/

RUN mkdir -p /etc/nginx/sites-enabled/
RUN ln -s /etc/nginx/sites-available/nginx-app.conf /etc/nginx/sites-enabled/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```bash
docker build -t PROJECT-nginx .
```

```bash
docker run --rm -p 4000:80 PROJECT-nginx

# host not found in upstream "django:8000" error가 나면 정상이다.
```

## 4. set up docker-compose

```bash
# ~/project-server

vi docker-compose.yml
```

```yml
# ~/project-server/docker-compose.yml

version: "3"
services:
  nginx:
    container_name: nginx
    build: ./nginx
    image: PROJECT-nginx
    restart: always
    ports:
      - "80:80"
    volumes:
      # 호스트의 PROJECT 폴더를 nginx container의 /srv/docker-server와 마운트
      - ./PROJECT:/srv/docker-server
      # 호스트의 log 폴더를 nginx container의
      - ./log:/var/log/nginx
    depends_on:
      - django

  django:
    container_name: django
    build: ./PROJECT
    image: PROJECT
    restart: always
    command: gunicorn config.wsgi:application --bind 0.0.0.0:8000
    ports:
      - 8000:8000
    env_file:
      - ./.env.prod
    volumes:
      - ./PROJECT:/srv/docker-server
      - ./log:/var/log/guinicorn
```

````

```Dockerfile
# ~/docker_server/nginx/Dockerfile

FROM nginx:latest

COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx-app.conf /etc/nginx/sites-available/

RUN mkdir -p /etc/nginx/sites-enabled/
RUN ln -s /etc/nginx/sites-available/nginx-app.conf /etc/nginx/sites-enabled/

# EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
````

```Dockerfile
# project-server/PROJECT/Dockerfile

FROM python:3.9.4

ENV PYTHONUNBUFFERED 1

RUN apt-get -y update
RUN mkdir /srv/docker-server

ADD . /srv/docker-server

WORKDIR /srv/docker-server

RUN pip install --upgrade pip
RUN pip install -r requirements.txt

# EXPOSE 8000
# CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

```bash
docker-compose up -d --build
```

<br>

---

<br>

## use jc21/nginx-proxy-manager

> [nginx-proxy-manager](https://nginxproxymanager.com/)

> [docker-bub : jc21/nginx-proxy-manager](https://hub.docker.com/r/jc21/nginx-proxy-manager)

```json
// ~/project-server/config.json

{
  "database": {
    "engine": "mysql",
    "host": "db",
    "name": "npm",
    "user": "amamov",
    "password": "Passw0rd!",
    "port": 3306
  }
}
```

```yml
# ~/project-server/docker-compose.yml

version: "3"
services:
  app:
    image: "jc21/nginx-proxy-manager:latest"
    restart: always
    ports:
      # Public HTTP Port:
      - "80:80"
      # Public HTTPS Port:
      - "443:443"
      # Admin Web Port:
      - "81:81"
    environment:
      # These are the settings to access your db
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "amamov"
      DB_MYSQL_PASSWORD: "Passw0rd"
      DB_MYSQL_NAME: "npm"
      DISABLE_IPV6: "true"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db
  db:
    image: "jc21/mariadb-aria:latest"
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: "amdinpassword!"
      MYSQL_DATABASE: "npm"
      MYSQL_USER: "amamov"
      MYSQL_PASSWORD: "Passw0rd@!"
    volumes:
      - ./data/mysql:/var/lib/mysql
```

```bash
docker-compose up -d --build
```

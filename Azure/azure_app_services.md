# DRF + React 앱 배포

> Docker + Azure App Services + S3(Azure Storage) + DB

- [Azure portal](https://portal.azure.com/#home)
- [AWS Console](https://aws.amazon.com/ko/console)

<br>

## Static & Media 파일 올리기

static 파일과 media 파일을 배포할 때 [django-storages](https://django-storages.readthedocs.io/en/latest/) 라이브러리를 사용하면 AWS-S3, Azure-strage, Google-cloud-storage 등을 쉽게 구축할 수 있다.

### Azure Storage 사용하기

1. `$ pip install "django-storages[azure]"` : `django-storages` 라이브러리 설치

2. `backend/config/storages.py` 파일을 만들고 설정을 위해 객체를 정의한다.

```python
# backend/config/storages.py

from storages.backends.azure_storage import AzureStorage


class StaticAzureStorage(AzureStorage):
    azure_container = "static"


class MediaAzureStorage(AzureStorage):
    azure_container = "media"
```

3. `backend/config/prod.py` 파일을 다음과 같이 설정한다.

```python
# backend/config/prod.py

import os
from .common import *

DEBUG = False

ALLOWED_HOSTS = os.environ.get("ALLOWED_HOSTS", "").split(",")

STATICFILES_STORAGE = "config.storages.StaticAzureStorage"
DEFAULT_FILE_STORAGE = "config.storages.MediaAzureStorage"

## 소스 코드에 공개하면 절대 안된다. (환경 변수로 지정)
AZURE_ACCOUNT_NAME = os.environ["AZURE_ACCOUNT_NAME"] # collectstatic 명령시에는 직접 작성
AZURE_ACCOUNT_KEY = os.environ["AZURE_ACCOUNT_KEY"] # collectstatic 명령시에는 직접 작성

LOGGING = {
    "version": 1,
    "disable_exiting_loggers": False,
    "handlers": {
        "console": {
            "level": "ERROR",
            "class": "logging.StreamHandler",
        },
    },
    "loggers": {
        "django": {
            "handlers": ["console"],
            "level": "ERROR",
        },
    },
}
```

4. [Azure](https://portal.azure.com/?l=en.en-us#home)에 접속하여 `Storage accounts` 서비스의 `Containers`를 사용한다.
   1. `Storage accounts`를 생성한다.
   2. 생성한 `Storage accounts`에서 "static" 이름의 `Container`를 생성한다. (`Public access level`(공용 엑세스 수준) : Bolb(누구나 파일을 읽을 수 있도록))
   3. 생성한 `Storage accounts`에서 "media" 이름의 `Container`를 생성한다. (`Public access level`(공용 엑세스 수준) : Bolb(누구나 파일을 읽을 수 있도록))

`$ python manage.py collectstatic --settings=config.settings.prod` 명령으로 `Storage accounts`에 빌드할 수 있다.

### AWS S3 사용하기

1. `$ pip install django-storages`

2. Azure Storage와 비슷한 방법으로 올리면 된다. 필자의 프로젝트중 hotel-api repository에 관련 파일이 있다.

<br>

---

<br>

## Django 프로젝트를 Dockerizing하기

### 빠르게 보기 + DB

```python
# backend/prod.txt

-r common.txt

django-storages
gunicorn
psycopg2-binary
# mysqlclient
```

```Dockerfile
# backend/Dockerfile

FROM ubuntu:18.04

# default set up
RUN apt-get update && apt-get install -y python3-pip && apt-get clean

# use mysql set up
#RUN apt-get update && \
#    apt install -y gcc python3-dev python3-pip mysql-client-core-5.7 libmysqlclient-dev && \
#    apt-get clean

WORKDIR /djangoproject
ADD . /djangoproject
RUN pip3 install -r requirements.txt

ENV PYTHONUNBUFFERED=1

EXPOSE 80
CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:80"]
```

```.dockerignore
/media
/static
db.sqlite3
```

```python
# backend/config/settings/prod.py

import os
from .common import *

# Django
SECRET_KEY = os.environ["SECRET_KEY"]

DEBUG = False

ALLOWED_HOSTS = os.environ.get("ALLOWED_HOSTS", "").split(",")

# CORS
CORS_ALLOWED_ORIGINS = os.environ.get("CORS_ALLOWED_ORIGINS", "").split(",")


# DRF
REST_FRAMEWORK["DEFAULT_RENDERER_CLASSES"] = [
    "rest_framework.renderers.JSONRenderer",
]

# Static & Media : AWS S3
STATICFILES_STORAGE = "config.storages.S3StaticStorage"
DEFAULT_FILE_STORAGE = "config.storages.S3MediaStorage"
AWS_ACCESS_KEY_ID = os.environ["AWS_ACCESS_KEY_ID"]
AWS_SECRET_ACCESS_KEY = os.environ["AWS_SECRET_ACCESS_KEY"]
AWS_S3_REGION_NAME = os.environ["AWS_S3_REGION_NAME"]  # ap-northeast-2 버킷 리전
AWS_STORAGE_BUCKET_NAME = os.environ["AWS_STORAGE_BUCKET_NAME"]  # 버킷이름
AWS_S3_CUSTOM_DOMAIN = (
    f"{AWS_STORAGE_BUCKET_NAME}.s3.{AWS_S3_REGION_NAME}.amazonaws.com"  # 버킷에 대한 URL 도메인
)
AWS_DEFAULT_ACL = os.environ["AWS_DEFAULT_ACL"]  # "public-read" 버켓에 대한 엑세스 권한

# Database : default : postgredb
DATABASES = {
    "default": {
        "ENGINE": os.environ.get("DB_ENGINE", "django.db.backends.postgresql"),
        "HOST": os.environ["DB_HOST"],
        "PORT": os.environ["DB_PORT"],
        "NAME": os.environ["DB_NAME"],
        "USER": os.environ["DB_USER"],
        "PASSWORD": os.environ["DB_PASSWORD"],
    }
}


# Logging
LOGGING = {
    "version": 1,
    "disable_exiting_loggers": False,
    "handlers": {"console": {"level": "ERROR", "class": "logging.StreamHandler",},},
    "loggers": {"django": {"handlers": ["console"], "level": "ERROR",},},
}
```

[DB 셋업하기↓](#postgresql-db와-연동하고-docker를-통해-마이그레이션-수행하기)

**Azure에서 기본적으로 PostgreSQL DB 만들 때 DB_NAME의 기본값은 postgres이고 이는 수정이 가능하다.**

1. `$ docker build -t amamov/myapp:1.0.0 .`

2. Container 실행

```shell
$ docker run --rm --publish 9000:80 \
    -e SECRET_KEY="" \
    -e ALLOWED_HOSTS="" \
     -e CORS_ALLOWED_ORIGINS="" \
    -e AWS_ACCESS_KEY_ID="" \
    -e AWS_SECRET_ACCESS_KEY="" \
    -e AWS_S3_REGION_NAME="" \
    -e AWS_STORAGE_BUCKET_NAME="" \
    -e AWS_DEFAULT_ACL="" \
    -e DB_NAME= \
    -e DB_USER= \
    -e DB_PASSWORD= \
    -e DB_HOST= \
    -e DB_PORT= \
    -it amamov/myapp:1.0.0 sh
```

3. 수행할 명령어

```shell
$ python3 manage.py collectstatic --settings=config.settings.prod
$ python3 manage.py migrate --settings=config.settings.prod
$ python3 manage.py createsuperuser --settings=config.settings.prod
```

### 천천히 음미하기

현재 프로젝트구조가 다음과 같다고 하자.

- `my-project`

  - `backend`
    - `config`
    - `manage.py`
    - `app1`
    - `app2`
    - `requirements.txt`
    - `Dockerfile`
    - `.dockerignore`
  - `frontend`

```Dockerfile
# backend/Dockerfile

FROM ubuntu:18.04

RUN apt-get update && apt-get install -y python3-pip && apt-get clean

WORKDIR /djangoproject
ADD . /djangoproject
RUN pip3 install -r requirements.txt

# ENV PYTHONUNBUFFERED=1 환경변수 설정을 하면 파이썬 표준 출력 과정에서 버퍼링을 하지 않고 바로 출력할 수 있다.
ENV PYTHONUNBUFFERED=1

## docker run 할 때 -e으로 환경 변수 설정을 해준다.
# ENV AZURE_ACCOUNT_NAME=
# ENV AZURE_ACCOUNT_KEY=

EXPOSE 80
CMD ["python3", "manage.py", "runserver", "0.0.0.0:8000"]
```

```Dockerfile
# backend/.dockerignore

/media
/static
db.sqlite3
```

이 상태에서 `$ docker build -t amamov_dj .` 명령으로 빌드해준다. (`amamov/app:1.0.0`처럼 바로 tag해도 된다.)

빌드 후에 다음 명령으로 환경 변수를 설정하여 로컬에서 서버를 열 수 있다.

```shell
docker run --rm --publish 9000:8000 \
    -e DJANGO_SETTINGS_MODULE=config.settings.prod  \
    -e AZURE_ACCOUNT_NAME=amamov \
    -e AZURE_ACCOUNT_KEY="1f9pfWaKKin7cnC4OeUye5SxaqwUn911j2SjG+qdYubOyg==" \
    -it amamov_dj
```

<br>

이제 python3이 아닌 gunicorn을 사용하여 서버를 열어보자.

`requirements.txt`에 gunicorn을 추가한다.

```Dockerfile
# backend/Dockerfile

FROM ubuntu:18.04

RUN apt-get update && apt-get install -y python3-pip && apt-get clean

WORKDIR /djangoproject
ADD . /djangoproject
RUN pip3 install -r requirements.txt

ENV PYTHONUNBUFFERED=1

EXPOSE 80
CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:80"]
```

Dockerfile을 수정하고 다시 빌드한 후 run하면 서버가 열린다.

<br>

---

<br>

## PostgreSQL DB와 연동하고 Docker를 통해 마이그레이션 수행하기

1. [Azure Database for PostgreSQL servers](https://portal.azure.com/)의 DB에서 `Single Server`를 선택하고 리소스 그룹을 선택하고(static, media와 같은 그룹을 선택하면 좋다.) DB 서버를 만든다.

   - `Connection security`에서 `Add current client IP address`를 체크해준다.

2. `psycopg2-binary` 라이브러리를 추가한다.

```python
# backend/prod.txt

-r common.txt

django-storages[azure]
gunicorn
psycopg2-binary
```

3. 설정 파일에 DB 설정을 추가한다.

```python
# backend/config/prod.py

import os
from .common import *

DEBUG = False

ALLOWED_HOSTS = os.environ.get("ALLOWED_HOSTS", "").split(",")

STATICFILES_STORAGE = "config.storages.StaticAzureStorage"
DEFAULT_FILE_STORAGE = "config.storages.MediaAzureStorage"

AZURE_ACCOUNT_NAME = os.environ["AZURE_ACCOUNT_NAME"]
AZURE_ACCOUNT_KEY = os.environ["AZURE_ACCOUNT_KEY"]

DATABASES = { # DB 설정 추가
    "default": {
        "ENGINE": os.environ.get("DB_ENGINE", "django.db.backends.postgresql"),
        "NAME": os.environ.get("DB_NAME", "postgres"),
        "USER": os.environ["DB_USER"], # Admin username
        "HOST": os.environ["DB_HOST"], # Server name
        "PASSWORD": os.environ["DB_PASSWORD"],
    }
}

CORS_ALLOWED_ORIGINS = os.environ.get("CORS_ALLOWED_ORIGINS", "").split(",") # CORS 설정 추가

LOGGING = {
    "version": 1,
    "disable_exiting_loggers": False,
    "handlers": {
        "console": {
            "level": "ERROR",
            "class": "logging.StreamHandler",
        },
    },
    "loggers": {
        "django": {
            "handlers": ["console"],
            "level": "ERROR",
        },
    },
}
```

4. docker로 빌드하고 서버 실행해보기
   `$ docker build -t amamov_dj .` 명령어로 빌드한 후 다음 명령어로 환경변수 설정을 하고 shell을 실행한다.

**Azure에서 기본적으로 PostgreSQL DB 만들 때 DB_NAME의 기본값은 postgres이고 이는 수정이 가능하다.**

```shell
docker run --rm --publish 9000:80 \
    -e DJANGO_SETTINGS_MODULE=config.settings.prod \
    -e AZURE_ACCOUNT_NAME=amamov \
    -e AZURE_ACCOUNT_KEY="1f9pfTYFh78+S1XlYnD1am11j2SjG+qdYubOyg==" \
    -e ALLOWED_HOSTS=localhost \
    -e DB_HOST=amamov.postgres.database.azure.com \
    -e DB_USER=amamov@amamov-todolist-db \
    -e DB_PASSWORD=ss981205@! \
    -e DB_NAME=postgres \
    -e CORS_ALLOWED_ORIGINS="http://localhost:9000" \
    -it amamov/todolist_dj:0.3 sh
```

shell을 실행하고 다음 명령으로 마이그레이션을 한다.

```shell
$ python3 manage.py migrate
$ python3 manage.py createsuperuser
```

이제 다음 명령으로 서버를 구동해보자.

```shell
docker run --rm --publish 9000:80 \
    -e DJANGO_SETTINGS_MODULE=config.settings.prod \
    -e AZURE_ACCOUNT_NAME=amamov \
    -e AZURE_ACCOUNT_KEY="1f9pfTYFh78+S1XlYnD1am11j2SjG+qdYubOyg==" \
    -e ALLOWED_HOSTS=localhost \
    -e DB_HOST=amamov.postgres.database.azure.com \
    -e DB_USER=amamov@amamov-todolist-db \
    -e DB_PASSWORD=ss981205@! \
    -e DB_NAME=postgres \
    -e CORS_ALLOWED_ORIGINS="http://localhost:9000" \
    amamov/todolist_dj:0.3
```

<br>

---

<br>

## Azure PaaS 서비스에 Dockerizing Django 서비스 배포하기

1. **`$ docker push amamov/instagram_dj:0.1`으로 Docker Hub에 push한다.**

   - `$ docker tag amamov_dj amamov/instagram_dj:0.1`으로 `amamov/instagram_dj:0.1` 이미지를 만들고 push한다.

2. Azure에서 `App Services`서비스를 이용하여 App을 생성한다.

   - `Publish` : `Docker Container` 선택
   - `OS` : `Linux` 선택
   - `Docker` 탭 : `Image source`는 `Docker Hub` 선택, `Image and tag`는 `amamov/instagram_dj` 선택
   - 생성 후 App 설정의 `컨데이너 설정` 탭에서 Docker Container 이미지를 업데이트할 수 있다.

3. 생성한 App에 환경 변수를 설정해준다.

   1. 만들어진 App에 들어가서 설정-`Configuration`(구성) 탭에 들어간다.
   2. `New application setting`을 클릭해서 환경변수를 입력한다.
      - `ALLOWED_HOSTS`에는 App 호스트을 입력하면 된다.(ex. amamov.azurewebsites.net) (`*` : 모든 호스트를 허용)

4. DB에서 App service의 ip를 허용해준다.

   1. `App service`의 `Properties`(속성)의 `Additional Outbound IP Address`를 복사한다.
      - `Virtual IP Adress`보다 안전하다.
   2. 복사한 IP 주소를 `DB Service`(Azure Database for PostgreSQL server) - `Settings` - `Connection security` - `Fire rule name`(방화벽 규칙)에 추가한다. (하나하나 세심하게 지정해야 한다.)
      - 이름은 `AppService01, 02, ..`이런식으로 지장해주고 IP를 하나하나 넣어준다. 시작 IP와 종료 IP는 동일하게 지정해주면 된다.

<br>

---

<br>

## React 프로젝트를 Azure Storage에 배포하기

React를 배포하는 방법엔 여러가지 방법(githubio, netlify 등)이 있다. 이중에 Azure Storage를 사용하여 배포하는 방법을 진행해보자.

1. VS-Code에서 `Azure Storage` Extension을 다운받고 storage account를 생성한다.

   1. `command + shift + a` 단축키로 들어갈 수 있다.
   2. azure 익스텐션에 들어가서 로그인해준다.
   3. **`+`버튼을 누르고 storage account를 만들어준다.**

2. frontend 폴더를 열고 `.env.production`파일을 만들고 실제 빌드할 호스트 환경변수를 설정한다.

   - `REACT_APP_API_HOST="https://amamov-instagram.azurewebsites.net`

3. `$ yarn build`로 프런트엔드 프로젝트를 빌드한다.

   - `$ yarn global add serve`로 로컬에서 서버를 열 수 있는 serve 패키지를 다운받는다.
   - frontend 폴더에서 `$ serve -s build`로 build 폴더를 서빙할 수 있다.

4. VS-Code의 azure 익스텐션에 들어가서 `+`버튼 옆의 배포 버튼(`↑`)을 누르고 build 폴더를 배포한다.
   - Azure에서 `Storage Account`에서 배포된 것을 확인할 수 있다.

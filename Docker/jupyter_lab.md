# Docker를 사용하여 Jupyter LAB 서비스 구축

## 볼륨 마운트 옵션 사용해 로컬 파일 공유하기 (Volume Mount)

도커 이미지로 컨테이너를 생성하면 이미지는 읽기 전용이 되며 컨테이너의 변경사항만 별도로 각 컨테이너의 정보로 보존한다.

예를 들어 mysql 이미지의 경우 이미지에는 mysql을 실행하는데 필요한 애플리케이션 정보만 들어있고, 컨테이너를 생성하여 mysql 컨테이너에는 쓰기모드가 가능하여 여러 데이터가 저장된다.

하지만 만일 도커 컨테이너를 삭제한다면, 컨테이너 계층의 데이터도 모두 삭제 된다.

그렇기 때문에 데이터의 영속성을 유지해야 하는데 이때 볼륨을 통해 쉽게 활용할 수 있다.

- 볼륨을 활용하는 방법
  1. 호스트와 볼륨을 공유
  2. 볼륨 컨테이너를 활용
  3. 도커가 관리하는 볼륨을 생성

### 호스트 볼륨 공유

```shell
docker run -e MYSQL_ROOT_PASSWORD=1234 -v /Users/chul/Documents/Docker/volume:/var/lib/mysql mysql
```

해당 경로에 파일이 생성된 것을 확인 할 수 있다.

혼동할 수 있는 점은 `/var/lib/mysql` 을 동기화 하는 것이 아닌 완전히 같은 디렉토리라는 것이다.

호스트 디렉토리가 없다면, 생성하고 컨테이너 내부의 디렉토리는 삭제하게 된다.

만일 컨테이너 내부에 디렉토리가 존재하고, 호스트 볼륨공유를 통해 호스트 디렉토리를 지정하면 컨테이너 디렉토리는 덮어씌워진다. (Mount)

```shell
docker run -v <호스트 경로>:<컨테이너 내 경로>:<권한>
# docker run -v /tmp:home/user:ro
```

- 권한의 종류
  - `ro` : 읽기 전용
  - `rw` : 읽기 및 쓰기

#### nginx 볼륨 마운트하기

```bash
sudo docker run -d -p 80:80 --rm -v /var/www:/usr/share/nginx/html:ro nginx
curl 127.0.0.1
# /var/www 안에 index.html 파일 생성
# echo helloWorld > /var/www/index.html # 써도 되고
cd /var/www/ && vi index.html
curl 127.0.0.1
```

<br>

---

<br>

## Jupyter LAB 환경 구축하기

### [Jupyter Docker Hub](https://hub.docker.com/r/jupyter/datascience-notebook)

현재 디렉토리를 사용하여 notebook 컨테이너 실행

```bash
mkdir ~/jupyternotebook
chmod 777 ~/jupyternotebook
cd ~/jupyternotebook

# "$PWD":... 의미 : 현재 디렉토리에 마운트
sudo docker run --rm -p 8080:8888 -e JUPYTER_ENABLE_LAB=yes -v "$PWD":/home/jovyan/work:rw jupyter/datascience-notebook:9b06df75e445
```

실행하면 나오는 링크를 통해 접속한다.

```bash
http://127.0.0.1:8080/token=??????????????????
```

주피터랩 서버로 접속해서 work으로 접속하고 새 노트북 생성하고 코드를 작성한다.

```python
print("Hello Python Jupyter Notebook")
```

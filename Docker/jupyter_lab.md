# Docker를 사용하여 Jupyter LAB 서비스 구축

## [Jupyter Docker Hub](https://hub.docker.com/r/jupyter/datascience-notebook)

## 볼륨 마운트 옵션 사용해 로컬 파일 공유하기 (Volume Mount)

- 도커 컨테이너에서 작성되거나 수정된 파일

  - 컨테이너가 파기된다면?
    - 호스트에서도 함께 삭제된다.
    - 호스트 쪽 파일 시스템에 마운트한다면 컨테이너에서 삭제해도 호스트 파일 시스템에 남아있게 된다.
    - 이때 사용하는 것이 데이터 볼륨
    - 마운트한다. ("USB로 연결한다.")

- 윈도우와 도커 간의 공유 기능

```shell
docker run -v <호스트 경로>:<컨테이너 내 경로>:<권한>
# docker run -v /tmp:home/user:ro
```

- 권한의 종류
  - `ro` : 읽기 전용
  - `rw` : 읽기 및 쓰기

### nginx 볼륨 마운트하기

```bash
sudo docker run -d -p 80:80 --rm -v /var/www:/usr/share/nginx/html:ro nginx
curl 127.0.0.1
# /var/www 안에 index.html 파일 생성
# echo helloWorld > /var/www/index.html # 써도 되고
cd /var/www/ && vi index.html
curl 127.0.0.1
```

### Jupyter LAB 환경 구축하기

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

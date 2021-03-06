# ec2 = django (virtualenv) + gunicorn + nginx

## 1. EC2 인스턴스 생성 후 셋팅

1. [aws console](https://console.aws.amazon.com/)
2. 우측 상단 지역이 Seoul 인지 확인
3. EC2 검색
4. Instances > Launch Instances
5. Ubuntu Server 20.04 LTS (64-bit x86)
6. t2.micro (Free tier eligible)
7. Storage 설정 (Free Tier 은 30GB까지)
8. Security Group > Add Rule

   1. HTTP
   2. HTTPS
   3. Custom -> TCP -> 8000 ->0.0.0.0/0, ::/0

9. Review and Launch > Launch

   1. Create a new key pair
   2. 이름은 원하는 이름 설정
   3. Download Key Pair
   4. Launch Instance

10. Elastic IP 받기

    1. Network & Security > Elastic IPs
    2. Allocate Elastic IP address
    3. Allocate
    4. Associate Elastic IP address
    5. Instance > 아까 생성된 인스턴스 선택

    6. Associate

11. SSH 연결하기

    1. `chmod 400 다운받은_PEM_FILE.pem`
    2. `ssh -i 다운받은_PEM_FILE ubuntu@아까_받은_Elastic_IP`

12. 필요한 패키지 설치

    1. `sudo apt update && sudo apt -y upgrade`
    2. `sudo apt install -y python3 python3-pip vim git nginx`
    3. `sudo pip3 install virtualenv`
    4. `sudo reboot` # 재부팅

13. ec2에 project 셋업

    1. `git clone GIT_PROJECT_REPO`
    2. `cd GIT_PROJECT_REPO` # 앞으로 GIT_PROJECT_REPO를 PROJECT라고 부르겠음
    3. `virtualenv venv --python=python3` # 가상환경 생성
    4. `source venv/bin/activate` # 가상환경 활성화
    5. `pip3 install -r requirements.txt` # 패키지 다운
       - `pip3 install gunicorn` # requirements.txt에 있겠지만 혹시나 다운
    6. `python manage.py runserver 0.0.0.0:80`
    7. `http://My_Elastic_IP`에 접속해서 프로젝트가 뜨는지 확인
    8. `deactivate` # 가상환경 탈출
    9. `venv/bin/gunicorn config.wsgi -b 0.0.0.0` # 여기서 config는 `django-admin startproject config`의 config를 의미
    10. `http://My_Elastic_IP`에 접속해서 프로젝트가 뜨는지 확인

## 2. gunicorn 셋업

1. `sudo vi /etc/systemd/system/gunicorn.service` # 서비스 등록 스크립트 생성

```js
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
Type=notify
RuntimeDirectory=gunicorn
WorkingDirectory=/home/ubuntu/PROJECT
ExecStart=/home/ubuntu/PROJECT/venv/bin/gunicorn config.wsgi
ExecReload=/bin/kill -s HUP \$MAINPID
KillMode=mixed
TimeoutStopSec=5
PrivateTmp=true
EnvironmentFile=/etc/gunicorn/env.conf

[Install]
WantedBy=multi-user.target
```

2. `sudo vi /etc/systemd/system/gunicorn.socket`

```js
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock
SocketUser=www-data

[Install]
WantedBy=sockets.target
```

3. `sudo mkdir /etc/gunicorn`

4. `sudo vi /etc/gunicorn/env.conf`

```python
# 환경 변수 여기에 입력
DB_NAME=django
DB_USER=django
DB_PASSWORD='p@ssword!'
DB_HOST=localhost
...
```

5. 생성한 서비스들 실행

   1. `sudo systemctl daemon-reload`
   2. `sudo systemctl enable --now gunicorn.socket`
   3. `sudo systemctl enable --now gunicorn`

   > 만일 Django 서비스를 재배포해야할 경우 `sudo systemctl restart gunicorn.service` 명령어를 사용하면 된다.

6. 테스트

   1. `curl --unix-socket /run/gunicorn.sock http` # 실행시 HTML 코드가 나오면 성공

7. `systemctl status gunicorn` # 상태 확인

8. `sudo vi /var/log/syslog` # 구동 실패시 에러 로그는 확인 가능.

## 3. nginx 셋업

1. `sudo apt -y install nginx` # 아까 설치했지만...

   - `sudo chmod 644 /var/log/nginx`
     - nginx 로그가 쌓이는 곳을 다른 곳에서 접근할 수 있도록 설정
   - `sudo chown -R ubuntu:ubuntu /usr/share/nginx/html`
     - nginx가 사용하는 홈 디렉터리가 /usr/share/nginx/html인데, 이 디렉터리를 ec2-user(ubuntu)가 사용할 수 있도록 만들어준다.

2. `sudo vi /etc/nginx/sites-available/PROJECT_NAME`

```nginx
server {
    listen 80;
    server_name My_Elastic_IP;

    location = /favicon.ico { access_log off; log_not_found off; }

    location /static/ {
        root /home/ubuntu/PROJECT/config;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

3. `sudo ln -s /etc/nginx/sites-available/PROJECT_NAME /etc/nginx/sites-enabled` # sites-enabled에 바로가기 추가

   - [리눅스 심볼릭 링크](https://server-talk.tistory.com/140)
   - [nginx sites-available vs sites-enabled](https://forteleaf.tistory.com/entry/nginx-site-enabled-site-availablemd)

4. `sudo nginx -t` # nginx 설정 문법 검사 및 재기동

5. `sudo systemctl restart nginx` # nginx 설정 문법 검사 및 재기동

6. `http://My_Elastic_IP` 접속시 Django 화면이 보이면 성공!

- [과정중 bug fix 레퍼런스](https://youtu.be/Z-eTvYwWhuc)

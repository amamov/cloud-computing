# EC2

## ec2 주피터 노트북을 사용하여 환경 셋팅하기

1. `sudo apt install python3-pip`

2. `sudo pip3 install notebook`

3. `jupyter notebook` 비밀번호 설정

**비밀번호 해시값 메모해두기**

```shell
ubuntu@...
Python 3.8.5 (default, Jan 27 2021, 15:41:15)
>>> from notebook.auth import passwd
>>> passwd()
Enter password:
Verify password:
'asadas:$argon2idas$v=19$..............w'
>>> quit()
```

4. `jupyter notebook --generate-config`

5. `sudo vi /home/ubuntu/.jupyter/jupyter_notebook_config.py`

```python
# 맨 마지막에 다음을 추가
c = get_config()
c.NotebookApp.password = u'asadas:$argon2idas$v=19$..............w'
c.NotebookApp.ip='172.22.38.188' # 자신의 ip 입력 (ec2 ip 주소 말고)
```

6. `sudo jupyter-notebook --allow-root --ip=0.0.0.0`

7. ec2 인바인드 규칙에서 8888포트 접속을 허용해준다.

8. `sudo netstat -nap | grep 8888` # 8888 포트에 실행되고 있는 서버를 확인한다.

9. `sudo kill -9 16120` # 해당 포트를 꺼준다.

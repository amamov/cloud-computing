# systemd

systemd는 최종적으로 모든 프로세스들을 관리하는 init 시스템이다.

여기서 init 프로세스는 시스템 부팅 과정의 최초 프로세스이자 시스템이 종료될 때 까지 계속 실행되며 모든 프로세스의 부모 노릇을 하는 프로세스다.

> 데몬(daemon)이란?
> 멀티태스킹 운영체제에서 데몬은 백그라운드에서 돌면서 여러 작업을 하는 프로그램이다.
> 데몬은 네트워크 요청, 하드웨어 동작, cron처럼 주기적인 작업을 실행하는 등 다양한 목적으로 사용된다.

- systemd에는 여러가지 Unit 파일이 존재하할 수 있으며 각 Unit은 이름만으로도 그 특징을 알 수 있다.

  - Service unit : `.service` 시스템 서비스
  - Target unit : `.target` systemd unit의 그룹
  - Socket unit : `.socket` 내부 프로세스 통신 소켓

- Unit은 세 가지 섹션으로 구성되어 있다.
  - **Unit**
    - Unit의 type에 의존적이지 않는 일반적인 옵션을 포함
    - 이 옵션은 Unit을 설명하고, Unit의 동작을 정의
    - 다른 Unit과의 Dependency를 설정
  - **Unit type (ex. service, socket, target, ...)**
    - 특정 Unit이 type-specific 지시자를 가지고 있다면 해당 옵션에 대한 항목을 참조하여 Unit 파일을 작성하거나 파악해야 함
  - **Install**
    - install 관련 정보를 담고 있음
    - systemctl enable과 systemctl disable에 의한 동작이 기술되어 있음

## Commands

- `systemctl status <서비스 명>`
  - 서비스의 현재 상태를 체크
- `systemctl start <서비스 명>`
  - 서비스의 시작을 명령
- `systemctl stop <서비스 명>`
  - 서비스의 정지를 명령
- `systemctl restart <서비스 명>`
  - 서비스의 재시작을 명령
- `systemctl reload <서비스 명>`
  - 서비스의 갱신을 명령
- `systemctl kill <서비스 명>`
  - 즉시 서비스를 중지하고, 관련 프로세스 모두 종료
- `systemctl enable <서비스 명>`
  - boot 시 서비스 시작 활성화
- `systemctl disable <서비스 명>`
  - boot 시 서비스 시작 비활성화
- `systemctl is-enable <서비스 명>`
  - boot 시 실행되도록 설정되어 있는지 여부 확인
- `systemctl is-active <서비스 명>`
  - 현재 실행되고 있는지 여부 확인
- `systemctl list-dependencies <서비스 명>`
  - 행당 서비스와 의존성 관계에 있는 리스트를 출력
- `systemctl list-units --type <타입>`
  - 특정 타입에 해당하는 Unit의 리스트를 출력
- `systemctl list-unit-files`
  - 시스템에 있는 모든 Unit의 리스트와 상태를 출력
- `systemctl list-sockets`
  - 리스닝(Listening)하는 소켓(socket) 관련 목록을 출력

<br>

## Unit 섹션 옵션

- **Description**

  - Unit에 대한 전반적인 설명을 기술
  - 이 영역은 systemctl status 명령어를 사용할 때 표시됨

- **Documenttation**

  - Unit에 대한 문서를 참조할 만한 URI 리스트를 기재함

- **Requires**

  - Requires에 정의된 Unit 리스트에 의존성이 있음을 말함
  - **Requires 리스트에 있는 Unit은 이 Unit과 같이 활성화 됨**
  - 만약 Requires에 있는 Unit이 fail이 된다면 이 Unit도 활성화하지 못함

- **Wants**

  - Requires 보다는 약한 의존성을 나타냄
  - Wants에 있는 Unit이 fail이 되더라도 이 Unit을 활성화 할 수 있음

- **After**

  - After에 정의된 Unit이 활성화된 이후에 Unit이 활성화되어야 함
  - Requires와는 다르게 After는 After에 명시된 Unit을 활성화 시키지는 않음
  - Before는 After와 반대되는 기능의 옵션임

- **Conflicts**
  - Requires와 반대로 여기에 명시된 Unit이 활성화되어 있다면 이 Unit을 활성화할 수 있음

## Unit type 섹션 옵션

- **Type**

  - Type Unit의 시작 Type을 지정함
    - simple : 기본적인 형태이며 ExecStart에 있는 것이 메인 프로세스임
    - forking : 이미 시작된 ExecStart 프로세스에서 자식(child) 프로세스를 생성하고 그 자식 프로세스가 서비스의
  - main 프로세스가 됨. 프로세스가 시작되면 부모 프로세스는 사라짐
    - oneshot : simple과 유사. 다음 Unit이 시작되기 전에 사라짐
    - dbus : simple과 유사. 메인 프로세스가 D-Bus라는 이름을 얻게 될 경우에만 다음 Unit이 시작됨
    - notify : simple과 유사. `sd_notify()` Function을 이용해서 메시지가 전달된 이후에만 다음 Unit이 시작됨
    - idle : simple과 유사. 모든 job이 끝날 때가지 실질적인 실행이 지연됨

- **ExecStart**

  - Unit이 시작되고 난 후 실행될 스크립트 커맨드를 정의

- **ExecStop**

  - Unit이 정지되고 난 후 실행될 스크립트 커맨드를 정의

- **Restart**

  - 이 옵션이 정의되면 해당 프로세스가 모두 종료된 후 서비스가 재시작됨

- **RemainAfterExit**

  - True로 설정하게 되면 프로세스가 종료된 후에도 서비스가 활성화되어 있는 것으로 간주함(기본값 false)

## Install 섹션 옵션

- **Alias**

  - 이 Unit의 별명을 설정할 수 있으며 각 별명은 space로 구분됨

- **RequiredBy**

  - 이 Unit에게 의존성이 있는 Unit 리스트임
  - 이 Unit이 활성화되면 RequiredBy 리스트에 있는 Unit들은 Require를 얻음

- **WantedBy**

  - 이 Unit에게 Wants를 갖는 Unit 리스트임
  - 이 Unit이 활성화되면 WantedBy 리스트에 있는 Unit들은 Want를 얻음

- **Also**

  - Also 리스트에 있는 Unit은 이 Unit이 삭제되거나 설치될 때 같이 삭제되거나 설치됨

- **DefaultInstance**

  - Unit이 활성화 될 때 인스턴스화 될 Unit을 제외함

<br>

## Example

```python
# /etc/systemd/system/서비스명.service (gunicorn.service)

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

```python
# /etc/systemd/system/서비스명.service (gunicorn.socket)

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock
SocketUser=www-data

[Install]
WantedBy=sockets.target
```

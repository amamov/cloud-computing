# docker-compose.yml 문법

- 확장자는 `.yml`
- key-value는 `:` 기호로 구분 (딕셔너리)
- 블록 내에서는 두 칸 더 들여쓰기
- 목록은 `-` 기호 사용
- 주석은 `#` 기호 사용

```yml
version: "3"

services:
  mysql:
    image: mysql:5.7
    ports:
      - 3306:3306
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=1205
      - MYSQL_DATABASE=wp
      - MYSQL_USER=wp
      - MYSQL_PASSWORD=wp

  wordpress:
    image: wordpress
    ports:
      - 4001:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db:3306
      - WORDPRESS_DB_USER=wp
      - WORDPRESS_DB_PASSWORD=wp
      - WORDPRESS_DB_NAME=wp

volumes:
  db_data: {}
```

## Version

```yml
version: "{버전}"
```

docker 문법의 버전을 의미한다. 각 버전벌로 필요로하는 docker-engine이 다르고 지원하는 문법이 다르다.

> ex. compose file format 3.6에 대응되는 docker engine release는 18.02.0+이다.

## Service

실행할 container 정의

```yml
services:
  # docker run --name {컨테이너 이름}
  { 컨테이너 이름 }:
    # 컨테이너에서 사용할 이미지 이름과 테그, 이미지가 없으면 자동으로 pull
    image: { 이미지 이름 }

    # 이미지를 자체 빌드 후 사용, image 속겅 대신에 사용한다.
    # 별도의 dockerfile이 필요하다.
    build: { dockerfile build 할 시 디렉토리 }

    # 컨테이너와 연결할 포트(들)
    ports:
      - "{호스트 포트(외부 포트)}:{컨테이너 포트}"

    # 컨테이너에서 사용할 환경변수(들)
    environment:
      - { 환경 변수 이름 } = { 값 }
      - DB_PASSWORD=ldhnaslkdn

      { 환경 변수 이름 } : { 값 }
      DB_USER : amamov

    # 마운트하려는 디렉터리(들)
    volumes:
      - { 호스트 디렉터리 }:{ 컨테이너 디렉터리 }
      - ./app/log:/var/log

    # 다른 컨테이너와 연결할 때 사용 (레거시)
    # 일반적으로는 docker-compose 내부에서 모든 컨테이너가 서로 연결될 수 있기 때문에 요즘에는 잘 사용하지 않는다.
    link:
      - { 연결할 컨테이너 이름 }:{ 해당 컨테이너에서 참조할 이름 }
      - mysql:db

    # 컨테이너들의 실행 순서를 정의할 수 있다.
    # 현재 블록의 컨테이너를 django라고 하자.
    depends_on:
      - mysql # django 컨테이너는 mysql 컨테이너에 의존하고 있기 떄문에 mysql 컨테이너가 먼저 실행이되고 django 컨테이너가 나중에 실행된다.

    restart: always

    driver: { 네트워크 이름 }
```

### 컨테이너의 실행시 타이밍 문제와 restart 옵션

테이터베이스 컨테이너가 시작한다고 해서 준비 과정이 모두 끝나는 것은 아니다.

데이터베이스 컨테이너가 시작하고

데이터베이스 초기화가 진행되고야 비로소

데이터베이스가 준비가 완료된다.

![docker-compose](../../images/d5.png)

따라서 이 경우, 컨테이너가 예기치 않게 종료 되었을 떄 어떻게 할 것인지 결정이 필요하다.

```yml
services:
  web:
    ...
    restart: always
```

- `no` : (기본 값) 다시 시작하지 않음

- `always` : 항상 다시 시작

- `on-failure` : 오작동하면서 종료되었을 때만 다시 시작

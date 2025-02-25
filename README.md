# InfluxDB 

### 개요
이 문서는 InfluxDB 2.0을 Docker Compose를 이용하여 손쉽게 배포하고 실행하는 방법을 설명합니다.
Docker의 secrets 기능을 활용하여 보안성을 높였으며, 프로젝트 구조를 명확하게 정리하였습니다.

- 🔗 공식 문서: InfluxDB Docs
- 🐳 Docker Hub: InfluxDB Docker Image

### 0. 작업할 폴더 생성 및 경로 이동

Influxdb 도커 설정할 폴더 생성 

```bash
$ mkdir influxdb
$ cd influxdb
```

### 1. docker-compose.yml 작성

도커 파일 위 문서에 나온 내용대로 작성한다

```yaml
# compose.yaml
services:
  influxdb2:
    image: influxdb:2
    ports:
      - 8086:8086
    restart: always # 항상 재실행 설정
    environment:
      DOCKER_INFLUXDB_INIT_MODE: setup
      DOCKER_INFLUXDB_INIT_USERNAME_FILE: /run/secrets/influxdb2-admin-username
      DOCKER_INFLUXDB_INIT_PASSWORD_FILE: /run/secrets/influxdb2-admin-password
      DOCKER_INFLUXDB_INIT_ADMIN_TOKEN_FILE: /run/secrets/influxdb2-admin-token
      DOCKER_INFLUXDB_INIT_ORG: <초기ORGANIZATION명>
      DOCKER_INFLUXDB_INIT_BUCKET: <초기버킷값>
    secrets:
      - influxdb2-admin-username
      - influxdb2-admin-password
      - influxdb2-admin-token
    volumes:
      - type: volume
        source: influxdb2-data
        target: /var/lib/influxdb2
      - type: volume
        source: influxdb2-config
        target: /etc/influxdb2

secrets:
  influxdb2-admin-username:
    file: ./env/.env.influxdb2-admin-username
  influxdb2-admin-password:
    file: ./env/.env.influxdb2-admin-password
  influxdb2-admin-token:
    file: ./env/.env.influxdb2-admin-token

volumes:
  influxdb2-data:
  influxdb2-config:

```

![1000009784.jpg](attachment:7387ab55-ac5f-42ef-a4c8-3d275fe6602a:1000009784.jpg)

이 내용으로 작성하는데 공식문서와 다른점은 다음과 같다.

- secrets 부분 현재 경로에서 env 폴더로 인증 계정 연결처리
    - influxdb2-admin-username:
        - file: ./env/.env.influxdb2-admin-username
    - influxdb2-admin-password:
        - file: ./env/.env.influxdb2-admin-password
    - influxdb2-admin-token:
        - file: ./env/.env.influxdb2-admin-token

### 2. 디렉터리 구조에 따른 계정 설정

프로젝트 구조는 다음과 같다

```bash
~/Dockers/influxdb/
├── docker-compose.yml
├── env
│   ├── .env.influxdb2-admin-password
│   ├── .env.influxdb2-admin-token
│   └── .env.influxdb2-admin-username
├── influxdb2-config
└── influxdb2-data

```

위 설정한 screts 내용을 반영하기 위해 계정 정보를 생성한다

```java
$ mkdir env
$ echo <계정아이디> > ./env/.env.influxdb2-admin-username  
$ echo <계정비밀번호> > ./env/.env.influxdb2-admin-password
$ echo <계정토큰값> > ./env/.env.influxdb2-admin-token

# Docker는 보안상 Secrets 파일의 권한을 엄격히 체크합니다.
$ sudo chmod 600 env/.env.influxdb2-admin-*
```

### 3. docker compose 실행 & 접속

도커 컴포즈 이용하여 실행해본다. 첫 실행이니 만큼 -d 하지 말고 실행해본다

```bash
$ docker-compose pull
$ docker-compose up
```

로그 내용 잘 보고 잘 로그 잘 찍히고 잘 실행 되면 [http://localhost:8086](http://localhost:8056) 접속 후 위 계정 정보에 따라 접속 잘 되면 정상적으로 잘 적용이 된 것이다!

![image.png](attachment:6e68233e-8164-45e2-95fd-7eb54b21510f:image.png)

![image.png](attachment:ee0540bb-ff5f-4e82-b706-325a44d4eb1f:image.png)

![image.png](attachment:f14f144e-7238-40a9-9034-57045d32fdf7:image.png)

### 기타

설정 중 잘 안되었을 경우 다음 명령어로 기존 볼륨 설정과 시크릿 설정등 모두 초기화하는 설정으로 컨테이너 중단을 해야 꼬임 없이 초기 값 잘 지정하여 

1. **컨테이너를 완전히 중지 및 삭제**

```bash
$ docker compose down --volumes --remove-orphans
```

2. **Docker Compose가 기존의 캐시를 버리고 다시 읽도록 빌드**

```bash
$ docker compose up --force-recreate
```

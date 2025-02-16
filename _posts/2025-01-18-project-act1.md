---
layout: post
title: "배포 과정 정리"
categories: [troubleshooting]
project: act
---

# GCP 로 배포하기

### 환경설정

- GCP 에서 VM 을 만들어주고 나면, SSH로 접속해서 라이브러리를 설치해주면 된다.
- 백엔드 서버 먼저 배포했다.

```bash
$ sudo apt install docker.io
```

### 프로젝트 폴더에 Dockerfile 생성

- Docker 환경을 설정하기 위한 Dockerfile과 docker-compose.yml 파일을 루트 디렉토리에 추가해야 한다.
- 이때 .env 파일을 .dockerignore 파일에 추가하고, SSL 인증서가 있는 certs 파일을 볼륨에 추가해야 한다.

##### docker-compose.yml 파일 작성하기

```yml
version: "3"
services:
app:
  build: .
  ports:
    - "443:443"
  environment:
    - NODE_ENV=production
  volumes:
    - .:/usr/src/app
    - /usr/src/app/node_modules
    - ./certs:/usr/src/app/certs:ro
    # 위 부분은 ./certs 디렉토리를 컨테이너의 /usr/src/app/certs 디렉토리에 읽기 전용(ro)으로 마운트하는 코드다.
  command: npm start
```

- 보안상 인증서 파일을 Docker 이미지에 직접 포함시키는 것은 좋지 않다. 대신 위 코드처럼 볼륨을 사용하여 인증서 파일을 컨테이너에 마운트하는 것이 더 안전하다.

### 이후 Docker 환경에서 Docker 이미지를 빌드하고 컨테이너를 시작하면 된다.

## 깃 레포에 커밋 한 후

- 인스턴스에서 pull 해보기
- 먼저 인스턴스에 도커 환경이 잘 있는지 확인을 해보자:

  ```bash
  name@actapp8:~$ sudo docker ps
  CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
  ```

  - 위와 같이 뜨면 잘 있다는 뜻.
  - 좀 전에 머지한 도커 파일들을 pull 하자. 이때 관리자 권한으로 pull 해야지 편하게 할 수 있다

```bash
$ sudo su
root@actapp8:/home/name/foldername/backend# git pull origin main
```

## **이제 빌드 시작**

도커 이미지 빌드

```bash
docker-compose build
```

도커 컨테이너 실행: app 실행할 때 아래 명령어를 치면 된다.

```bash
docker-compose up -d
```

로그 확인 (에러 확인용)

```bash
docker-compose logs -f
```

컨테이너 중지:

```bash
docker-compose down
```

## Docker 환경에서는 Node 설치 X

- Dockerfile에서 npm install을 실행하므로, 호스트 시스템에서 별도로 실행할 필요가 없다.
- 서버 구동도 Dockerfile이나 docker-compose.yml에 정의되어 있다.

### 우리 앱에서 필요 없었던던 부분

**현재 기술 스택**: React Native 와 Node.js 백엔드 (도커 환경에서 잘 동작한다.)

- Docker Compose: 빠른 개발, 배포 가능

#### **배포 환경 둘러보기**

- bootjar : 현재 앱에서 필요 X
- 쿠버네티스: 현재 앱에서 필요 X
  - 현재 앱에서는 사용자 수가 적기 때문에 단일 서버 도커로 커버 가능함.
  - 따라서 쿠버버네티스 패키지 관리자인 Helm 도 필요 없음
- Kafka: 현재 앱에서 필요 X
  - 분산 스트리밍 플랫폼으로, 대규모 스트림을 처리하는 데 사용.
  - 실시간 데이터 스트리밍이 필요하거나, 대규모 이벤트 처리나 로그 집계가 필요하거나, 마이크로서비스 간 비동기 통신이 필요한 경우 사용하도록.
  - _만약 우리 앱에서 사용자 추천 알고리즘이 실시간 데이터 처리를 필요로 한다면 Kafka가 유용할 수 있다._

## 배포 오류

1.  컨테이너가 시작됐다가 바로 종료됨
    <div style="display: flex; gap: 10px;">
         <img src="images/pj/pj1.png" width="600" height="100">
    </div>

    - 로그확인 -> 파일 이름 문제였다; vs 로 고쳐도 반영 안 됐음... 어쩔 수 없이 로컬에서의 폴더 위치에서 git bash 열어서 이름 수동으로 수정!
    - 유저 깃 로그에서 직접 대문자로 수정 후 컨테이너 재빌드했더니, 빌드 완료!

## 배포 상태 확인

- 443 포트 확인하는 명령어 : sudo lsof -i :443
- 컨테이너 성공적으로 시작, 443 포트가 도커 프록시에 의해 사용중인 것이 확인됨

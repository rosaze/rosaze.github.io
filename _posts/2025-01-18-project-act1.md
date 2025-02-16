---
layout: post
title: "배포 과정 정리"
categories: [troubleshooting]
project: act
---

# GCP 로 배포하기

### 환경설정

- GCP 에서 VM 을 만들어주고 나면, SSH로 접속해서 라이브러리와 필요한 패키지를 설치하면 된다.
  - 다운로드 파일에 실행 권한 부여하고, 설치가 성공적으로 완료되었는지 확인하기!
- 백엔드 서버 먼저 배포했다.

```bash
$ sudo apt install docker.io
$ sudo apt-get install -y curl
$ sudo chmod +x /usr/local/bin/docker-compose
```

### 프로젝트 폴더에 Dockerfile 생성

- 서버 구동, 환경 관련 정보는 Dockerfile이나 docker-compose.yml에 정의되어 있다.
- Docker 환경을 설정하기 위한 Dockerfile과 docker-compose.yml 파일을 루트 디렉토리에 추가해야 한다.
- 두 파일의 NODE_ENV 를 development 또는 production 으로 둘 다 동일하게 맞춰야 함.
- 이때 .env 파일을 .dockerignore 파일에 추가하고, SSL 인증서가 있는 certs 파일을 볼륨에 추가해야 한다.[^1]

##### docker-compose.yml 파일 작성하기

```yml
version: "3.4"
services:
  actbackend:
    image: actbackend
    build:
      context: .
      dockerfile: ./Dockerfile
    environment:
      NODE_ENV: development
    volumes:
      - .:/usr/src/app
      - /usr/src/app/node_modules
      - ./certs:/usr/src/app/certs:ro
      # 위 부분은 ./certs 디렉토리를 컨테이너의 /usr/src/app/certs 디렉토리에 읽기 전용(ro)으로 마운트하는 코드다.
    ports:
      - 443:443
      - 9229:9229
    command: ["node", "app.js"]
    restart: always
```

- 보안상 인증서 파일을 Docker 이미지에 직접 포함시키는 것은 좋지 않다. 대신 위 코드처럼 볼륨을 사용하여 인증서 파일을 컨테이너에 마운트하는 것이 더 안전하다.
- 버전 관리 관련 파일들이나 개발 환경 설정 파일들을 .dockerignore 파일에 추가해야 한다.

### 이후 Docker 환경에서 Docker 이미지를 빌드하고 컨테이너를 시작하면 된다.

## 깃 레포에 커밋 한 후

**인스턴스에서 pull하기.**

- 좀 전에 머지한 도커 파일들을 pull 해보자.
- 인스턴스에서 pull 할 때 현재 사용자가 .git 디렉토리의 파일을 수정할 권한이 없다고 떠서 소유권을 변경한 뒤에 다시 pull을 시도했어야 했다.

```bash
$ sudo chown -R host:host /home/name/foldername/ac.t_backend
$ sudo chmod -R u+rw host:host /home/name/foldername/ac.t_backend
```

**docker-compose 빌드 하기**

```bash
$ docker-compose build
```

>                       빌드 완료!!!

- 이제 인스턴스에 도커 환경이 잘 있는지 확인을 해보자:

  ```bash
  name@actapp8:~$ sudo docker ps
  CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
  ```

  - 잘 있다는 뜻.

## 도커 컨테이너 실행

app 실행할 때 아래 명령어를 치면 된다.

```bash
docker-compose up -d
```

로그 확인 (에러 확인용)

```bash
docker-compose logs -f
```

컨테이너 중지:[^2]

```bash
docker-compose down
```

<hr>
**Docker 환경에서는 Node 설치 X**

But 컨테이너 내부에서 의존성을 설치했어야 됐다.

```bash
$ docker-compose run --rm actbackend npm install
```

### 우리 앱에서의 배포

**현재 기술 스택**: React Native 와 Node.js 백엔드 (도커 환경에서 잘 동작한다.)
기능들은 복잡하지만 아직 마이크로서비스 수준까지는 아님.

- Docker Compose: 빠른 개발, 배포 가능
- 원격 서버에서 직접 이미지를 빌드하고 실행했기 때문에 Docker Hub를 사용하지 않았다.
- Docker Hub를 사용하는 경우:
  - 여러 서버에 동일한 환경을 배포할 때
  - CI/CD(지속적 통합/지속적 배포) 파이프라인을 구축할 때

### 배포 환경 둘러보기

- **bootjar** : 현재 앱에서 필요 X
- **쿠버네티스**: 현재 앱에서 필요 X
  - 현재 앱에서는 사용자 수가 적기 때문에 단일 서버 도커로 커버 가능함.
  - 쿠버네티스 실행하려면 상당한 컴퓨팅 리소스 필요.
  - 따라서 쿠버네티스 패키지 관리자인 Helm & Ingress 도 필요 없음
  - 자동 스케일링이 필요하다면 나중에 도입하기로
- **Kafka**: 현재 앱에서 필요 X
  - 분산 스트리밍 플랫폼으로, 대규모 스트림을 처리하는 데 사용.
  - 실시간 데이터 스트리밍이 필요하거나, 대규모 이벤트 처리나 로그 집계가 필요하거나, 마이크로서비스 간 비동기 통신이 필요한 경우 사용하도록.
  - _만약 우리 앱에서 사용자 추천 알고리즘이 실시간 데이터 처리를 필요로 한다면 Kafka가 유용할 수 있다._

<hr>

### Docker Network:

- 날씨 추천 AI 모델을 배포한 flask api는 현재 서비스에서 핵심 기능 중 하나이기 때문에 같은 Docker 네트워크에 두는 것이 낫다고 판단했다. 서버 개발 초보자이기 때문에 환경 관리에 쏟는 시간을 최대한 줄이고 싶었다.
  - 즉 장기적인 관점에서 Docker 네트워크 내에서 직접 호출하는 방식을 선택.

#### 모델을 기존 백엔드 프로젝트에 통합하고 VM 인스턴스에 배포.

- 깃헙에서 날씨 추천 모델을 클론하여 weather_model 이라는 디렉토리에 저장함.
- 다음, docker-compose 파일을 수정하여 두 개의 컨테이너를 정의하고 관리하도록 함.(actbackend,actflaskapi )
- 서비스되는 포트가 다르다.
  - HTTPS 서버가 443 포트에서 실행
  - Gunicorn 서버를 통해 5003 포트에서 실행

<div style="display: flex; gap: 10px;">
         <img src="images/pj/cloud3.png" width="600" height="220">
    </div>

- 성공적으로 컨테이너가 생성됐다. (act_backend-actflaskapi-1와 act_backend-actbackend-1)

## 배포 오류

1.  컨테이너가 시작됐다가 바로 종료됨
    <div style="display: flex; gap: 10px;">
         <img src="images/pj/pj1.png" width="600" height="100">
    </div>

    - 로그확인 -> 파일 이름 문제였다; vs 로 고쳐도 반영 안 됐음... 어쩔 수 없이 로컬에서의 폴더 위치에서 git bash 열어서 이름 수동으로 수정![^3]
    - 유저 깃 로그에서 직접 대문자로 수정 후 컨테이너 재빌드했더니 해결됐다.

2.  Flask api 와의 통신 문제:

- 컨테이너에 모델을 추가한 이후부터는 , Node.js 백엔드 컨테이너에서 Flask API를 호출할 때, 로컬호스트(localhost) 대신 **컨테이너 이름(actflaskapi)**을 사용해야한다.
  - Docker Compose에서 컨테이너는 같은 네트워크(act_network)에 배포되기 때문에, 컨테이너 간 통신은 컨테이너 이름을 통해 이루어진다.
- Flask API의 기본 URL을 http://actflaskapi:5003/api/v1로 설정했는데, 이는 Docker 컨테이너 내부에서의 통신을 위해 사용된 주소이다. 위 주소로 했더니 404 None 에러가 안 뜬다.
- 결론적으로, 외부에서 테스트할 때는 호스트 머신의 IP와 포트 매핑을 해야 한다.

### 배포 상태 확인

- 시스템의 443 포트 여부 확인, 프로세스 확인

```bash
$ sudo lsof -i :443
$ ps aux | grep node
```

- 컨테이너 성공적으로 시작, 443 포트가 도커 프록시에 의해 사용중인 것이 확인됨.

---

{: data-content="footnotes"}
[^1]: Docker 환경에서 HTTPS 서버를 실행하기 위함이다.
[^2]: 컨테이너, 네트워크, 볼륨 모두 제거하려면 docker-compose down -v
[^3]: 자주 발생하는 에러라고 한다.

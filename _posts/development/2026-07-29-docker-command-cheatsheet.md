---
layout: single
title: "Docker 명령어 치트시트 - 컨테이너 실행부터 볼륨 백업까지"
date: 2026-07-29 20:00:00 +0900
categories: development
header:
  teaser: /assets/images/2026/07/29/docker-cheatsheet-hero.png
---

Docker 명령은 하나씩 보면 단순하지만 `run`, `start`, `exec`, `attach`처럼 비슷한 명령이 많아 다시 찾게 된다. 자주 쓰는 명령을 이미지, 컨테이너, 네트워크, 저장소, Compose 순서로 묶었다.

![터미널에 컨테이너와 포트 서비스 볼륨을 연결한 Docker 작업 구조](/assets/images/2026/07/29/docker-cheatsheet-hero.png){: .align-center }

삭제 명령은 마지막에 따로 모았다. 특히 volume은 컨테이너와 수명이 다르므로 지우기 전에 백업 여부를 먼저 확인해야 한다.

## 가장 먼저 확인할 것

이미지는 실행 환경을 만드는 읽기 전용 템플릿이고, 컨테이너는 그 이미지를 실행한 인스턴스다.

| 구분 | 이미지 | 컨테이너 |
| --- | --- | --- |
| 역할 | 실행 환경 템플릿 | 실행 중인 인스턴스 |
| 변경 | 레이어가 고정됨 | 쓰기 가능한 변경 레이어를 가짐 |
| 삭제 | `docker rmi` | `docker rm` |

Docker 엔진 연결부터 확인한다.

```bash
docker --version
docker info
docker context show
```

`docker info`가 서버 정보를 반환하면 CLI가 Docker 엔진과 통신하는 상태다.

## 이미지 관리

```bash
docker pull ubuntu:24.04
docker images
docker images ubuntu:24.04
docker build -t myapp:1.0 .
docker rmi myapp:1.0
```

빌드 캐시를 배제하고 전체 로그를 남기려면 다음처럼 실행한다.

```bash
docker build --no-cache --progress=plain -t myapp:1.0 . 2>&1 | tee docker-build.log
```

## 컨테이너 실행과 상태

```bash
docker run -d --name web -p 18080:8080 myapp:1.0
docker ps
docker ps -a
docker logs web
docker logs -f web
docker stats --no-stream web
docker stop web
docker start web
docker rm web
docker rm -f web
```

### `docker run`에서 자주 쓰는 옵션

| 옵션 | 뜻 |
| --- | --- |
| `-d` | 백그라운드 실행 |
| `--name NAME` | 컨테이너 이름 지정 |
| `--rm` | 종료 시 컨테이너 자동 삭제 |
| `-p 호스트:컨테이너` | 포트 매핑 |
| `-e KEY=VALUE` | 환경 변수 전달 |
| `-v SRC:DST` | bind mount 또는 volume 연결 |
| `-it` | 표준입력과 가상 터미널 사용 |

컨테이너 이름은 정지 상태에서도 중복될 수 없다. 셸 문법을 명령으로 전달할 때는 실행 파일이 아니므로 `sh -c`로 감싼다.

```bash
docker exec web sh -c 'pwd; ls -la /'
```

## `run`, `start`, `exec`, `attach` 차이

| 명령 | 하는 일 | 주의점 |
| --- | --- | --- |
| `docker run IMAGE CMD` | 새 컨테이너 생성 후 PID 1 실행 | 기존 컨테이너를 재사용하지 않음 |
| `docker start CONTAINER` | 정지된 컨테이너의 PID 1 재실행 | `-a` 없이는 출력에 연결하지 않음 |
| `docker exec CONTAINER CMD` | 실행 중인 컨테이너에 새 프로세스 추가 | 메인 프로세스와 별개 |
| `docker attach CONTAINER` | PID 1의 표준입출력에 직접 연결 | `Ctrl+C`가 컨테이너를 끝낼 수 있음 |

대화형 셸은 대부분 `exec`가 안전하다.

```bash
docker exec -it web sh
```

`attach`에서 컨테이너를 종료하지 않고 빠져나오려면 `Ctrl+P`, `Ctrl+Q`를 차례로 누른다.

## 포트 매핑

`-p 18080:8080`은 호스트의 `18080` 포트를 컨테이너의 `8080` 포트에 연결한다. Dockerfile의 `EXPOSE`는 포트를 문서화할 뿐 실제 연결을 만들지 않는다.

```bash
# 외부 인터페이스에도 열림
docker run -d -p 18080:8080 myapp:1.0

# 같은 컴퓨터에서만 접근
docker run -d -p 127.0.0.1:18080:8080 myapp:1.0
```

포트 충돌은 호스트와 Docker 양쪽에서 찾는다.

{% raw %}
```bash
lsof -nP -iTCP:18080 -sTCP:LISTEN
docker ps --format 'table {{.Names}}\t{{.Ports}}'
```
{% endraw %}

## Bind mount와 volume

`-v SOURCE:TARGET`에서 SOURCE가 경로면 bind mount, 이름이면 Docker volume이다.

| 예시 | 종류 | 원본 위치 |
| --- | --- | --- |
| `-v "$PWD/site:/app/site:ro"` | Bind mount | 호스트 디렉터리 |
| `-v "mydata:/data"` | Volume | Docker 관리 저장공간 |

Bind mount는 개발 중인 소스처럼 호스트에서 직접 편집할 파일에, volume은 데이터베이스처럼 Docker가 수명을 관리할 데이터에 잘 맞는다.

```bash
docker volume create mydata
docker volume ls
docker volume inspect mydata
```

### volume 백업과 복원

```bash
# 백업
docker run --rm -v mydata:/source:ro -v "$PWD/backup:/backup" \
  alpine:latest tar -czf /backup/mydata.tar.gz -C /source .

# 복원
docker volume create mydata-restored
docker run --rm -v mydata-restored:/target -v "$PWD/backup:/backup:ro" \
  alpine:latest tar -xzf /backup/mydata.tar.gz -C /target
```

복원한 데이터가 정상인지 확인한 뒤 원본 volume을 삭제한다.

## Dockerfile에서 자주 보는 지시어

| 지시어 | 역할 |
| --- | --- |
| `FROM` | 기반 이미지 또는 빌드 stage 지정 |
| `WORKDIR` | 이후 명령의 작업 디렉터리 |
| `COPY` | 빌드 context의 파일 복사 |
| `RUN` | 빌드 시점 명령 실행 |
| `ENV` | 이미지와 컨테이너에 남는 환경 변수 |
| `EXPOSE` | 사용 포트 문서화 |
| `USER` | 실행 사용자 지정 |
| `HEALTHCHECK` | 컨테이너 내부 상태 점검 |
| `ENTRYPOINT` | 고정 실행 명령 |
| `CMD` | 기본 명령 또는 기본 인자 |

멀티 스테이지 빌드를 쓰면 컴파일러와 원본 소스를 최종 이미지에서 뺄 수 있다.

```dockerfile
FROM golang:1.26-alpine AS builder
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/app ./cmd/app

FROM alpine:3.23
COPY --from=builder /out/app /usr/local/bin/app
USER nobody
EXPOSE 8080
ENTRYPOINT ["/usr/local/bin/app"]
```

## Docker Compose

여러 옵션을 매번 입력하는 대신 실행 구성을 YAML로 고정한다.

```bash
docker compose up --build
docker compose up -d --build
docker compose ps
docker compose logs -f
docker compose down
docker compose down -v
```

```yaml
services:
  web:
    build:
      context: .
    ports:
      - "${HOST_PORT:-18082}:${SERVER_PORT:-8080}"
    environment:
      SERVER_PORT: "${SERVER_PORT:-8080}"
    restart: unless-stopped
```

`docker compose down -v`는 volume까지 삭제한다. 데이터를 남겨야 한다면 `-v`를 붙이지 않는다.

## 정리 명령은 마지막에

```bash
docker container prune
docker image prune
docker image prune -a
docker volume prune
docker system prune -a
```

`prune`은 조건에 맞는 대상을 한꺼번에 지운다. 실행 전에 `docker ps -a`, `docker images`, `docker volume ls`로 대상을 확인한다.

## 자주 만나는 오류

| 증상 | 원인 | 대응 |
| --- | --- | --- |
| `port is already allocated` | 호스트 포트가 사용 중 | `lsof`나 `ss`로 확인하고 호스트 포트 변경 |
| `container name is already in use` | 정지된 컨테이너도 이름을 점유 | 기존 컨테이너 재사용 또는 삭제 |
| `volume is in use` | 컨테이너가 volume을 연결 중 | 연결한 컨테이너부터 정리 |
| `curl` 종료 코드 18 | 마운트 파일을 쓰는 중 읽음 | 임시 파일에 쓴 뒤 `mv`로 원자적 교체 |

## 참고 자료

- [Docker CLI reference](https://docs.docker.com/reference/cli/docker/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker Compose](https://docs.docker.com/compose/)

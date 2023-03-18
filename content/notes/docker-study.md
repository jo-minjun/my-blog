---
title: "Docker 스터디"
author: ["조민준"]
date: 2023-01-04T23:51:30+09:00
draft: false
categories: ["DevOps"]
tags: ["Docker"]
ShowToc: true
---

# Docker

### Docker란

- 애플리케이션 개발, 실행, 공유를 위한 오픈 플랫폼이다.
- 호스트 시스템과 격리된 환경에서 애플리케이션을 패키징하고 실행할 수 있게 해준다. (컨테이너)
  - 협업 시 각 로컬에 개발환경을 설치하지 않아도 된다.
  - 서버 관리에 편리하다.

![1](/images/notes/docker-study/1.svg)

[https://docs.docker.com/get-started/overview/](https://docs.docker.com/get-started/overview/)

## 도커 명령어

- 크게 4가지 종류의 명령어가 있다.
  - Registry 관련
  - Image 관련
  - Container 관련
  - Compose 관련

명령어의 자세한 옵션과 설명은 아래 문서를 참조

[docker](https://docs.docker.com/engine/reference/commandline/docker/)

### Registry 관련

**login**

```bash
docker login
```

- Registry에 로그인한다.

**logout**

```bash
docker logout
```

- Registry에서 로그아웃한다.

**search**

```bash
docker search [OPTIONS] <TERM>
```

- Registry에 있는 이미지를 검색한다.

| Option      | Default | Description                            |
| ----------- | ------- | -------------------------------------- |
| --filter -f |         | key=value 포맷으로 검색을 필터링 한다. |

- stars: star의 개수 (int)
- is-automated: 자동 빌드 여부 (boolean)
- is-official: 공식 여부 (boolean) |
  | --limit | 25 | 검색 결과의 최대 개수 |
  | --no-trunc | | 검색 결과 텍스트를 생략하지 않고 전부 보여준다. |

**pull**

```bash
docker pull [OPTIONS] <IMAGE>
```

- Registry에서 이미지를 내려 받는다.
- <IMAGE>에 사용자 명을 지정하지 않으면 공식 이미지를 내려 받는다.

**push**

```bash
docker push [OPTIONS] <IMAGE>
```

- 이미지를 Registry에 업로드 한다.

### Image 관련

**build**

```bash
docker image build [OPTIONS] [Dockerfile PATH | URL]
```

- Dockerfile을 이용해서 이미지를 빌드한다.

**ls**

```bash
docker image ls [OPTIONS]
```

- 이미지 목록를 보여준다.

**rm**

```bash
docker image rm [OPTIONS] <IMAGE> [IMAGE...]
```

- 하나 또는 하나 이상의 이미지를 제거한다.

| Option     | Default | Description               |
| ---------- | ------- | ------------------------- |
| --force -f |         | 이미지를 강제로 제거한다. |

**tag**

```bash
docker image tag SOURCE_IMAGE TARGET_IMAGE
```

- 이미지에 태그를 설정한다. (IMAGE_ID에 별칭을 부여한다.)
- 숫자 및 `_` `-` `.` 으로 이름을 시작할 수 없다.

### Container 관련

**commit**

```bash
docker container commit [OPTIONS] CONTAINER
```

- 컨테이너의 변경사항을 이미지로 생성한다.

| Option       | Default | Description               |
| ------------ | ------- | ------------------------- |
| --author -a  |         | 커밋한 사용자를 작성한다. |
| --message -m |         | 커밋 메시지를 작성한다.   |

**diff**

```bash
docker container diff CONTAINER
```

- 컨테이너의 변경사항을 확인한다.
- A: 추가, C: 변경, D: 삭제

**exec**

```bash
docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

- `docker exec` 명령어와 같다.
- 실행 중인 컨테이너에 명령어를 실행한다.

| Option           | Default | Description                     |
| ---------------- | ------- | ------------------------------- |
| --detach -d      |         | 명령어를 백그라운드로 실행한다. |
| --interactive -i |         | 표준입력을 유지한다.            |
| --tty -t         |         | 터미널(pseudo-TTY)을 할당한다.  |

**logs**

```bash
docker container logs CONTAINER
```

- `docker logs` 명령어와 같다.
- 컨테이너의 로그를 보여준다.

| Option          | Default | Description                      |
| --------------- | ------- | -------------------------------- |
| --follow -f     |         | 로그를 계속 추적하면서 출력한다. |
| --timestamps -t |         | 시간 데이터를 보여준다.          |

**ls**

```bash
docker container ls [OPTIONS]
```

- 컨테이너 목록을 보여준다.

| Option    | Default           | Description               |
| --------- | ----------------- | ------------------------- |
| --all -a  | running container | 모든 컨테이너를 보여준다. |
| --size -s |                   | 사이즈를 같이 보여준다.   |

**prune**

```bash
docker container prune
```

- stop 상태인 모든 컨테이너를 제거한다.

**rename**

```bash
docker container rename CONTAINER NEW_NAME
```

- 컨테이너 이름을 변경한다.

**rm**

```bash
docker container rm [OPTIONS] CONTAINER [CONTAINER...]
```

- 하나 또는 하나 이상 컨테이너를 제거한다.

| Option     | Default | Description                           |
| ---------- | ------- | ------------------------------------- |
| --force -f |         | 동작 중인 컨테이너를 강제로 제거한다. |

**run**

```bash
docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

- 이미지를 컨테이너로 생성하고 실행한다.

| Option           | Default | Description                                                    |
| ---------------- | ------- | -------------------------------------------------------------- |
| —detach -d       |         | 컨테이너의 ID를 출력하고 백그라운드로 실행한다.                |
| --interactive -i |         | 표준입력을 유지한다.                                           |
| --tty -t         |         | 터미널(pseudo-TTY)을 할당한다.                                 |
| --name           | random  | 컨테이너에 이름을 지정한다.                                    |
| --env -e         |         | 환경변수를 설정한다.                                           |
| --publish -p     |         | host(port):container(port) 포맷으로 publish와 bind를 설정한다. |
| --volume -v      |         | 볼륨을 마운트 시킨다.                                          |
| --rm             |         | 종료되면 해당 컨테이너를 삭제한다.                             |

**start, restart**

```bash
docker container start [OPTIONS] CONTAINER [CONTAINER...]
```

- 하나 또는 하나 이상의 컨테이너를 시작한다.
- 이미 실행 중인 컨테이너를 다시 시작하려면 `restart`를 사용한다.

**stop**

```bash
docker container stop [OPTIONS] CONTAINER [CONTAINER...]
```

- 하나 또는 하나 이상의 컨테이너를 중지시킨다.

### Compose 관련

**up**

```bash
docker compose up
```

- 컴포즈 파일의 컨테이너들을 생성하고 시작한다.

**down**

```bash
docker compose down
```

- 컨테이너를 중단하고 제거한다.

| Option        | Default | Description                        |
| ------------- | ------- | ---------------------------------- |
| --rmi         |         | 서비스에 사용된 이미지를 제거한다. |
| --volumnes -v |         | 이름이 지정된 volume을 제거한다.   |

## Dockerfile

- Dockerfile을 이용해서 Docker 이미지를 빌드할 수 있다.
- `docker image build` 명령어를 사용해서 Dockerfile에 명시된 command line을 수행하도록 할 수 있다.

```bash
docker image build [Dockerfile 경로]
```

**Format**

- Dockerfile 포맷은 다음과 같다.

```docker
# Comment
INSTRUCTION arguments
```

- INSTRUCTION은 대/소문자를 구분하지 않지만, 대문자로 작성하는 것이 컨벤션이다.
- Dockerfile은 반드시 `FROM` INSTRUCTION으로 시작해야 한다.

**Environment replacement**

- 환경변수는 `$variable_name` 또는 `${variable_name}` 방식으로 사용할 수 있다.
- `${variable_name}` 는 다음과 같은 연산자를 지원한다.
  - `${variable_name:-word}` 는 `variable_name` 이 정의되어있지 않다면 `word` 로 대체된다.
  - `${variable_name:+word}` 는 `variable_name` 이 정의되어 있다면 `word` 가 그 값으로 대체되고 정의되어있지 않다면 빈 문자열로 대체된다.

**FROM**

```docker
FROM [--platform=<platform>] <IMAGE> [AS <name>]
FROM [--platform=<platform>] <IMAGE>[:<TAG>] [AS <name>]
FROM [--platform=<platform>] <IMAGE>[@<DIGEST>] [AS <name>]
```

- 생성할 이미지의 베이스 이미지를 설정한다.
- 멀티 플랫폼 이미지를 참조할 때 `--platform` 사용하여 플랫폼을 특정할 수 있다.
  - linux/amd64
  - linux/arm64
  - windows/amd64
  - …
- <image> 뒤에 `TAG`와 `DIGEST`는 선택적으로 사용한다. 둘 다 생략했다면 `TAG`로 latest가 사용된다.
- AS를 사용해서 빌드 단계에 이름을 줄 수 있다.

**RUN**

```docker
RUN <command> # shell 형식
RUN ["executable", "param1", "param2"] # exec 형식
```

- 현재 이미지의 새 레이어에서 실행되고 결과를 커밋되고, 커밋된 이미지는 Dockerfile의 다음 스텝에서 사용된다.
- RUN 명령어는 두 가지 방식을 따른다.
  - shell 형식
    - 내부적으로 shell 명령어를 호출하여 <command>를 호출한다.
  - exec 형식
    - 사용자가 executable(`/bin/sh`, `/bin/bash`…) 을 명시하여 명령어를 실행할 수 있다.

**CMD**

```docker
CMD <command> param1 param2 # shell 형식
CMD ["executable", "param1", "param2"] # exec 형식
```

- 컨테이너가 실행될 때 수행될 default 명령어를 설정한다.
- 컨테이너 실행시 override가 가능하다.
- CMD는 두 가지 방식을 따른다.
  - shell 형식
    - 내부적으로 shell 명령어를 호출하여 <command>를 호출한다.
  - exec 형식
    - 사용자가 executable(`/bin/sh`, `/bin/bash`…) 을 명시하여 명령어를 실행할 수 있다.

**ENTIRYPOINT**

```docker
ENTRYPOINT command param1 param2 # shell 형식
ENTRYPOINT ["executable", "param1", "param2"] #exec 형식
```

- 컨테이너가 실행될 때 가장 먼저 수행되는 명령어를 지정한다.
- ENTIRYPOINT 명령어는 두 가지 방식을 따른다.
  - shell 형식
    - 내부적으로 shell 명령어를 호출하여 <command>를 호출한다.
  - exec 형식
    - 사용자가 executable(`/bin/sh`, `/bin/bash`…) 을 명시하여 명령어를 실행할 수 있다.

**LABEL**

```docker
LABEL <key>=<value> <key>=<value> <key>=<value> ... # 한 줄에 작성
LABEL <key>=<value> \ # 여러 줄에 작성
			<key>=<value> \
			<key>=<value> \
			...
```

- 키=밸류 방식으로 이미지에 메타데이터를 추가한다.
- LABEL은 기본 또는 상위 이미지의 LABEL을 현재 이미지에 상속 받는다.
- 이미지의 라벨은 `docker image inspect` 명령어로 확인할 수 있다.

**EXPOSE**

```docker
EXPOSE <port> [<port>/<protocol>...]
```

- Docker에게 컨테이너가 런타임에서 어떤 네트워크 포트를 사용할 지 알려준다.
- TCP, UDP를 사용할 수 있고, 명시하지 않는다면 TCP가 사용된다.
- 실제로 포트를 공개하지는 않지만 `docker run -P` 명령어를 사용하면 호스트의 랜덤 포트가 컨테이너의 EXPOSE로 명시한 포트에 매핑된다.

**ENV**

```docker
ENV <key> <value>
ENV <key>=<value> ...
```

- 환경변수 <key>를 <value>로 설정한다.
- 컨테이너 실행 시 `docker container run —env` 명령어로 변경할 수 있다.

**ADD**

```docker
ADD [--chown=<user>;<group>] <src>... <dest>
ADD [--chown=<user>;<group>] ["<src>",... "<dest>"]
```

- <src>의 파일, 디렉토리, 리모트 파일의 URL을 <dest>에 추가한다.
- `*`과 `?` 과 같은 패턴을 사용할 수도 있다.

```docker
# hom으로 시작하는 모든 파일 추가
ADD home* /dir/
# ?는 단일 문자 대체
ADD hom?.txt /dir/
```

**COPY**

```docker
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

- 이미지에 호스트의 파일이나 디렉토리를 복사한다.
- `ADD`는 대상이 압축파일인 경우 해체하여 복사하는 등 기능을 제공하지만, COPY는 복사만 수행한다.

**WORKDIR**

```docker
WORKDIR /path/to/workdir
```

- Dockerfile에서 정의한 명령을 실행하기 위한 디렉토리를 지정하며, 경로가 존재하지 않으면 생성한다.

**ARG**

```docker
ARG <name>[=<default value>]
```

- Dockerfile에서 사용할 변수를 정의한다.
- `ENV`와 달리 Dockerfile 내부에서만 사용 가능하다.

**HEALTHCHECK**

```docker
HEALTHCHECK [OPTIONS] CMD command # 사용할 명령을 지정 (curl 등)
HEALTHCHECK NONE #기본 이미지에서 상속된 healthcheck 사용 안함
```

- 컨테이너가 잘 동작하는지 확인한다.
- 옵션은 다음과 같다.

| Option           | Default | Description                |
| ---------------- | ------- | -------------------------- |
| --interval=n     | 30s     | 헬스 체크 간격             |
| --timeout=n      | 30s     | 헬스 체크 타임아웃 기준    |
| --retries=n      | 3       | 타임아웃 횟수              |
| --start_period=n | 0s      | 컨테이너 실행 후 대기 시간 |

## Compose file

[Compose specification](https://docs.docker.com/compose/compose-file/)

### Compose file versions

| Reference file                                                                | What changed in this version                                                          |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| https://docs.docker.com/compose/compose-file/ (most current, and recommended) | https://docs.docker.com/compose/compose-file/compose-versioning/#versioning           |
| https://docs.docker.com/compose/compose-file/compose-file-v3/                 | https://docs.docker.com/compose/compose-file/compose-versioning/#version-3            |
| https://docs.docker.com/compose/compose-file/compose-file-v2/                 | https://docs.docker.com/compose/compose-file/compose-versioning/#version-2            |
| Version 1 (Deprecated)                                                        | https://docs.docker.com/compose/compose-file/compose-versioning/#version-1-deprecated |

### Compose specification

- Compose specification은 도커가 다중 컨테이너 애플리케이션을 정의하기 위해 만든 **새로운 표준 규격**이다.
- **YAML(YML)**을 이용해서 다음과 같은 항목을 정의한다.
  - service(필수), network, volume, config, secret
- Compose file의 이름은 compose.yaml 또는 docker-compose.yaml을 사용한다.
- 만약 둘 다 존재하는 경우 Compose spec의 컨벤션인 **compose.yaml을 권장**한다.

### Compose 애플리케이션 모델

- Compose file은 플랫폼에 의존하지 않는 컨테이너 집합 기반 애플리케이션을 정의한다.
- **서비스(service)**
  - 애플리케이션 컴포넌트를 구성한다.
  - 컨테이너를 실행해서 플랫폼에 구현되는 추상 개념이다.
  - 어떤 서비스는 런타임 또는 플랫폼에 의존적인 **설정(config)**을 필요로 한다.
- **네트워크(network)**
  - 서비스간 통신을 구성한다.
  - 서로 연결된 서비스 컨테이너 간에 IP 라우팅을 위한 플랫폼 기능 추상체이다.
- **볼륨(volume)**
  - 서비스는 볼륨에 데이터를 저장하고 공유한다.
- config와 secret을 이용해서 컨테이너에 필요한 정책과 보안을 설정할 수 있다.

### Profile

- 프로필을 사용해서 환경에 맞게 Compose 애플리케이션 모델을 조정할 수 있다.
- `services`는 요소로 서비스 `name`을 제공하고 그 하위에 `profiles` 속성을 제공한다.
  - profiles 속성으로 프로필 목록을 정의한다.
  - profiles 속성이 설정되지 않은 서비스는 항상 활성화 된다.
  - 특정 서비스를 실행하는 경우 지정한 프로필이 활성화 된다.

```yaml
services:
	# 모든 프로필에서 활성화 된다.
	foo:
		image: foo

	# test 프로필에서 활성화 된다.
	bar:
		image: bar
		profiles:
			- test

	# test 및 debug 프로필에서 활성화 된다.
	baz:
		image: baz
		depends_on:
			- bar
		profiles:
			- test
			- debug
```

### service의 구성 요소

- service의 주요 하위 요소
  - `build` `image` `command` `container_name` `depends_on` `environment` `expose` `ports` `healthcheck` `volumes`
- 다른 구성 요소는 문서를 참고

**build**

- 컨테이너 이미지를 생성하기 위한 빌드 구성을 지정한다.
- `build` 요소는 문자열 값을 가지거나 하위 요소를 가질 수 있다.
- 아래와 같이 build에 문자열 값을 가지면 Dockerfile의 context만 가질 수 있다.
  ```yaml
  services:
  	webapp:
  		build: ./dir
  ```
- `build` 요소의 하위 요소는 다음과 같다.
  - context: Dockerfile의 context를 지정한다.
  - dockerfile: 사용할 Dockerfile의 이름을 지정한다.
  - args: Dockerfile ARG 값을 정의한다.
  - …
  ```yaml
  services:
  	webapp:
  		build:
  			context: ./dir
  			dockerfile: webapp.Dockerfile
  			args:
  				- GIT_COMMIT=cdc3b19
  ```

**image**

- 컨테이너를 시작할 이미지를 지정한다.
- `[<registry>/][<project>/]<image>[:<tag>|@<digest>]` 방식으로 기술해야 한다.

```yaml
image: redis
image: redis:5
image: library/redis
image: docker.io/library/redis
image: my_private.registry:5000/redis
```

**command**

- 컨테이너 이미지(CMD)에 선언된 기본 명령을 재정의 한다.

```yaml
services:
	webapp:
		command: ["bundle", "exec", "thin", "-p", "3000"] # exec 형식
		command: bundle exec thin -p 3000  # shell 형식
```

**container_name**

- container_name은 컨테이너의 이름을 지정한다.

```yaml
services:
	webapp:
		container_name: my-web-container
```

\***\*depends_on\*\***

- 서비스 간의 시작 및 종료 종속성을 기술한다.
- 두 가지 방법으로 기술할 수 있다.
- Short syntax
  - 종속성 서비스 이름만 지정한다.
  ```yaml
  services:
    web:
      build: .
      depends_on:
        - db
        - redis
    redis:
      image: redis
    db:
      image: postgres
  ```
  - 위 예제는 아래와 같은 동작을 의미한다.
    - `web` 보다 `db` 및 `redis`가 빨리 생성된다.
    - `web` 이 `db` 및 `redis`보다 빨리 제거된다.
- Long syntax
  - 이 방법을 사용하면 추가 필드를 사용할 수 있다.
  - `condition`: 종속성이 충족된 것으로 간주되는 조건
    - `service_started`: (default) 의존하는 서비스가 먼저 시작됨
    - `service_healthy`: 의존하는 서비스가 먼저 시작되고, healthy 상태임
    - `service_completed_successfully`: 의존하는 서비스가 성공적으로 종료됨
  ```yaml
  services:
    web:
      build: .
      depends_on:
        db:
          condition: service_healthy
        redis:
          condition: service_started
    redis:
      image: redis
    db:
      image: postgres
  ```
  - 위 예제는 아래와 같은 동작을 의미한다.
    - `web` 이 실행되기 전에 `db` 가 healthy 상태이고 `redis`가 시작된 상태이다.

**environment**

- 컨테이너에 설정된 환경변수를 정의한다.
- 두 가지 방법으로 환경변수를 정의할 수 있다.
- Map syntax
  ```yaml
  environment:
    RACK_ENV: development
    SHOW: "true"
    USER_INPUT:
  ```
- Array syntax
  ```yaml
  environment:
    - RACK_ENV=development
    - SHOW=true
    - USER_INPUT
  ```

**expose**

- 컨테이너에서 노출해야 하는 포트를 정의한다.
- 호스트 내부의 다른 컨테이너들만 엑세스가 가능하다.

```yaml
expose:
  - "3000"
  - "8000"
```

**ports**

- 컨테이너 포트를 노출한다.
- `[HOST:]CONTAINER[/PROTOCOL]`

```yaml
ports:
  - "3000" # 호스트의 랜덤 포트, 컨테이너의 3000번 포트
  - "3000-3005" # 컨테이너의 포트 번호 범위내에서 할당
  - "8000:8000"
  - "9090-9091:8080-8081"
  - "49100:22"
  - "127.0.0.1:8001:8001"
  - "127.0.0.1:5000-5010:5000-5010"
  - "6060:6060/udp"
```

```
expose vs ports
- ports는 호스트와 컨테이너의 포트를 바인딩 시킨다.
- ports는 호스트 포트와 컨테이너 포트를 모두 노출시키기 때문에
	호스트 내부 컨테이너 간에는 노출된 포트로 접근할 수 있지만,
	호스트 외부에서는 컨테이너와 바인딩된 포트로 접근해야 한다.

- expose는 호스트 포트를 공개하지 않고 컨테이너의 포트만 공개한다.
- 따라서 호스트 외부에서는 컨테이너에 접근할 수 없고 컨테이너 끼리만 접근이 가능하다.
```

**healthcheck**

- 서비스 컨테이너가 healthy 상태인지 확인한다.
- healthcheck의 하위 구성 요소는 아래와 같다.

| Element      | Description                                                                    |
| ------------ | ------------------------------------------------------------------------------ |
| disable      | true 또는 false로 healthcheck 여부를 설정                                      |
| test         | 컨테이너 상태를 확인하기 위한 명령 정의. exec 방식과 shell 방식 모두 사용 가능 |
| interval     | 헬스 체크 간격                                                                 |
| timeout      | 헬스 체크 타임아웃 기준                                                        |
| retries      | 타임아웃 횟수                                                                  |
| start_period | 컨테이너 시작 후 대기 시간                                                     |

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

**volumes**

- 서비스 컨테이너에서 엑세스하는 마운트 호스트 경로 또는 정의한 볼륨을 기술한다.
- 마운트가 단일 서비스에서만 사용되는 경우 최상위 volumes 요소 대신 services 하위 요소로 선언할 수 있다.
- 여러 서비스가 볼륨을 재사용하려면 최상위 volumes 요소에서 정의된 볼륨을 기술해야 한다.
- `HOST_VOLUME:CONTAINER_PATH:[ACCESS_MODE]`
  - `HOST_VOLUME`: 호스트 경로 또는 최상위 volumes 요소에서 정의한 볼륨 이름
  - `CONTAINER_PATH`: 컨테이너의 경로
  - `ACCESS_MODE`: 목록은 `,`으로 구분된다.
    - `rw` : 읽기 및 쓰기(기본값)
    - `ro` : 읽기 전용
    -

```yaml
services:
  backend:
    image: awesome/database
    volumes:
      - db-data:/etc/data

  backup:
    image: backup-service
    volumes:
      - db-data:/var/lib/backup/data

volumes:
  db-data:
    driver_opts:
      device: /host/path/to/volume
```

```yaml
services:
  backend:
    image: awesome/database
    volumes:
      - /dir1:/etc/data

  backup:
    image: backup-service
    volumes:
      - /dir2:/var/lib/backup/data:ro
```

---
title: "Linux 주요 커맨드와 옵션들"
author: ["조민준"]
date: 2023-01-06T00:01:11+09:00
draft: false
categories: ["DevOps"]
tags: ["Linux"]
ShowToc: true
---

## Linux 주요 커맨드와 옵션

**커맨드 라인 단축키**

- ctrl + a: 커서를 라인 가장 앞으로 옮긴다.
- ctrl + e: 커서를 라인 가장 뒤로 옮긴다.
- ctrl + k: 커서를 기준으로 뒤쪽을 모두 지운다.

### 유틸리티

**piping, redirect**

```bash
<COMMAND> | <COMMAND>
```

- `|`를 기준으로 앞 커맨드의 표준 출력을 뒷 커맨드의 표준 입력으로 사용한다.
- `curl -s https://apigateway.dev1.meshdev.io/neogeo/management/info | jq`

```bash
<COMMAND> < <FILE>

<COMMAND> > <FILE>
<COMMAND> 1> <FILE> # 위 명령어와 같다.
<COMMAND> 2> <FILE> # 커맨드의 에러 내용을 파일에 덮어 쓴다.

<COMMAND> >> <FILE>
```

- `<`는 뒷 파일의 내용을 커맨드의 입력으로 사용한다.
- `>`는 앞 커맨드의 결과를 파일 등에 덮어 쓴다.
- `>>`는 앞 커맨드의 결과를 파일 등에 추가한다.

```bash
cat << EOT > template.json
{
  "tag": ""
}
EOT

cat template.json
```

**grep**

- 지정한 패턴이나 문자열에 매칭되는 내용만 출력한다.

```bash
grep [OPTIONS] [PATTERN] [FILE]
```

| -c     | 패턴이 일치하는 행의 수를 출력한다.    |
| ------ | -------------------------------------- |
| -i     | 대소문자를 구분하지 않는다.            |
| -v     | 패턴이 일치하는 않는 내용만 출력한다.  |
| -A <n> | 패턴 일치 줄 이후 n개 라인을 출력한다. |
| -B <n> | 패턴 일치 줄 이전 n개 라인을 출력한다. |

**watch**

- COMMAND의 결과를 계속해서 보여준다.

```bash
watch [OPTIONS] <COMMAND>
```

| -d     | 변경된 부분을 표시한다.  |
| ------ | ------------------------ |
| -n <m> | m초 주기로 리프레쉬한다. |

**time**

- 특정 명령어 또는 프로그램의 수행 시간을 보여준다.

```bash
time <COMMAND>
```

- real: 총 수행 시간
- user: CPU가 사용자 영역에서 보낸 시간
- sys: CPU가 커널 영역에서 보낸 시간

**xargs**

- 앞 커맨드의 수행 결과를 뒤 커맨드의 인자로 넘긴다.

```bash
<COMMAND> | xargs <COMMAND>
```

- 아무 커맨드를 입력하지 않으면 `echo`가 수행된다.

### 파일/DIR

**cat**

- 파일 이름을 받아서 내용을 터미널에 출력한다.

```bash
cat [OPTIONS] <FILE_NAME>
```

```bash
cat name # name 파일의 내용을 터미널에 출력한다.
cat name1 name2 # name1, name2 파일의 내용을 터미널에 출력한다.
```

| -b  | 줄번호를 출력한다. 비어있는 행은 포함하지 않는다. |
| --- | ------------------------------------------------- |
| -n  | 줄번호를 출력한다. 비어있는 행을 포함한다.        |
| -s  | 2개 이상의 빈 행을 한 행으로 출력한다.            |

**touch**

- 파일의 날짜와 시간을 수정하는 명령어다.
- 빈 파일을 생성하기 위해 자주 사용된다.

```bash
touch [OPTIONS] <FILE_NAME> # 파일이 없으면 생성하고, 접근 시간, 상태 변경 시간, 수정 시간을 현재로 변경
```

**mkdir**

- 디렉토리를 생성한다.

```bash
mkdir [OPTIONS] <DIRECTORY_NAME>
```

| -m --mode   | 권한을 함께 부여한다.          |
| ----------- | ------------------------------ |
| -p --parent | 상위 디렉토리를 같이 생성한다. |

**rm**

- 파일이나 디렉토리를 삭제한다.

```bash
rm [OPTIONS] <FILE_NAME | DIRECTORY_NAME>
```

| -f --force     | 삭제 여부를 묻지 않는다.                                              |
| -------------- | --------------------------------------------------------------------- |
| -r --recursive | 해당 디렉토리와 해당 디렉토리 하위의 모든 파일과 디렉토리를 삭제한다. |

**tree**

- 디렉토리의 하위 구조를 계층적으로 보여준다.

```bash
tree [OPTIONS] <DIRECTORY_NAME>
```

| -L <N> | 하위 구조의 레벨을 지정한다. (해당 레벨까지 출력) |
| ------ | ------------------------------------------------- |
| -d     | 파일은 제외하고 디렉토리만 출력한다.              |

**echo**

- 텍스트를 터미널에 출력한다.
- 텍스트에 특수문자가 있는 경우 `"<text>"`로 명시해주어야 한다.

```bash
echo [OPTIONS] <TEXT>
```

**sed**

[Linux sed command help and examples](https://www.computerhope.com/unix/used.htm)

- 파일을 수정하는 stream editor이다.

```bash
sed OPTIONS... [SCRIPT] [INPUT_FILE]
```

| -i  | 결과를 터미널에 출력하지 않고 파일에서 처리한다. |
| --- | ------------------------------------------------ |
| -r  | 정규식을 사용한다.                               |
| -n  | 적용 부분만 구분해서 출력한다.                   |

| 's/문자1/문자2/'    | 문자1을 문자2로 대체한다. (/은 (공백)     | . 으로 사용해도 된다.) |
| ------------------- | ----------------------------------------- | ---------------------- |
| 'n,ms/문자1/문자2/' | n ~ m 번 줄에서 문자1을 문자2로 대체한다. |
| 'n,mp'              | n ~ m 번 줄을 출력한다.                   |
| 'n,md'              | n ~ m 번 줄을 지운다.                     |

```bash
gsed 's/"tag": ./"tag": "latest"/g' template.json > output.json

cat output.json
```

```bash
file: mail

billy@example.org
tom@example.org
jay@example.org
root@example.org
```

- billy와 tom의 example.org만 example.com으로 변경하고 싶은 경우

```bash
gsed -i -r 's/^(billy|tom)@example.org/\1@example.com/' file
```

- 1~2번 라인만 출력하고 싶은 경우

```bash
gsed -n '1,2p' file
```

- 1~2번 라인을 지우고 출력하고 싶은 경우

```bash
gsed '1,2d' file
```

**tee**

- 입력과 출력을 동시에 한다.

```bash
tee [OPTIONS] [FILE]
```

- 출력을 redirection할 경우 파일이 root 권한이라면 실패한다.
- 이 경우 tee를 사용할 수 있다.

```bash
sudo echo "TEXT" > ROOT_FILE # permission denied
sudo echo "TEXT" | tee ROOT_FILE
```

**less**

- 내용을 터미널에 출력한다.
- 위 → 아래, 아래 → 위 방향으로 이동할 수 있다.
- 종료하려면 q를 눌러야 한다.

```bash
less file
```

**more**

- 파일의 내용을 터미널에 출력한다.
- 위 → 아래 방향으로만 이동 가능하다.
- 가장 아래로 이동하면 more가 종료된다.

```bash
more file
```

**ls**

- 디렉토리에 있는 파일이나 디렉토리를 출력한다.

```bash
ls [OPTIONS] [DIRECTORY]
```

| -a  | 모든 파일과 디렉토리를 보여준다.                             |
| --- | ------------------------------------------------------------ |
| -l  | 사용자의 권한, 소유자, 크기, 날짜 등 디테일 정보를 보여준다. |

**which**

- 특정 명령어의 위치를 출력한다.

```bash
which [OPTIONS] COMMAND
```

| -n <m> | 옵션은 모든 위치에서의 명령어를 찾는다. |
| ------ | --------------------------------------- |

**head**

- 파일의 앞 부분을 출력한다.

```bash
head [OPTIONS] FILE
```

| -n <m> | 파일의 앞 부분 m줄 만큼 출력한다. |
| ------ | --------------------------------- |

**tail**

- 파일의 끝 부분을 출력한다.

```bash
tail [OPTIONS] FILE
```

| -n <m> | 파일의 끝 부분 m줄 만큼 출력한다.                                                  |
| ------ | ---------------------------------------------------------------------------------- |
| -f     | 파일의 끝부터 10줄을 출력하고, 새로 입력되는 정보를 계속해서 출력한다.             |
| -F     | 파일이 변경되어도 계속해서 추적하며 출력한다. (삭제되어도 다시 파일이 생기면 출력) |

**tar**

- 여러개의 파일과 디렉토리를 하나의 파일로 묶거나 해제한다.

```bash
tar [OPTIONS] <TARGET>
tar cvf name.tar ./dir # tar 생성
tar xvf name.tar # tar 해제
```

| c   | tar 파일 생성                      |
| --- | ---------------------------------- |
| x   | tar 파일 해제                      |
| v   | 생성 또는 해제 시 파일 리스트 출력 |
| t   | tar에 포함된 내용 확인             |
| f   | 파일 이름 지정                     |

**gzip / gunzip**

- gzip: 파일을 압축한다.
- gunzip: gz파일을 해제한다.

```bash
gzip [OPTIONS] <TARGET>
```

| -<n> | n(1 ~ 9)은 압축 속도이다. 압축 속도가 빠르면 압축률이 낮아진다. (1: 빠름, 9: 느림) |
| ---- | ---------------------------------------------------------------------------------- |
| -c   | 압축 결과를 출력하고 원본은 유지한다.                                              |
| -d   | 압축을 해제한다.                                                                   |
| -v   | 압축 시 자세한 정보를 출력한다. (진행률 등)                                        |
| -r   | 디렉토리의 모든 파일을 압축한다.                                                   |

```bash
gunzip [OPTIONS] <TARGET>
```

| -l  | 압축 파일 정보를 출력한다.                  |
| --- | ------------------------------------------- |
| -v  | 해제 시 자세한 정보를 출력한다. (진행률 등) |
| -r  | 디렉토리의 모든 파일을 해제한다.            |

**zip / unzip**

- zip: 파일 또는 디렉토리를 압축한다.
- unzip: zip 파일을 해제한다.

```bash
zip [OPTIONS] <ZIP_NAME.zip> <TARGET...>
```

| -<n>    | 압축 정도 (0: store only ~ 9: compress better) |
| ------- | ---------------------------------------------- | --- | --- | ------------------------------------------------------------------ |
| -e      | 압축 파일에 암호 적용                          |
| -s <n(k | m                                              | g   | t)> | 분할 압축 (k: kilobytes, m: megabytes, g: gigabytes, t: terabytes) |
| -q      | 메시지 출력 제한                               |
| -r      | 하위 디렉토리의 파일과 숨겨진 파일을 포함한다. |

```bash
upzip [OPTIONS] <TARGET>
```

| -d  | 특정 디렉토리에 zip을 해제한다. |
| --- | ------------------------------- |

**chmod**

```bash
chmod [OPTIONS] <MODE> <TARGET>
```

- 파일 또는 디렉토리의 권한 등을 변경한다.

| -R  | 지정한 모드를 하위 디렉토리 및 파일에 전부 적용한다. |
| --- | ---------------------------------------------------- |

| 000 | 모든 사용자가 r/w/x 불가능                        |
| --- | ------------------------------------------------- |
| 440 | 소유자 및 그룹은 r 가능, 그 외에는 불가능         |
| 664 | 소유자 및 그룹은 r/w 가능, 그 외에는 r만 가능     |
| 755 | 소유자는 r/w/x 가능, 그룹 및 그 외에는 r/x만 가능 |
| 777 | 모든 사용자가 r/w/x 가능                          |

### 네트워크

**curl**

```bash
curl [OPTIONS] <URL>
```

- 여러 통신 프로토콜을 이용해서 데이터를 전송한다.

| -X <METHOD> | HTTP METHOD를 지정한다.                      |
| ----------- | -------------------------------------------- |
| -H <HEADER> | HEADER 값을 지정한다.                        |
| -d <BODY>   | BODY에 들어갈 값을 지정한다.                 |
| -T <FILE>   | PUT 방식으로 파일을 업로드 한다.             |
| -b <COOKIE> | 쿠키를 지정한다.                             |
| -s          | 진행 상태, 에러 메세지 등을 보여주지 않는다. |
| -v          | 헤어 등의 데이터를 추가로 보여준다.          |

**wget**

```bash
wget [OPTIONS] <URL>
```

- 웹에서 파일 다운로드를 한다.

| -c        | 중단된 파일 다운로드를 다시 시작한다. |
| --------- | ------------------------------------- |
| -b        | 파일 다운로드를 백그라운드로 한다.    |
| -O <NAME> | 파일의 이름을 지정한다.               |

**nc**

```bash
nc [OPTIONS] [HOST] [PORT]
```

- TCP 또는 UDP 프로토콜을 사용하는 네트워크 환경에서 데이터를 읽고 쓴다.
- 일반적으로 서버의 포트 상태를 확인과 접속 가능 여부를 확인하기 위해 사용한다.

| -z  | 포트 스캔만 진행한다.    |
| --- | ------------------------ |
| -v  | 더 많은 정보를 출력한다. |

```bash
nc -z google.com 80
```

**telnet**

```bash
telnet [OPTIONS] [HOST] [PORT]
```

- 원격으로 호스트에 접속한다.

| -l <ID> | 접속할 ID를 지정        |
| ------- | ----------------------- |
| -a      | 현재 사용자를 ID로 지정 |

- `telnet`만 입력하면 LINEMODE로 진입한다.

| ?      | 사용법을 출력한다.                     |
| ------ | -------------------------------------- |
| logout | 사용자가 로그아웃하며 접속을 해제한다. |
| open   | 지정한 호스트로 연결한다.              |
| quit   | 텔넷을 종료한다.                       |
| status | 텔넷의 상태를 출력한다.                |
| …      |                                        |

```bash
telnet google.com 80
GET / HTTP/1.1
```

**openssl**

```bash
# rsa 개인키 생성
openssl genrsa [-des3] -out PRIVATE_KEY_NAME.key [BIT_SIZE;1024, 2048]

# rsa 공개키 생성
openssl rsa -in PRIVATE_KEY_NAME.key -pubout -out PUBLIC_KEY_NAME.key

# 인증서 생성
openssl req -new -key PRIVATE_KEY_NAME.key [-days n] -out certification.csr

# HTTPS 통신
openssl s_client -connect google.com:443
```

- SSL/TLS 프로토콜을 이용하기 위한 오픈소스 라이브러리이다.
- `# rsa 비밀키 생성`
  - 공개키 암호화 방식의 개인키를 생성한다.
  - `-des3` 옵션을 넣으면 des3 대칭키 알고리즘으로 한 번 더 암호화 해준다.
    - 비밀키를 추가로 요구한다.
- `# rsa 공개키 생성`
  - 공개키 암호화 방식의 공개키를 생성한다.
- `# 인증서 생성`
  - 개인키로 서명한 인증서를 생성한다.
  - `-days` 옵션으로 유효한 기간을 설정할 수 있다.
  - 인증서는 공개키와 발급자 정보를 식별하는 정보를 가지고 있다.
  - 이 명령어를 수행하면 몇 가지 정보를 요청한다. (국가, 회사, 이메일 등)

**ab**

```bash
ab [OPTIONS] <HOST>[:PORT][/PATH]
```

- 아파치가 제공하는 HTTP 서버 성능 검증 도구이다.

| -n <m> | 요청의 전체 수, m번 만큼 전체 요청을 수행한다. |
| ------ | ---------------------------------------------- |
| -c <n> | 동시에 요청하는 수, n개의 요청을 동시에 한다.  |
| -H     | 요청 헤더를 지정한다.                          |
| -C     | 요청 쿠키를 지정한다.                          |
| -T     | 요청 Content-type을 지정한다.                  |

```bash
ab -n 10 -c 2 https://apigateway.dev1.meshdev.io/neogeo/management/info
```

**nslookup**

```bash
nslookup [-type=<TYPE>] DOMAIN_NAME [DNS]
```

- DNS 서버에서 도메인의 정보를 조회한다.

| -type=soa | origin DNS를 조회한다. |
| --------- | ---------------------- |

**host**

```bash
host [OPTIONS] NAME
```

- DNS 서버에서 도메인의 정보를 조회한다.

**netstat**

```bash
netstat [OPTIONS]
```

- 네트워크 연결상태, 인터페이스 상태 등을 보여준다.

| 옵션 | 설명                              |
| ---- | --------------------------------- |
| -a   | 모든 소켓 확인한다.               |
| -r   | 라우팅 테이블 확인한다.           |
| -n   | 호스트 이름을 ip 주소로 보여준다. |
| -t   | TCP 소켓을 확인한다.              |
| -u   | UDP 소켓을 확인한다.              |
| -p   | PID/program name을 확인한다.      |

**traceroute**

```bash
traceroute [OPTIONS] HOST
```

- HOST까지 가는 네트워크 경로를 확인해준다.

### 프로세스

**top**

```bash
top [OPTIONS]
```

- 실시간으로 시스템의 프로세스 상태를 확인한다.

| -b     | 배치 모드                                 |
| ------ | ----------------------------------------- |
| -n <m> | m 후에 인터렉션 종료 (top 화면을 나간다.) |
| -d <n> | 화면 새로고침 주기                        |

| shift + p | CPU 사용률 내림차순      |
| --------- | ------------------------ |
| shift + m | 메모리 사용률 내림차순   |
| shift + t | 프로세스 런타임 내림차순 |
| k         | PID 작성시 kill          |

**ps**

```bash
ps [OPTIONS]
```

- 현재 실행중인 프로세스의 목록과 상태를 보여준다.

| -A  | 모든 프로세스를 출력한다.                        |
| --- | ------------------------------------------------ |
| -f  | 풀 포맷으로 보여준다. (UID, PPID 등)             |
| -r  | 현재 실행 중인 프로세스를 보여준다.              |
| -e  | 커널 프로세스를 제외한 모든 프로세스를 출력한다. |

**kill / pkill**

```bash
kill [SIGNAL] PID
```

- PID를 이용해서 프로세스에 signal을 보낸다.

```bash
pkill [SIGNAL] NAME
```

- 프로세스 이름을 이용해서 프로세스에 signal을 보낸다.

| 9   | SIGKILL | 강제 종료 시그널               |
| --- | ------- | ------------------------------ |
| 15  | SIGTERM | 정상 종료 시그널 (termination) |

```
kill vs terminate
- kill
	- It is more like pressing PC power and reset button.
		It wont save any logs or other data.
- terminate
		- it will store all your data before shutting down
		(write data from RAM to disk, logs, etc)
```

**pgrep**

```bash
pgrep [OPTIONS] [PATTERN]
```

- 패턴에 일치하는 프로세스 정보를 출력한다.

| -u <USERNAME> | 특정 유저가 실행시킨 프로세스를 검색한다. (이름) |
| ------------- | ------------------------------------------------ |
| -U <UID>      | 특정 유저가 실행시킨 프로세스를 검색한다. (UID)  |
| -g            | 특정 그룹이 실행시킨 프로세스를 검색한다. (이름) |
| -G            | 특정 그룹이 실행시킨 프로세스를 검색한다. (이름) |
| -l            | 프로세스 이름을 같이 출력한다.                   |

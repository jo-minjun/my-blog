---
title: "Linux 배포판 별 패키지 매니저"
author: ["조민준"]
date: 2023-01-05T20:45:12+09:00
draft: false
categories: ["DevOps"]
tags: ["Linux"]
ShowToc: true
---

## 배포판 별 패키지 매니저

### alpine

참고: [Working with the Alpine Package Keeper (apk)](https://docs.alpinelinux.org/user-handbook/0.1a/Working/apk.html)

```bash
apk [<OPTIONS>...] COMMAND [<ARGUMENTS>...]
```

- 존재하는 리포지터리(repository)는 다음과 같다.
  - main
    - 공식적으로 지원하는 패키지들
  - community
    - testing 리포지터리에서 테스트된 패키지들
  - testing
    - 새롭거나, 손상됐거나, 오래된 테스트가 필요한 패키지들

**Updating repository**

```bash
apk update
```

- 리포지터리 인덱스를 업데이트한다.

**Searching**

```bash
apk search [<OPTIONS>...] PATTERN...
```

- 리포지터리에서 PATTERN을 검색한다.

| Option           | Description                        |
| ---------------- | ---------------------------------- |
| --description -d | 설명에서 PATTERN을 검색한다.       |
| --exact -e       | 패키지 이름을 정확하게 매칭시킨다. |

**Installing**

```bash
apk add [<OPTIONS>...] PACKAGES...
```

- 패키지를 설치한다.
- 이미 존재하면 업그레이드를 시도한다.

**Upgrading**

```bash
apk upgrade [<OPTIONS>...] [<PACKAGES>...]
```

- 설치된 패키지를 업그레이드 한다.
- 특정 패키지가 명시되지 않으면 설치된 패키지 중 가능한 패키지를 업그레이드 한다.

**Removing**

```bash
apk del [<OPTIONS>...] PACKAGES...
```

- 설치된 패키지를 제거한다.

### centos(Amazon Linux 2, Amazon Linux 2022)

참고: [Chapter 9. Yum Red Hat Enterprise Linux 7 | Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-yum#doc-wrapper)

```bash
yum [<OPTIONS>...] COMMAND [<ARGUMENTS>...]
```

**Searching**

```bash
yum search PATTERN
```

- 패키지 이름은 모르지만 관련 용어를 알고 있을 때 편리한 패키지 검색 명령어이다.

**Listing**

```bash
yum list all
```

- 설치 되었거나 설치 가능한 패키지 목록을 보여준다.

```bash
yum list installed
```

- 설치된 패키지 목록을 보여준다.

```bash
yum list available
```

- 설치 가능한 패키지 목록을 보여준다.

**Installing**

```bash
yum install PACKAGE
```

- 패키지를 설치한다.

**Updating packages**

```bash
yum check-update
```

- 설치된 패키지 중 업데이트가 가능한 패키지를 확인한다.

```bash
yum update PACKAGE
```

- 한 개 이상 패키지에 대한 업데이트를 진행한다.
- 패키지를 입력하지 않으면 모든 패키지에 대해 업데이트를 진행한다.

**Removing**

```bash
yum remove PACKAGE
```

- 패키지를 제거한다.

**History**

```bash
yum history list
```

- 실행되었던 yum 관련 명령어들을 확인한다.

**Rollback**

```bash
yum history rollback HISTORY_ID
```

- history에서 확인한 ID로 해당 명령어를 수행하기 전으로 되돌린다.

### ubuntu(debian)

참고: [Ubuntu Manpage: apt - command-line interface](https://manpages.ubuntu.com/manpages/xenial/man8/apt.8.html#description)

```bash
apt [options] command
```

**Updating**

```bash
apt update
```

- 리포지터리의 설치 가능한 목록을 업데이트 한다.

**Listing**

```bash
apt list PATTERN
```

- 패키지 이름으로 목록을 검색해서 보여준다.
- `apt -i list` 로 설치된 목록을 확인 할 수 있다.

**Searching**

```bash
apt search PATTERN
```

- 패키지 설명, 이름 등으로 목록을 검색해서 보여준다.

**Installing**

```bash
apt install PACKAGE
```

- 패키지를 설치한다.

**Removing**

```bash
apt remove PACKAGE
```

- 패키지를 제거한다.

**Upgrading**

```bash
apt upgrade [PACKAGE]
```

- 설치된 패키지를 업그레이드 한다.

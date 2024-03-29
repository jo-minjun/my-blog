---
title: "Shell Script"
author: ["조민준"]
date: 2023-01-09T23:30:12+09:00
draft: false
categories: ["Dev"]
tags: ["Linux"]
ShowToc: true
---

## Shell Script

- [쉘 스크립트로 만든 UP-DOWN 게임](https://github.com/jo-minjun/shell-script-up-down-game)

### Shell이란

운영체제에서 커널과 사용자 사이를 이어주는 역할을 하는 명령어 해석기이다.

Shell은 `bash` `sh` `csh` `zsh`등이 있다.

### Shell Script란

운영체제의 Shell을 이용해서 Shell 명령어들을 순차적으로 실행시켜주는 스크립트이다.

Shell Script를 사용하기 위해서는 다음과 같이 시작해야 한다.

```bash
#!/bin/bash

#!/usr/bin/env bash
#!/usr/bin/env python3
```

- 위와 같이 `#!`으로 시작하여 Shell의 경로를 선언해준다. 이를 **쉬뱅**이라 한다.
- **쉬뱅**은 어느 인터프리터가 스크립트의 명령어를 해석할 지 가리킨다.

## 변수

### 변수

```bash
number=1
string="string"

echo "$number"
echo "$string"

echo "${number}"
echo "${string}"
```

- 변수는 위와 같이 공백을 사용하지 않고 선언한다.
- 변수명은 대소문자를 구분한다.
- 변수명은 숫자를 포함할 수 있으나, 숫자로 시작할 수 없다.
- 변수에 숫자를 대입해도 문자열로 취급된다.
- 변수는 `$변수명` 또는 `${변수명}`으로 사용할 수 있다.

### 배열

```bash
array=("a" "b" "c")
array+=("d")

echo "${array[0]}"
echo "${array[*]}"
echo "${#array[*]}"
```

- 배열은 각 원소를 공백으로 구분한다.
- 원소를 추가할 경우 `+=`으로 한다.
- `변수명[index]`를 사용해서 특정 인덱스(0 ~ n)의 원소에 접근할 수 있고, `변수명[*]`으로 모든 원소에 접근할 수 있다.
- `#변수명[*]`으로 원소의 수를 확인할 수 있다.

### 미리 정의된 변수

| 변수    | 설명                                 |
| ------- | ------------------------------------ |
| $0      | 쉘 스크립트의 파일명                 |
| $#      | 쉘 스크립트에 전달된 인자의 수       |
| $$      | 쉘 스크립트의 PID                    |
| $1 ~ $n | 쉘 스크립트에 전달된 인자 값         |
| $\*     | 쉘 스크립트에 전달된 인자들의 문자열 |

## 비교 연산자

### 변수 비교 연산자

| 연산자                | 설명                                         |
| --------------------- | -------------------------------------------- |
| -z ${변수A}           | 변수A의 문자열 길이가 0이면 참               |
| -n ${변수A}           | 변수A의 문자열 길이가 0이 아니면 참          |
| ${변수A} -eq ${변수B} | 변수A와 변수B의 값이 같으면 참               |
| ${변수A} -ne ${변수B} | 변수A와 변수B의 값이 다르면 참               |
| ${변수A} -gt ${변수B} | 변수A의 값이 변수B의 값보다 크면 참          |
| ${변수A} -ge ${변수B} | 변수A의 값이 변수B의 값보다 크거나 같으면 참 |
| ${변수A} -lt ${변수B} | 변수A의 값이 변수B의 값보다 작으면 참        |
| ${변수A} -le ${변수B} | 변수A의 값이 변수B의 값보다 작거나 같으면 참 |
| 연산1 -a 연산2        | 연산1과 연산2가 모두 참이면 참               |
| 연산1 -o 연산2        | 연산1과 연산2중 하나라도 참이면 참           |

### 파일/디렉터리 비교 연산자

| 연산자        | 설명                               |
| ------------- | ---------------------------------- |
| -d ${A}       | A가 디렉터리면 참                  |
| -e ${A}       | A파일이 존재하면 참                |
| -L ${A}       | A파일이 심볼릭 링크면 참           |
| -r ${A}       | A파일에 읽기 권한이 존재하면 참    |
| -w ${A}       | A파일에 쓰기 권한이 존재하면 참    |
| -x ${A}       | A파일에 실행 권한이 존재하면 참    |
| -s ${A}       | A파일의 크기가 0보다 크면 참       |
| -f ${A}       | A파일이 존재하면 참                |
| ${A} -nt ${B} | A파일이 B파일보다 최신 파일이면 참 |
| ${A} -ot ${B} | A파일이 B파일보다 이전 파일이면 참 |
| ${A} -ef ${B} | A파일이 B파일과 같은 파일이면 참   |

- 주로 `-d` `-f` 를 자주 사용한다.

## 제어문

### 분기문

```bash
if [ 비교 연산자 ]; then
	# 실행
elif [ 비교 연산자 ]; then
	# 실행
else
	# 실행
fi
```

- `if`로 시작하고 `fi`로 끝난다.
- 분기문에서 비교 연산은 `[ 비교 연산자 ]; then` 구분을 사용한다.

```bash
case target in
	값1)
		# 실행
	;;
	값2|값3)
		# 실행
	;;
	*)
		# 실행
	;;
esac
```

- `case`로 시작하고 `esac`로 끝난다.
- `;;` 를 이용해서 break를 할 수 있다.

### 반복문

반복문은 공통적으로 `do`로 시작하고 `done`으로 끝난다.

`continue`와 `break`를 이용해서 반복문을 제어할 수 있다.

```bash
while (비교 연산자); do
	# 실행
done
```

```bash
for i in ${array[*]}; do
    # 실행
done
```

- 배열의 각 요소에 접근한다.

```bash
for i in {0..10}; do
    # 실행
done
```

- 0 ~ 10 범위의 값을 접근한다.

```bash
for (( i = 0; i < 10; i++)); do
    # 실행
done
```

- 0 ~ 10 범위의 값을 접근한다.

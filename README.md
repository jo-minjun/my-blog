# my-blog

## 개요

- hugo를 이용한 블로그입니다.
  - 기술 관련 내용, 기록하고 싶은 기억, 그냥 끄적거리고 싶은 내용을을 블로깅합니다.
- github.io를 사용합니다.
  - https://jo-minjun.github.io/

## 배포

- github action을 사용합니다.
- 배포 흐름을 관리하려면 `.github/workflows/deploy-hugo.yaml`을 수정해야 합니다.

## 블로그 로고 관리

- `static/logo.png` 이미지 파일을 변경합니다.
- 블로그 링크를 공유하여 이미지가 변경되었는지 확인할 수 있습니다.
- 실제 변경 반영까지 시간이 걸릴 수 있습니다.

## 컨텐츠 이미지 관리

- `static/images/doodles` 또는 `static/images/notes`에 삽입할 이미지를 위치시킵니다.
- 컨텐츠 파일에서 컨텐츠 종류에 따라 `![이미지 개요](/images/notes/파일 이름)` 또는 `![이미지 개요](/images/doodles/파일 이름)` 으로 이미지를 삽입합니다.
  - e.g. `![znode](/images/notes/kafka/znode.png)`

## 컨텐츠 파일 생성

- hugo 블로그는 블로그 파일에 메타 정보를 포함하는 헤더가 있습니다.
- 이 헤더는 복사해서 사용할 수 있으나, hugo를 설치하면 헤더가 포함된 파일을 생성하는 명령어를 지원합니다.
  - `brew install hugo`

```shell
# notes
hugo new content/notes/{filename}.md --kind notes
# doodles
hugo new content/doodles/{filename}.md --kind doodles

# cat {filename-hello}.md
---
title: "Filename hello"
author: ["조민준"]
date: "yyyy-MM-dd'T'HH:mm:ss+09:00"
draft: false
categories: ["..."]
tags: []
ShowToc: true
---
```

## 로컬 서버 구동

- 로컬에서 hugo 서버를 구동하려면 `hugo`를 설치해야 합니다.
  - `brew install hugo`
- 아래 명령어를 실행합니다.

```shell
hugo server
```

- 브라우저에서 `http://localhost:1313`에 접근합니다.

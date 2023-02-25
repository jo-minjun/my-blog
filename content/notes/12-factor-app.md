---
title: "12 factor app (15 factor app)"
author: ["조민준"]
date: 2023-02-24T00:02:51+09:00
draft: false
categories: ["Dev"]
tags: ["SaSS", "MSA"]
ShowToc: true
---

## Introduction

12 factor app은 서비스형 소프트웨어를 구성하기 위한 방법론입니다.

- 개발 언어/프레임워크에 상관없는 이식성과 플랫폼 호환성 향상을 위한 디자인 원칙입니다.
- 애플리케이션의 수평적 확장이 용이하도록 합니다.

> - 애플리케이션 확장 가능성
> - CI/CD에 용이성
> - 플랫폼간 이식성
> - 기본적인 기대치, 정책 설정

## 1. Codebase

### 형상관리 시스템에서 하나의 코드베이스를 관리하면서, 다수에 배포한다.

하나의 코드베이스에 여러개의 애플리케이션 코드가 있다면 12 factor app 위반입니다.

코드베이스는 모든 배포에 사용되지만 각 배포는 다른 버전이 사용될 수 있습니다.

![codebase](../12-factor-app/codebase.png)

## 2. Dependencies

### 의존성을 명시적으로 선언하고 분리한다.

의존성은 명시적으로 선언되어, 신규 개발자 또는 시스템 설정을 편리하게 해야합니다.

대부분의 프로그래밍 언어는 패키징 시스템을 제공하기 때문에 새로 설정을 해야할 때는 언어와 패키지 매니저만 설치하면 됩니다.

![dependencies](../12-factor-app/dependencies.png)

## 3. Config

### 설정값을 환경에 저장한다.

애플리케이션 설정값은 배포 환경에 따라 달라지는 값들입니다.

- 데이터베이스 또는 Backing 서비스를 처리하는 리소스
- Amazon S3 또는 트위터와 같은 외부 서비스에 대한 인증 정보
- 배포 환경 호스트 이름과 같은 값

![config](../12-factor-app/config.png)

## 4. Backing services

### Backing service

Backing 서비스는 데이터베이스와, 메시징, 메일 서비스 등 통해 연결된 모든 서비스입니다.

12 factor app은 Backing 서비스를 모두 리소스로 취급하고, 설정에서 값을 읽어서 처리하여 느슨하게 연결합니다.

![backing-service](../12-factor-app/backing-service.png)

## 5. Build, release, run

### 빌드와 실행 단계를 엄격하게 구분한다.

코드베이스는 3단계를 거쳐 배포되고, 엄격하게 구분되어야 합니다.

1. Build: 지정된 코드 버전을 사용하여 의존성을 가져오고 컴파일합니다.
2. Release: 컴파일된 결과물과 현재 배포 환경의 설정을 연결합니다. Release 단계의 결과물은 즉시 실행될 수 있습니다.
3. Run: 애플리케이션을 실행합니다.

코드 변경은 반드시 빌드 단계에서만 이루어져야만 하며 만들어진 Release 결과는 변경될 수 없고, 이전 버전으로 롤백이 가능해야합니다.

## 6. Processes

### 애플리케이션을 하나 이상의 Stateless 프로세스로 실행한다.

애플리케이션은 실행 환경에서 하나 이상의 프로세스로 실행됩니다.

상태는 데이터베이스와 같은 상태 저장 서비스에 저장해야 하며, 애플리케이션은 Stateless하게 유지해야 합니다.

![processes](../12-factor-app/processes.png)

## 7. Port binding

### 포트 바인딩을 통해 서비스 제공을 한다.

애플리케이션은 포트를 바인딩하여 서비스를 제공해야 합니다.

포트를 통해 서비스를 제공함으로써 다른 애플리케이션의 Backing 서비스가 될 수 있습니다.

## 8. Concurrency

### 프로세스 모델을 통해 수평적 확장을 한다.

애플리케이션은 리소스 추가를 통한 수직 확장 뿐만 아니라, 수를 늘리는 수평적 확장이 가능해야 합니다.

6. Processes를 준수함으로써 확장하거나 축소할 수 있습니다.

## 9. Disposability

### 빠른 시작과 그레이스풀 셧다운으로 안정성을 최대화한다.

배포와 수평 확장시 빠른 애플리케이션 구동을 위해 필요합니다.

종료 시그널을 받은 애플리케이션은 새로운 요청을 받지 않고, 기존 요청을 처리한 후 안정적으로 종료되어야 합니다.

## 10. Dev/prod parity

### 개발, 스테이징, 상용 환경을 최대한 비슷하게 유지한다.

Local에서는 H2 database를 사용하고 상용에서는 MySQL을 사용하는 것과 같은 차이를 줄이는 것입니다.

12 factor app은 개발과 상용 환경 사이의 차이를 줄여 지속적인 배포가 가능하도록 해야합니다.

## 11. Logs

### 로그를 이벤트 스트림으로 처리한다.

애플리케이션은 로그에 관여하면 안되며, 단순히 버퍼링없이 출력할 뿐입니다.

애플리케이션은 언제든지 생성되고 삭제될 수 있습니다. 따라서 이벤트는 별도 저장소에 보관되는 것이 좋습니다.

![logs](../12-factor-app/logs.png)

## 12. Admin processes

### 어드민/관리 작업을 일회성 프로세스로 실행해야 한다.

개발자는 종종 일회성으로 애플리케이션 관리 작업을 수행해야 하며, 작업을 스크립트화하여 한번에 실행할 수 있도록 해야합니다.

- 데이터베이스 마이그레이션
- 일회성 스크립트 실행

관리 스크립트는 애플리케이션과 같은 코드베이스에서 같은 설정 값을 사용해야 합니다.

---

> 케빈 허프만이 “Beyond the 12 factor app”을 통해 MSA 환경에 적합한 3가지 요소를 제시했다.

## 13. API first

### API 스펙 정의를 우선으로 한다.

API 스펙을 먼저 정의하여 어떤 스키마로 통신할지 결정해야 합니다.

API first를 통해 클라이언트와 서버가 동시에 작업을 진행할 수 있습니다.

## 14. Telemetry

### 애플리케이션 및 리소스를 모니터링한다.

애플리케이션 및 CPU, RAM등 리소스를 모니터링하여 성능, 이벤트 및 헬스 체크 등을 확인할 수 있습니다.

서비스 관리 및 경고 알람 트리거 설정에 도움을 줍니다.

## 15. Security

### 보안 정책이 적절한지 확인한다.

API, DB 등 보안 정책이 적절한지 확인해야 합니다.

API는 OAuth 등으로 보호되어야 하며 HTTPS를 이용해서 노출시켜야 합니다.
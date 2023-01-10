---
title: "Introduce Kubernetes"
author: ["조민준"]
date: 2023-01-10T21:51:30+09:00
draft: false
categories: ["DevOps"]
tags: ["Kubernetes"]
ShowToc: true
---

## What, Why Kubernetes?

### What

- 쿠버네티스는 컨테이너들을 운영, 관리하는 **컨테이너 오케스트레이터**이다.
- 컨테이너 오케스트레이터는 개별 컨테이너의 배포, 관리, 확장, 네트워킹을 자동화해준다.

### Why

- 물리 서버에서 동작하는 서비스는 리소스 관리를 효율적으로 할 수 없다.
  - 3개의 서비스에 트래픽이 몰리는 시간대가 다르다.
  - 각 서비스는 최소 트래픽 때 0.5대, 최대 트래픽 때 3개의 서버가 사용된다.
  - 이 경우 총 9대의 서버가 사용된다.
- 배포시에도 비효율적이다.
  - 중단이 가능한 경우 모든 서비스를 내린 후, 업데이트하여 다시 올린다.
  - 중단이 불가능하면 서비스를 하나씩 내리고 하나씩 업데이트하여 다시 올린다.

→ 쿠버네티스를 사용하면 리소스 관리를 효율적으로 할 수 있다.

- 시간대 별로 평균 서버 필요량을 예측하여 다음과 같은 운용이 가능하다.
  - 평균 4개의 서버
  - A서비스에 트래픽이 몰리는 경우 3개 서버에 A서비스 할당, 1개 서버에 B, C 서비스 할당
- 서버에 장애가 발생한 경우에 유연하게 대처할 수 있다.
  - 쿠버네티스에는 Auto Healing 기능이 있다.
  - 여분의 서버 1개가 있는 경우 자동으로 여분 서버에 서비스를 옮긴다.
- 배포할 때는 Deployment Object를 통해서 자동화 해준다.

→ 서버가 효율적인 개수를 유지하고 자동화 됨으로써 유지보수 비용이 감소한다.

## Overview

![1](../introduce-kubernetes/1.png)

- 서버 한대는 Master, 나머지 서버는 Node가 된다.

→ 이것이 연결되어 하나의 쿠버네티스 클러스터가 된다.

- Master는 쿠버네티스의 기능을 컨트롤하고 Node는 리소스를 제공한다.
  - 클러스터의 리소스를 늘리고 싶다면 Node를 추가하면 된다.
- namespace를 이용해서 쿠버네티스 오브젝트를 독립된 공간으로 분리시켜준다.
- namespace에는 쿠버네티스 최소 배포 단위인 Pod가 있고, Pod에 IP를 할당되도록 연결되는 Service가 있다.
  - Service는 다른 namespace와는 연결될 수 없다.
- Pod에는 여러 Container가 있을 수 있다. 또한 Volume을 마운트해서 Pod의 데이터가 증발되지 않도록 한다.
- namespace에는 ResourceQuota와 LimitRange를 설정해서 namespace의 리소스의 양을 제한시킬 수 있다.
- ConfigMap과 Secret으로 Pod의 컨테이너에 환경변수를 설정할 수 있게 한다.
- 여러가지 컨트롤러는 Pod들을 관리한다.
- Replication Controller, ReplicaSet은 Pod가 죽으면 다시 구동시키거나 스케일 인/아웃을 해준다.
- Deployment는 배포 후에 Pod를 새 버전으로 업그레이드하고, 문제가 생기면 롤백 해준다.
- DaemonSet은 한 Node에 하나의 Pod만 사용되도록 해준다.
- CronJob은 특정 Job을 주기적으로 수행되도록 해준다.
  - Job은 특정 작업만 하고 종료되는 것이다.

## Components

![2](../introduce-kubernetes/2.png)

- master 노드
  - kube-apiserver
  - etcd
  - kube-scheduler
  - kube-controller-manager
- worker 노드
  - kubelet
  - kube-proxy

### kube-apiserver

- 쿠버네티스 클러스터의 API를 사용할 수 있게 해준다.
- kubectl과 같은 클라이언트로부터 요청 받아낸다.

### etcd

- key-value 형식의 데이터 저장소이다.

### kube-scheduler

- Node들의 리소스 상태와 kube-apiserver를 확인하면서, Pod에 Node 정보를 할당한다.

### kube-controller-manager

- Controller들을 실행한다.

### kubelet

- kube-apiserver를 확인하면서 Pod에 자신의 Node 정보가 할당된 것이 있으면 Pod를 생성한다.
  - 컨테이너를 생성하고 kube-proxy에 네트워크 생성 요청을 한다.

### kube-proxy

- 네트워크 규칙을 관리하고 컨테이너가 네트워크를 사용할 수 있도록 한다.

## 실습 환경 구축

- 강의에서 소개된 환경이 아닌 minikube를 사용한다.

```bash
minikube start \
  --driver='docker' \
  --kubernetes-version='stable' \
  --nodes=3

# 실습을 위해 dashboard 사용
# 실제 업무에서는 secret 등의 값 노출을 막기위해 dashboard를 잘 사용하지 않는다.
minikube dashboard
```

- 위와 같은 방법으로 master 1개, worker node 2개를 구축하고 dashboard를 사용할 수 있다.

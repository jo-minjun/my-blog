---
title: "Kubernetes: pod와 service"
author: ["조민준"]
date: 2023-01-10T21:51:30+09:00
draft: false
categories: ["DevOps"]
tags: ["Kubernetes"]
ShowToc: true
---

## Pod

### container

- Pod에는 container가 여러개 있을 수 있으며, localhost로 접근할 수 있다.
- Pod가 생성될 때는 IP가 할당되며, 이 IP를 통해 Pod에 접근할 수 있다.
  - 쿠버네티스 클러스터 내에서만 IP로 접근 가능하다.
- Pod가 재생성되면 IP 주소가 바뀐다.

```yaml
apiVersion: v1
# 하나의 Pod
kind: Pod
metadata:
	name: pod-1
spec:
	# 여러 개의 container
	containers:
		- name: container1
			image: image1
			ports:
			- containerPort: 8000
		- name: container2
			image: image2
			ports:
			- containerPort: 8080
```

### label

- Pod 뿐만 아니라 다른 오브젝트에도 사용할 수 있지만, Pod에서 가장 많이 사용된다.
- 목적에 따라 오브젝트를 분류하고 분류된 오브젝트만 연결하기 위해서 사용한다.
- key:value

```bash
Pod1                      Pod2                      Pod3
type:web                  type:db                   type:server
env:dev                   env:dev                   env:dev
================================================================
Pod4                      Pod5                      Pod6
type:web                  type:db                   type:server
env:prod                  env:prod                  env:prod
```

- 위와 같은 dev/prod 환경에서 web/db/server 종류가 있는 경우
  - 웹 개발자가 web만 보고 싶다면 type:web label인 Pod만 서비스에 연결해서 확인하면 된다.
  - prod환경 운영자는 env:prod label인 Pod만 서비스에 연결해서 확인하면 된다.

```yaml
# Pod 생성

apiVersion: v1
kind: Pod
metadata:
	name: pod-2
	labels:
		type: db
		env: dev
spec:
	containers:
		- name: container
			image: image
```

```yaml
# Service 생성

apiVersion: v1
kind: Service
metadata:
	name: svc-1
spec:
	selector:
		type: web
	ports:
		-port: 8080
```

### node scheduler

- Pod는 Node들 중 하나에 올라가야 한다.
- 수동으로 지정하는 방법과 자동으로 지정되는 방법이 있다.
- 수동
  - Node를 생성할 때 label을 설정하고 Pod를 만들 때 Node를 선택한다.
  - **NodeSelector**, NodeAffinity, Pod Affinity, Anti-Affinity, Toleration, Taint…
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
  	name: pod-3
  spec:
  	nodeSelector:
  		hostname: node1 # node에 지정한 label의 key, value
  	containers:
  		- name: container
  			image: image
  ```
- 자동
  - Pod가 요구하는 리소스 할당량과 각 Node의 리소스 할당 가능량을 계산해서 scheduler가 선택한다.

## Service

- Service는 자신의 IP를 가지고 있다.
- Service를 Pod에 연결해 놓으면 Service의 IP를 가지고 Pod에 접근할 수 있다.
  - Pod의 IP를 가지고 접근할 수도 있지만, Pod의 IP는 변경될 수 있다.

### ClusterIP

- 가장 기본적인 Service이다.
- 클러스터 내에서만 접근이 가능한 IP이다.
  - 클러스터 내의 오브젝트에서는 접근 가능하지만 외부에서는 접근할 수 없다.
- 하나의 Pod 뿐만아니라 여러개의 Pod를 연결할 수 있다.
  - 요청이 오면 여러개의 Pod로 분산도 해준다.
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
  	name: svc-1
  spec:
  	# label을 이용해서 Pod 연결
  	selector:
  		app: pod
  	# 9000번 포트로 요청이 들어오면 8080번 포트로 연결이도 된다.
  	ports:
  		- port: 9000
  			targetPort: 8080
  	type: ClusterIP
  ```
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
  	name: pod-1
  	labels:
  		app: pod
  spec:
  	containers:
  		- name: container
  			image: tmkube:app
  			ports:
  				-containerPort: 8080
  ```

### NodePort

- Service에 IP가 포함되어 있어 ClusterIP와 같은 기능이 포함되어 있다.
- 클러스터 내부 모든 Node에 포트를 할당하여 Node의 IP로 접근을 하면 Service로 트래픽이 전달된다.
  - Service는 다시 자신에게 연결된 Pod로 트래픽을 전달한다.
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
  	name: svc-2
  spec:
  	selector:
  		app: pod
  	ports:
  		- port: 9000
  			targetPort: 8080
  			nodePort: 30000
  	type: NodePort
  	# 이 옵션을 사용하면 Service가 요청이 들어온 Node의 Pod로 트래픽을 전달한다.
  	externalTrafficPolicy: local
  ```

### Load Balancer

- NodePort의 특징을 포함하고 있다.
- Load Balancer라는 오브젝트가 생성되어 각 Node에 트래픽을 분산 시킨다.
- 외부에서 Load Balancer에 접근하기 위한 IP는 기본적으로 설정되어 있지 않다.
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
  	name: svc-3
  spec:
  	selector:
  		app: pod
  	ports:
  		- port: 9000
  			targetPort: 8080
  			nodePort: 30000
  	type: LoadBalancer
  ```

### ExternalName

- 클러스터 외부 서비스에 접근하기 위해 사용하는 서비스이다.

---
title: "Kubernetes: 애플리케이션 설정을 체계적으로 관리하기"
author: ["조민준"]
date: 2023-09-23T02:15:52+09:00
draft: false
categories: ["DevOps"]
tags: ["Kubernetes"]
ShowToc: true
---

> 쿠버네티스 개발전략을 읽고 작성한 내용입니다.  
> https://product.kyobobook.co.kr/detail/S000200103262

- 쿠버네티스를 통해서 설정을 외부화하여 관리하는 방법을 알아보겠습니다.
- 여기서 설정은 데이터베이스 url, username, password, 얼마전에 작업한 뉴렐릭 라이센스 키 같은 값들을 말합니다. 이러한 설정값들은 변경되거나 추가 삭제되는 경우가 많기 때문에 애플리케이션 외부에서 관리하는 것이 유리합니다.
  - 외부화된 설정: Externalized configuration

## 환경 변수와 실행 인자

### 실행 인자

- 우선 실행 인자와 환경 변수에 대해 알아보겠습니다
- prod 프로필을 사용하려면 아래와 같이 실행 인자로 환경 정보를 넘겨줍니다.

  ```bash
  $ java -Dspring.profiles.active=prod -jar my-app.jar
  ```

  - 위 스크립트에서 `-Dspring.profiles.active=prod`를 실행 인자라고 합니다. 이 실행 인자를 이용해서 애플리케이션에 환경 정보를 넘겨 줄 수 있습니다.
  - `application-prod.yaml` 과 같이 미리 정의된 설정 파일을 선택하여 사용할 수 있습니다.

- 그런데 위와 같은 방법은 애플리케이션을 직접 실행하는 환경에서 적합합니다. 쿠버네티스를 사용하는 환경에서는 적합하지 않은데요.
- 쿠버네티스 환경에서는 파드 명세서에 실행 인자를 정의하여 환경 정보를 넘겨줄 수 있습니다.
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
  	name: my-pod-prod
  spec:
  	containers:
  	- name: my-container
  		image: alswns9288/my-app:1.0.0
  		ports:
  		- containerPort: 8080
  			name: web
  			protocol: TCP
  		args: ["-spring.profiles.active=prod"]
  # ...
  ```

### 환경 변수

- 파드 명세서에 환경 변수를 정의하여 설정을 관리할 수 있습니다
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
  	name: my-pod-prod
  spec:
  	containers:
  	- name: my-container
  		image: alswns9288/my-app:1.0.0
  		ports:
  		- containerPort: 8080
  		env:
  		- name: SPRING_PROFILES_ACTIVE
  		  value: "prod"
  # ...
  ```
  - 파드 명세서에 name, value 를 정의하여 환경 정보를 넘겨줄 수 있습니다.

## 컨피그맵을 이용하여 여러 설정값 한번에 관리하기

- 실행 인자와 환경 변수는 여러 파드가 동일한 설정값을 사용하는 경우에는 적합하지 않습니다. 이런 경우에는 컨피그맵을 사용하면 설정값을 편리하게 관리할 수 있습니다.
- 컨피그맵은 설정값을 key-value 형태로 쿠버네티스 클러스터에 저장하고 파드에서 사용할 수 있도록 도와주는 역할을 합니다.

```yaml
# 컨피그맵
apiVersion: v1
kind: ConfigMap
metadata:
	name: my-config
data:
	APPLICATION_KEY: "configmap value"
```

### **컨피그맵 사용**

**<컨피그맵 참조>**

```yaml
# 파드
apiVersion: v1
kind: Pod
metadata:
	name: my-pod
spec:
	containers:
	- name: my-container
		image: alswns9288/my-app:1.0.0
		ports:
		- containerPort: 8080
			name: web
			protocol: TCP
		env:
		- name: APPLICATION_KEY
			valueFrom:
				configMapKeyRef:
					name: my-config
					key: APPLICATION_KEY
```

**<컨피그맵을 파일로 참조>**

```yaml
# 컨피그맵
apiVersion: v1
kind: ConfigMap
metadata:
	name: my-config
data:
	my-property: |
		APPLICATION_KEY: "configmap value"
```

```yaml
# 파드
apiVersion: v1
kind: Pod
metadata:
	name: my-pod
spec:
	containers:
	- name: my-container
		image: alswns9288/my-app:1.0.0
		ports:
		- containerPort: 8080
			name: web
			protocol: TCP
		volumeMounts:
		- name: property
			mountPath: /app/resources
		volumes:
		- name: property
			configMap:
				items:
				- key: my-properaty
					path: application.yaml
```

- **<컨피그맵 참조>** 방법과 **<컨피그맵을 파일로 참조>** 방법의 차이는 무엇일까요?
  - **<컨피그맵 참조>** 방법은 파드를 시작할 때 환경변수를 시스템에 설정하고 파드를 시작합니다. 따라서 컨피그맵을 변경해도 파드의 환경변수 값이 변경되지 않습니다.
    - 변경된 컨피그맵을 적용하려면 파드를 재시작해야합니다.
  - 반면에 **<컨피그맵을 파일로 참조>** 방법은 파일을 마운트하는 방법입니다. 컨피그맵을 변경하면 마운트된 파일이 변경됩니다. 따라서 파드를 재시작하지 않아도 변경된 컨피그맵을 사용할 수 있습니다.

> 파드 명세서에서 `env` 가 아닌 `envFrom` 을 사용할 수도 있습니다.  
> 이 방식은 `env` 처럼 컨피그맵의 특정 key를 참조하는 것이 아닌, 컨피그맵의 전체 키를 참조합니다.

## 시크릿을 이용해 민감한 설정값 관리하기

- 컨피그맵으로 관리하기 민감한 값은 시크릿으로 관리할 수 있습니다
- 시크릿은 민감하게 관리해야 하는 설정값들을 별도로 관리할 수 있게 해줍니다.
- 시크릿을 생성하는 대표적인 2가지를 알아보겠습니다.

### 파일로 시크릿 생성

```bash
$ cat secretfile
secret value
```

```bash
$ kubectl create secret generic my-secret --from-file=APPLICATION_KEY=./secretfile
```

### 명세서로 시크릿 생성

```yaml
# 시크릿
apiVersion: v1
kind: Secret
metadata:
  name: my-secret-manifest
data:
  APPLICATION_KEY: c2VjcmV0IHZhbHVlCg==
```

- 명세서를 사용할때는 base64로 인코딩된 값을 사용해야합니다.

### 파드에서 시크릿 사용

```bash
# 파드
apiVersion: v1
kind: Pod
metadata:
  name: my-pod2
spec:
  containers:
    - name: my-container
      image: 'alswns9288/my-app:1.0.0'
      ports:
        - containerPort: 8080
          name: web
          protocol: TCP
      env:
        - name: APPLICATION_KEY
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: APPLICATION_KEY
```

### 시크릿 조회

```bash
$ kubectl get secret my-secret -o yaml
apiVersion: v1
data:
  APPLICATION_KEY: c2VjcmV0IHZhbHVlCg==
kind: Secret
metadata:
  creationTimestamp: "2023-09-19T16:12:05Z"
  name: my-secret
  namespace: default
  resourceVersion: "5320"
  uid: ccf7e058-755d-4587-ba2b-0d98ff3c8a05
type: Opaque
```

- 위와 같이 시크릿을 조회하면 base64로 인코딩된 설정값을 조회할 수 있습니다.

```bash
$ echo c2VjcmV0IHZhbHVlCg== | base64 -D
secret value
```

- 이렇게 base64로 되어 있으면 어떻게 민감한 값을 관리할 수 있을까 의문이 들 수 있습니다.
- 쿠버네티스는 플러그인으로 암호화 설정을 하거나, 시크릿에 대한 권한 제한으로 처리하도록 가이드하고 있습니다.

---
title: "kubectl command"
author: ["조민준"]
date: 2023-01-09T23:51:30+09:00
draft: false
categories: ["DevOps"]
tags: ["Kubernetes"]
ShowToc: true
---

## kubectl command

- 쿠버네티스 API를 사용하는 CLI 도구이다.

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

- [command]
  - 하나 이상의 리소스에서 수행하는 동작을 지정한다.
  - ex) `create` `get` `describe` `delete`
- [TYPE]
  - 리소스 타입을 지정한다.
  - 대소문자를 구분하지 않으며 단수형, 복수형, 약어를 지정할 수 있다.
  - ex) `pod` `pods` `po`
- [NAME]
  - 하나 이상의 리소스의 이름을 지정한다.
  - 대소문자를 구분하며 리소스 이름을 지정하지 않으면 모든 리소스가 대상이 된다.
  - 리소스가 모두 동일한 TYPE인 경우
    - ex) `kubectl get pod name1 name2`
  - 리소스 타입을 개별로 지정하는 경우
    - ex) `kubectl get pod/name1 replicaset/name2`
- [flags]
  - 플래그를 지정한다.
  - ex) `-A`

### 주요 명령어

| command      | description                                                                 | example                                                                                                                        |
| ------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| get          | 하나 이상의 리소스를 보여준다.                                              | kubectl get pod                                                                                                                |
| edit         | 서버의 리소스를 수정한다.                                                   | kubectl edit pod name1                                                                                                         |
| delete       | 파일 또는 리소스를 삭제한다.                                                | kubectl delete -f file.yaml kubectl delete pod name1                                                                           |
| scale        | deployment, replicaset 등의 scale을 조정한다.                               | kubectl scale replicaset name1 --replicas=3                                                                                    |
| top          | CPU/memory 등 리소스 상태를 보여준다.                                       | kubectl top pod kubectl top node                                                                                               |
| describe     | 리소스의 상세 정보를 보여준다.                                              | kubectl describe -l key=value                                                                                                  |
| logs         | pod의 로그를 보여준다.                                                      | kubectl logs -f pod_name                                                                                                       |
| exec         | container에 커맨드를 실행시킨다.                                            | kubectl exec -it pod_name -- /bin/bash                                                                                         |
| port-forward | pod의 포트로 local 포트를 포워드한다.                                       | kubectl port-forward pod_name 13231:80 kubectl replicaset/replicaset_name 13231:80 kubectl deployment/deployment_name 13231:80 |
| cp           | 파일 또는 디렉터리를 복사한다. pod_name을 명시하지 않으면 local로 설정된다. | kubectl cp pod_name:path path                                                                                                  |
| apply        | 리소스를 생성하거나 업데이트한다.                                           | kubectl apply -f file.yaml                                                                                                     |
| config       | kubeconfig(~/.kube/config) 파일을 관리한다.                                 | kubectl config view kubectl config use-context dev1                                                                            |

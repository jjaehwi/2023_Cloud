# 기초 kubectl 실습

- [kubectl syntax](#kubectl-syntax)

- [kubectl basic operations](#kubectl-basic-operations)

- [kubectl best practices](#kubectl-best-practices)

---

## kubectl syntax

- 기본 구조 : **`kubectl <command> <type> <name> <flags>`**

- `command`

  - 쿠버네티스 리소스에 대해 수행할 작업

  - (ex) create, get, describe, execute, delete ...

- `type`

  - 관리하고자 하는 쿠버네티스 리소스의 종류

  - (ex) pods, nodes, namespaces, deployments, services ...

- `name`

  - 해당 타입의 쿠버네티스 리소스의 이름

- `flags`

  - CLI 명령에 대한 옵션

  - (ex) -f, -s, -o

---

## kubectl basic operations

- **List resources** : 리소스 리스트 보기

  - `kubectl get <type>`

    - 쿠버네티스 리소스의 리스트 출력

    - `-o wide` : 해당 리소스와 관련된 추가적인 정보 출력

- **Describe resources** : 리소스 자세한 정보 확인

  - `kubectl describe <type> <name>`

    - 해당 쿠버네티스 리소스의 디테일한 정보 출력

- **Create or Change resources** : 리소스 생성 및 교체

  - `kubectl create <type> <name>`

    - 리소스 생성

  - `kubectl apply <type> <name>`

    - 리소스 생성 및 교체 (Recommended)

  - `kubectl create/apply -f <file name>.yaml`

    - yaml 파일을 활용한 리소스 생성 및 교체

- **Delete resources** : 리소스 삭제

  - `kubectl delete <type> <name>`

    - 리소스 삭제

  - `kubectl delete -f <file name>.yaml`

    - yaml 파일에 해당하는 리소스 삭제

- **Execute a command** : Pod 에서 실행되는 컨테이너로 명령어 전달

  - `kubectl exec <name of pod> --<command>`

    - **Pod 에서 실행되는 `컨테이너로 명령어 전달`**

- **Display pod logs** : Pod 의 로그 정보 확인

  - `kubectl logs <name of pod>`

  - Pod 에 대해서만 수행 가능

```
# counter.yaml 예시
apiVersion: v1
kind: Pod
metadata:
    name: counter
spec:
    containers:
        - name: count
          image: busybox
          args: [/bin/sh, -c, 'i=0; while true; do echo "$i: $(data)"; i=$((i+1)); sleep 1; done']

# counter.yaml 파일 작성 후 배포
kubectl apply -f counter.yaml

# exec 명령어 예시 (ls 명령어 전달)
kubectl exec counter --ls

# logs 명령어 예시
kubectl logs counter
```

---

## kubectl best practices

- **Know about `namespaces`**

  - `kubectl <command> <type> <name> -n <namespace name>`

    - 해당 네임스페이스에 해당하는 리소스에 대해 명령 수행

  - `kubectl <command> <type> <name> --all-namespaces`

    - 모든 네임스페이스에 해당하는 리소스에 대해 명령 수행

  - `kubectl <command> <type> <name>`

    - 네임스페이스를 지정하지 않으면 현재 context 의 기본으로 설정된 namespace 가 된다.

```
# namespace 목록 확인
kubectl get namespaces
```

- **Display `manual page` of resource** : 리소스의 메뉴얼 확인

  - `kubectl explain <type>`

  - 해당 리소스에 대한 manual page 출력

    - 리소스 각 필드의 타입, kind, apiVersion 등을 확인할 수 있다.

- **Use simple aliases** : 쿠버네티스 리소스의 `축약어` 사용 가능

  - `kubectl api-resources` 를 통해 각 리소스에 대한 축약어를 확인할 수 있다.

  - nodes = no

  - pods = po

  - deployments = deploy

  - services = svc

  - namespaces = ns

  - replicationcontrollers = rc

  - (ex) kubectl get svc

- `kubectl <command> --help` 를 통해 <command> 에 대한 설명과 사용할 수 있는 옵션을 확인할 수 있다.

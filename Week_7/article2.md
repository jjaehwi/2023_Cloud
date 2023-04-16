# Application deployment & Scale Out

- [Pod 타입으로 배포](#pod-타입)

- [ReplicaSet 타입으로 배포](#replicaset-타입)

- [Deployment 타입으로 배포](#deployment-타입)

---

## Pod 타입

- `run 명령어`를 이용하여 Pod 배포하기

  - `kubectl run [Pod name] --image=[container image name] --port=[port number]`

  - `kubectl get pods -o wide` 명령어를 이용하여 배포된 Pod 확인하기

```
# nginx-pod 라는 이름, nginx 이미지, container port number 를 80 으로 갖는 Pod 배포하기
kubectl run nginx-pod --image=nginx --port=80
```

---

- `delete 명령어` 를 이용하여 Pod 제거하기

```
# nginx-pod 라는 이름을 가진 Pod 제거
kubectl delete pod nginx-pod
```

---

- `yaml 파일` 을 이용하여 Pod 배포하기

  - `kubectl create/apply -f [File name]`

```
# nginx-pod.yaml 파일
apiVersion: v1
kind: Pod
metadata:
    name: nginx-pod
spec:
    containers:
        - name: nginx-container
          image: nginx:1.14
          ports:
            - containerPort: 80
              protocol: TCP

# nginx-pod.yaml 파일을 이용하여 배포하기
kubectl create -f nginx-pod.yaml
```

---

- `describe 명령어` 를 이용하여 상세 정보 출력하기

  - `kubectl describe pod [Pod name]`

  - namespace, label, IP address, container 의 상세 정보 확인 가능

```
# nginx-pod 라는 이름의 Pod 상세 정보 출력하기
kubectl describe pod nginx-pod
```

---

- `exec 명령어` 를 이용하여 해당 Pod 의 container 에 command 명령 실행하기

  - `kubectl exec [Pod name] -- [command]`

  - `-it` : interactive terminal

```
# exec 명령어를 통해 nginx 컨테이너에 접속 후 index.html 파일 수정하기
kubectl exec -it nginx-pod -- /bin/bash
echo "Hello world" > /usr/share/nginx/html/index.html
curl [Pod IP]:[port]
```

---

## ReplicaSet 타입

- `yaml 파일`을 이용하여 배포

  - `kubectl apply -f [File name]`

```
# rs-nginx.yaml 파일
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: rs-nginx
spec:
    replicas: 3
    selector:
        matchLabels:
            app: webui
    template:
        metadata:
            name: nginx-pod
            labes:
                app: webui
        spec:
            containers:
                - name: nginx-container
                  image: nginx
                  ports:
                    - containerPort: 80
                      protocol: TCP

# yaml 파일을 이용하여 ReplicaSet 배포
kubectl apply -f rs-nginx.yaml
```

---

- `scale 명령어` 를 통해 replicas 수 변경하기

  - `kubectl scale replicaset [ReplicaSet name] --replicas=[변경하는 수]`

```
# replicas 3 개에서 2 개로 변경하기
kubectl scale replicaset rs-nginx --replicas=2
```

---

- `describe 명령어` 를 통해 ReplicaSet 상세 정보 출력

  - `kubectl describe replicaset [ReplicaSet name]`

```
# ReplicaSet rs-nginx 상세 정보 출력
kubectl describe replicaset rs-nginx
kubectl describe rs rs-nginx
```

---

- **ReplicaSet 의 가장 큰 장점** : `Pod 의 가용성을 안정적으로 일정하게 유지`한다.

  - **Pod 를 강제적으로 지워져도 ReplicaSet 에 의해 새로운 Pod 가 생성된다.**

  - **Pod 를 replicas 수를 초과하게 생성하더라도 ReplicaSet 에 의해 Pod 가 다시 삭제된다.**

    - ReplicaSet 이 관리하는 대상이 되어야한다.

    - `Pod 의 label 이 ReplicaSet 의 SELECTOR 의 key value 값이 되어야함.`

```
# replicas 수를 초과하게 생성하여 삭제되는 것 확인해보기
kubectl run testpod --image=nginx --labels=app=webui --port=80
```

---

- `delete 명령어` 로 배포한 ReplicaSet 제거하기

  - `kubectl delete replicaset [ReplicaSet name]`

```
# rs-nginx 라는 이름의 ReplicaSet 삭제하기
kubectl delete replicaset rs-nginx
kubectl delete rs rs-nginx
```

---

## Deployment 타입

- `create 명령어` 를 이용하여 Deployment 배포하기

  - `kubectl create deployment [Deployment name] --image=[container image name] --replicas=[개수]`

```
# create 명령어로 Deployment 배포
kubectl create deployment nginx-pod --image=nginx --replicas=2
```

---

- `yaml 파일` 을 이용하여 Deployment 배포하기

  - `kubectl create/apply -f [File name]`

```
# deploy-nginx.yaml 파일
apiVersion: apps/v1
kind: Deployment
metadata:
    name: deploy-nginx
spec:
    replicas=3
    selector:
        matchLabels:
            app: webui
    template:
        metadata:
            name: nginx-pod
            labes:
                app: webui
        spec:
            containers:
                - name: nginx-container
                  image: nginx

# deploy-nginx.yaml 파일로 Deployment 배포
kubectl apply -f deploy-nginx.yaml
```

---

- `scale 명령어` 로 replicas 수 조정하기

  - `kubectl scale deployment [Deployment name] --replicas=[개수]`

```
# deploy-nginx 라는 이름의 Deployment 의 replicas 수 2 로 변경하기
kubectl scale deployment deploy-nginx --replicas=2
```

---

- `describe 명령어` 로 Deployment 상세 정보 출력

  - `kubectl describe deployment [Deployment name]`

---

- **ReplicaSet 에 문제가 발생했을 경우 자동으로 ReplicaSet 을 복제**하여 `명시된 replicas 개수에 대한 가용성을 보증`한다.

```
# ReplicaSet 을 인위적으로 delete 해보고 다시 생기는 것 확인해보기
kubectl delete replicaset [ReplicaSet name]
```

---

- `delete 명령어` 를 이용하여 Deployment 제거하기

  - `kubectl delete deployment [Deployment name]`

```
# delete 명령어로 nginx-pod 라는 이름의 Deployment 제거
kubectl delete deployment nginx-pod
```

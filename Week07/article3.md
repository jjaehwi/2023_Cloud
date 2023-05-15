# Service Deployment

- [Service](#service)

  - [Label](#label)

  - [Selector](#selector)

  - [Service type](#service-type)

  - [Port 종류](#port-종류)

- [ClusterIP Service](#clusterip-service)

- [NodePort Service](#nodeport-service)

---

## Service

- `Pod 의 논리적 집합`

  - Pod 집합에 접근할 수 있는 정책을 정의하는 추상적 개념

  - `Label`, `Selector` 를 이용하여 Pod 집합 결정

- 서비스를 통해 **클러스터 내 / 외부에서 Pod 로 접근 가능**

---

### Label

<img width="511" alt="스크린샷 2023-04-17 오전 2 09 07" src="https://user-images.githubusercontent.com/87372606/232328830-62e7e369-cecc-43e3-aa8e-91f97b6459fe.png">

- 쿠버네티스 `오브젝트 (Pod, Deployment, ...) 에 붙는 key / value 쌍`

- 주로 오브젝트를 묶거나 **선택하기 위해 사용**

- 의미있고 관련성 높은 특징으로 식별하기 위한 **일종의 태그 역할**

---

### Selector

<img width="493" alt="스크린샷 2023-04-17 오전 2 09 37" src="https://user-images.githubusercontent.com/87372606/232328871-1d99e4c9-7962-4fc9-a79f-c36dec6702a8.png">

- 어떤 오브젝트를 선택할지 표현하는 방법

- `특정 label 을 선택하여 해당하는 오브젝트만 선택`

---

### Service type

- `ClusterIP (default)`

  - 서비스를 클러스터 내부 IP 에 노출시킴

  - `클러스터 내부에서만 접근 가능`

- `NodePort`

  - **각 노드 IP 주소의 특정 port 에 서비스를 노출**시킴

  - 포트 당 하나의 서비스를 사용하며, **30000 ~ 32767 범위 내의 포트를 사용**

  - `<Node IP 주소>:<NodePort> 를 요청하여 클러스터 외부에서 접근할 수 있다.`

- `LoadBalancer`

  - 클라우드 공급자의 로드밸런서를 사용하여 서비스를 클러스터 외부에 노출시킨다.

  - (ex) AWS, GCP, Azure

  - minikube 환경에서는 지원하지 않음

---

### Port 종류

<img width="609" alt="스크린샷 2023-04-17 오전 2 14 39" src="https://user-images.githubusercontent.com/87372606/232329187-6a82a87d-851a-4bbd-8d2b-7edb26013685.png">

- 그림에서 보면 외부에서 31000 포트에 트래픽이 들어올 때,

  - 노드 포트와 서비스 포트가 mapping 되어서 서비스로 전달이 되고,

  - 서비스에서는 자신에게 연결되어 있는 Pod 에게 트래픽을 전달한다.

    - 서비스에서 targetPort 를 80 으로 지정함으로 해당 Pod 를 찾을 수 있게된다.

---

## ClusterIP Service

<img width="535" alt="스크린샷 2023-04-17 오전 2 43 47" src="https://user-images.githubusercontent.com/87372606/232331369-02a2c704-8f63-4be2-a7e8-77a0be414eb4.png">

```
# nginx-deployment.yaml 생성
apiVersion: apps/v1
kind: Deployment
metadata:
    labels:
        app: nginx
    name: nginx-pod
spec:
    replicas: 2
    selector:
        matchLabels:
            app: nginx
    template:
        metadata:
            labels:
                app: nginx
            name: nginx
        spec:
            containers:
                - name: nginx
                  image: nginx
                  ports:
                    - containerPort: 80
                      protocol: TCP


# nginx-deployment.yaml 파일로 Deployment 배포 후

# kubectl exec -it [Pod name] -- bash` 로 접근하여 index.html 파일 수정한다.

# vim 편집기를 먼저 설치해야하기 때문에 apt update 후 apt install vim -y 로 설치

# index.html 파일이 있는 /usr/share/nginx/html 로 접근

# vim index.html 로 index.html 파일 수정

<h1>Hello! This is a Pod1<h1>

# 마찬가지로 2 번째 Pod 에 접근해서 Pod2 로 index.html 파일 수정
```

- `yaml 파일` 을 이용하여 Service 배포하기

  - kubectl apply -f [File name]

```
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
    name: nginx-service
    labels:
        app: nginx
spec:
    ports:
        - port: 8080
          protocol: TCP
          targetPort: 80
    selector:
        app: nginx

# Service 배포
kubectl apply -f nginx-service.yaml
```

---

- `expose 명령어` 를 사용하여 해당 Deployment 의 ClusterIP 서비스 생성

  - `kubectl expose deployment [Deployment name] --name=[Service name] --type=[Service type] --port=[port] --target-port=[targetPort]`

```
# nginx-service2 ClusterIP 서비스 생성
kubectl expose deployment nginx-pod --name=nginx-service2 --type=ClusterIP --port=8080 --target-port=80
```

---

- `kubectl get endpoints [Service name]` 을 통해 endpoint 를 확인해 볼 수 있다.

```
# 방금 배포한 nginx-service endpoint 확인하기
kubectl get endpoints nginx-service
```

---

- `curl 명령어` 를 통해 클러스터 내부에서 접근하여 확인 할 수 있다.

  - `curl [Service IP 주소]:[port 번호]`

```
# Service IP:port 번호로 수정한 index.html 에 접근하기
curl 10.102.232.27:8080

그러면 아까 Pod 에서 수정한 index 파일이 Pod 1, 2 에 대해 번갈아가며 나오게 된다.
이는 서비스 내 자체적인 loadbalancing 알고리즘에 의해 출력되는 것이다.
```

<img width="452" alt="스크린샷 2023-04-17 오전 2 39 09" src="https://user-images.githubusercontent.com/87372606/232330546-d5f605de-727e-4903-b5c6-b7095d74cc21.png">

---

## NodePort Service

<img width="588" alt="스크린샷 2023-04-17 오전 2 44 22" src="https://user-images.githubusercontent.com/87372606/232331500-22f193d0-eaf2-47c6-bc50-262049a9366e.png">

<img width="519" alt="스크린샷 2023-04-17 오전 2 52 35" src="https://user-images.githubusercontent.com/87372606/232331989-b61d77d9-1522-416c-bd21-d14c883ee4fb.png">

- `yaml 파일` 을 이용하여 NodePort Service 배포

  - `kubectl apply -f [File name]`

```
# nginx-service-nodeport.yaml 파일
apiVersion: v1
kind: Service
metadata:
    name: nginx-service-nodeport
    labels:
        app: nginx
spec:
    type: NodePort
    ports:
        - nodePort: 31000 (안쓰면 랜덤)
          port: 8080
          protocol: TCP
          targetPort: 80
    selector:
        app: nginx

# yaml 파일을 이용하여 NodePort Service 배포
kubectl apply -f nginx-service-nodeport.yaml
```

---

- `expose 명령어` 를 이용하여 NodePort Service 배포

  - `kubectl expose deployment [Deployment name] --name=[Service name] --type=[Service type] --port=[port] --target-port=[targetPort]`

```
# expose 명령어를 이용하여 nginx-service-nodeport2 이름의 NodePort Service 배포
kubectl expose deployment nginx-pod --name=nginx-service-nodeport2 --type=[NodePort] --port=8080 --target-port=80
```

---

- `describe 명령어` 를 이용하여 NodePort Service 상세 정보 출력

  - `kubectl describe svc [NodePort Service name]`

- `curl 명령어` 를 통해 클러스터 **내부 / 외부 에서 접근하여 확인** 할 수 있다.

  - `curl [Service IP 주소]:[port 번호]`

  - `curl [NodePort IP 주소]:[NodePort 번호]`

<img width="658" alt="스크린샷 2023-04-17 오전 2 56 28" src="https://user-images.githubusercontent.com/87372606/232332179-4abf28cb-695d-4e66-ae5f-e48770442c34.png">

- 웹 브라우저 주소창을 통해서 접속할 수도 있다.

<img width="658" alt="스크린샷 2023-04-17 오전 2 57 08" src="https://user-images.githubusercontent.com/87372606/232332214-4fc4c132-e604-4936-a80b-2f5ec2d33b00.png">

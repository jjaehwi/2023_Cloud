# Basic 리소스 와 yaml 파일 형식

- [Configuration File](#kubernetes-의-configuration-file)

  - [YAML format](#yaml-yaml-aint-markup-language)

- [Label, Selector](#label-selector)

- [어플리케이션 배포 방식](#어플리케이션-배포-방식)

  - [PoD 타입](#pod-타입)

  - [ReplicaSet 타입](#replicaset-타입)

  - [Deployment 타입](#deployment-타입)

- [Visual map of a kubernetes deployment](#a-visual-map-of-a-kubernetes-deployment)

---

## Kubernetes 의 Configuration File

- Cluster 에 원하는 서비스를 구동 하는 방법은 이에 대한 원하는 상태를

  - `직접 요구`하거나

  - 해당 내용을 `파일로 정리`하여

  - APIServer 를 통해 파일에 기술한 정보대로 Master 내의 etcd 에 원하는 상태정보를 표현한 `Object (Resource)` 를 만들게 하는 것이다.

### YAML (YAML Ain’t Markup Language)

- 사람이 읽기 쉬운 데이터 직렬화 양식

- Syntax: `strict indentation (들여쓰기)`

- [YAML VALIDATOR](https://onlineyamltools.com/validate-yaml)

- **_5 Fields of Kubernetes Definition File_**

  - **`apiVersion`**

    - 오브젝트를 생성하기 위해 사용하고 있는 **쿠버네티스 API 버전**

    - api 서버가 기능이 추가되면서 리소스를 만들어달라고 요청받고 access 를 할 때 분류를 잘 해야했기 때문에 생겨남.

    - `오브젝트` : 원하는 상태가 저장된 쿠버네티스 API Server상의 객체

  - **`kind`**

    - 어떤 종류의 쿠버네티스 `리소스`인지 기술 (ex) deployment 라는 종류의 리소스를 만들어달라고 요청

  - **`metadata`**

    - `오브젝트를 구별`해줄 수 있는 데이터

    - 일반적으로 데이터에 관한 구조화된 데이터, 기초적인 설명….

    - name, label, namespace, annotations ...

  - **`spec`**

    - 오브젝트로 생성하고자 하는 리소스의 (kind 에 대한) 구체적인 정보

  - **`status`**

    - 현재 오브젝트의 상태

    - 쿠버네티스에 의해 자동으로 생성

    - `의도하는 상태 (desired status)` 와 `현재 상태 (actual status)` 를 꾸준히 비교하고 업데이트

- `kubectl explain [쿠버네티스 리소스]` 를 통해 해당 쿠버네티스 리소스의 정보, 관련 field에 대한 설명 및 데이터 타입 확인 가능

  - (ex) kubectl explain pods

```
(ex) deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
    name: nginx-deployment
spec:
    selector:
        matchLabels:
            app: nginx
    replicas: 2
    template:
        metadata:
            labels:
                app: nginx
        spec:
            containers:
                - name: nginx
                  image: nginx:1.14.2
                  ports:
                    - containerPort: 80
```

---

## Label, Selector

- `Kubernetes 내부에서의 객체간 연계, 연결을 위한 방법`

- `Labels` are `key/value pairs` that you can **attach to objects** like pods

  - 포드와 같은 개체에 연결할 수 있는 키/값 쌍

  - 의미 있고 관련성 있는 정보를 설명하는 데 도움을 준다.

- `Selectors` are a way of expressing **how to select objects based on their labels**

  - Selectors 는 labels 을 기반으로 어떻게 객체를 선택할지 표현하는 방법

  - labels 이 지정된 기준과 동일한지 또는 집합 내에 적합한지 여부

  - `Selectors allows us to filter objects based on labels.`

---

## 어플리케이션 배포 방식

- bare metal = physical server (real hardware machine)

- software 적인 server = Virtual machine

  1. control 할 서버

  2. pod 가 올라갈 서버

- `PoD 타입`

  - 어플리케이션을 pod 의 형태로 배포한다.

- `ReplicaSet 타입`

  - 어플리케이션을 ReplicaSet 형태로 배포한다.

  - ReplicaSet은 실행되는 pod 개수에 대해 **지정한 replicas 개수만큼 항상 실행 될 수 있도록 관리**한다.

- `Deployment 타입`

  - `Deployment` : 1개 이상의 Pod를 관리하는 컨트롤러의 종류 중 하나

  - replicaset resource 를 생성시키고 그 후 생성된 replicaset resource 정보에 따라 PoD resource 가 해당 개수만큼 생성되어 최종적으로 실제 PoD 가 생성되게 한다.

---

## PoD 타입

- `run` 명령어를 이용하여 Pod 배포하기

  - `kubectl run [Pod name] --image=[container image name] --port=[port number]`

  - pod 의 리소스에 IP 와 ports 번호가 잇는데 ip 를 안써주면 자동으로 할당

  - `kubectl get pods` 를 이용하여 배포된 Pod 확인

```
# Pod 배포 예시
kubectl run nginx-pod --image=nginx --port=80

# 배포된 Pod 확인
kubectl get pods -o wide
```

---

- `yaml 파일`을 이용하여 Pod 배포하기

  - yaml 파일 작성 후 `kubectl apply -f [File name]`, `kubectl create -f [File name]` 을 통해 배포

  - 지금은 API server 에 yaml 파일로 요청하는 것

  - yaml 파일을 누가 까야할지 어떤 워커 노드에게 처리시킬 것인지는 스케쥴러가 써준다.

  - `스케쥴러`도 **kubelet 이 보낸 상태정보를 API 서버를 통해서 워커 노드들의 상태**를 알고있고, 스케쥴러가 어느 워커 노드에게 스케쥴을 할당할지 (한가한 노드들) 결정한다.

  - 이런 정보들은 status 정보에 작성된다.

  - 할당되면 워커노드는 kubelet 을 통해 api server 를 계속 지켜보고 있다가 자기에게 할당된 일거리가 있으면 처리한다.

  - docker 엔진에게 일거리를 주고 docker 엔진은 image 를 다운받고 port 와 ip 주소들을 할당한다.

```
# nginx-pod.yaml 예시

apiVersion: v1
kind: Pod
metadata:
    name: nginx-pod
spec:
    container:
        - name: nginx-container
          image: nginx
          ports:
            - containerPort: 80
              protocol: TCP

# 배포 예시
kubectl create -f nginx-pod.yaml
```

<img width="810" alt="스크린샷 2023-04-16 오전 12 22 48" src="https://user-images.githubusercontent.com/87372606/232233932-3705e660-6fcc-409f-8188-c2c725d74934.png">

---

- `kubectl describe pod [Pod name]` 을 통해 namespace, label, IP address, container의 상세 정보 확인 가능

```
# nginx-pod 상세 정보 확인 예시
kubectl describe pod nginx-pod
```

---

- `kubectl exec [Pod name] --[command]` 로 해당 Pod 의 container 에 [command] 명령 실행

```
# exec 명령어를 통해 nginx 컨테이너에 접속 후 index.html 파일 수정

kubectl exec -it nginx-pod --/bin/bash
echo "Hello world" > /usr/share/nginx/html/index.html
exit 후
curl ip:port 로 확인
```

<img width="582" alt="스크린샷 2023-04-16 오전 12 32 13" src="https://user-images.githubusercontent.com/87372606/232234359-30b7f40d-c4aa-4489-a3b4-4b7003d01df0.png">

---

- `kubectl delete pod [Pod name]` 으로 **Pod 를 삭제**할 수 있다.

---

### ReplicaSet 타입

- **똑같은 내용의 pod 를 두 개 만들고 싶으면** 또 다시 pod object 를 만들면 되는데, 이는 번거로우니 `replicaset 이라는 오브젝트`를 새로 만들었다.

- yaml 파일을 이용하여 `kubectl create -f [File name]`, `kubectl apply -f [File name]` 으로 배포할 수 있다.

```
# rs-nginx.yaml 예시
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name:: rs-nginx
spec:
    replicas: 2
    selector:
        matchLabels:
            app: webui
    template:
        metadata:
            name: nginx-pod
            labels:
                app: webui
        spec:
            container:
                - name: nginx-container
                  image: nginx:1.14
                  ports:
                    - containerPort: 80
                      protocol: TCP

# yaml 파일을 이용하여 배포
kubectl apply -f rs-nginx.yaml
```

- spec 의 구성요소는 replicas, selector, template 등등

  - template 안에 metadata, spec ,,, metadata 안에 name 과 labels,,

  - spec 이라고 하는 컨테이너를 만들건데 webui 라는 label 을 붙혀라…

  - 그리고 그 webui 라고 하는 label 이 붙은 pod 를 replicas 에 적은 개수만큼 만들어라.

  - label 딱지 붙이는 pod 에 대한 spec 을 작성하지 않으면 없다고 오류가 발생하게됨.

---

- `kubectl describe replicasets [ReplicaSet name]` 을 통해 ReplicaSet 상세 정보를 출력할 수 있다.

```
# ReplicaSet 상세 정보 출력 예시
kubectl describe replicaset rs-nginx
```

<img width="901" alt="스크린샷 2023-04-16 오전 12 42 37" src="https://user-images.githubusercontent.com/87372606/232234832-228232e5-0767-4bff-8651-6ec74c9aef53.png">

---

- ReplicaSet 은 replicas 수를 유지한다.

  - watch 명령어를 이용하여 Pod 리스트의 변화를 관찰한다.

  - 나머지 세션에서는 Pod 를 인위적으로 삭제해본다.

  - 지정한 Pod 는 삭제되었지만 ReplicaSet 에 의해 `새로운 Pod 가 생성`되는 것을 확인할 수 있다.

  - 지정한 replicas 수를 초과해도 마찬가지로 `생성되는 Pod 를 다시 삭제`한다.

<img width="582" alt="스크린샷 2023-04-16 오전 12 47 33" src="https://user-images.githubusercontent.com/87372606/232235069-bc30a850-24be-46cd-a0d3-5da7b55d683a.png">

<img width="635" alt="스크린샷 2023-04-16 오전 12 49 03" src="https://user-images.githubusercontent.com/87372606/232235135-6a572e97-0e29-4ca3-9e74-e5e708c15888.png">

---

- `kubectl delete replicasets [ReplicaSet name]` 로 생성한 ReplicaSet 을 삭제할 수 있다.

---

### Deployment 타입

- 훨씬 복잡한 스펙들에 대해 얘기할 수 있는 리소스, 여러 스펙들을 묶음으로 만들어놓았다.

- deployment 는 replicas 를 만들고 replicas 는 pod 를 만든다.

  - 즉 아랫단에 있는 리소스들을 만드는 것. final 리소스는 pod 니까 pod 가 만들어진다.

  - `high level 의 리소스를 정의하면 low level 단계적으로 만들어내는 과정`으로 쿠버네티스가 일을 한다.

  - abstract 한 object (상위 레벨의 simple 한 spec 으로 표현하면…) 를 만들면 단계적으로 low level 로 갈 수록 구체적으로 작성되는 것.

---

- `create` 명령어를 이용하여 Deployment 배포

  - `kubectl create deployment [Deployment name] --image=[container image name] --replicas=2`

  - Deployment 로 배포 시 replicas 수를 관리하기 위한 ReplicaSet 을 자동으로 생성한다.

```
# Deployment 배포 후 확인 예시
kubectl create deployment nginx-pod --image=nginx:1.14 --replicas=2
kubectl get deployment, replicaset, pod -o wide
```

<img width="630" alt="스크린샷 2023-04-16 오전 1 03 11" src="https://user-images.githubusercontent.com/87372606/232235839-772a1023-3ed2-4f66-abff-04aece45903b.png">

<img width="317" alt="스크린샷 2023-04-16 오전 1 03 26" src="https://user-images.githubusercontent.com/87372606/232235880-cdea34ab-425b-4eb5-88ac-8fd4c0e380e6.png">

---

- `yaml 파일` 을 이용하여 Deployment 배포

  - kubectl create -f [File name], kubectl apply -f [File name]

```
# deploy-nginx.yaml 예시
apiVersion: apps/v1
kind: Deployment
metadata:
    name: deploy-nginx
spec:
    replicas: 2
    selector:
        matchLabels:
            app: nginx-pod
    template:
        metadata:
            name: nginx-pod
            labels:
                app: nginx-pod
        spec:
            containers:
                - name: nginx-container
                  image: nginx:1.14

# Deployment 배포
kubectl create -f deploy-nginx.yaml
```

---

- `kubectl describe deployment [Deployment name]` 으로 Deployment 정보를 상세 출력할 수 있다.

```
# Deployment 상세 출력
kubectl describe deployment nginx-pod
```

<img width="379" alt="스크린샷 2023-04-16 오전 1 14 09" src="https://user-images.githubusercontent.com/87372606/232237291-133bf06a-522d-4938-9ed9-97582cd9b7bd.png">

---

- Deployment 는 ReplicaSet에 문제가 발생했을 경우 **_자동으로 ReplicaSet을 복제하여 명시된 replicas 개수에 대한 가용성 보증_**

  - `kubectl delete replicasets [ReplicaSet name]` 으로 지워서 확인해본다.

<img width="749" alt="스크린샷 2023-04-16 오전 1 23 21" src="https://user-images.githubusercontent.com/87372606/232237710-3eb144f1-fb08-4c4a-ac45-962bb39e9508.png">

---

- `kubectl delete deployment [Deployment name]` 으로 생성한 Deployment 를 삭제할 수 있다.

---

## A visual map of a Kubernetes deployment

<img width="429" alt="스크린샷 2023-04-16 오전 1 26 47" src="https://user-images.githubusercontent.com/87372606/232237889-23eed2fd-60b6-45a2-bb86-b5abb2414727.png">

- deployment 오브젝트를 정의하면

  - deployment 리소스가 만들어지고

  - deployment 컨트롤러가 이것을 보고 있다가 replicasets 를 만들어내고

  - replicasets 컨트롤러가 이것을 보고 있다가 pods 를 만들어낸다.

  - 이것을 스케쥴러가 보고있다가 워커 노드 넘버를 매긴다.

  - kubelet 이 통해 api server 를 보고 있다가 자기 노드에 대한 일거리라면 가져가서 일을 한다.

- **`watch mechanism` : API 서버가 watch 메커니즘을 갖고 있어서 클라이언트들은 자기에게 뭔가 할당되었을때, 인지할 수 있다.**

```
The API server is the only component that can directly talk to etcd, and only it can make changes to it.
So you can assume that etcd exists (hidden) behind the API server in this diagram.

Also, I'm talking about only two main controllers (Deployment and ReplicaSet) here. Others would also work similarly.

The steps below describe what happens when you execute the kubectl create command:

Step 1:
When you use the kubectl create command, an HTTP POST request gets sent to the API server, which contains the deployment manifest.
The API server stores this in the etcd data store and returns a response to the kubectl.

Steps 2 and 3:
The API server has a watch mechanism and all the clients watching this get notified.
A client watches for changes by opening an HTTP connection to the API server, which streams notifications.
One of those clients is the Deployment controller.
The Deployment controller detects a Deployment object, and it creates a ReplicaSet with the current specification of the Deployment.
This resource gets sent back to the API server, which stores it in the etcd datastore.

Steps 4 and 5:
Similar to the previous step, all watchers get notified about the change made in the API server—this time the ReplicaSet Controller picks up the change.
The controller understands the desired replica counts and the pod selectors defined in the object specification, creates the pod resources, and sends this information back to the API server, storing it in the etcd datastore.

Steps 6 and 7:
Kubernetes now has all the information it needs to run the pod, but which node should the pods run on?
The scheduler watches for pods that don't have a node assigned to them yet, and starts its process of filtering and ranking all nodes to choose the best node to run the pod on.
Once the node is selected, that information gets added to the pod specification.
And it gets sent back to the API server and stored in the etcd datastore.

Steps 8, 9, and 10:
All the steps until now occur in the control plane itself.
The worker node has yet to do any work. The pod's creation isn't handled by the control plane, though.
Instead, the kubelet service running on all the nodes watches for the pod specification in the API server to determine whether it needs to create any pods.
The kubelet service running on the node selected by the scheduler gets the pod specification and instructs the container runtime in the worker node to create the container.
This is when a container image gets downloaded (if it's not already present) and the container actually starts running.
```

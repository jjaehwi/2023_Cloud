# K8S Volume 개요 및 emptyDir 과 hostPath

- [Volume 개요](#volume-개요)

- [emptyDir](#emptydir)

  - [emptyDir 의 특성](#emptydir-의-특성)

- [hostPath](#hostpath)

---

## Volume 개요

- kubernetes 의 storage : Volume

- persistent volume 는 Volume 의 종류 중 하나

- Kubernetes supports many types of volumes.

- `Ephemeral volume` types (사라지는 스타일 ex. emptyDir) have a **lifetime of a pod**, but `persistent volumes` **exist beyond the lifetime of a pod**.

- When a pod ceases to exist, **Kubernetes destroys ephemeral volumes**; however, **Kubernetes does not destroy persistent volumes.**

  - `emptyDir 같은 ephemeral volumes` 은 `Pod 가 사라지면 같이 사라지고` (**컨텐츠가 유지되지 않는 것**) Pod 가 새로 만들어지면 새롭게 만들어지는 저장소

  - `persistent volumes` 은 `외부에 따로 존재하는 저장소` 이고, Pod 가 외부에 mount 되어서 사용하다가 Pod 가 사라져도 Volume 은 persistent 하게 유지된다.

- Ephemeral volume 이나 persistent volumes 외에 **다른 스타일의 volumes** 들이 많이 있다.

- 실제 하드웨어적인 storage 가 있을 때, 그 부분을 persistent volumes 로 만들어서 mount 하여 사용할 수 있다.

- public 클라우드 사업자들에게 있는 volumes 를 나의 클라우드에 붙혀서 사용도 가능하다.

  - **awsElasticBlockStore** : AWS 의 block storage 를 사용하는 것

  - **azureDisk** : MS AZURE 의 public 클라우드에 있는 storage

  - **azureFile** : MS AZURE 의 file 형태의 storage

  - **cephfs** : storage 를 mount 하기 위해 다른 관리 기술들을 붙혀서 만드는데, 이러한 관리 체계의 오픈 소스 프로젝트

  - **cinder** : 오픈스택의 storage 에 관련된 cinder 프로젝트 를 volume 으로 사용할 때

  - **gcePersistentDisk** : 구글의 persistent disk

- `hostPath` : **local 한 storage (서버의 storage)**

- storage 를 Pod 에서 돌아가는 application 의 데이터를 저장하고 읽는 용도로 사용하기도 하지만...

  - `configMap` : application 의 configuration 에 관련된 정보 를 따로 관리하기 위한 방법에 대해 정리됨.

    - storage 의 특성상 쿠버네티스 시스템에 돌아가는 어떤 application 에 세팅할 정보가 필요할 때

    - (ex) 응용 프로그램에 돌아가는 nginx 를 설치하고 미리 세팅할 값들이 있을 때

  - `secret` : 암호 코드 같은 key 값을 따로 저장할 때, 체계에 대한 개념에 대해 정리됨.

    - ID / password 같은 정보 를 관리

    - (ex) 웹 서버에 로그인 하기 위한 아이디 / 패스워드 정보

- `downwardAPI` : 클라우드 인프라의 정보를 알고싶을 때, API server 를 통해 명령을 내리고 정보를 받아오고 하는데

  - 거꾸로 명령을 내려 부르는 것

  - Pod 의 이름, Pod 의 IP, Pod 가 실행되는 노드의 이름 등 실제로 파드가 생성 및 실행이 되기전에는 알 수 없는 속성들도 존재

  - 이런 속성들을 컨테이너에서 실행 중인 애플리케이션에서 알아내기 위해 사용됨.

  - Downward API 는 단순히 환경변수, 또는 파일 (downwardAPI 볼륨을 통해) 로 위와 같은 속성들을 컨테이너에서 손쉽게 사용할 수 있도록 하는 기능

---

## emptyDir

- Pod 가 생성될 때, emptyDir 이 생성될 수 있다.

- `emptyDir` : 쿠버네티스가 돌고 있는 환경에서 Pod 가 생성될 때 기본적인 메모리를 이용하여 임시적으로 만들어 놓은 storage

```
// emptyDir configuration example
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```

- container 가 돌아가는 Pod 를 만드는데, containers 의 name 과 image 같은 정보를 적어준다.

  - `추가적으로 storage 가 필요할 때, Volume 에 관한 정보를 적어줘야한다.`

    - 컨테이너가 만들어지고 `volumes:` 에 해당하는 정보를 가진 Volume 이 붙어야한다 라고 하는 것

    - `emptyDir` 라고 하는 것은 쿠버네티스가 돌고 있는 환경에서 메모리를 이용하여 임시적으로 만들어 놓은 storage 이다.

  - Volume 에 대한 정보를 volumes 에 적으면 항상 **containers 의 volumeMounts 의 mountPath** 를 적어야한다.

    - `만든 Volume 을 만들어진 container 의 어떤 위치에 생성 (mount) 할 것인지 적는 것`

    - 이 때, mount 하려는 Volume 의 이름과 volumes: 에 대한 정보로 만든 Volume 의 이름이 같아야한다.

---

### emptyDir 의 특성

- An emptyDir volume is `first created when a Pod is assigned to a node`, and `exists as long as that Pod is running` on that node

  - Pod 가 생겨날 때, **쿠버네티스가 돌고있는 환경의 메모리를 이용해서 emptyDir 성격의 Volume 이 붙지만** 이는 `임시적이라 Pod 가 죽으면 같이 사라진다.`

- `RAM 형태`의 디렉토리이다. (RAM-backed filesystem)

- `같은 Pod 안에 두 컨테이너`가 있고, 둘 다 emptyDir 을 사용할 때, `공유`가 될 수 있다.

---

## hostPath

- emptyDir 과 성격이 비슷하지만 사라지는 면에서 다르다.

- `hostPath` : A hostPath volume mounts a file or directory `from the host node's filesystem into your Pod.`

- RAM 을 가지고 쓰는 것이 아닌 local 한 하드 디스크에 쓰는 것이므로 `Pod 가 사라져도 데이터가 남아있다.`

```
// hostPath configuration example
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
```

- Pod 의 containers 를 정의할 때, `volumeMounts` 로 **어떤 Volume 이 어디에 mount 될지** 적는다.

- `volumes:` 로 Volume 에 대한 정보를 정의한다.

  - name 은 volumeMounts 에 적은 이름과 같은 Volume 이름 이어야 한다.

  - `hostPath` 를 통해 path 를 정해준다.

    - **노드에 붙은 local 한 하드 디스크 (메모리) 의 어떤 디렉토리에 있는 부분을 Volume 으로 사용할지 적는 것.**

    - host 에 있는 어떤 부분 중 어떤 path 에 있는 애를 쓸지 적는 것.

    - RAM 을 가지고 쓰는 것이 아닌 하드 디스크에 쓰는 것이므로 `Pod 가 사라져도 데이터가 남아있다.`

    - **새로운 Pod 가 생겨날 때, hostPath 의 위치에 붙일 수 있고 못 붙일 수도 있다.**

      - 서버가 여러개면 Pod 는 어디에 생성될지 모르기 때문. (어느 워커 노드에 생성될지 모르는 것)

- `persistentVolume` 은 **외부에 있어서 내가 찾아서 사용할 수 있지만** `hostPath` 는 **`local 에 존재하는 특정 공간을 Volume 으로 사용하는 것`**이기 때문

---

`References`

- [Kubernetes.io 의 Storage/Volume 문서](https://kubernetes.io/docs/concepts/storage/volumes/)

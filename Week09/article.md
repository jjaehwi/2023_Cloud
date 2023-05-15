# Kubernetes Volumes

- 지금까지 쿠버네티스에서 application 이 돌아가게 하는 container 를 Pod 에 deploy 하고, Pod 자체가 Node 에 deploy 되는 내용과 서비스를 만드는 내용에 대해 공부했다.

- `storage 에 관련된 내용을 공부`한다.

- `Container 에서 Mount Volume 하는 방법에 대해 학습`한다.

---

- [기초용어](#기초용어)

- [How to persist data in Kubernetes using volumes?](#how-to-persist-data-in-kubernetes-using-volumes)

  - [The need for Volumes](#volume-의-필요성)

  - [Storage 요구사항](#storage-요구사항)

- [Persistent Volume](#persistent-volume)

  - [Local vs Remote Volume Types](#local-vs-remote-volume-types)

- [PersistentVolumeClaim](#persistentvolumeclaim)

- [중간 정리](#중간-정리-persistentvolume-persistentvolumeclaim)

- [Why so many abstractions?](#why-so-many-abstractions)

- [ConfigMap and Secret](#configmap-and-secret)

- [Storage Class](#storage-class)

---

## 기초용어

- `Drive` : **storage device**

- `Volume` : **Logical 한 drive**, **단일 파일 시스템을 갖춘 단일 storage 영역**

  - single accessible storage area with a single file system

  - (ex) Windows c:, d:, ,,,

- Mount Volume : 특정 위치에 연결 시키는 것

- **컨테이너에 어떻게 논리적인 drive 인 Volume 을 mount 시킬 수 있을까..?**

  - 컨테이너에 Volume 을 mount 시키면, 컨테이너의 root directory 에 붙는 것이 아니고 내가 원하는 위치 (컨테이너 디렉토리 상)에 붙힐 수 있다.

---

## How to persist data in Kubernetes using volumes?

- 프로그램이 파일에 data 를 적었고 그 파일이 존재하는 곳이 특정 Volume (directory) (drive) 인데, Pod 는 없어질 수도 있고 다시 생성될 수도 있다.

  - Pod 가 사용했던 정보가 어떤 파일에 저장이 되어있을텐데, 문제가 되어 Pod 가 사라졌을 때, 그동안 썼던 data 를 어떻게 유지할까?

  - `persistent 한 Volume` 이 만들어져야한다.

<img width="616" alt="스크린샷 2023-05-01 오후 11 20 05" src="https://user-images.githubusercontent.com/87372606/235466025-7dad1829-b837-4b2d-a39b-12d5d7963b4e.png">

1. `Persistent Volume`

2. `Persistent Volume Claim`

3. `Storage Class`

- 위 3 가지의 개념을 통해 data 를 persist 하게 보존할 수 있는 체계를 갖출 수 있다.

---

### Volume 의 필요성

- Pod 안에 프로그램 (application) 을 돌리고 있을 때, application 은 data 를 mysql engine 이 돌아가는 Pod 에 추가한다.

- 그러면 mysql (데이터베이스) 가 업데이트 된다.

- 이 때, **Pod 가 문제가 생겨서 사라져버리고, 대체할 다른 Pod 가 생겨났을 때, data 는 유지될 수 없다**.

  - Pod 자체가 새로 만들어지게 되면 그 전 data 는 사라지고 original 프로그램만 다시 돌아가기 때문이다.

- `data 가 persistent 하지 못하다.`

<img width="625" alt="스크린샷 2023-05-01 오후 11 29 44" src="https://user-images.githubusercontent.com/87372606/235467679-ed194c4b-2a49-4f43-9cbb-086dc3fbc0df.png">

<img width="625" alt="스크린샷 2023-05-01 오후 11 29 49" src="https://user-images.githubusercontent.com/87372606/235467669-5519583a-a39d-4856-b7b8-362bbfe8f623.png">

<img width="625" alt="스크린샷 2023-05-01 오후 11 30 37" src="https://user-images.githubusercontent.com/87372606/235467802-33712b50-6f59-4d34-9bd6-cb161994833f.png">

- **이런 문제를 막기 위해 등장한 `storage 요구사항`**

---

### Storage 요구사항

1. `storage 는 Pod 의 lifecycle 에 영향을 받지 않아야한다.`

2. `storage 는 어떤 Node 에서든지 접근 가능해야한다.`

   - **Pod 가 새로 만들어질 때, 원래 있던 노드에 만들어질 수도 있지만 스케쥴러에 의해 아무 워커 노드 중 적당한 곳에 가서 Pod 가 만들어질 수도 있다.**

   - 그렇기 때문에 **storage 가 특정한 Node (server) 에 있고 그 Node 에서만 접근할 수 있게되면 문제가 생길 수 있다.**

<img width="625" alt="스크린샷 2023-05-01 오후 11 35 41" src="https://user-images.githubusercontent.com/87372606/235468578-3eb7e9c5-05e1-4318-b32b-76637cf254bb.png">

3. `storage 는 cluster 가 crash 되더라도 survive 해야한다.`

   - cluster 자체가 문제가 될 수도 있고 그 와중 storage data 는 survive 해야하는 경우가 발생할 수 있다.

<img width="625" alt="스크린샷 2023-05-01 오후 11 37 27" src="https://user-images.githubusercontent.com/87372606/235468852-2d5b682a-9083-4469-a82a-41f8fa25f826.png">

- 위 요구사항을 만족할 수 있는 storage 시스템을 쿠버네티스에게 만들어줘야한다.

---

## Persistent Volume

<img width="616" alt="스크린샷 2023-05-01 오후 11 46 24" src="https://user-images.githubusercontent.com/87372606/235470341-a21e5dd2-e75b-484d-9bb7-5f472090fa7e.png">

- CPU 나 메모리 처럼 하나의 `새로운 리소스`로써 Volume 을 생각한 것

- 리소스이므로 만들기 위해서 yaml 파일로 spec 을 써서 creation 한다.

<img width="616" alt="스크린샷 2023-05-01 오후 11 51 51" src="https://user-images.githubusercontent.com/87372606/235471267-d8369a55-0220-4815-a067-0fddbd7b2c18.png">

- 실질적인 storage 는 다양한 형태로 있을 수 있다.

  - 쿠버네티스 클러스터에는 서버 여러개가 묶여있는 형태이고, 여러 서버 중 storage 만 가지고 있는 서버가 있을 수 있고, storage 가 많이 붙어있는 서버도 있을 수 있다.

  - 이런 것들은 모드 `local disk (local 한 서버)` 인 것이다.

  - 또는 클러스터에 네트워크로 연결해서 network filesystem 이라는 spec 을 통해 `nfs 형태의 서버`도 존재할 수 있다.

  - 또는 클라우드 사업자들 (아마존, 구글) 이 제공하는 storage 를 이용할 수도 있다. (`Cloud-storage`)

  - 여기서 **어떤 종류의 storage 를 원하고, 어떻게 create 하고 manage 할 것인지에 대해 생각**해야한다.

<img width="616" alt="스크린샷 2023-05-01 오후 11 52 38" src="https://user-images.githubusercontent.com/87372606/235471400-fe76404e-4864-4723-880e-05e3ba94e574.png">

<img width="616" alt="스크린샷 2023-05-01 오후 11 52 52" src="https://user-images.githubusercontent.com/87372606/235471436-3e9f7ccb-8e11-4d78-b39b-3bf51bc54295.png">

<img width="616" alt="스크린샷 2023-05-01 오후 11 53 06" src="https://user-images.githubusercontent.com/87372606/235471466-fb7d6d61-2157-435e-8b94-50bce93532f6.png">

- [kubernetes.io](https://kubernetes.io/docs/concepts/storage/) 에서 어떻게 yaml 파일을 작성해야하는지 나와있다.

- **capacity: storage: 에 대한 내용이나 accessModes 에 대한 내용은 공통적**이다.

<img width="616" alt="스크린샷 2023-05-02 오전 12 00 22" src="https://user-images.githubusercontent.com/87372606/235472732-e4cbb62c-67ac-46c5-a95d-dd8a0a44cec7.png">

- **내가 쓰고자하는 storage 에 대한 내용을 PersistentVolume yaml 파일로 define**를 할 수 있다.

- `PersistentVolume 의 yaml 파일을 통해 정의된 storage 는 쿠버네티스 클러스터 안에 논리적으로 존재`한다.

  - 실질적으로는 local disk, nfs server, cloud-storage 가 될 수도 있는 것

  - PersistentVolume yaml 파일을 통해 만든다고 하면 클러스터 안에 만들어지는 것

- PersistentVolume 은 클러스터 안에서 누구나 접근 가능해야하는 공통 자원이기 때문에 어떤 namespace 던지 access 가능하다. (`Accessible to the whole cluster`)

- namespace 안에 존재하는 것이 아니고 클러스터 안에 별도로 존재하는 자원이다. (`outside of the namespaces`)

---

### Local vs Remote Volume Types

- local volume type 은 요구사항 2, 3 을 만족하지 못한다.

  - 요구사항 1, 2 는 만족하게 할 수 있다.

- Pod 가 죽어도 접근가능하고, 클러스터 안에서는 다 쓸 수 있게끔 하는 것은 가능하지만, `근본적으로 remote 에 있으면 더 안전`하다.

---

## PersistentVolumeClaim

- 쿠버네티스의 PersistentVolume 은 별도의 관리자가 만들어 내는 것이다.

- **Pod 가 Volume 의 data 를 필요로 할 때 `Volume 은 미리 만들어져있어야한다.`**

  - **PV are resources** that need to be there `BEFORE`

<img width="619" alt="스크린샷 2023-05-02 오전 12 37 22" src="https://user-images.githubusercontent.com/87372606/235479464-ad9d9d15-cb92-4e37-91ed-b1fdf65e690f.png">

- storage 가 administrator 에 의해 다양한 방법에 의해 준비가 되고, **Pod 입장에서 `PersistentVolume` 을 사용하는 방법은 `PersistentVolumeClaim` 이라는 새로운 개념 (object) 를 통해 사용한다.**

  - pv 를 만드는 것과 사용하는 것을 `decoupling` 한 것이다.

  - pv 는 구체적인 Volume 의 type 이나 디테일한 parameter 들을 세팅해야하는데, 사용자 입장에서 그것까지 신경쓰지 않아도 되도록 한 것이다.

  - 사용자가 원하는 용량이 어떻고, 어떤 accessMode 를 원하는지와 같은 `기본적인 것들에 대한 것만 요구하도록 한 것`이다.

- `PersistentVolumeClaim` : 쓰는 사람 입장에서 필요한 것들을 별도의 spec 파일로 define 하도록 한 것

  - **직접 pv 의 spec 을 써서 요청하는 것이 아닌 pvc 라는 새로운 spec 을 통해 요청**하는 것

<img width="619" alt="스크린샷 2023-05-02 오전 12 44 31" src="https://user-images.githubusercontent.com/87372606/235480661-698f67a6-e43f-4a5c-806e-0f6b44ac3357.png">

- 쿠버네티스가 `PersistentVolumeClaim 에 요구된 요구 내용`과 `미리 관리자가 준비해놓은 Volume 의 종류`를 비교를 해서 (요청한 것과 실제 갖고있는 것을 비교) **맞는 쪽으로 연결을 시킨다.**

---

## 중간 정리 (PersistentVolume, PersistentVolumeClaim)

- `PersistentVolume` 을 만드는 것은 관리자가 PersistentVolume 을 documentation 한 것에 맞춰서 PersistentVolume yaml 파일에 구체적인 Volume 정보를 써서 생성하는 것

- `PersistentVolumeClaim` 은 Pod 가 갖고 싶은 Volume 의 종류를 define 하는 것

- 이 둘을 `매칭` 시키는 것은 `쿠버네티스`가 내부적으로 동작해서 한다.

---

<img width="619" alt="스크린샷 2023-05-02 오전 12 44 53" src="https://user-images.githubusercontent.com/87372606/235480719-c9580b83-846f-41a3-97ed-28fe54b57e5c.png">

<img width="625" alt="스크린샷 2023-05-02 오전 1 15 26" src="https://user-images.githubusercontent.com/87372606/235485725-76694b15-7947-4df1-955c-94ff8e7dff31.png">

- PersistentVolumeClaim 에 쓰고 싶은 Volume 을 적으면 Pod 는 이를 이용하여 요청한다.

- volumes 라는 field 에다가 Pod 는 어떤 Volume 을 붙힐지 define 한다.

  - 여기서 persistentVolumeClaim field 의 claimName field 에다가 미리 기술되어있는 PersistentVolumeClaim 의 name 을 적어서 Volume 생성을 요청한다.

- `Pod 를 creation 할 때, Volume 중 PersistentVolumeClaim 에 쓰여있는 spec 대로 있는 Volume 이 있는지 찾아서 Pod 에 붙힌다.`

- 이렇게 **만든 Volume 을 container 가 사용**하는데 `container 의 어떤 path 에 mount 할 지 volumeMounts filed 에 define` 한다.

  - `어떤 Volume 을 어떤 위치에 mount 할 지 표시하는 것`

  - **Apps can access the mounted data here: “/var/www/html”**

<img width="619" alt="스크린샷 2023-05-02 오전 12 56 02" src="https://user-images.githubusercontent.com/87372606/235482607-41a9f782-87c2-4a59-be52-4b17546f633b.png">

<img width="619" alt="스크린샷 2023-05-02 오전 12 56 10" src="https://user-images.githubusercontent.com/87372606/235482629-eb780e71-227f-4d0c-b43c-01a3a0614586.png">

<img width="619" alt="스크린샷 2023-05-02 오전 12 56 19" src="https://user-images.githubusercontent.com/87372606/235482652-e8ee749d-6635-4672-ae5a-25f90c9280ad.png">

<img width="619" alt="스크린샷 2023-05-02 오전 12 56 28" src="https://user-images.githubusercontent.com/87372606/235482681-97b22445-103f-49f3-857e-5a9132a170f2.png">

- `Claim must be in the same namespace!` (claim 은 같은 namespace 에 존재한다.)

  - Pod 는 여러 개 만들어지는데, Pod 들이 공통적으로 원하는 Volume 의 type 에 대해서 pvc 로 기술해놓을 수 있는데, 이것은 같은 그룹 내에서만 미리 정의해놓은 pvc 스펙에 따라 요청할 수 있는 것이다.

  - 다른 namespace 에 있는 애들이 pvc 를 만들어도 (동일한 이름일지라도) 서로 보이지 않는다.

  - `실제 스토리지 (pv) 는 namespace 에 관계없고, claim 하는 스펙 자체는 동일한 namespace 에서만 유효하다.`

<img width="625" alt="스크린샷 2023-05-02 오전 12 59 25" src="https://user-images.githubusercontent.com/87372606/235483153-d7c4f3af-0f8d-44f8-99c4-3902ce5b7891.png">

---

## Why so many abstractions?

- 구체적으로 적는 것 대신 `공통적으로 표준화된 방식으로 쓰는 것을 abstract` 이라고 한다.

- Data should be safely stored

- 사용자 입장에서는 actual storage 를 set up 할 필요가 없다.

---

## ConfigMap and Secret

- 두 가지 특별한 `data local Volume` : 일반적인 Volume 과 다르다.

  - pv 나 pvc 를 통해 만들지 않는다.

  - `쿠버네티스가 스스로 creation 하고 관리`한다.

1. `ConfigMap` : 설정하는데 쓰이는 parameter 같은 것들을 담아두는 특별한 파일

   - 프로그램이 초기화될 때, 세팅해줘야하는 프로그램 dependent 한 세팅 값과 같은 정보들을 configuration file 이라 하는 special 한 Volume 을 통해 정보를 적어두고 사용한다.

2. `Secret` : certification, 인증 파일, 암호키 처럼 secret 한 것들은 일반적인 Volume 처럼 관리하지 않고 특별하게 관리한다.

<img width="625" alt="스크린샷 2023-05-02 오전 1 13 09" src="https://user-images.githubusercontent.com/87372606/235485387-2af9da92-9736-46d7-abad-ba837f75862b.png">

<img width="625" alt="스크린샷 2023-05-02 오전 1 14 04" src="https://user-images.githubusercontent.com/87372606/235485516-95ec6901-f8d6-4e74-9e4a-d7e92bbf5290.png">

<img width="625" alt="스크린샷 2023-05-02 오전 1 17 34" src="https://user-images.githubusercontent.com/87372606/235486040-92c00938-92f9-4fda-bbd3-7af6c08dab19.png">

---

## Storage Class

- 지금까지 배운 것

1. Admins configure storage : 관리자가 storage 를 준비시킨다.

2. Create Persistent Volumes : Volume 이 만들어지면

3. K8S Users claim PV using PVC : 유저는 pvc 를 통해 pv 를 요청 (claim) 한다.

<img width="625" alt="스크린샷 2023-05-02 오전 1 24 11" src="https://user-images.githubusercontent.com/87372606/235487188-2bb27dda-1fee-4b11-bf87-46180cee2a43.png">

- `Challenge`

  - 만약 쿠버네티스 안에 여러 가지 응용 프로그램이 돌아가면서 Pod 들이 생성되고 사라지고 할텐데, Pod 별로 원하는 Volume 의 종류를 pvc 를 통해 요청한다.

  - **pvc 를 생각해서 pv 가 미리 준비가 되어있어야하는데**

  - **새로운 Pod 가 만들어질 때, 쓰여있는 pvc 를 읽어보니 준비되지 않은 Volume 을 요청할 때, 관리자는 빠르게 뒷 편에서 맞는 pv 를 준비**해야한다.

  - 이 작업이 한 두개이면 괜찮지만 많아지게 될 경우 (hundreds of applications) deploy 되고 각각에 맞춰 storage 가 준비될 때, `시간 소모가 심하고 복잡`하다.

---

- `Storage Class 개념` : **Persistent Volumes 를 미리 만들어두는 것이 아니라 dynamic 하게** 하기 위해 (필요할 때만 만들어지도록 하는 것) 도입된 개념

<img width="625" alt="스크린샷 2023-05-02 오전 1 25 56" src="https://user-images.githubusercontent.com/87372606/235487447-9c65ffb6-925b-460e-b65e-2dda49dbe115.png">

- internal, external, provisioner 를 define 해서 configeMap 을 세팅해놓고 그 때 그 때 필요한 storage 를 storage Class 라는 정보로 요청하면 그거에 맞는 것이 dynamic 하게 생성되도록 한다.

- `그 때 그 때 만들어지도록 자동화된 툴을 둔 것.`

  - 자동으로 준비시킬 수 있게끔 하는 연결고리를 만든 것

  - `abstract 한 style : 실제 detail 보다는 대강 그 종류라면 그 상황에 맞춰 준비시킬 수 있는..`

  - 사람이 manually providing 하던 것들을 `abstraction`

  - parameters for that storage

<img width="625" alt="스크린샷 2023-05-02 오전 1 30 56" src="https://user-images.githubusercontent.com/87372606/235488223-092a54b6-481c-4491-ada2-9f2f48872328.png">

- pvc 에서는 volumeClaim 에서 PersistentVolume 을 얘기했는데, 이제 class name 을 얘기한다.

  - `Volume 은 존재해야하는 것`이지만, `class name 을 얘기하는 것은 존재하지 않더라도` `class name 을 통해 Storage Class config 에 적힌 spec 을 dynamic 하게 요청`하는 것

<img width="625" alt="스크린샷 2023-05-02 오전 1 33 19" src="https://user-images.githubusercontent.com/87372606/235488569-de2e70c9-a68b-4876-a1cf-5a1eff180728.png">

- `Claim 입장에서 Persistent Volume 을 spec 에 쓰게 되면 만들어져 있는 것을 바로 요청하는 것이 되고, Storage Class 를 쓰게 되면 요청됐을 때 필요한 것을 dynamic 하게 만들어낸다.`

---

`Reference`

- DCN LAB

- TechWorld with Nana

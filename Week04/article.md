# 쿠버네티스 (Kubernetes) 기초

- [가상화의 발전](#가상화의-발전)

  - [컨테이너 오케스트레이션 플랫폼](#container-orchestration-platform)

  - [컨테이너 엔진과 오케스트레이션](#container-engine-orchestration)

- [쿠버네티스](#kubernetes)

  - [쿠버네티스 기본 구조](#쿠버네티스-기본-구조)

  - [Kubernetes Resources](#쿠버네티스-리소스)

  - [Control Plane](#control-plane-master)

  - [Data Plane](#data-plane-worker)

---

## 가상화의 발전

<img width="961" alt="스크린샷 2023-04-02 오전 1 06 54" src="https://user-images.githubusercontent.com/87372606/229301777-000c5337-1439-409c-91ce-2daa30cb81c5.png">

- 기존 일반 서버에 어플리케이션을 돌릴 때, 기본적인 **응용프로그램 (App) 이외에도 라이브러리들 (Bin / Libs)**이 같이 컴퓨터에서 돌아가게 된다.

  - 방법 1 : Hypervisor 를 통해 VM 을 만들기 위한 소프트웨어를 설치하고 Guest OS 를 설치하여 돌렸다. 이는 Guest OS 가 추가로 돌기 때문에 overhead 가 있다.

  - 방법 2 : 컨테이너 엔진 (컨테이너를 위한 가상화 소프트웨어) 을 통해 그 위에 App 과 Bin / Libs 를 돌릴 수 있다.

    - 리눅스에서 제공하는 namespaces 와 cgroup 을 통해 분리가 가능하다.

---

### Container Orchestration Platform

<img width="961" alt="스크린샷 2023-04-02 오전 1 10 15" src="https://user-images.githubusercontent.com/87372606/229301956-6216b8f5-c0f1-43e7-9b37-9d7eff1ac350.png">

- 환경이 커지게 되면 인프라에 필요한 기본적인 서버들이 여러개 존재하고, 예전과 달리 VM 을 돌리기 위해 환경을 활용하는 것이 아닌 이 위에 `다양한 App 과 Libs 들이 합쳐진 컨테이너들을 돌리고자 했다.`

- 여러 대의 서버가 묶어져 있는 환경에서 돌아갈 때, 이 **동일한 환경에서 통해 개발 (Development), 시험 (Test), 운영 (Production) 단계로 `separation`** 할 수 있다.

  - **수 많은 컨테이너들을 같이 돌릴 수 있고 관리를 하기 위해 관리하는 `플랫폼`이 필요하다.**

- VM 인프라를 만들어서 서비스를 쓰다가 더 가볍고 여러 부수적인 기능들이 잘 되는 새로운 환경을 고민 : container orchestration software

---

### Container Engine, Orchestration

<img width="961" alt="스크린샷 2023-04-02 오전 1 18 17" src="https://user-images.githubusercontent.com/87372606/229302364-115ae3be-0431-4d51-977b-82c7bef44396.png">

- `컨테이너 엔진 (Container Engine)` : **가상화 시켜주는 엔진**, **_컨테이너가 돌아갈 수 있는 환경을 만들어주는 엔진_**

  - operating system 을 서로 완전히 분리된 것 처럼 해서 application 입장에서 독차지하고 있는 것처럼 보이게 해주는 기술

  - `Docker` --> `CRI - O`

- `컨테이너 오케스트레이션 (Container Orchestration)` : **가상화 된 것을 `종합적으로 관리`하는 소프트웨어**, **_컨테이너의 배포 관리_**

  - 컨테이너 오케스트레이션의 목적은 여러 컨테이너의 배포 프로세스를 최적화 하는데 있으며, 이것은 컨테이너와 호스트의 수가 증가함에 따라 점점 더 가치가 생긴다. 이러한 유형의 자동화를 오케스트레이션이라고 합니다.

  - `deployment` (컨테이너를 만들어서 동작시키는 것), `scaling` (규모를 가변화 하는 것), `connet` (컨테이너끼리의, 외부로의 연결) 를 관리

  - script, Manual, static configuration 으로 하는 방식에서 발전

  - Mesos --> Docker swarm --> **`Kubernetes`**

---

## 쿠버네티스 (Kubernetes)

- Google partnered with the `Linux Foundation` to form the **_Cloud Native Computing Foundation (CNCF)_** to offer Kubernetes as an open standard

- Frequently abbreviated `k8s` : Greek for helmsman or pilot (선장)

- **Kubernetes is an `open-source system` for `automating deployment`, `scaling`, and `management of containerized applications`.**

  - **쿠버네티스는 `컨테이너화된 응용 프로그램의 배포, 확장 및 관리를 자동화하는 오픈 소스 시스템`이다.**

---

### 쿠버네티스 기본 구조

<img width="701" alt="스크린샷 2023-04-02 오전 1 39 10" src="https://user-images.githubusercontent.com/87372606/229303445-3478705b-cf89-4ee6-b781-87ecbac93fbe.png">

- VM 에 올려야 할 여러가지 App 들이 있을 때, 오픈스택을 통해서 VM 을 만들고 요청하고 VM 이 만들어지면 그 위에 소프트웨어를 돌리면된다.

- 컨테이너 입장에서도 마찬가지로 App 을 돌리기 위한 Bin / Libs 를 합쳐놓은 패키지화 되어있는 것 (돌려야 할 응용프로그램들) 들이 있는데, **쌓여있는 컨테이너들을 워커 노드 (서버) 들에게 보내야 한다.**

- 쿠버네티스가 하는 기본적인 관리는 컨테이너를 실을 수 있는 공간 (CPU 메모리) 이 있는 노드를 찾아서 올려주는 것이다. (`Container 들을 Deploy 하는 것`)

  - `종합관리를 해주는 소프트웨어 : kubernetes Master`

- 쿠버네티스가 돌아가는 환경에는 `master` 가 있어야하고, 일거리가 돌아갈 수 있는 `worker node` 가 있어야한다.

  - worker node 의 수는 규모에 따라 달라질 수 있다.

  - master 는 한 개와 백업, 백백업 해둬서 3개 정도..

  - **`Master Node` 와 `Worker Node` 로 구성되어 있는 기본적인 단위를 `Cluster` 라고 한다.**

  - `쿠버네티스는 Cluster 들을 관리하는 것`

  - 적절한 노드를 찾아내는 것 : scheduled, 노드에 할당하는 것 : packed

- `Kubernetes runs anywhere` : Cloud Native 의 장점 (`portability`)

  - laptop (마스터 노드와 워커 노드가 분리되지 않고 통합되어있는 교육용 환경 : minikube)

  - Globally distributed data centers (데이터 센터)

  - Major cloud providers (클라우드 사업자)

<img width="981" alt="스크린샷 2023-04-02 오전 1 55 31" src="https://user-images.githubusercontent.com/87372606/229304215-69f4845e-17f3-49e2-83c8-235b16c2c627.png">

- `worker node` 에는 **`컨테이너 엔진`**, **`Pod`**, **`kubelet`**, **`Kube - proxy`** 가 있다.

  - 기본적인 HW 위에 OS 가 깔려있고 그 위에 `가상화하는 컨테이너 엔진 (ex. Docker)` 가 돌아간다.

  - 그리고 그 위에 `Pod` 가 있어서 컨테이너가 돌아간다.

    - Pod 안에 컨테이너가 돌아가고 있는 것이다.

  - **Master 노드의 여러 컴포넌트 들과 짝이 되어 통신하는 agent software** (백그라운드에서 돌아가고 있는 소프트웨어) 가 `kubelet` 이다.

  - **네트워킹을 도와주는 agent** 인 `Kube-proxy`

- `master` 에는 **`API Server`, `Scheduler`, `Controller Manager`, `etcd`** 가 있다.

  - 명령어의 표준화된 포맷인 **RESTful API** 가 있는데, 이런 **API 를 handling 하는 서버가 `API server`** 이다.

    - 그래서 Master 에 RESTful API 로 요청을 하면, 명령을 이해하고 작업을 할 수 있다.

  - **API server 가 정보를 저장하는 저장소를 필요**로 하는데 그 **데이터베이스가 `etcd`** 이다.

    - 시스템의 상태 (어느 노드에 어떤 pod 가 돌고 있는지에 대한 정보, 누가 요청을 한지에 대한 정보) 가 저장되어있다.

  - 보통은 명령을 내려서 worker node 가 일을 하게 할텐데 **_특이하게도 kubelet 이 API server 를 통해 etcd 에 있는 정보를 읽는다._**

    - etcd 가 칠판과 같은 역할인 것이다. kubelet 이 API server 를 통해 칠판을 보고 자기 상황과 판단하여 행동을 하는 것이다.

    - 사람이 API server 에게 어떤 상태를 만들어 달라고 요청을 하는 것이고 그 내용이 etcd 에 저장되면 worker node 의 kubelet 들이 이것을 보고 동작하는 것이다.

  - `Scheduler` 는 **요청된 컨테이너화된 workload 를 어느 노드에 배치시킬지를 결정, Kubelet 과 정보교류를 통해 각 노드의 상황을 바탕으로 결정한다.**

    - 칠판에 Pod A 를 돌려야한다고 써져있는데 누구에게 돌려야할지가 안적혀있으면 스케쥴러는 누가 돌려야할지 판단하여 칠판에 적는다.

  - `Controller Manager` 는 etcd 의 상태정보를 보고 **여러 필요한 일들을 해주는 확장될 수 있는 기능들에 대해 종합적으로 모아 놓은 것이다.**

---

<img width="577" alt="스크린샷 2023-04-15 오후 8 05 42" src="https://user-images.githubusercontent.com/87372606/232210221-3c2f6536-9857-4dbf-8e8c-f5ca961a5a0a.png">

- `큐브 컨트롤 명령 보내기`

  - 직접 `REST API` 로 명령을 보내던가, `클라이언트 (CLI)` 를 통해 보내던가, `SDKs (프로그램 코드상에서 필요할 때)` 를 통해 보낸다.

  - kubectl 명령어를 사용하는 것으로 실습 진행

- control plane 에서 내가 원하는 것을 kube - apiserver 에 요청하는 것이 아닌 kube - apiserver 가 알아들을 수 있는 형태로 바꿔서 보내야한다.

  - **PoD 에 대한 스펙을 적은 것을 데이터로 만들어 달라고 요청하면 (API 서버에) 그러면 etcd 에 데이터를 쓰고 스케쥴러가 그 데이터를 보고 API 서버에 올려놓고 그 스케줄을 확인한 워커노드가 일을 한다.**

  - kubelet 들은 주기적으로 apiserver 를 주시하는데 자기에게 일이 할당되면 보고 동작을 하는 것.

- 그래서 **`Declarative`** 한 스타일 인 것이다.

---

### 쿠버네티스 리소스

<img width="726" alt="스크린샷 2023-04-15 오후 8 15 53" src="https://user-images.githubusercontent.com/87372606/232210652-d0714cd2-d0b5-49ea-b9bf-1919585f6d27.png">

- 흔히 쓰일 만한 표준적인 리소스들을 define 해뒀다. 워커노드는 표준적인 리소스기 때문에 명령을 보고 수행할 수 있다. 하지만 커스텀된 리소스라면 이를 처리할 애들도 필요하다.

  - customer 가 리소스를 define 할 수도 있다. `CRD (Customer Resource Define)`

  - CRD 를 보고 일할 사람도 필요한데 이게 `controller` 이다.

- pod 여러개를 만들고 싶으면 replacaset 을 만든다.

  - 레플리카 셋이란? 레플리카 셋은 일정 개수의 포드를 유지하는 컨트롤러이다.

  - 쿠버네티스의 기본 단위인 포드는 여러 개의 컨테이너를 추상화해 하나의 어플리케이션으로 동작하도록 만드는 컨테이너 묶음

---

### Control Plane (Master)

<img width="908" alt="스크린샷 2023-04-15 오후 8 20 35" src="https://user-images.githubusercontent.com/87372606/232210834-635398b0-f054-4147-a103-eda6f06edbb8.png">

**_kube - apiserver_**

- `모든 구성요소들과 communicate 할 수 있는 component` ( 모두 apiserver 를 거쳐서 간다 )

  - `api objects (리소스 오브젝트)` 들은 apiserver 에 쓸 수 있는 메뉴들인 것 (pods, services, deployments ,,,,)

  - (ex) pods 라는 오브젝트, pods 라는 실제 만들어진 것

  - (ex) services 라는 이름의 오브젝트, 실제 만들어진 일반적인 services

- apiserver 를 이용하는 방법은 http 메세지를 만들어보내던가 sdks 를 통해 보낼 수 있지만 `kubectl command` 를 통해 할 것이다.

  - `kubectl create` (services, cronjobs, deployments, quotas) 를 통해 오브젝트 에 대한 명령을 작성하면 **kubelet 이라는 애가 동작하여 오브젝트 주문 대로 만들어준다.**

  - `kubectl get` 을 통해 만들어졌는지 알 수 있다.

**_kube-scheduler_**

- `pods 의 스케쥴링을 담당`한다.

  - Kubernetes 스케줄링이란 kubelet 이 Pod 를 실행할 수 있도록 Pod 가 Node 에 적합한지 확인하는 것을 의미.

  - Scheduler 는 새로 생성된 모든 Pod 나 예약되지 않은 다른 Pod 에 대해서 실행할 최적의 Node 를 선택한다.

**_kube-controller-manager_**

- `the core control loops` (**명세대로 동작이 제대로 됐는지 확인**하는) 를 가지고 있다.

  - (ex) pods 2 개를 만들어달라고 요구했다면 (replica set), 명세를 확인하고 2개가 아닐 때 2개가 되도록 동작을 해준다.

  - **원하는 것과 실제된 것을 확인해봐서 안맞으면 맞게끔 맞춰주는 것**

**_cloud – controller - manager_**

**_etcd_**

- `데이터베이스` (key - value 스타일 로 저장), 중요하기 때문에 2중, 3중화한다.

---

### Data Plane (Worker)

<img width="908" alt="스크린샷 2023-04-15 오후 8 27 03" src="https://user-images.githubusercontent.com/87372606/232212182-5153e5d1-5c03-4659-bb73-c6ed7fa619e9.png">

**_kube - proxy_**

**_kubelet_**

- PodSpecs 이 돌아가게 해준다.

**_container runtime_**

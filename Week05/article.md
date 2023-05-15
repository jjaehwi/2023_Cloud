# 쿠버네티스 설계 개념

- 목표 : 쿠버네티스를 깊이 이해하기

- [컨테이너를 어떻게 배포 하는지](#how-to-deploy-a-workload)

  - [Principle 1](#princlple-1-declarative-apis)

  - [Principle 2](#principle-2-there-are-no-hidden-internal-apis)

  - [Principle 3](#principle-3-meet-the-user-where-they-are-supporting-legacy-applications)

  - [Principle 4](#principle-4-workload-portability)

- [왜 Portability 가 중요할까](#why-workload-portability)

- [정리](#principles-정리)

---

## How to deploy a workload?

- 쿠버네티스는 컨테이너화 되어있는 서비스들을 시스템에 deploy 하는 것을 자동화했다.

  - 어느 환경이던 상관없이 deploy 를 할 수 있다.

- 쿠버네티스는 컨테이너화 되어있는 일거리들을 deploy 하고 잘 deploy 됐는지 monitoring 하는 것을 스스로 한다.

  - 오픈스택은 VM 을 만든 후 일일이 monitoring 하고 문제 발생시 명령을 내려야했다.

<img width="655" alt="스크린샷 2023-04-02 오후 7 43 45" src="https://user-images.githubusercontent.com/87372606/229348070-44908b9c-131b-4c70-b21c-fb1e36ffc513.png">

- 컨테이너 X 라는 것이 있을 때, 노드 B 에 deploy 하게 끔 한다.

<img width="971" alt="스크린샷 2023-04-02 오후 7 44 50" src="https://user-images.githubusercontent.com/87372606/229348112-f00cef3e-5cba-45c0-a105-6b37cd61e2f6.png">

- 만약 사람이 API 를 통해 노드 B 에 명령을 내리게 될 경우 문제점은 노드 B 에 문제가 생겼을 경우 유저가 노드 B 의 상태를 계속 지켜봐야하는데 이런 방법이 복잡하다 (Complex).

  - 유저가 자체적으로 logic 을 설계해야하기 때문에 자동화의 방법에서는 좋지 못한 방법이다.

  - You : provide exact set of instructions to drive to desired state

  - System : executes instructions

  - You : monitor system, and provide further instructions if it deviates. (모니터링을 하고, 필요하다면 추가적으로 명령을 함)

- 쿠버네티스는 기존의 클라우드 오케스트레이션 시스템과 달리 `declarative` 한 방식을 선택했다.

  - 내가 원하는 상태를 API 를 통해 declaration 하면 그 다음은 자동화된다.

---

### Principle 1. Declarative APIs

- **`Kubernetes APIs are declarative`** : 원하는 것을 서술하는 식

  - rather the imperative : 세부적인 명령 대신에

  - You : `define desired state` (원하는 상태를 define 만 하면 된다)

  - System : `works to drive towards that state` (시스템은 이 상태가 되도록 스스로 일을 한다)

<img width="697" alt="스크린샷 2023-04-02 오후 7 54 24" src="https://user-images.githubusercontent.com/87372606/229348482-24361ce2-682f-450f-853e-208697031bc7.png">

<img width="697" alt="스크린샷 2023-04-02 오후 7 55 48" src="https://user-images.githubusercontent.com/87372606/229348545-b301d8db-4a23-4ccf-9858-9ef6eb7b04fc.png">

- You : 삭제될 때까지 `kube API 서버에 (on kube API server)` 유지되는 `API 객체 생성 (create API object)`

- System : `모든 구성 요소가 병렬로 작동하여 해당 상태로 구동` (all components work in parallel to drive to that state)

  - 상태를 기술한 yaml 이라는 파일을 create 하라고 명령을 하면 (object 를 creation)

  - Pod A 에 대한 것을 master 안의 etcd 에 저장을 하면 노드 B 는 그것을 보고 Pod A 를 만들게 된다.

<img width="742" alt="스크린샷 2023-04-02 오후 8 00 00" src="https://user-images.githubusercontent.com/87372606/229348740-2f2e9621-f411-466d-8d67-c8b798781326.png">

- 만약 노드 B 안의 Pod A 가 고장이나면 시스템은 자동으로 master 의 상태정보를 계속 보다가 만족이 되지 않았기 때문에 노드 A 에 다시 Pod A 를 만들게 된다.

<img width="989" alt="스크린샷 2023-04-02 오후 8 02 42" src="https://user-images.githubusercontent.com/87372606/229348854-531b3bb6-e343-4a62-a116-71eefccd6888.png">

- 마스터가 Pod A 에 대한 상태정보를 써놓고 있다가 노드에게 명령을 내리는 스타일이라면 이건 이전의 사람이 명령을 내리는 방식과 똑같은 것이다.

  - 사람대신 마스터가 모니터링하고 에러를 catch 하기 때문에 복잡해지는 것은 마찬가지

  - 두 번째 `Princlple` : **`The Kubernetes control plane is transparent.`**

---

### Principle 2. There are no hidden internal APIs.

- **`쿠버네티스 control plane (master) 도 역시 명령을 내리지 않는 스타일로 하자. (transparent)`**

  - 투명하게 상태정보만 써놓기만 하고 별도의 내부 hidden API 가 없게 하자.

  - `즉, API server 뒷단에서 다시 Master가 대신 명령하지 않는다.`

- Master : `define desired state of node` (**상태정보만 define**)

- Node : `works independently to drive itself towards that state` (**각 워커 노드가 상태정보를 보고 그 상태가 되게끔 동작한다.**)

<img width="672" alt="스크린샷 2023-04-02 오후 8 12 27" src="https://user-images.githubusercontent.com/87372606/229349222-46c15769-ac1f-4f05-9ce0-5b60a601c9a3.png">

<img width="672" alt="스크린샷 2023-04-02 오후 8 12 55" src="https://user-images.githubusercontent.com/87372606/229349240-11f6bb4b-2c09-41b2-986d-2027ef5c0227.png">

<img width="672" alt="스크린샷 2023-04-02 오후 8 13 05" src="https://user-images.githubusercontent.com/87372606/229349248-3a93f4ee-8e85-4587-91da-bf6f381f1994.png">

<img width="672" alt="스크린샷 2023-04-02 오후 8 13 24" src="https://user-images.githubusercontent.com/87372606/229349261-73341ef9-7e17-4947-a38a-0d5e1014414b.png">

- All components watch the Kubernetes API, and figure out what they need to do.

  - 마스터가 명령을 내리는 것이 아니라 `노드가 watching` 을 하는 것,

    - 마스터안의 스케쥴러도 마찬가지로 마스터 안의 상태정보가 생기면 스케쥴러가 어디 워커 노드가 좋을지를 API Server 에 생긴 object 에 작성한다.

- 장점

  - `Declarative API` provides the same benefits to internal components (인간과 master 에서의 장점이 똑같이 master 와 node 사이에 적용된다.)

  - `Simpler`, **`more robust system`** that can easily recover from failure of components. (문제 발생 시 컴포넌트만 바꾸면 되기 때문에 더 안정적인 시스템이 된다.)

  - **`Makes Kubernetes composable and extensible.`**

    - 새로운 상태정보를 define 할 수 있고, 상태정보를 보고 action 을 취하는 control logic 도 작성가능하기 때문.

---

### Principle 3. Meet the user where they are. (Supporting Legacy Applications)

- Principle 3, 4 는 `Kube API Data` 에 관한 것 이다.

  - Kubernetes API has `lots of data` that is interesting to workloads (쿠버네티스 API 를 통해 상태정보를 define 하는데 data 의 종류가 다양하다.)

- `Secrets` – **Sensitive info** stored in KubeAPI (certification 에 관련된 key, password 와 같은 중요한 정보들)

  - (ex) passwords, certificates, etc.

- `ConfigMap` – **Configuration info** stored in KubeAPI

  - (ex) application startup parameters, etc.

- `DownwardAPI` – **Pod information** in KubeAPI

  - (ex) name/namespace/uid of my current pod.

<img width="397" alt="스크린샷 2023-04-02 오후 8 25 22" src="https://user-images.githubusercontent.com/87372606/229349810-fffa4890-7794-43df-8e16-87b0c2b60530.png">

- 데이터를 API 서버에 두고 기존 응용 프로그램들이 읽어가도록 하면 기존 응용 프로그램들을 변형시켜야했다. (App must be modified to be Kubernetes aware.)

<img width="397" alt="스크린샷 2023-04-02 오후 8 31 34" src="https://user-images.githubusercontent.com/87372606/229350077-1be894b7-7d7e-4322-831e-2ddd02d6980c.png">

- `기존 응용 프로그램들을 수정없이 돌릴 수 있게 하자.` (If app can load config or secret data from file or environment variables it doesn’t need to be modified!)

- 컨테이너가 API 서버에 있는 데이터를 읽어오는 것이 아니라 별도의 Secret volume (storage) 에서 가져오게끔 하게 했다.

  - Pod 이 응용 프로그램 임무를 완수하고 삭제될 수 있기 때문에 `remote storage` 가 등장했다.

<img width="412" alt="스크린샷 2023-04-02 오후 8 41 51" src="https://user-images.githubusercontent.com/87372606/229350590-0c0bdf49-b850-4325-a142-ad2519459e48.png">

<img width="431" alt="스크린샷 2023-04-02 오후 8 41 32" src="https://user-images.githubusercontent.com/87372606/229350581-b666e59f-ddee-4ca2-a9a5-a7ef4067c057.png">

<img width="552" alt="스크린샷 2023-04-02 오후 8 43 00" src="https://user-images.githubusercontent.com/87372606/229350646-7e3e475a-2d3a-4457-858a-cb5260d3264a.png">

<img width="552" alt="스크린샷 2023-04-02 오후 8 43 09" src="https://user-images.githubusercontent.com/87372606/229350652-5bf361ee-15cb-47aa-a128-edd397c604bd.png">

- Pod 를 define 할 때, storage 정보 (remote storage) 를 쓸 수 있게끔 해놨다.

  - 그러면 control 하는 소프트웨어가 동작해서 하드웨어적인 physical storage 를 준비시킨다.

  - 그리고 container for pod 가 생성이 되면 mount 시킨다.

  - 이렇게 해도 문제는 pod 는 일을 끝내면 없어지는데 응용 프로그램이 돌아가면서 생성한 데이터를 찾을 수 없었다.

    - Principle 4 개념 등장

---

### Principle 4. Workload portability

- `PoD 의 이식성을 강화 (스토리지 측면에서도)`

- Decouple storage implementation from storage consumption

---

## Why Workload Portability?

- 쿠버네티스의 기본이 `Cloud Native` (어디서든지 돌아갈 수 있게끔 하자) 하게 container 를 돌리기 위해. 그러기 위해서는 storage 에 관련된 것도 abstraction 해야했다.

- Decouple distributed system application development form cluster implementation

---

## Principles 정리

1. Kube API `declarative` over imperative.

2. `No hidden internal APIs`

3. Meet the user where they are: `Remote storage`

4. `Workload portability` : PV/PVC

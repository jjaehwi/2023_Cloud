# 오픈스택기초 (Open Source Cloud OS)

- [오픈스택이란 무엇인가?](#오픈스택이란-무엇인가-what-is-openstack)

  - [오픈스택의 역사](#openstack-release-plan)

  - [오픈스택의 구성요소](#오픈스택-구성요소-openstack-component)

---

## Intro

- 가상화에 대한 이해를 바탕으로 가상화 기능이 들어있는 `수 많은 서버들을 종합적으로 관리하는 대표적인 클라우드 관리 소프트웨어`인 `오픈스택`에 대해 배운다.

- `퍼블릭 클라우드 (public cloud)` 를 만들어서 외부에게 서비스 하는 회사 (ex. 아마존 AWS, MS AZURE, KT, NHN, 네이버 등)

- 클라우드 OS 중 `오픈소스`인 것.

  - 큰 기업이 아니면 클라우드를 만들어서 제공할 수 없기 때문에 오픈소스로 개발

  - 여러 사람이 같이 개발하는 오픈소스

  - 여러 개의 클라우드 오픈소스들이 존재했는데, 과도기를 지나 표준화된 것이 오픈스택이다.

  - **오픈스택은 Virtual Machine 서비스를 제공하기 위해 시작**되었다. 현재는 컨테이너를 쓰는 기능들도 들어가있다.

  - **쿠버네티스는 처음부터 컨테이너에 대한 클라우드 관리 소프트웨어**였다.

---

## 오픈스택이란 무엇인가? (What is OpenStack?)

<img width="444" alt="스크린샷 2023-03-28 오전 12 45 29" src="https://user-images.githubusercontent.com/87372606/227993303-4328eb44-ea80-42cf-ae8b-ebeea062301a.png">

- `오픈스택` : 오픈스택은 **`cloud operation system (클라우드 관리 시스템)`** 이다. 데이터센터를 통해 compute, storage, networking 과 같은 하드웨어 자원들을 모아놓은 대규모 pool 들을 컨트롤 할 수 있다.

  - 이런 것을 관리하려면 인증하는 과정과 `API` 를 통해서 관리해야한다.

  - 대쉬보드가 존재해서 GUI 를 통해 control 이 가능하다. (through web interface)

- `오픈스택`은 Bare Metal (일반 서버), Virtual Machines (서버 위에 가상화 기능을 넣어 만든 machine), 컨테이너 로 이루어진 **클라우드 환경**을, 연결해주는 `네트워크 기능과 저장소 (storage) 를 같이 공유`하는 `인프라 소프트웨어` 이다.

- 그 인프라 위에 연결하는 포인트들을 보면 많은 툴 기능들이 있어서 섞어서 사용 가능하다. (ex. 쿠버네티스, CloudFoundary(Paas), Terraform...)

  - 이런 기능들은 제 3의 서비스를 위한 소프트웨어이다.

- 본인들의 인프라를 위한 툴로는 OpenStack SDK, Horizon Web UI 등 과같은 툴들이 있다.

- 오픈스택이라는 것으로 Bare Metal, Virtual Machines, 컨테이너 라는 것 이외의 네트워크, 저장소 자원까지 합쳐서 `공유 / 관리 할 수 있게 해주는 플랫폼 소프트웨어`인데, 제 3의 다른 서비스 소프트웨어들과 본인들이 만든 다양한 툴과도 섞어서 사용 가능하다.

  - Bare Metal, Virtual Machines, 컨테이너와 같은 infrastructure 들을 제공해주는 것을 `infrastructure - as - a - service functionality` 라 한다.

  - infrastructure 를 제공하는 것 외의 `보조 기능`들을 제공하는데 fault management 나 service management 기능이 있다.

---

### OpenStack Release Plan

<img width="849" alt="스크린샷 2023-03-28 오전 1 00 39" src="https://user-images.githubusercontent.com/87372606/227997460-0e7830a9-3c38-45c8-a4f9-b4c0d0b5957c.png">

- 현재는 zed 까지 왔고 2015 ~ 2016 년 엄청 과도기였다. (2023-03-22 Antelope, 2023-10-04 Bobcat ...)

- 계속해서 업그레이드와 기능 추가가 되고 있다는 것.

---

### 오픈스택의 구성요소 (OpenStack component)

<img width="421" alt="스크린샷 2023-03-28 오전 1 03 47" src="https://user-images.githubusercontent.com/87372606/227998206-a69ce343-1362-4cf1-8cd8-35dc26b0aadf.png">

- `Compute Node` 안에 **Hypervisor 에게 요청하여 Virtual Machine 들이 만들어진다.**

- 이것들을 컨트롤하고 요청하기 위한 `Controller Node` 가 존재한다.

  - 수 많은 VM 이 돌아가는 환경을 **control 해주는 SW** 들 이 `Controller Node` 에 있고 그 짝으로 `Compute Node` 에 **control 명령을 듣고 수행할 수 있는 Hypervisor 와 다른 SW** 들이 있을 것이다.

  - 오픈스택을 공부하는 것은 Hypervisor 에게 어떤 식으로 명령어를 작성하여 요청하고 어떤 SW 가 그 명령어를 수행하는지에 대해 배우는 것이다.

  - Hypervisor 에서 요청들에 대해 관리할 수도 있지만 처음부터 Controller Node 에서 요청하는 것을 모아서 적정하게 처리할 수 있다. (Controller Node 에서 각 Compute Node 에 대한 상태정보를 갖고 있어서 그것을 보고 명령을 처리한다.)

    - 그렇기 때문에 Controller Node 에 현재 **Compute Node 의 상태들을 관리하기 위한 저장소, 보관소 등이 필요**하다. **요청들을 순서대로 처리하기 위한 queue, buffer 가 필요**할 것이다. 이런 생각들을 오픈스택 만드는 사람들이 모여서 정리하고 구현한 것이 오픈스택이다.

<img width="421" alt="스크린샷 2023-03-28 오전 1 12 09" src="https://user-images.githubusercontent.com/87372606/228000373-44e0600e-e54e-42cb-a605-903e7b056950.png">

- `Virtual Machine == Instance`

  - 하드웨어를 emulation 해서 만들어진 **Virtual HW 위에 Machine 이 되기 위해 OS (Guest OS)** 가 필요하다.

  - **CD - ROM 처럼 구워져있는 소프트웨어**들을 `image` 라고 한다. Guest OS 의 image 를 갖고 있어야한다. **보조기억장치 같은 HW** 를 `storage` 라고 한다.

- 버전이 upgrade 되면서 `nova`, `glance`, `swift` 라는 이름이 새로 정의되었다. (각 프로젝트 이름을 붙힌 것)

- E 버전 정도 가니까 인증이 된 사람만 사용할 수 있게 그 **인증**에 관한 것을 관리해주기 위한 프로젝트로 `keystone` 이 정의 되었다.

  - 또한 API 로 정의된 명령어로 모든 작업을 수행하는데 간단한 **GUI** 를 만들어 준 것이 `horizon` 이다. (대쉬보드 역할)

- 더 upgrade 되면서 `quantum (neutron)` 이라는 **네트워크에 관련**된 프로젝트가 등장했다.

  - swift 라는 storage 말고도 `cinder` 라는 **storage** 가 등장했다.

- 이런식으로 계속 고도화 되어 나간다. 오픈스택 전문가가 되려면 이거보다 더 넓은 관계도와 프로젝트 사이 주고받는 명령어에 대한 이해를 해야한다.

<img width="913" alt="스크린샷 2023-03-28 오전 1 34 27" src="https://user-images.githubusercontent.com/87372606/228005993-6e7aa575-6490-4257-978a-459b5130ffd5.png">

- `controller node` 에 존재하는 것

  - nova (Compute) 안에는 Queue 가 있다. nova 에게 요청을 하려면 api 를 call 하면 된다. 그렇게 명령을 받으면 queue 로 들어간다.

  - 그리고 누구한테 수행시킬건지 nova - scheduler 가 scheduling 한다.

  - 그리고 상태정보를 관리해야하기 때문에 Nova database 가 존재한다.

  - nova - console 은 직접 명령을 내릴 수 있는 서브적인 것이다. 결정하기 위한 nova - conductor, 인증과 관련된 nova - consoleauth, nova - cert 가 존재한다.

- 결과적으로 실제 일을 해주는 `compute node` 에 존재하는 nova - compute 에 의해 compute node 안의 Hypervisor 를 통해 Instance 가 생성된다.

<img width="877" alt="스크린샷 2023-03-28 오전 1 34 09" src="https://user-images.githubusercontent.com/87372606/228005876-8dfff7a3-d57f-4c06-a6e2-0f2a0daddc66.png">

- CORE 쪽은 완성이 되어서 안정화되어있고 새롭게 기여하기 힘들지만, 외부의 보조적인 기능들에 대해서는 비교적 접근 가능하다.

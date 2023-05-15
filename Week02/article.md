# 오픈스택기초 (Open Source Cloud OS)

- [오픈스택이란 무엇인가?](#오픈스택이란-무엇인가-what-is-openstack)

  - [오픈스택의 역사](#openstack-release-plan)

- [오픈스택의 구성요소](#오픈스택-구성요소-openstack-component)

  - [Nova](#nova---compute-프로젝트)

  - [Swift, Cinder](#storage-swift-cinder-프로젝트)

  - [Glance](#glance-프로젝트)

  - [Keystone](#keystone-프로젝트)

  - [Horizon](#horizon-프로젝트)

  - [Neutron](#neutron-프로젝트)

  - [오픈스택 네트워크 구조](#오픈스택-네트워크-구조)

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

## 오픈스택의 구성요소 (OpenStack component)

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

---

### Nova - Compute 프로젝트

<img width="673" alt="스크린샷 2023-03-28 오후 5 59 29" src="https://user-images.githubusercontent.com/87372606/228185066-056f9b13-2da6-4ba4-9e91-304f4a1c866c.png">

- **`Nova` 는 `Hypervisor 를 관리하는 API 를 제공`**한다.

- `Instance (VM) 생성 관리`

- **Virtual Machine 이 만들어지는 compute node 를 어떻게 control** 할 것인가에 대해..

- **제어 기능은 Control Node 에 위치**하여 **compute node 관리**

  - API, DB, Conductor, Scheduler 부분이 Control Node 부분

- Guest OS 의 SW 는 CD - ROM 같은 곳에 존재할 것이고, 이것을 HW 에 가져와서 돌려야하기 때문에 별도로 존재하고 Glance 나 Cinder 가 존재해서 불러서 사용하는 것이다. (RESTFUL API 인 HTTP 방식으로 call 한다.)

  - API 에서 unbuntu 와 같은 OS 를 깔아서 VM 을 만들어달라고 요청을 받으면 HTTP 통신으로 Glance 나 Cinder 에 요청해서 Compute 에 보내게 된다. Compute 에서 image 를 받아 돌아간다.

- VM 을 만들어달라고 한 사람이 인증된 사람인지 ID, PW 를 만들고, 확인하는 역할을 하는 별도의 서버가 Keystone 이고 API 에서 HTTP 통신으로 부른다.

- VM 이 혼자 돌아가는 것이 아니기 때문에 외부와의 연결을 위한 Network 기능에 대한 부분을 가지고있다.

  - Neutron 프로젝트가 생성되고 나서는 직접 메세지를 보내는 것이 아닌 HTTP 방식으로 통신한다.

---

### Storage (Swift, cinder) 프로젝트

<img width="785" alt="스크린샷 2023-03-28 오후 6 09 52" src="https://user-images.githubusercontent.com/87372606/228187867-f64f6f1d-8826-40df-bcbf-d1aac837291f.png">

- `클라우드 스토리지 서비스 - 스토리지 및 스토리지 API 를 제공`

- 블록 스토리지 (하드디스크) : `Cinder` - `정형성` 저장 적합 (고정필드, 엑셀 등)

  - 문제시 **포맷할 수 없으므로 새로 생성**

- 오브젝트 스토리지 : `Swift` - `비정형성` 저장 적합 (이미지 등)

  - **사용자 계정별 저장공간 제공 가능**

- 공유파일 스토리지 : `Manila`

  - NFS & CIFS 공유 파일 서비스

- `Ceph` 의 RDB, RADIOS : 분산 스토리지를 위한 다른 오픈 소스 서비스로 모든 종류의 스토리지 관련 서비스제공 프로젝트

  - 오픈스택의 프로젝트가 아니고 Storage 관련된 것만 전문적으로 처리하기 위해 만들어진 프로젝트

---

### Glance 프로젝트

<img width="574" alt="스크린샷 2023-03-28 오후 6 20 20" src="https://user-images.githubusercontent.com/87372606/228190640-b033b837-c5cd-4d0e-8b77-4bda156ac908.png">

- CD - ROM 같은 기능, `오픈스택에서 운영체제 이미지를 관리`

- 다양한 Hypervisor에서 사용할 수 있는 `VM 이미지를 관리`하고, `VM 에 설치된 OS 보관 및 관리`

- glance - api로 이미지를 등록, 삭제 및 관리

- glance - api를 사용하여, glance - registry, glacne database 에 이미지를 관리

- 이미지 등록시, glance - registry 를 통하여 glance - database 에 등록

- 등록된 이미지 사용시, glance - database 에 바로 사용 요청

---

### Keystone 프로젝트

<img width="685" alt="스크린샷 2023-03-28 오후 6 21 09" src="https://user-images.githubusercontent.com/87372606/228190855-56d4be0d-f8ef-47f8-b2b1-9c24b14bd05d.png">

- 물리 서버 내 Compute, 이미지, 네트워크, 스토리지와 같은 자원들에 대한 `인증관리`

- 사용자 인증을 통하여 물리 서버 내의 자원을 사용할 수 있도록 관리

- `Provides Auth for` : 인증을 해주는 것, 모든 프로젝트에서 Keystone 프로젝트를 통해 사용자가 맞는지 확인하는 과정을 거친다.

---

### Horizon 프로젝트

<img width="633" alt="스크린샷 2023-03-28 오후 6 23 21" src="https://user-images.githubusercontent.com/87372606/228191413-0f216950-776e-4fe6-a15f-31102f9677f1.png">

- 오픈스택 `대시보드 서비스`

- Horizon 은 사용자가 `웹 UI 를 통하여` **인스턴스 생성, 삭제 및 관리등을 쉽고 빠르게 처리**할 수 있도록 해주는 `웹 서비스`

- Horizon 은 **아파치 웹 서버**를 사용하며, 대시보드는 **파이썬 장고 프레임워크**로 구현

- 클라우드를 관리해주는 오픈스택의 수 많은 기능들을 쓰고 싶으면 직접 하나하나 접속할 수 있는 URI 를 알아내고 명령어를 치면 되는데, 명령어로 바꿔주는 웹 서버가 있어서 그 서버에 접속해서 보여주는 화면을 클릭하면 그것을 커맨드로 바꿔준다.

  - 웹 서버를 만들어서 접속할 수 있도록 한 것

  - RESTFUL API 에 정의된 포맷대로 명령을 문자로 보낸다. (http 의 url, parameters) 이런 터미널로 하는 작업을 GUI 로 할 수 있게 해주는 것

---

### Neutron 프로젝트

<img width="456" alt="스크린샷 2023-03-28 오후 6 32 50" src="https://user-images.githubusercontent.com/87372606/228193929-455f730a-b7fd-4705-a129-6f1169b8fc12.png">

- `네트워크 서비스` 인 Neutron 은 Folsom 버전에서 Quantum 이라는 이름으로 오픈되었다.

  - 그 후에 Grizzly 버전에서 Havana 버전으로 오면서 Neutron 으로 변경되었다.

- **기존 오픈스택의 네트워크 서비스는 `Nova-network 가 담당`을 하였으나, SDN 개념이 들어오면서 `별도의 네트워크 프로젝트`로 분리되었다.**

- Neutron 은 Neutron 서버, 에이전트, 플러그인, 메시지 큐, 네트워크 프로바이더, 데이터베이스로 구성

- Neutron 은 **다양한 네트워크 플러그인과 네트워크 모델을 지원**

---

### 오픈스택 네트워크 구조

<img width="767" alt="스크린샷 2023-03-28 오후 6 35 27" src="https://user-images.githubusercontent.com/87372606/228194620-7c0d6758-a4fd-4053-b3e0-9883bce63aaf.png">

- Controller Node 안에 Controll 기능과 외부랑 접속할 수 있는 기능을 같이 넣은 것이고, VM 이 만들어 지는 부분과 Storage 가 있는 부분이 따로 있는 그림

- 컨트롤 기능들은 별도의 서버에서 컨트롤 기능들을 동작시킨다.

- Controller Node 에서 Control 에 관련된 기능들은 Controller Stack 에 모아두고 Network 관련 기능들은 Networking Stack 에 모아둔다.

  - Network 컨트롤 기능은 Newturon Server 에서 하고 그 control 에 의해 실제로 network 에 가담되는 애들에 대한 기능들을 모아둔 것.

  - 그래서 외부로 나갈 수 있는 것. (`External Network Access`)

  - 이 그림에선 Controller Node 안에 정의되어 있지만 이 Networking Stack 은 또 다른 서버에 존재해도 된다.

- `Internal Admin / API Subnet` : **내부적 관리**를 위한 접속망

  - Nova Compute 가 Hypervisor 에게 VM 에 대한 정보를 요청하는데, 이는 일반적인 데이터가 아니고 특별한 control 데이터

- `Tenant Data Network` : 각 개인이나 집단이 VM 을 여러개 갖고 있을 때, VM 이 여기저기 떨어져 있을 때, 소유한 VM 끼리 통신하기 위한 네트워크

  - Tenant : 각 VM 머신 사용자에 대한 논리적인 / 분리되어있는 집단

- `Storagee Subnet(s)` : **스토리지에 대한 Access** 를 위한 접속망

- `Corporate Network` : 클라우드 전체가 외부랑 통신하기 위한 네트워크

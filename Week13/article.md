# Application 업데이트 및 마이크로 서비스 구조

- [Update 종류](#update-종류)

  - [Blue Green Update](#blue-green-update)

  - [Canary Update](#canary-update)

  - [Rolling Update](#rolling-update)

- [MSA (Micro Service Architecture)](#msa-micro-service-architecture)

  - [기존 Web App 구조](#기존-web-app-구조)

  - [마이크로 서비스](#마이크로-서비스)

---

## Update 종류

- `blue green update`

- `Canary Update`

- `Rolling Update`

---

### Blue Green Update

<img width="550" alt="스크린샷 2023-06-11 오후 8 27 35" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/1eb1fff4-d4f6-49e0-b216-4bc3e452a411">

- 구버전 (Blue) 과 신버전 (Green) 을 **나란히 배포하고 신버전으로 일제히 전환**하는 업데이트 방법

- blue 로 서비스되던걸 green 으로 바꾸려면 : Service 를 바꾸면 됨

- green 은 미리 deploy 시켜놓고, 이쪽으로 한번에 가려면 (가는지 안가는지 결정하는애가 서비스 였기 때문에..) **서비스 안의 selector 를 blue 에서 green 으로 바꾸고 apply 한다.**

- 분배 정책을 세팅할 수 있는 **_Load Balancer_**

---

### Canary Update

<img width="550" alt="스크린샷 2023-06-11 오후 8 27 57" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/d55e307a-f6fa-4d54-b62c-4e2b8d228036">

- blue green 은 한꺼번에 바꾸는 방식이었다..

- **신버전에 일부를 보내고 정상적이라면 전체를 업데이트하는 방식**

---

### Rolling Update

<img width="550" alt="스크린샷 2023-06-11 오후 8 28 18" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/8cfd4370-ff3b-4b41-a544-837576dde48e">

<img width="550" alt="스크린샷 2023-06-11 오후 8 28 27" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/8a2aa2cb-dd3e-4806-81f6-b37a8606e783">

<img width="550" alt="스크린샷 2023-06-11 오후 8 28 35" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/8d895edc-2973-44b6-9aa2-229138689f2c">

<img width="550" alt="스크린샷 2023-06-11 오후 8 28 41" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/0124618d-2d17-47a4-b98a-4bb01138c384">

- 가장 typical 한 방식, `무중단으로 업데이트`

- **blue green, canary 는 load balancer 를 통해 가능**했지만, **rolling update 는 deployment 를 통해 바로 가능함… 내재되어있기 때문 (update 의 장점도 가지고 있는 것)**

---

## MSA (Micro Service Architecture)

<img width="712" alt="스크린샷 2023-06-11 오후 8 30 57" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/f6ff60a1-d9eb-434e-a083-90d885b05bbe">

- `WAS` : 웹 어플리케이션 서버 (interact 가 있는)

  - 웹 어플리케이션 서버에서 MSA 적용 필요성 (ex. tomcat, jboss, websphere, jeus)

---

### 기존 Web App 구조

<img width="537" alt="스크린샷 2023-06-11 오후 8 32 03" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/54a633b3-3aff-4497-92e1-4467b31dbed8">

- 일반적인 구조 : APP 을 구동했을 때 상호작용하는 UI 파트, 그에 맞는 service, 내부적인 잠깐의 저장을 위한 레포지토리, 메인 정보는 뒷단에 db 에 저장되어있음.

<img width="537" alt="스크린샷 2023-06-11 오후 8 32 54" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/4acc1720-8fa7-4d5c-9773-0a2f6b964d9d">

- `scaling 이 필요`해지고, **둘 사이를 왓다갔다하기 위해 `로드 밸런싱` 필요**, 시스템의 안정성 확보

<img width="537" alt="스크린샷 2023-06-11 오후 8 33 04" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/9ebc3fd5-4e50-4b1c-ad76-73a2e76b7d96">

- 새로운 서비스를 추가하기 위해, `B 서비스 추가` -> **UI 도 수정되어야하고 레포지토리도 새로 만들어져야하고, db 도 새로 추가되어야한다.**

<img width="537" alt="스크린샷 2023-06-11 오후 8 34 06" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/9cc30a55-73f4-494e-a223-2fc3af1dbf07">

<img width="537" alt="스크린샷 2023-06-11 오후 8 34 16" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/629bfec3-c5e1-4d9d-b397-f8cb1a569f98">

- 외부와 시스템 연계 / 통합을 위해 (db 까지 연동)

- **외부와의 시스템이 너무 많아지면 문제가 생긴다.**

- `문제점`

  - 코드가 너무 커져서 유지보수의 어려움

  - 시스템을 분리를 희망

  - DB도 분리 희망

  - 연계시스템이 변경 시 전체 영향

  - 연계 시스템이 장애가 발생시 전체 서비스도 장애

  - 사소한 수정인데도 전체 배포를 하고, QA를 다시해야함

  - 새로운 걸 추가하기위해 기존 로직/데이터를 변경하면 무슨 문제가 생길지 모르는 두려움

  - 새로운 버전/기술을 적용하기 어려움

- `마이크로 서비스가 등장`했다.

---

### 마이크로 서비스

- `마이크로 서비스`란?

  - **작고** (small)

  - **API 로 다른 서비스와 연계**하며 (communicate with APIs)

  - **자율적**이며 (autonomous)

  - **한 가지 일을 잘하는데 초점을 맞춘 서비스** (focused on doing one thing well)

- `장점`

  - **Technology Heterogeneity** : 기술 다양성 (기존 코드가 뭐던 상관없음)

  - **Resilience** : 안정성, 회복력

  - **Scaling** : 스케일링이 쉬움

  - **deployment 가 쉬움**

  - **Organizational Alignment** : 조직 구성이 쉬움

  - **composition** 이 쉬움 (옛날 거 갖다 쓸 수 있음)

  - **replacing 이 쉬움** (딴걸로 갈아끼우면 됨 API 연동만 잘 되면..)

---

### 확장

<img width="676" alt="스크린샷 2023-06-11 오후 8 40 04" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/e57e2a29-9f43-4f0d-ba9c-a4c1064d3f12">

- x : horizontal duplication (개수 늘리는거)

- y : functional decomposition (새로운 기능 추가)

- z : data partitioning (db partitioning 하는 것, data 를 쪼개서 늘리는 것)

<img width="527" alt="스크린샷 2023-06-11 오후 8 41 53" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/bd66d2a0-ad98-426a-ba0b-1b01eb1385b6">

<img width="494" alt="스크린샷 2023-06-11 오후 8 42 06" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/3e7185d0-61a2-43c5-9327-b1bf40c392e0">

- 기능을 늘리면 각자 새로운 서비스 컴포넌트를 늘리면되고

- db 를 늘리면 db 를 추가하면되고

- 스케일링 하려면 개수를 늘리면 된다.

- `단점`

  - 많이 늘어나게되면 운영하는게 복잡해짐 (`complexity`) : 더 늘어나게 되면 `API GW(gateway) 의 개념 도입`

---

<img width="545" alt="스크린샷 2023-06-11 오후 8 43 28" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/94239a7a-9bc3-4201-b2be-d1890c4626fe">

- `공통적으로 필요한 기능을 모음`

  - load balancing + 알파 -> **security, logging , version 등등 공통된 기능을 가지고 있는 것**

<img width="545" alt="스크린샷 2023-06-11 오후 8 45 07" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/cbebfc0a-f525-48e3-b753-80d6c80f6ee9">

- 들어오는 입구를 다르게 해서 스케일링

- 가장 모던한 방식 : 뒷단에서 개발한 것을 별도의 레지스트리에 올려놓으면 자동으로 찾아가게끔 했다.

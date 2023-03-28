# 컨테이너 개요

- [컨테이너 기초 개념](#컨테이너-개요)

- [VM vs Container](#vm-vs-container)

  - [Namespaces](#namespaces)

  - [Cgroups](#cgroups)

- [컨테이너의 Architecture 기본](#컨테이너의-기본적인-architecture)

  - [컨테이너의 장점](#컨테이너-장점)

## 컨테이너 (Container) 기초 개념

<img width="704" alt="스크린샷 2023-03-29 오전 12 50 50" src="https://user-images.githubusercontent.com/87372606/228295020-8b517195-17ed-4cb7-9041-14103679f18b.png">

- `컨테이너`는 **어떤 환경에서나 실행하기 위해 필요한 모든 요소를 포함하는 소프트웨어 패키지**이다.

  - 컨테이너는 이러한 방식으로 운영체제를 가상화하며 프라이빗 데이터 센터에서 퍼블릭 클라우드 또는 개발자의 개인 노트북에 이르기까지 **어디서나 실행된다.** (`Portable`)

  - 프로세스가 필요로 하는 라이브러리, 프로세스를 돌리면서 세팅해야하는 configuration 들을 `표준화된 방법으로 package 화 (standard Packaging)` 해놓은 것

  - OS 위에서 namespace 와 Cgroup 을 통해 일반적인 프로세스 보다 더 `isolation`

  - 클라우드 인프라를 관리하는 일과 거기에 돌아가는 프로세스에 대한 일을 분리해놓았다. (`Separation of Concerns`)

    - 한번에 한 가지만 걱정해도 괜찮도록 한 것

---

### VM vs Container

<img width="549" alt="스크린샷 2023-03-29 오전 1 34 05" src="https://user-images.githubusercontent.com/87372606/228308029-c428526f-bdd6-4f3c-bda1-0acecbc9d8a5.png">

- **_VM_**

  - HOST OS 위에 Guest OS가 동작하는 구조

  - HOST OS 위에서 Hypervisor를 통해 자원을 가상화 하여 VM을 동작

- **_Container_**

  - HOST OS에서 프로세스를 위한 공간을 별도로 분리

  - 기본적인 Binary, Library 만을 guest os 대신 사용

- VM 은 `portability` (옮겨다니는 것) 가 안좋다. 또한 `scalability` 가 안좋다 (확장성) (scale out == 넓히는 것)

- 컨테이너 기반이 VM 기반보다 `portability`, `scalability` 보다 좋다.

  - 추가적으로 `Declarative` 하다…

- **컨테이너 기반이 `portability` 한 이유**

  - 미리 `패키징`했기 때문! 패키징했는데 얘네들은 (리눅스 엔진과 같은 입장에서 보면) 동일한 컨테이너 엔진이 깔려있으면 서버가 달라도 어디서든지 돌아가기 때문이다.

  - common 한데 돌 수 있게끔해주는 binary 파일같은 것들을 압축해놨기 때문에 protable 하게 되는 것.

- 컨테이너는 Guest OS 위에 **리눅스 OS 가 갖고 있는 namespace 와 Cgroup 기능을 통해 각각 패키지화 되어있는 컨테이너들을 돌릴 수 있게 해주는 `Container Runtime`** 이 있다.

  - Container Runtime (컨테이너 엔진) 의 예로는 `Docker 엔진`이 있다.

  - Container Runtime 은 패키지화 되어있는 프로그램을 풀어서 돌아갈 수 있게 해준다.

  - Container Runtime 은 **namespace 와 Cgroup 을 통해 separation / isolation** 을 해준다.

<img width="621" alt="스크린샷 2023-03-29 오전 1 30 33" src="https://user-images.githubusercontent.com/87372606/228307289-56574372-466c-4833-9b22-afb0ab2cf79b.png">

---

### Namespaces

- VM 에서는 각 게스트 머신별로 독립적인 공간을 제공하고 서로가 충돌하지 않도록 하는 기능을 갖고 있다. **리눅스에서는 이와 동일한 역할을 하는 `namespaces 기능을 커널에 내장`하고 있다.**

  - mnt (파일시스템 마운트) : 호스트 파일시스템에 구애받지 않고 독립적으로 파일시스템을 마운트하거나 언마운트 가능

  - pid (프로세스) : 독립적인 프로세스 공간을 할당

  - net (네트워크) : namespace 간 network 충돌 방지, 중복 포트 바인딩 (독립된 network stack 을 갖도록 해준다)

  - ipc (systemV IPC) : 프로세스간의 독립적인 통신통로 할당

  - uts (hostname) : 독립적인 hostname 할당

  - user (UID) : 독립적인 사용자 할당

<img width="830" alt="스크린샷 2023-03-29 오전 1 04 24" src="https://user-images.githubusercontent.com/87372606/228300421-9a2336a4-bfa0-4525-8cc9-0688e0c01f0f.png">

- OS 위에 프로세스가 여러개 돌아가는데, 이 그림에선 프로세스 5,6,7,8 과 프로세스 9 를 각각을 namespace 로 논리적으로 서로 다르게 grouping 했다.

  - 각 그룹 안에서는 유저가 세팅하는 다른 ID 값으로 보이게 한다.

  - 각 분리되어있는 그룹 안에서는 상대를 볼 수 없게 OS 가 막는 역할을 한다.

---

### cgroups

<img width="793" alt="스크린샷 2023-03-29 오전 1 08 37" src="https://user-images.githubusercontent.com/87372606/228301875-a124421b-b28e-4ff9-bfc4-633cb5a11984.png">

<img width="471" alt="스크린샷 2023-03-29 오전 1 26 54" src="https://user-images.githubusercontent.com/87372606/228306406-b8e75983-6587-404b-a112-7570cc8b6841.png">

- 하나의 namespace 안에 있는 프로세스가 이용할 수 있는 CPU 메모리의 용량을 제한한다. 각자 쓸 수 있게 나눠주는 control 을 한다.

---

## 컨테이너의 기본적인 Architecture

<img width="874" alt="스크린샷 2023-03-29 오전 1 10 36" src="https://user-images.githubusercontent.com/87372606/228302402-e2e33d1f-a68c-4586-9e58-7416c3619ffe.png">

- 컨테이너 런타임을 통해 컨테이너화 되어있는 패키지들이 구동되는데, 이 전에 패키지화 하는 과정이 필요하다. 즉 컨테이너를 만들어 내는 동작이 필요한 것.

  - original 프로그램이 어느 저장소에 저장이 되어 있을 것이다. (Container Registry)

  - 프로그램이 돌아가는데 필요한 라이브러리나 세팅들을 같이 묶어서 Images 로 만든다. (build)

- 컨테이너는 두가지 측면이 있는 것이다.

  - `컨테이너 만드는 동작` : 필요한 것을 묶어서 하나로 패키지화 하는 것

  - `컨테이너 구동하는 동작` : 패키징 되어있는 것을 풀어서 실제로 프로세스로 실행되게 run 시키는 것

<img width="655" alt="스크린샷 2023-03-29 오전 1 39 36" src="https://user-images.githubusercontent.com/87372606/228309265-0fbc3892-1104-42b8-b666-7af703dbdfe3.png">

---

### 컨테이너 장점

**_1. 애플리케이션의 빠른 배포_**

- 컨테이너 형식이 표준화됨

- 개발자와 관리자의 업무를 분리 (개발자는 컨테이너 내부만, 관리자는 컨테이너 관리만)

- 개발 테스트 및 수정이 빠름

**_2. 설치 및 확장이 쉽고, `이식성`이 좋음_**

- 다양한 인프라에 설치가능(베어메탈, VM, 타 IaaS)

- 실험환경에서 쉽게 실제 동작하는 환경으로 이동 가능

- 컨테이너 규모 확장 및 축소가 쉽고 빠름

- 컨테이너 실행 / 종료가 빠름

**_3. 오버헤드가 낮음_**

- 하이퍼바이저가 필요 없음

- 서버 자원을 더 효과적으로 사용 가능

- 장비 및 라이센스에 소비되는 비용 감소

**_4. 관리가 쉬움_**

- 수정된 부분만 소규모로 적용

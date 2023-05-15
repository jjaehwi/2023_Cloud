# Week 1 \_ Virtualization (가상화)

## HW virtualization for VM (Virtual Machine)

- 오픈소스로써 많이 사용되는 가상화 소프트웨어인 `KVM`, `QEMU`, `Libvirt` 에 대해 이해하기

### Virtualization

- 컴퓨터에서 CPU, Memory, I/O 등의 물리적인 리소스를 `추상화 (Abstraction)` 하는, 즉 **물리적인 자원을 논리적인 자원으로 보이게 하는 기술**

---

#### Hypervisor

- 호스트 컴퓨터에서 `다수의 운영 체제 (operation system) 를 동시에 실행하기 위해` **하드웨어 자원을 논리적플랫폼 (platfrom) 으로 가상화 해주는 소프트웨어**

- 논리적인 자원으로 **가상화된 환경 위에 여러 OS 를 동시에 운영**할 수 있다.

<img width="413" alt="스크린샷 2023-03-08 오전 12 29 36" src="https://user-images.githubusercontent.com/87372606/223468768-9245d7a2-aa64-447c-a599-5c261afdcd5a.png">

- 하드웨어 하나를 여러개의 하드웨어로 분리해주는 것, 그래서 그 하드웨어 위에 OS 를 깔고 응용 프로그램을 돌리는 것. 각각의 독립된 여러 머신이 있는 것처럼 보임.

**_Hypervisor 분류_**

- **설치 방식에 따른 분류**

**Type 1 (Native or Bare metal)**

- 호스트의 `하드웨어 상에서 바로 동작`, 게스트 OS 를 관리

- CPU 가 가상화를 지원해야한다. (Intel VT, AMD - V 등등)

- XenServer or Vmware ESX, Hyper - V (MS사)

- `KVM (Linux 에 같이 포함됨)`

<img width="172" alt="스크린샷 2023-03-08 오전 12 39 47" src="https://user-images.githubusercontent.com/87372606/223471680-f67dfdea-ff64-4a05-9f30-dfb951b7c63c.png">

**Type 2 (Hosted)**

- `호스트의 OS 상에서 Hypervisor 동작` (바로 H/W 에 올라갈 수 없고 기본이 되는 OS (Host OS) 가 있어야 한다.)

- Hypervisor 는 OS 를 통해 H/W 를 호출

- `VirtualBox (ORACLE 사)`, VMware Desktop

- `QEMU`

<img width="172" alt="스크린샷 2023-03-08 오전 12 40 03" src="https://user-images.githubusercontent.com/87372606/223471787-427f0b8a-e5ca-4d37-a794-d61d3a577255.png">

**<정리>**

- Hypervisor 가 여러 제품들이 있다.

- `OS 를 깔고 Hypervisor 를 돌리는 경우`도 있지만 `하드웨어 위에서 바로 Hypervisor 를 돌리는 경우`도 있다.

- `리눅스에 포함된 KVM` 이 있다.

---

#### Full Virtualization vs Para Virtualization

1. **Full Virtualization (전가상화)**

- Full Virtualization 의 경우 완벽하게 Virtualization 하기 때문에 OS 자체를 별도 변화 (수정) 할 필요 없다. (패키징된 리눅스 제품(우분투, 센토스)을 그냥 쓰면 된다.)

- Guest OS 를 원 OS 의 변경없이 사용 가능한 것

2. **Para Virtualization (반가상화)**

- Para Virtualization 의 경우 full 로 Virtualization 한 것이 아니기 때문에 가짜로 만든 환경에 맞추기 위한 작업이 필요하다. 그래서 Binary Patching 이 필요함. (패키징된 리눅스 제품 외 다른 코드를 같이 써야한다.)

- Guest OS 에서 직접 하드웨어 제어가 안되고 하이퍼바이저를 통해 제어, 그래서 Guest OS 는 원래 OS 로 부터 일부 수정된 것들이 사용됨 (ex. QEMU 와 함께 KVM 을 운영할 수 있는 것)

<img width="317" alt="스크린샷 2023-03-08 오전 1 11 07" src="https://user-images.githubusercontent.com/87372606/223480311-ee718963-9fde-45d5-90dc-8d7c8c415d89.png">

---

#### KVM (Kernel based Virtual Machine)

- `KVM` is a **Linux kernel module.**

- It is a type 1 hypervisor that is a `full virtualization` solution for Linux on x86 hardware containing virtualization extensions (Intel VT or AMD-V).

- These technologies provide the ability for a **slice of the physical CPU to be directly mapped to the vCPU.** (실제 CPU 코어들을 KVM 이 잘라서 vcpu (virtual cpu) 로 만들어 준다.)

- 정리 : KVM 은 리눅스 커널에 같이 배포되는 Hypervisor 인데, 얘는 CPU 를 full virtualization 한다. 그리고 CPU 가 제공하는 VT-x 나 AMD-V 같은 CPU 를 자르는 기능들을 활용해서 virtual cpu 를 만든다.

<img width="182" alt="스크린샷 2023-03-08 오전 12 47 49" src="https://user-images.githubusercontent.com/87372606/223473967-d15e317f-3fd5-46fa-826a-b8d627925cec.png">

- `리눅스 안에 포함`되어 있다.

- **커널과 실제 프로그램이 돌아가는 부분을 `User Space`** 라고 한다. 리눅스 virtual machine 에서 OS (Linux kernel) 가 있고 그 위에 유저 프로그램 (User Space Processes) 이 돌아간다. 돌아갈 때, `QEMU 도 있고 KVM` 도 있다. 둘을 같이 쓴다는 것 이다.

- 원하는 프로그램을 돌리는 것은 CPU 에 명령어를 내리는 것이다. CPU 의 명령어들을 유저가 요청하면, 실제 하드웨어가 수행해야하는데 제일 아래 있기 때문에, **User Space 에서 명령을 내리면 명령이 리눅스 커널로 가고, 커널은 KVM 을 통해 하드웨어로 넘긴다.** 그 과정에서 하드웨어에서 수행되는 명령과 유저가 요청한 명령이 잘 맞지 않으면 중간에서 변화를 시켜줘야 한다. 명령어가 잘 맞지 않을 때 translation 하는 역할을 QEMU 가 했었다. QEMU 는 그 역할 말고도 하드웨어와 똑같은 역할을 하는 소프트웨어를 만드는 (하드웨어를 흉내내는, emulation) 역할을 한다.

- 최근엔 하드웨어 자체에서 유저가 요청한 명령을 직접 수행하도록 하는 기능이 지원됨. (Intel-VT) 그런 하드웨어를 잘 지원할 수 있도록 만든 소프트웨어가 KVM 이다.

**<정리>**

- CPU 관련된 것은 가상화를 KVM 이 하고, 그 밖에 다른 I/O 장치에 관한 것은 QEMU 가 한다. 서로 잘 하는 것에 중점을 맞춰 둘이 같이 쓰는 것.

- 자세한 그림

<img width="341" alt="스크린샷 2023-03-08 오전 12 49 03" src="https://user-images.githubusercontent.com/87372606/223474301-c504d2c8-3642-4c49-9f29-c7cba3f3db98.png">

---

##### QEMU (Quick Emulator) - HW emulator

- QEMU can run on its own and `emulate all of the virtual machines resources`, **as all the emulation is performed in software it is extremely slow.**

- To overcome this, `QEMU allows you to use KVM` as an accelerator so that the physical CPU virtualization extensions can be used. (느리기 때문에 KVM 이 같이 쓰이는 것.)

- disk, network, VGA, PCI, USB, serial/parallel ports, 등등은 명령 처리 보단 덜 급하기 때문에 emulation 방식으로 사용된다.

---

#### Libvirt (virtualization management library)

- Libvirt is quite effective and it can manage a lot of hypervisors altogether.

- 가상화 플랫폼(하이퍼바이저)을 관리하는 오픈소스 라이브러리, `다양한 하이퍼바이저 환경을 통합적으로 관리하기 위해 탄생`

- virual machine 을 만들고 싶으면 KVM 과 같은 하이퍼바이저 에게 요구를 하는데, KVM 이 유일한 하이퍼바이저는 아니기 때문에, 이 요구를 `공통/통일된 인터페이스`로 요구하기 위해 탄생함.

- API 를 제공해준다. 외부에서 call 하면 Libvirt 가 알맞게 맞춰준다.

<img width="272" alt="스크린샷 2023-03-08 오전 1 27 51" src="https://user-images.githubusercontent.com/87372606/223485527-e5ff148c-022a-49b8-99ee-46bb5f1b5a93.png">

---

### VM vs Container

**_VM_**

- HOST OS 위에서 하이퍼바이저 를 통해 자원을 가상화 하여 VM 을 동작

- HOST OS 위에 Guest OS 가 동작하는 구조

**_Container_**

- HOST OS 에서 프로세스를 위한 공간을 별도로 분리

- 기본적인 Binary, Library 만을 Guest OS 대신 사용

- HOST OS 에서 돌아가면 서로 보이기 때문에 (그냥 프로그램 여러개 돌리는 꼴) 그렇게 되지 않기 위한 컨테이너 화 (OS 를 가상화 하는 것) 해주는 기술이 필요하다.

  - (ex) Docker Engine

<img width="272" alt="스크린샷 2023-03-08 오전 1 33 17" src="https://user-images.githubusercontent.com/87372606/223487098-a4391191-64cf-44ef-8077-cb0024bc4354.png">

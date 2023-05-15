# Kubernetes Design Principles: Understand the why Part 3,4

- 5 주차의 Kunernetes Design Principle 3,4 추가 설명 - storage 관점에서

- [Principle #3 : Meet the user where they are](#principle-3--meet-the-user-where-they-are)

- [Principle #4 : Workload portability](#principle-4--workload-portability)

-

## Principle #3 : Meet the user where they are.

- **`Meet the user where they are. (i.e. Supporting Legacy Applications)`**

  - 기존 응용 프로그램들을 수정없이 돌릴 수 있도록 하자

  - legacy application 들이 돌아갈 수 있어야한다.

- 쿠버네티스에서 돌리려고하는데 hurdles 이 너무 크면 안된다. (`minimize hurdles`)

- adoption 이 조금씩 늘어나야한다. (`increase adoption`)

<img width="886" alt="스크린샷 2023-05-15 오후 2 11 42" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/cddee983-61a2-423f-bce0-0006f9fee580">

- 전에는 App must be modified to be 쿠버네티스 aware 였는데 ... 이는 불편하니까 수정없이 사용할 수 있게 하기 위해 `remote storage` 를 사용하기로함.

<img width="916" alt="스크린샷 2023-05-15 오후 2 14 49" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/e551a24c-644d-47e0-bc83-13c16178277c">

- **remote storage 를 Pod 의 yaml 파일에 직접 선언하게 하면 매번 고쳐야했다.**

  - 그래서 Pod definition 자체가 `no longer portable` 했다.

---

## Principle #4 : Workload portability

- **`Pod 의 이식성을 강화` (스토리지 측면에서도)**

- **Volume 을 활용했기 때문에 여기서 돌아가던게 저기서도 돌아갈 수 있다**. (pv, pvc 라는 개념)

  - `쓰는 사람 입장에서는 pv, pvc 만 define` 되어있다.

  - **pv 만 사용할 때마다 환경에 맞춰서 만들어지기만 하면 portability 해지는 것.**

  - **Decouple storage `implementation` from storage `consumption`**

    - Decouple distributed system application development form cluster implementation

    - **_Make Kubernetes a true abstraction layer, like an OS._**

---

## 결론

- Pod 에서 pvc 를 쓰면 pvc 에서는 storage: 에 pv 이름과 storageClass 을 적는다.

<img width="810" alt="스크린샷 2023-05-15 오후 2 27 00" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/f3106d00-9c8a-4302-af11-0e85f40b7161">

- Volume 을 만지는 사람이 pv 와 storageClass 를 실제로 만들고 (여기엔 실제 저장소가 있는 위치가 구체적으로 define 되어있다.)

- 만들어지면 container 가 Pod 와 pvc 는 바꾸지 않고, StorageClass 와 pv 만 바꿔서 storage 를 변동할 수 있다. (실제 remote storage 를 관리하는 사람이 하는 것)

- pv 는 실제 준비 되어있는 것이고, 사람은 class 로만 얘기할 수 있는 것.

- **사용자 입장에서 보면 뒷 단에서 이뤄지는 역할을 신경쓰지 않고 storage 를 가져다가 쓸 수 있게 된 것..**

  - workerNode 가 portable 해졌다. `pvc 를 통해 뒷 단 과 decouple 된 것.`

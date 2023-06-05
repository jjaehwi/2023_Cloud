# Kubernetes Stateful Set

- [StatefulSet](#statefulset)

  - [Deployment 와 StatefulSet 비교](#deployment-와-statefulset-비교)

  - [Using StatefulSet](#using-statefulset)

  - [Scale Up/Down of stateful set](#scale-updown-of-stateful-set)

- [Headless Service](#headless-service)

  - [Headless Service for StatefulSet](#headless-service-for-statefulset)

- [Limitations (한계점)](#limitations-한계점)

---

## StatefulSet

- **_pod 가 생겨나고 사라질 때, 고유한 id 가 붙고 이전 상태를 기억하지 않는다._**

  - `stateless` 한 것 (죽었다 살아나도 과거와 상관없이 독립되게 돌아가는 것)

  - `statefulSets` 는 **_state 가 유지되어야하는 것_** (**과거에 있었던 상태를 이어받아 돌아가야하는 것**)

  - state 를 이어나가는 요구사항을 반영한 새로운 deployment 를 만든 것

  - `stable`, `persistent storage` : **Pod 가 죽었다가 살아나도 이전에 쓰던 storage 를 계속 쓸 수 있도록 하자.**

---

### Deployment 와 StatefulSet 비교

<img width="898" alt="스크린샷 2023-06-05 오후 8 54 30" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/57b8658f-714d-4cd4-8203-db49d9964401">

- Volume 을 쓰는 입장에서 PVC 를 얘기하면 실제 PV 와 연결된다.

  - **Deployment** 의 Pod template 에 의해 Pod 가 3 개 만들어지고 (Volume spec 에 대해 정의되어 있다면) storage 가 연결됨.

- `statefulSets` 는 **spec 에 Pod 에 대한 spec (Pod template) 이 있고, Volume 에 대한 정보가 있다. (PVC template)**.

  - 그러면 결과는 최오른쪽 사진이 된다. (**Pod 끼리도 0, 1, 2 로 나뉨**)

  - `Deployment` 에서는 **identifier 가 순서가 정해지지 않고 그 때 그 때 바뀌었지만**, `statefulSet` 에서는 **순서있게 이름이 붙는 것**이다.

  - 거기에 **같이 붙은 Volume 도 자기 고유의 Volume 이 붙게된다.** (**_각 Pod 마다 PVC 를 통해 PV 가 만들어지고 붙는 것_**)

- `차이점` : **Pod 에 순서있게 고유번호가 붙고, Volume 자체도 각 고유의 Pod 에 서로 다른 PVC 가 붙고 결과적으로 storage 가 붙는다.**

  - 문제가 생겨서 첫번째 Pod 가 죽었을 떄, identifier 는 (quiz-o) 유지될까 -> 유지된다.

  - 그리고 이전에 붙었던 Volume 에 그대로 붙게된다.

- `Guarantees about the ordering and uniqueness of these Pods.`

  - **_Pod 가 생겨날 때, 순서와 uniqueness 가 보장됨_**

- These pods are created from the same spec, but `are not interchangeable` : each has a `persistent identifier` that it maintains across any rescheduling.

  - Pod 의 스펙은 template 에 써있는대로 같은 스펙으로 여러 개 만들고 고유번호만 달라진다.

  - **_동일한 스펙으로 만들어졌지만 각자 고유한 번호를 갖고있는 것_**

  - `persistent identifier` : **죽었다 살아나도 (rescheduling 이 되더라도) 유지되는 고유번호**

- Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. (동일한 스펙으로 만드는 것은 Deployment 와 동일)

- Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. (**Pod 별로 유지되는 고유한 identify 를 가지는 것은 차이점**)

- storage 가 붙은 다음에 떼지지 않고 계속 유지되어야하는 (workload 에 persistent 하게 제공하고 싶은) 프로그램에서 사용됨. (If you want to use storage volumes to provide persistence for your workload, you can user a statefulSet as part of the solution.)

---

### Using StatefulSet

<img width="923" alt="스크린샷 2023-06-05 오후 9 15 44" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/1ec6f032-8c16-453f-8702-6fd39223ca78">

- kind 제외 Deployment 와 같다.

1. **stable** 하고, **unique** 한 network identifier

2. **stable** 하고, **persistent** 한 storage

3. **순서를 갖고 (Ordered)** **graceful deployment** and **scaling**

4. 프로그램이 **순서를 갖고 (Ordered)** **rolling 한 (순차적인) 자동 업데이트**가 되길 원하는 경우

- ex. db 엔진

---

### Scale Up/Down of stateful set

<img width="840" alt="스크린샷 2023-06-05 오후 9 23 24" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/3bea6a60-146f-4e9f-82b4-d48689985684">

- statefulSet 에서 replicas 3 으로 쓰여있던걸 1 로 바꾸면 어떤일이 발생할까

- Pod 2 개가 사라진다. 하지만 **마지막에 만들어진거부터 순서대로 사라진다.**

  - 때문에 가장 뒤에 있는 것이 마지막에 만들어졌기 때문에 가장 뒤에거부터 2개가 사라진다.

  - storage 는 남아있다. (`persistent 한 Volume` 이기 때문)

- 그렇기 때문에 **_다시 scale up 을 해서 replicas 를 3 으로 하면 다시 1, 2 가 만들어져서 storage 에 붙는다._** (`사라졌다 다시 생겨도 다시 전 상태처럼 잘 작동`하는 것)

- `Deployment` 는 **Pod 가 사라졌다 생기면 이전과 다른 identifier 이기 때문에 stateless 하지만 `StatefulSet` 은 아니다**

---

## Headless Service

<img width="140" alt="스크린샷 2023-06-05 오후 9 28 35" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/ab189839-a106-47b0-aa5d-9ed78636c214">

- `clusterIP: None` 이란 무엇인가?

  - `Headless Service` : **clusterIP 주소가 없는 Service**

  - **_Pod 끼리 묶어서 변하지 않는 IP 를 줘야하는데 묶어놓기만 하고 IP 를 주지않은 것!_**

    - 찾아가려면 `이름으로 찾아갈 수 있다.` (**IP 는 붙히지말고 이름을 붙혀달라는 뜻**)

    - `DNS 서버에 이름을 통해 물어보면 IP 를 알 수 있고 이제 찾아갈 수 있게 되는 것`이다. (이게 `DNS 동작`)

    - **자동으로 이름 (DNS name) 은 부여**되기 때문

<img width="799" alt="스크린샷 2023-06-05 오후 9 31 11" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/dff79ea1-18fb-4607-a81a-8a8ac35211ab">

- **Service 에 이름이 붙어 있는 것**을 확인할 수 있다.

- Deployment 가 만들어질 때 3 개의 Pod 가 만들어지고 하나의 이름이 만들어진다.

- 반면 `StatefulSet` 은 **각자 이름이 붙는다.** 이 이름은 Service 를 만들어야 붙는데 (ClusterIP 는 Pod 만 만들었을 때 생기는 것이 아니라 Service 가 만들어질때 생성되는 것)

  - clusterIP 가 생길 때 이름도 생기게된다.

  - **Service 를 만들 때, statefulSet 으로 만들어져있으면 이름이 만들어지는데** **이 이름은 각자 따로 붙게 되고**, **이 Pod 들을 묶으면 Pod 들을 묶은 Service 에 대한 이름도 생성**된다.

- `정리` : **_StatefulSet 이므로 각각 마다 이름이 존재하고, 또한 묶어서 접근할 수 있는 이름도 존재한다._**

---

### Headless Service for StatefulSet

<img width="872" alt="스크린샷 2023-06-05 오후 9 37 18" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/8966d24b-3fa0-4e85-865a-1bc020a5cfb2">

- `Headless Service` 이기 때문에 **_각각에 대해 이름을 통해 찾아가는 방식으로 access 해야한다._**(**clusterIP 주소로 찾아가는 것이 아닌** **이름으로부터 direct 로 Pod 로 찾아가는 방식**인 것)

- `StatefulSet` 에서는 **_headless 방식으로 service 로 만들어서 개별적인 Pod 로 접근할 때 `dns entry 를 통해 access`_** 하면 된다.

  - **headless Service 를 만들면 각각에 대해 dns entry 가 만들어지기 때문**

- `정리` : statefulSet 에서 만들어져있는 각각 Pod 들을 외부에서 접근하려면 clusterIP 같은 것을 할당해야하는데 이 방법은 Pod 들을 다 묶는다. 하지만 statefulSet 은 이런 스타일이 아니라 각 Pod 의 고유 identifier 가 존재하기 때문에 묶는 것이 아니라 각각에 대해 이름을 붙여서 접근하고 statefulSet 에 Headless Service 를 만들면 각각에 대해 dns entry 가 만들어져서 각 Pod 에 접근할 수 있고, 묶음에 대한 이름도 만들어져서 묶음에 대해 접근도 가능하다.

  - **개별접근도 가능하고 Pod 들의 묶음에서 아무거나에나 접근도 가능**한 것.

  - **주소가 안붙는 대신 각각에 이름이 붙는 것.**

  - 형식 : 고유한 identifier + 자동으로 붙게되는 이름

  - 이런 `이름과 Pod 에 대한 IP 에 대한 mapping 이 자동으로 dns 서버에 저장`이 된다. (그런 기능이 Cluster 안에 설치되어있어야한다. -> 별도의 **coreDNS 라는 프로젝트**가 있다. -> 이름을 갖고 접근하는 스타일의 프로젝트)

---

## Limitations (한계점)

1. The storage for a given Pod must either be provisioned by a PersistentVolume Provisioner based on the requested storage class, or `pre-provisioned by an admin` (**_관리자가 미리 준비해둬야함_**)

   - 스토리지를 따로 준비해서 쓸 수 있게끔 **pv 로 정의해주면 statefulSet spec 에 적어서 mount 해서 쓰게 된다.** (관리자와 협의를 통해 미리 마련되어야한다.)

   - **Pod 를 deleting 해도 Volume 은 없어지지 않는다.**

   - 아예 **StatefulSet 을 지운다면**,,, 순차적으로 지우더라도 **PV 와 PVC 는 남아있게 된다. (단지 잘 정리된 상태로 남아있는 것)** (Deleting and/or scaling a StatefulSet down will not delete the volumes associated with the StatefulSet.)

   - Deployment object 가 사라지면 거기에 딸려있던 Pod object 도 사라지고 생겼던 Pod 도 사라진다. 하지만 PV 와 PVC 는 남아있다. (**storage 를 지우는 작업도 필요**)

2. StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods.

   - StatefulSets 는 현재 포드의 네트워크 ID 를 책임지는 Headless Service 가 필요하다.

3. StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. **To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.**

   - `StatefulSet 이 없어져도 Pod 들이 다 지워지는 것이 보장되지않는다.` 그래서 **0 으로 scale down 을 통해 지워야한다.**

   - **storage 까지 옵션을 줘서 지우는 것이 original StatefulSet 자체에는 존재하지 않다. 하나하나 scale down 을 통해 지워야 함.**

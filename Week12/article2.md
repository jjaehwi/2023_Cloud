# Job

- `Job 을 통해 Pod` 를 만드는 것과 `그냥 Pod` 를 만든는 것의 차이

- **`Job 으로 Pod 를 만들면 Pod 가 시간이 되면 사라진다.`**

  - 0/1 상태에서 1/1 상태가 될 때를 생각해보자..

  - 1개를 만들어야하는데 아직 0 개를 만든 것

  - 실제 Pod 가 만들어지는 것 -> 0 개 (kubelet 이 runtime 을 지나 image 를 다운 받는 것)

  - Pod 란 오브젝트는 1개 있는 것 (실제 Pod 를 만들으라고 하는 오브젝트는 생성되어있음)

- `Job` 은 **끝이 있는 프로그램을 Pod 로 돌릴 때 Job 으로 Pod 를 생성**한다. (**Job 을 통해 만들지 않으면 보통은 웹서버 처럼 계속 돌아가는 Pod 를 만듬**)

  - **_하나 이상의 포드를 만들고 지정된 수의 포드가 성공적으로 종료될 때까지 포드의 실행을 계속 재시도한다._**

- **_Pod 가 끝난다는 개념_** -> **응용 프로그램의 끝이 있는 경우 (끝이 있는 일거리) 응용의 실행 결과가 나오고 프로세스가 종료되는 것**

- **When a Job completes**, `no more Pods are created`, but the `Pods` are **usually not deleted either.**

---

- 예시

<img width="406" alt="스크린샷 2023-06-11 오전 5 21 53" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/8c36566f-f5d6-45fd-be30-3a4155e584b5">

- 원주율을 2000 자리 까지 찍고 끝나는 프로세스 같은 응용 프로그램을 돌릴 때 Job 을 사용한다. 이런 형태의 일거리를 Job 으로 돌리는 것.

- 돌리고자하는 프로그램의 특성을 잘 알고 있어야 Job 을 이해할 수 있다.

  - Job 으로 돌리는 task 에 대한 이해가 필요

- `restartPolicy`

  - Pod 에도 있는 spec 정보임, 없으면 default : **Always** -> 포드는 항상 살아있기 때문

  - Job 에서는 always 를 사용할 수 없다. (**Job 에서 만드는 Pod 는 restartPolicy 가 never 아니면 onFailure 이다.**)

  - **onFailure** : 돌리다가 문제가 생기면 계속 만들고 만들어지면 그만 만들어라.

  - **Never** : 문제되면 바로 멈춰라.

  - Deployment 로 돌렸다면 Pod 를 죽여도 계속해서 살아날 것임 왜냐 -> Always 니까

- `backoffLimit` : 문제가 생겨서 재시도 하더라도 어느정도까지만 재시도 할지 설정하는 것.

- `정리` : `Job 은 복잡한 걸 끝내면 Pod 를 끝내라고 하기 위해 만들어졌다.`

---

<img width="572" alt="스크린샷 2023-06-11 오전 5 25 25" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/4f5d0920-f8e9-416f-96f3-f7795f7a992e">

- `completions`, `parallelism` 이라는 스펙이 추가된 Job

  - 동시에 몇개를 돌려라… 에 대한 얘기

- (ex) 원주율 2000 자리 까지 찍는 걸 몇 번 반복할까 : 5 번 반복했으면 좋겠다. -> copletions: 5

  - parallelism 이 없으면 sequential 하게 1->2->3->4->5 이렇게 진행이 될텐데..

  - parallelism 이 설정되면 병렬적으로 Pod 를 생성하여 응용이 돌아가는 것처럼 1 -> 3 -> ? 과 2 -> 4 -> ? 가 같이 돌아가는 것처럼 된다.

    - (ex) completions: 5, parallelism: 2 -> 5 번을 해야하는데 동시에 최대 2 번까지는 같이 돌아가도록 허용하도록 하는 것

---

## + Role 추가 정리

- Role 은 뭘 할 수 있게끔 한걸까

  - `api 에 대한 권한`에 대한 것.

  - `ServiceAccount` 는 **Pod 에게 부여할 ID (identifier) 를 주기위해 만들어졌다. (Pod 가 만들어지면 ServiceAccount 를 주는 것)**

- 쿠버네티스의 운영에서 namespace 를 가지고 grouping 되는데

  1. namespace 의 연관이나 ServiceAccount 에 대한 연관을 가지고 얘가 뭘 할 수 있는지 없는지에 대해 얘기하는 것

  2. 그 **Account 가 뭘 할 수 있는지 없는지 규정해놓은 규정집 = `Role`**

  - **다양한 규정집 중 어떤 규정집을 Account 에 붙일 것인가 = `RoleBinding`**

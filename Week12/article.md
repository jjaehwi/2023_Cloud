# Role & Role binding

- [Kubernetes API Server](#kubernetes-api-server)

  - [Authentication](#authentication)

  - [Authorization](#authorization)

  - [Admission Control Plugin](#admission-control-plugin)

  - [Validating resource and store](#validating-resource-and-store)

- [Authentication (SA 개념)](#authentication-1)

- [Authorization (RBAC 개념)](#authorization-1)

---

## Kubernetes API Server

<img width="775" alt="스크린샷 2023-06-10 오후 8 25 44" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/ec0d5db0-7ce3-4287-8ea8-4aa299dd8698">

- When creating a resource from a JSON file, forexample, kubectl posts the file’s contents to the API server through an HTTP POST request. In the API server, it follows the following steps.

  - **3 단계를 겪고 etcd 에 저장되고 controller 들이 작업한다.**

  - API 서버에서 어떤 과정이 일어나는지는 `role & role binding` 과 관련이 있다.

1. 인증에 관련된 것은 인증에 관련된 플러그인을 갖다가 써라

2. 권한에 관한 것도 마찬가지로 plugin 소프트웨어를 만들어서 쓸 수 있도록 해놨다.

3. 이외 추가적인 단계를 거치도록 plugin 할 수 있다.

- 지금까지는 그냥 다 접근할 수 있게 해놨는데, 실제 쿠버네티스를 사용할 땐 인증, 권한, admission control 에 대한 plugin 을 가져다가 사용해야한다.

- 쿠버네티스의 여러 리소스들은 API 서버를 통해 배포되고 관리되었다.

- API 서버에 내부적으로 동작하는 3 가지 모듈

  1. `인증 (Authentication)` : 누가 지금 APi 서버에 접근하는지

  2. `권한 (Authorization)` : 확인 된 사람이 할 수 있는 동작이 무엇이있나

  3. `Admission Control` : final resource 에 접근하거나 etcd 에 접근할 때 추가적으로 해야할 작업에 대한 것.

---

### Authentication

1. **AUTHENTICATING THE CLIENT WITH AUTHENTICATION PLUGINS (Auth N)**

- authentication 은 인증에 관련된 것인데 쿠버네티스가 인증에 관련된 것은 제공하지않아서 별도로 아웃소싱해야한다.

- 여러개를 플러그인 할 수도 있어서 하나하나 다 통과해야함.

- Determines who is sending the request. It does this by inspecting the HTTP request.

  - 내가 누구인지도 http request 에 담아서 보내기 때문에 http request 메시지를 살펴봐야한다. http request 메시지 안에 인증에 관련된 값을 어떻게 포함시키냐는 플러그인 모듈에 따라 달라질 수 있다. (authentication 플러그인에 따라 달라지는 것)

---

### Authorization

2. **AUTHORIZING THE CLIENT WITH AUTHORIZATION PLUGINS (Auth Z)**

- Their job is **to determine whether the authenticated user can perform the requested action on the requested resource.**

  - 너가 할 수 있는 행동이 뭔지 확인해보자~ 하는 것

  - 어떤 액션을 어떤 리소스에 대해 할지 판단하는 것.

  - 오브젝트에 대해 너가 만들 수 있는지 없는지 권한이 있는지 파악하는 것. (액션을 취할 수 있는가?)

- `role base 된 access control`

- `namespace` : **논리적으로 남의 것은 안보이도록 격리해주는 것**

  - 네트워크 입장에서 보면 네트워크 스펙도 격리하도록 한 (서로 안보이도록) tenant 기술 (세입자 개념)

- namespace 로 구별이 되어있고 **내가 A 에 있으면 A 안에서 할 수 있는 행동만 보면 되지** B 쪽을 봐봐야 할 수 없으니까 A 만 보면 된다.

---

### Admission Control Plugin

3. **VALIDATING AND/OR MODIFYING THE RESOURCE IN THE REQUEST WITH ADMISSION CONTROL PLUGINS**

- These plugins can modify the resource for different reasons

- 2 까지 통과했으면 실제로 리소스를 만들거나 access, creation, modify 등등의 동작을 할 수 있다.

  - 하라고하는 것을 그대로 하는 것이 아닌 미리 사전에 등록해놓은 control 을 할 수 있다.

  - 다양한 admission control 이 존재하는데,,

    - AlwaysPullImages , ServiceAccount, NamespaceLifecycle, ResourceQuota 등등..

---

### Validating resource and store

4. VALIDATING THE RESOURCE AND STORING IT PERSISTENTLY

- 마지막으로 manifest 를 특별한 오류가 있는지 체크하고 그 정보를 etcd 에 기록한다. 그 다음 응답을 할 때 (Pod 에 만들어라~ ) 여기에 쓰인 정보를 통해 응답 하게 된다.

---

## Authentication

- API 서버로 요청해온 client 들의 신분을 확인하는 것

- client 를 구별하면 users 일수도, 시스템 내부에 있는 Pods (applications running inside them) 들 일 수도 있다.

  - 두 클라이언트에 따라 인증에 대한 방법이 달라진다.

---

1. `Users` are meant to be **managed by an external system**

- `user 인 경우` : ID / PW 와 같은 인증방법 자체가 **별도의 인증 시스템을 플러그인 할 수 있어서 별도 시스템에 의해 관리**된다.

  - API server 내에 오브젝트 (리소스) 들은 정의되어있는데 사람에 대한 인증과 관련된 오브젝트나 리소스는 정의되어있지 않다.

---

<img width="628" alt="스크린샷 2023-06-11 오전 4 34 53" src="https://github.com/jjaehwi/2023_OS/assets/87372606/c786fb15-feca-4e2b-bebd-e93580993ce1">

2. The `pods` **`use the service accounts`**, which are **_created and stored in the cluster as ServiceAccount resources_**

- `Pod 인 경우` : Pod 의 `ServiceAccount` 라는 오브젝트와 연관되어있다. **_ServiceAccount 안에 access 를 위한 토큰 (credential) 이 존재한다._**

  - Pod 나 개개인 user 들을 **sa 를 통해 그룹을 만들 수 있고 그룹이 만들어지면 그룹 정보를 통해 그룹 차원에서 access 가능한지 판단**한다.

- **Pod 는 namespace 와 연관**되어 있고 `default 로 ServiceAccount 라는 리소스가 만들어진다.` 그러면 `Pod 들은 생성될 때 자동으로 연결`되게 끔 된다.

  - **_namespace 를 만들면 ServiceAccount 가 만들어지는 것_** : **그 안에 Pod 를 만들면 자동으로 ServiceAccount 와 mapping** 되는 것

- `특별한 ServiceAccount 를 만들어서 연결`시킬 수도 있다.

  - `ServiceAccount` : Pod 를 위한 security 정보를 담고있는 애, 사람은 사람끼리 알아서 해결

  - 자동으로 생성되어 (default) 시스템적인 정보를 주는 것 (이름을 붙혀서 만들 수도 있음)

  - 필요에 따라 새로운 ServiceAccount 를 만들 수 있다.

    - **특별히 만든 ServiceAccount 는 해당 namespace 에서만 해당된다.**

    - **다른 namespace 의 Pod 와 ServiceAccount 를 연결시키는 것은 불가능**

  - **_새로운 ServiceAccount 에 Pod 를 붙히고 싶으면 Pod 내의 spec 필드에 어떤 sa 가 해당 Pod 에 연결될지 적는다._**

---

<img width="626" alt="스크린샷 2023-06-11 오전 4 43 17" src="https://github.com/jjaehwi/2023_OS/assets/87372606/4dc08683-1602-4449-b149-089301c09e23">

- `Image pull secrets` : 이미지가 있는 레지스트리에서 secret 에 대한 정보가 저장되는 곳

- `Mountable secrets` : Mountable 한 secret = 여기에 정리된 것만 mounting 될 수 있도록

- `Tokens` : Pod 가 ServiceAccount 와 연결되는데 **Pod 의 API 서버에 접근할 때 사용되는 토큰**

- The SERVICE ACCOUNT’S TOKEN is used to talk to the API server.

  - Pod 를 만들면 Pod 가 필요로하는 security 정보들이 ServiceAccount 안에 존재.

  - By setting the name of the `ServiceAccount` in the **spec.serviceAccountName field** in the pod definition, the **service account created is assigned to pods.**

---

## Authorization

<img width="661" alt="스크린샷 2023-06-11 오전 4 49 33" src="https://github.com/jjaehwi/2023_OS/assets/87372606/bed8e907-09c9-4c0b-9627-21b1462a3e9a">

- An authorization plugin such as `RBAC(Role Based Access Control)`, which runs inside the API server, **determines whether a client is allowed to perform the requested verb on the requested resource or not.**

  - `RBAC (role – based access control)` : **요청하는 resource 에 대해 어떤 행동을 취할지 정의한다.**

    - **_사용자가 수행할 수 있는 작업을 제어하는 방법._**

    - RBAC 는 각 사용자에게 하나 이상의 "역할"을 할당하고 각 역할에 서로 다른 권한을 부여하여 이를 수행.

  - HTTP method 에 대한

- `Roles` and `ClusterRoles`, which **specify which verbs can be performed on which resources.**

- `RoleBindings` and `ClusterRoleBindings`, which **bind the above roles to specific users, groups, or ServiceAccounts.**

  - 클러스터 전제로 가면 ClusterRoles, ClusterRoleBindings 이 됨.

- `Roles` define `what can be done`,

  - 어떤 행동을 할 수 있냐 : `Role` (무슨 일을 할 수 있냐 없냐 …) (할 수 있는지 없는지 어떻게 정의하나? -> Role 이라는 새로운 오브젝트를 통해)

- `Bindings` define `who can do it`

  - 누구를 role 에 바인딩할지 : `RoleBinding`

- 즉 role 을 만들고 누구를 role 에 바인딩할지 rolebinding 으로 정해주는 것

<img width="610" alt="스크린샷 2023-06-11 오전 4 52 56" src="https://github.com/jjaehwi/2023_OS/assets/87372606/231b379e-dd28-4cd1-8c60-98fbf4b0a84d">

- `rules` 은 리소스에 대한 action -> 위 그림에서 services 에 대한 리소스에 대해서는 get 이나 list 만 허용한다.

  - 이것은 service-reader 라고 하는 name 의 Role 인 것

  - **이 role 을 해당되는 account 에 연결**시키면 된다. 이 개념이 `RBAC` 이다.

- **어떤 리소스에 대해서 어떤 action 을 취할 수 있고 어떤 리소스에 대해서는 취할 수 없다. 라고 정의하는 것이 `role`**

- **이 role 을 어느 account 에 binding 할 수 있는지 정하는 것이 `RoleBinding`**

- 그림에서 보면

  - get, list 등 보는 것만 허용하는 role 을 만들어놓고, 어떤 특정 리소스에 이 role 을 binding 해야한다.

  - Pod 가 만들어지면 ServiceAccount 가 생겼기 때문에 이 account 에 role 을 binding 해야하는데 이 실습은 안보여줬고 role binding 하는 명령어를 찾아서 account 에 binding 하는 것을 실습에서 보여주자.

  - rolebinding 을 만들어서 sa 나 user 에 연결시키면 된다. (실습 영상 참고)

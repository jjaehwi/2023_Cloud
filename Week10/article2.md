# Configmap and Secret

- 응용프로그램이 돌아가고 있으면서 바꾸고 싶은 정보가 있다면 activation 에 영향을 주지 않으면서 업데이트할 수 있도록 하게 하자

  - 프로그램이 돌고있으면 도는 **프로그램에게 돌아가는데 필요한 정보들 (configuration) 을 dynamic 하게 바꿀 수 있도록 그런 정보**를 따로 떼어놓자 (`configMap`)

  - 이러한 정보들 중에 **비밀스러운 정보들 (password 같은) 더 secure 한 정보**들도 따로 떼어놓자 (`Secret`)

---

- [ConfigMap](#configmap)

- [Secret](#secret)

- [1. Environment variables](#1-environment-variables)

- [2. Volumes](#2-volumes)

- [정리](#정리)

---

## ConfigMap

<img width="766" alt="스크린샷 2023-06-05 오후 7 26 22" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/69e794ea-1924-4c3d-b6a4-419ed3d92e4f">

- `ConfigMap` 에서 URL 을 바꾸면 **_원래 돌고 있던 코드가 돌아가면서 필요한 정보가 바뀐 것을 알고 있다가 가져다가 적용한다._**

- DB 의 유저 아이디와 비밀번호와 같은 정보를 따로 떼어내서 갖고있고 Pod 와 연결되어서 가지고 가서 사용할 수 있게 한 것.

- Base64 로 인코딩 해놓은 것 (보이는 게 아스키코드 1234 처럼 안보임)

  - 암호화해놓은 것은 아님

- `요약` : **프로그램이 돌아가면서 필요한 정보들을 아주 중요한 secret 정보와 일반적인 정보들을 따로 두고 필요할 때 가져가면서 쓸 수 있도록 한 것**이다.

  - 어떻게 만들어낼 것인가?

  - **apiServer 에게 요청해서 만들어내야한다.**

  - yaml 파일로 만들어낼 수 있지만 yaml 파일로 만들어서 배포하면 비밀스러운 정보인데 목적에 모순된다.

  - yaml 파일로 만들지 않고, kubectl 커맨드로 수동적으로 쳐서 만들 수 있고, secret 파일을 따로 만들 수 있는 별도의 환경이 있다. (kubernetes 와는 별개의 일) --> 어쨋든 apiServer 를 통한다.

  - yaml 파일로 만들어서 apply 하지 않더라도 다른 방법으로 만들어져도 결과물은 읽어보면 yaml 파일 형식일 것이다.

  - **configMap 이나 Secret 의 spec 이 어떻게 생겼는지 알아야한다.**

  - 만들어졌다면 `어떻게 Pod 에 전달시킬지 알아야한다. (응용 Pod 에 Ejection)`

1. `환경변수 방법` (environmental variables)

   - 코드가 실행될 때, 환경변수를 세팅하는 방식으로 정보를 전달.

2. 일일이 환경변수를 세팅하는 것이 아니라, `통째로 정보가 저장되어있는 파일을 주는 방식`

   - `mount` 하는 것 (**_secret 이나 configuration 에 대한 정보를 갖고있는 파일을 special 한 Volume 처럼 취급하고 mount 시키는 것_**)

- 프로그램이 돌아가는데 어떤 설정값이나 secret 한 정보들이 필요할 텐데 이거는 별도로 내부 오브젝트로 만들어져야한다.

  - 그 정보를 만드는 방법은 control 명령과 yaml 파일을 통해 할 수 있지만 중요하므로 별도의 환경을 통해 만든다.

  - 이 정보를 담은 것을 Pod 와 연결시키는 방법에는 `1. 환경변수를 통해 세팅하는 방법`과 `2. 파일을 주는 방식으로 special 한 Volume 처럼 취급하여 mount 시키는 방법`이 있다.

---

## Secret

<img width="984" alt="스크린샷 2023-06-05 오후 7 55 38" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/7b4839b6-fdd3-4bcb-8f85-7dee55c78fd8">

- **`Secret` 은 오브젝트인데 `sensitive 한 data 를 담고있는 오브젝트`이다.**

  - 어떤식으로 만들어져라라고 하는 오브젝트가 아님 (이전에는 오브젝트를 만들면 그 오브젝트대로 만들어져라고 하기 위해 오브젝트를 만들었었음)

  - 그 오브젝트는 etcd 안에 들어가있다. 이것을 Pod 가 활용하는 방법은 **1. Application 에 돌아가는 요소들에 대해 환경변수를 설정해서 가져가도록 할 수 있고**, **2. 통째로 Volume 처럼 mount 하는 방법이 있다.**

- 암호화된 것이 아니라 아스키 값이 아닐 뿐이기 때문에 **별도의 암호화가 필요**하다.

  - Enable Other `Encryption` Methods for Secrets

- Secret 오브젝트에 접근해서 **데이터를 읽어가는 애들을 제한을 둬야한다.** (아무나 하게끔하면 안됨)

  - Enable or configure API `access rules` that restrict reading and writing the Secret

---

<img width="650" alt="스크린샷 2023-06-05 오후 8 12 40" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/15471b95-e415-4a7c-8f2a-56847086b93a">

- **_쿠버네티스 api 를 통해 만들고, 이 정보가 etcd 에 기록_** 된다. (apiserver 에 정보를 알려주면 etcd 에 base64 인코딩하여 기록한다.)

  - **프로그램이 돌다가 secret 정보가 필요해서 apiserver 에 요청하면 etcd 로 부터 가져와서 준다. (`환경변수 방법`)**

<img width="955" alt="스크린샷 2023-06-05 오후 8 14 39" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/7431191a-083b-4969-814c-4bf304e4f70f">

- apiserver 에 정보를 넣으면 **base64 로 인코딩되어서 etcd 에 저장**되는 것

  - no encryption 이어서 위험하다.

  - Like most Kubernetes objects, Secrets are stored in etcd. By default, no encryption is done at the Kubernetes layer before persisting Secrets to etcd.

<img width="955" alt="스크린샷 2023-06-05 오후 8 16 09" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/8ed1ba14-0c04-42eb-8521-d1a0098c9c36">

- **별도의 방법으로 암호화하는 key 를 더해서 encryption** 을 한다. (쿠버네티스의 기본이 아니고 기능확장을 위해 이 체계를 만든 것)

  - Vault : 키 하나만 알면 전부 파악할 수 있게 되기 때문에 이를 방지하기 위해 키가 계속 갱신되고 이를 수동으로 관리하기 힘들기 때문에 별도의 툴을 사용하여 관리하는 것

---

## 1. Environment variables

<img width="200" alt="스크린샷 2023-06-05 오후 8 18 14" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/6cf98242-5156-4546-8cd2-fd643966b7f1">

- name : 유저 라는 정보가 있는데 이 정보를 어디서 가져오냐 valueFrom

- 많은 항목 중 secretKeyRef: mysecret 에서 가져오고 key 레퍼런스는 dbuser 이다.

- 환경변수 값을 바꾸고 싶으면 secret 값

1. The **environment variable key** that will be available in the application

2. **The name of the Secret object** in Kubernetes

3. The **key in the Secret object that should be injected into the USER variable**

- `한계점` : 코드상보단 빠르지만 **업데이트 되기 위해 코드를 죽였다 살려야한다**. 아니면 코드상에서 working 하면서 바뀔 수 있는 툴이 있어야한다.

  - 돌아가던 코드를 멈췄다가 다시 돌려야한다. **왜냐면 환경변수를 다시 세팅한 것 이니까**

---

## 2. Volumes

<img width="227" alt="스크린샷 2023-06-05 오후 8 23 21" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/5fd8ff13-7ba1-4809-a262-7fecbdada298">

- env 가 없이 **_volumes 를 만들어서 그 Volume 을 mount_** 시켰다.

  - volumes 정보를 보면 `secret` 이라고 되어있다. 그리고 **volumeMounts 를 보면 mountPath 를 통해 mount 한 것**

1. Pod-level **volumes** available **for mounting**. Name specified must be referenced in the mount

2. The volume object to mount into the container filesystem

3. **Where in the container filesystem** the mount is made available

- `장점`

**_1. Secrets may be up-dated dynamically, `without the Pod restarting`._**

- 코드상에서 읽어가는 스타일 (**working 하던 상태에서도 업데이트** 될 수 있는 것)

- The application must **watch** the configuration files on disk and a**pply new configuration when the files change.**

**_2. side-car proxy required to get the updated secrets_**

- side-car proxy 를 두는데 Pod 안에 컨테이너를 하나만 두는 것이 아니라 메인 컨테이너를 도와주는 컨테이너를 하나 더 만드는 것.

  - 이 side car 를 통해 주기적으로 파일에 업데이트 된 것이 있는 지 확인해서 찔러줌

---

## 정리

- **secret 은 etcd 안에 만들어지고, 정보들은 key value 값**이다.

- **_Pod 에 연결되는 방법은_** `1. 환경변수`나 (코드를 만들 때 설정), `2. Volume 을 만들어서 mount` 시킨다.

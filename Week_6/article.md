# Kubernetes 의 기본적인 Networking 과 Service

- [Network Basic](#kubernetes-network-basic)

  - [IP addresses in Kubernetes](#ip-addresses-in-kubernetes)

- [K8S 에서 Service](#service)

  - [NodePort Service](#nodeport-service)

  - [단일 노드에 여러 개의 PoD 로 구성된 서비스](#단일-노드에-여러-개의-pod-로-구성된-서비스)

  - [여러 노드에 분산된 PoD 들로 구성된 서비스](#여러-노드에-분산된-pod-들로-구성된-서비스)

---

## Kubernetes Network Basic

- 쿠버네티스 클러스터 안의 포드가 만들어지면 IP 주소가 할당이 된다. (private 주소로 할당)

  - 보통 10 점대의 주소로 할당이 되는데 쿠버네티스를 처음 세팅할 때 정할 수 있다.

  - 서비스에 사용되는 `주소 풀`이 따로 있다.

  - private IP 주소는 로컬 네트워크에서 사용되고 public IP 주소는 네트워크 외부에서 사용된다.

    - public IP 주소는 네트워크 외부에서 통신하는 데 사용된다.

1. 모든 포드는 자체 IP 주소(PoD IP 주소)를 얻는다. `네트워킹 SW (CNI Pugin)` 가 할당한다.

2. 노드의 포드는 `NAT` 없이 모든 노드의 모든 포드와 통신할 수 있어야 한다.

   - 네트워킹 SW(CNI Pugin) 가능하게 해준다.

   - 서로 private 한 ip 주소를 알고 있다는 것

   - private 주소를 쓰고 있으면 이 private 주소를 가지고 외부와 통신을 할 수 없는데, private 주소에서 패킷을 외부로 보낼 때 NAT 기능을 통해 주소가 변환되어서 (외부에서도 알 수 있도록 바꿔서) 나간다.

     - 각자는 포트넘버로 구분, 외부에서 private IP 에 할당해놓은 주소가 있어서 이 주소를 통해 외부와 주고받고 외부에서 이 주소로 응답이 오면 각각은 포트넘버를 가지고 구분을 하는 것

3. Pod 내의 컨테이너는 네트워크 네임스페이스 (IP 및 MAC 주소) 를 공유하므로 루프백 주소를 사용하여 서로 통신할 수 있다.

- `정리` : 포드에는 주소가 붙는데 그런 주소를 가진 클러스터 안에 다른 포드들 끼리는 서로 NAT 없이 통신을 할 수 있다. 이것을 CNI 플러그인이 해준다.

- NAT 동작 그림

<img width="600" alt="스크린샷 2023-04-16 오전 2 42 54" src="https://user-images.githubusercontent.com/87372606/232244936-54a142f3-1091-4079-a29d-84a67047f485.png">

---

### IP addresses in Kubernetes

- Kubernetes 의 IP 주소와 관련된 용어

- 서비스, 포드, 컨테이너 및 노드가 IP 를 사용하여 통신한다.

- `Pod IP` : 지정된 포드에 할당된 IP 주소, `일시적`이다.

  - 포드가 사라졌다가 생기게 되면 IP 도 바뀌게 되는데, 그 전 주소에 대해 접근을 하고 있었다면 바뀌게 되면 접근을 못하게 될 수 있다.

  - Cluster IP 가 탄생

- `ClusterIP` : 서비스에 할당된 IP 주소, 이 주소는 서비스 기간 동안 stable 하다.

- `NodeIP` : 지정된 노드에 할당된 IP 주소

---

## Service

<img width="370" alt="스크린샷 2023-04-16 오전 2 35 38" src="https://user-images.githubusercontent.com/87372606/232244641-4a38104b-5837-4dfb-81d3-9b529810ffb3.png">

- 그림에서 Service B 에 포드가 3개 있고 label 은 app=B, Service A 에 포드 1개 label 은 app=A

  - 포드라는 이름의 리소스를 만들면 자동으로 만들어지는데, 어떤 오류로 인해 포드가 사라지면 다시 만들어진다. (포드라는 이름의 리소스를 만들면 포드 컨트롤러가 보고 포드를 만드는 것)

  - 사라졌다가 생기게 되면 IP 도 바뀌게 되는데… 그 전 주소에 대해 접근을 하고 있었다면 바뀌게 되면 접근을 못하게 될 수 있다.

  - 그렇다면 바뀌지 않는 주소에 대해 접속할 수 있도록 하자라고 생각했고 `클러스터 IP` 가 탄생했다.

- `Cluster IP` : **The IP address assigned to a `Service`. This address is stable for the lifetime of the Service**

  - `서비스 라고 하는 오브젝트`를 새로 define 한 것.

  - 서비스라는 오브젝트를 지우지 않는 한 변하지않는 주소가 클러스터 IP

- app=A 라고 하는 포드를 보면 이 포드에 대해 외부에서 지속적으로 접근하고 싶을 수 있기 때문에 서비스 A 라는 껍질을 하나 더 씌웠다.

  - 포드를 만들으면 주소가 계속 바뀔 수 있으니까 **_변하지 않는 주소가 필요하기 때문에_** `이 주소를 붙히는 동작이 서비스라는 오브젝트`이다.

---

## K8S 에서의 서비스란?

- `서비스는 포드의 집합`이다.

- 만약 레플리카 셋에 의해 포드 3 개가 생기게 되면 이 3 개의 포드는 전부 같은 애들이기 때문에 각각의 private IP 는 다르지만 이 같은 애들을 묶어서 서비스라고 정의를 해서 변하지 않는 주소를 할당했다.

- `logical set of Pods` (포드의 logical 한 set) (또 다른 abstraction,, 새로운 resource)

- The set of Pods targeted by a Service is usually determined by a `LabelSelector`

  - 누군가 이 서비스의 변하지 않는 주소에 대해 접근을 하면 `selector` 로 같은 `label` 을 가진 (같은 집합안에 있는) 포드들을 선택한다.

- `같은 포드들 끼리 묶어서 고유의 IP 를 부여해놓은 것이 Cluster IP 주소`이다. (서비스를 정의해서 그 서비스에 부여되는 주소)

  - `internal` 용이다. 지금까지의 주소들은 전부 `private 주소`이기 때문. (`Exposes the Service on an internal` IP in the cluster.)

  - 변하느냐 안변하느냐의 차이일 뿐이지 **private 이므로 외부에서는 access 를 할 수 없다.**

  - 외부로 가기 위해서는 private 한 곳에 주어진 public 주소가 필요하게 된다.

- **_바깥에서 접근할 수 있는 방식의 서비스_** : `NodePort` 라는 서비스, `LoadBalancer`, `ExternalName` 이라는 서비스

---

### NodePort Service

- 외부와 access 하기 위해 포드들과 서비스는 private 주소이기 때문에 할 수 없다.

  - Node 의 서비스, 포드들이 있는 것.

  - 안에서는 통신이 되는데 밖에서 접근하려면 문 앞에 public 주소가 있어야한다.

  - 나갈 때 바뀌어서 나가야 하는데 이게 NAT 동작이다. 그리고 돌아올 때도 바뀌어서 돌아온다. (NAT 동작 참고)

  - 문 앞에 있는 주소와 포트 값을 알려줘야 찾아올 수 있다.

- 그러면 Node 에게 먼저 접근을 해야하기 때문에 Node 의 주소를 통해야한다.

  - Node의 주소를 통해서 private 영역에 왔다면 이 안에서 서비스를 어떻게 찾아갈까 : port 를 통해 찾아가고, 이게 `NodePort` 이다.

- `NodePort Service` : 바깥에서 오기 위해선 NAT 기능이 필요하기 때문에 포트값이 필요하고 포트 값을 보고 찾아오면 내부의 동작에 의해 (table) 알아서 찾아가게된다.

  - `내부에서 서비스와 포드에 할당된 포트번호는 내부에서만 알고 있기 때문에 외부에서 알 수 없다.` 그래서 노드 포트 번호가 필요하다.

<img width="878" alt="스크린샷 2023-04-16 오전 2 55 06" src="https://user-images.githubusercontent.com/87372606/232245502-5afaf645-1740-4b9d-886c-c729030f1b8c.png">

- 외부에서 접근하려면 `노드의 IP` 에 접근을 한다. (VPN 으로 이 IP 에 바로 붙히기 때문에 지금은 private 주소 인 것)

  - 그 안에 서비스에 접근하기 위해 31000이라는 포트번호를 가지고 가야한다. (`NodePort 방식의 서비스`)

  - nginx 를 포드로 만들고 cluster IP 를 만들고 노드 IP 를 만들어서 위와같은 방식이 만들어지면 접근을 할 수 있게 된다.

  - `일종의 NAT 기능`을 사용한 것!

- 이 일이 자동으로 일어나게끔 쿠버네티스 네트워크 기능을 세팅할 수 있다.

  - 이게 `kube - proxy` 이다. (이러한 일이 일어나는 IP 테이블을 세팅한다.)

  - `type: NodePort` 를 쓰면 NodePort 서비스가 된다.

  - spec 에 들어갈 중요한 정보는 `selector` 가 된다.

  - port 정보 에는 추가적으로 노트 포트에 대한 정보가 들어가야한다. (구체적으로 작성하지 않으면 자동으로 할당된다.)

- 포드를 묶으면 포드가 죽었다 만들어졌다 하면서 주소가 계속 바뀔 수 있는데, 이런 각 PoD 의 주소들이 바뀌더라도 정리를 해주는 오브젝트가 자동으로 만들어진다. (`Endpoints 오브젝트`)

<img width="571" alt="스크린샷 2023-04-16 오전 2 59 10" src="https://user-images.githubusercontent.com/87372606/232245662-bb208200-65bf-4a11-8f23-90a4be82c2bb.png">

---

<img width="878" alt="스크린샷 2023-04-16 오전 2 58 30" src="https://user-images.githubusercontent.com/87372606/232245646-a0e80297-3d5c-4d84-920f-1e0caa23d76f.png">

- 어떤 label 이 붙은 애들을 선택할 것인지 명시하는 selector

  - 서비스 하는 껍질의 포트넘버를 80 으로 알려줌

  - 9376 이라는 포트를 가진 MyApp 이 label 인 포드들을 잡아서 서비스를 만듬.

  - 내부에서 접근하기 위해 IP 가 필요한데,, IP 는 자동으로 붙혀줌. (클러스터의 IP 는 뭐를 쓸지, 포드의 IP 는 뭐를 쓸지 설치할 때 설정을 해두고 그에따라 IP 가 붙음)

  - 직접 access 하기 위해 만들어진 service 의 상태를 kubectl 명령으로 (describe) 보면 상세 IP 주소를 보고 access 를 한다.

  - `포드의 포트 값 = 타겟 포트`

  - `서비스의 포트값 = 포트`

  - `외부에서 들어올 때 NAT 를 위한 포트 값 = 노드 포트`

---

- **_외부에서 노드에 붙어있는 IP 포트번호를 통해 접근하려고 하면,_**

  - 노드가 되건 VM 이 되건 IP 가 붙게되는데 (사람이 메뉴얼 하고 또 다른 소프트웨어 (VM 의 경우 뉴트론) 에 의해 IP 가 붙고)

  - `외부에서 액세스 하기 위해 포트 값 뭘 쓸지 추가로 얘기 해줘야해서 노드 포트 값이 생긴 것`이고,

  - 그리고 `내부 로 접근하면 서비스 포트와 ip 로 접근`하고

  - `타겟 포트 번호와 포드의 IP 를 통해 포드에 접근`한다.

<img width="941" alt="스크린샷 2023-04-16 오전 3 20 37" src="https://user-images.githubusercontent.com/87372606/232246792-5c811269-17e9-42b1-8963-35c8b7cdd3dd.png">

---

## 단일 노드에 여러 개의 PoD 로 구성된 서비스

<img width="876" alt="스크린샷 2023-04-16 오전 3 15 11" src="https://user-images.githubusercontent.com/87372606/232246587-4ead913d-6751-4e35-ae9c-477fbc58c5f4.png">

- 3개 중 하나 만 선택해서 가야하는데, (동시에 불가능) (round robin)

  - Load balancing 과 같은 이런 일은 누가하냐면 서비스를 만들게 되면 `Endpoints` (최신 판에선 Endpoint slice) 라는 오브젝트는 **주소가 계속 바뀌는 것을 찾아서 계속 업데이트**한다.

  - `EndPoint` 는 각각에 주소가 0.4, 0.2, 0.3 인데 이 목록을 만들어 놓는다. 만약 포드가 없어졌다 사라지면 `주소가 바뀌는데 이에 따라 계속 갱신`한다.

  - 이 Endpoint 오브젝트는 `서비스를 만드는 과정중에 자동으로 생성`된다.

- `EndPoint 를 보고 돌아가면서 접근하게끔 내부 시스템이 구현되어있는 것이다. (목록을 보고 roundrobin)`

## 여러 노드에 분산된 PoD 들로 구성된 서비스

<img width="876" alt="스크린샷 2023-04-16 오전 3 15 29" src="https://user-images.githubusercontent.com/87372606/232246597-c2836441-e418-483c-a199-f3485d799f5a.png">

- 방금은 하나에 노드에 3개의 pod 가 전부 존재했었느데, 이 경우는 워커 노드가 3개이고 PoD 3개를 만들려했더니 스케쥴러가 공평하게 하나씩 만들어버린 상황이다.

  - 포드가 3개 있는데 각각 다른 노드에 있는 경우

- `Challenge`

  - 노드 포트 서비스에 의해 만들면 노드가 3개니까 노드 IP 가 3 개 인데 어디에 만들어야할까?

  - 노드 포트 서비스 라는 오브젝트를 만들면 어떻게 연결을 시킬까?

- 서비스 IP 주소는 자동으로 변하지 않는 주소로 만들어지고 `노드 포트 서비스가 만들어질 때, 모든 노드에 연결되도록 만들어진다.`

  - 어떤 노드 주소로 들어오던지 간에 (포트 값 동일) 서비스가 될 수 있다.

- 문제는 load balancing 이 될까?

  - EndPoint 에 의해 포드들이 선택이 되는데, 아무 노드로 들어가더라도 **EndPoint 에 의해 포드들이 선택이 되면 다른 노드에 있다면**, `그냥 나왔다가 다시 그 포드가 있는 노드로 들어간다.` (그런 것을 해주는 게 IP table 이다.)

  - IP table 을 건들이는 애가 `Kube-proxy` : 쿠버네티스 에 있는 `네트워킹 소프트웨어`가 한다.

- `정리` : 노드 포트로 정의를 해뒀는데, 어떤 것을 선택해도되고, EndPoint 에 의해 포드가 결정되고 만약 다른 노드에 있었다면 다른 노드로 가서 결정된 포드로 간다. kube-proxy 가 ip table 을 잘 관리해서 어디를 가도 table 을 통해 갈 수 있는 것이다.
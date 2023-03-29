# 네트워크 기초

- `네트워킹` : ip 패킷이 만들어져서 정보를 교류하는 것.

  - ip 패킷이 전달되기 위해 구간마다 통신방식을 통해 전달이 되어야한다.

- 라우터에서 ip 주소를 보고 (내부적으로는 라우팅 테이블을 확인) `포워딩` 진행

- `IPv4 주소` : 32비트의 호스트를 위한 identifier, 라우터 인터페이스

- `인터페이스` : connection between host/reouter and physical link

  - 앞부분이 공통된 영역 = `서브넷`

    - 서브넷 : high order bits = 서브넷 파트, low order bits = 호스트 파트

  - **독립된 IP 주소는 각각의 인터페이스를 가지고있다.**

- 단말기들이 동일한 서브넷 주소 안에서는 스위치를 통해 자동으로 통신이 된다. (같은동네)
  서브넷이 서로 다른 경우 라우터를 통해 포워딩되어야한다. (다른동네)

      - 서브넷 마스크 : /숫자 --> 어느 비트까지 동일한 부분인지를 나타냄. (`CIDR 방법`)

---

## DHCP (dynamic host configuration protocol)

<img width="655" alt="스크린샷 2023-03-29 오후 2 06 11" src="https://user-images.githubusercontent.com/87372606/228432083-6cc43b2f-bb61-4ed7-8a8e-d2417a8a62cd.png">

- DHCP 서버가 있어서 요청하면 나눠줄 주소들을 저장해둔다.

- DHCP request 메세지를 뿌리면 서버는 나눠줌.

  - 자동주소설정은 DHCP 클라이언트가 동작하는 것이고 네트워크에는 DHCP 서버가 있어야하는 것.

- 클라우드도 마찬가지이다. **virtual 머신마다 주소가 붙어야하고 그 안에 DHCP 서버가 있어야한다.** (`뉴트론에서 만들어놓은 기능에 DHCP 서버 기능`이 있다.)

- 네트워크와 VM 을 연결하면 그 안에 자동으로 DHCP 서버가 만들어져서 주소가 할당될 수 있다.

- 서로 다른 virtual 네트워크에 있는 VM 끼리 통신하려면 `라우터`가 있어야 하고 라우터에 virtual 네트워크를 붙여야한다.

---

### Public IP, Private IP

<img width="655" alt="스크린샷 2023-03-29 오후 2 08 34" src="https://user-images.githubusercontent.com/87372606/228432363-9acadc55-adc0-4e40-815c-d52ed4fd9014.png">

- `public ip` : 서로 중복되지 않게 **고유의 주소**가 되도록 하는 주소

  - outside network 와 연결하기 위해 public 주소 필요

- `private ip` : **local 하게만 사용**한다. (`내부망`)

  - 내 맘대로 임의로 붙혔기 때문에 같은 주소값을 가질 수 있다.

  - 이왕이면 혼돈되지 않게 특정 주소 블록을 잘라놨다. (10.0.0.0 ~ 10.255.255.255, 172.16.0.0 ~ 172.31.255.255, 192.168.0.0 ~ 192.168.255.255)

---

## VPN (Virtual Private Network)

<img width="655" alt="스크린샷 2023-03-29 오후 2 11 21" src="https://user-images.githubusercontent.com/87372606/228432769-58fad984-98e6-41cc-9782-752d2674785f.png">

- public network 가 있는데 일반적인 인터넷에는 위험할 수 있다.

- VPN 은 private 하게 외부와 단절되어 있도록, 원격에서 가능하게 해준다.

- 원격으로 데이터를 보낼 때 겉에 여러 필드를 붙혀서 내용이 암호화되게 하고 터널을 거쳐 나갈 때 `복호화`되게 한다. (내가 만든 것에 껍질을 더 붙혀서 보낸다고 이해.)

---

## HTTP (HyperText Transfer Protocol)

<img width="655" alt="스크린샷 2023-03-29 오후 2 14 36" src="https://user-images.githubusercontent.com/87372606/228433234-d915e6cd-75cc-4a66-a38e-6cd8386d651f.png">

- `HTTP 프로토콜` : 어떻게 네트워크를 통해 가는지 단말이 서버한테 보내는 메세지, 서버가 응답하는 내용이 동작되는 표준적인 방법

- **서버 – 클라이언트 구조**

- `서버` : 항상 켜져있음, IP 고정,

- `클라이언트` : 서버와 소통, 간헐적으로 연결, 동적 IP 주소를 가짐, 클라이언트끼리 직접 소통 불가

<img width="486" alt="스크린샷 2023-03-29 오후 2 22 14" src="https://user-images.githubusercontent.com/87372606/228434282-ce1d345a-f735-4bcf-b133-90e289fe9626.png">

- 웹페이지는 오브젝트로 구성되어있다.

  - 오브젝트 : HTML 파일, JPEG 이미지, 오디오파일 등등

  - HTML 포멧 기반

  - URL(uniform resource locator) (리소스의 위치를 표현)

<img width="664" alt="스크린샷 2023-03-29 오후 2 22 30" src="https://user-images.githubusercontent.com/87372606/228434330-49357002-aed3-4972-9b9b-b97a82240f84.png">

<img width="962" alt="스크린샷 2023-03-29 오후 2 24 34" src="https://user-images.githubusercontent.com/87372606/228434607-b2abf90c-51e0-45af-8728-f120c7ab88bc.png">

- request 에서는 URL 이 중요한 정보가 되고, 응답으로는 URL 로 요청된 정보를 준다.

- HTTP 는 인터넷 웹 브라우징 할 때 요청하고 응답할 때 사용되는 프로토콜이고, 요청할 때 URL 이 중요하게 요구된다.

- 기능을 추가하기 위해서는 Method (GET, POST, HEAD commands) 의 단어에 맞는 행동을 하면 된다.

<img width="323" alt="스크린샷 2023-03-29 오후 2 24 13" src="https://user-images.githubusercontent.com/87372606/228434566-5429a4ab-ee9d-485f-8236-ae2bc456c346.png">

- **`HTTP is 'stateless' : HTTP 는 state 를 관리하지 않는다.`**

  - 서버가 과거의 클라이언트 요청에 대한 것을 기록해놓지 않는다.

---

## ⭐️ REST API (REpresentational State Transfer)

- API 또는 애플리케이션 프로그래밍 인터페이스는 **애플리케이션이나 디바이스가 서로 간에 연결하여 통신할 수 있는 방법을 정의하는 규칙 세트**이다.

- `REST API 는 REST (REpresentational State Transfer) 아키텍처 스타일의 디자인 원칙을 준수하는 API` 이다.

  - 이러한 이유 때문에 REST API 를 종종 RESTful API 라고도 한다.

- 가장 기본 레벨에서, `API` 는 **하나의 애플리케이션이나 서비스가 다른 애플리케이션이나 서비스 내의 `리소스에 액세스`할 수 있도록 해주는 메커니즘**이다.

  - **액세스를 수행하는 애플리케이션이나 서비스**를 `클라이언트`라고 하며, **리소스가 포함된 애플리케이션이나 서비스**를 `서버`라고 합니다.

- REST API 는 거의 모든 프로그래밍 언어를 사용하여 개발이 가능하며, 다양한 데이터 포맷을 지원할 수 있다.

- 유일한 요구사항은 이들이 아키텍처 제한사항으로도 알려진 다음의 6가지 REST 디자인 원칙에 맞아야 한다는 것이다.

**_1. Uniform (균일한 인터페이스)_**

- 요청이 어디에서 오는지와 무관하게, 동일한 리소스에 대한 모든 API 요청은 동일하게 보여야 합니다.

- REST API 는 사용자의 이름이나 이메일 주소 등의 동일한 데이터 조각이 오직 하나의 URI (Uniform Resource Identifier)에 속함을 보장해야 합니다.

  - URI로 지정한 리소스에 대한 조작을 통일되고 한정적인 인터페이스로 수행하는 아키텍처 스타일을 말한다.

- 리소스가 너무 클 필요는 없지만, 이는 클라이언트가 필요로 하는 모든 정보를 포함해야 합니다.

**_2. Client - Server 구조_**

- 클라이언트와 서버 애플리케이션은 서로 간에 완전히 독립적이어야 한다.

  - 클라이언트와 서버에서 개발해야 할 내용이 명확해지고 서로간 의존성이 줄어들게 됩니다.

- 유일한 정보는 요청된 리소스의 URI이며, 이는 다른 방법으로 서버 애플리케이션과 상호작용할 수 없습니다.

**_3. 캐싱 가능성_**

- REST 의 가장 큰 특징 중 하나는 HTTP라는 기존 웹표준을 그대로 사용하기 때문에, 웹에서 사용하는 기존 인프라를 그대로 활용이 가능하다는 것이다.

  - 따라서 HTTP 가 가진 `캐싱 기능`이 적용 가능합니다.

  - HTTP 프로토콜 표준에서 사용하는 Last-Modified 태그나 E-Tag 를 이용하면 캐싱 구현이 가능하다.

**_4. Self-descriptiveness (자체 표현 구조)_**

- REST 의 또 다른 큰 특징 중 하나는 REST API 메시지만 보고도 이를 쉽게 이해 할 수 있는 자체 표현 구조로 되어 있다는 것이다.

**_5. 계층 구조 아키텍처_**

- REST 서버는 다중 계층으로 구성될 수 있으며 보안, 로드 밸런싱, 암호화 계층을 추가해 구조상의 유연성을 둘 수 있고 PROXY, 게이트웨이 같은 네트워크 기반의 중간매체를 사용할 수 있게 한다.

**_6. Stateless (무상태성)_**

- REST 는 `무상태성 성격`을 갖습니다. 다시 말해 **작업을 위한 상태정보를 따로 저장하고 관리하지 않는다.**

- 세션 정보나 쿠키정보를 별도로 저장하고 관리하지 않기 때문에 API 서버는 들어오는 요청만을 단순히 처리하면 된다.

  - 때문에 서비스의 자유도가 높아지고 서버에서 불필요한 정보를 관리하지 않음으로써 구현이 단순해진다.

---

### REST API 동작 방식 / 디자인 가이드

<img width="648" alt="스크린샷 2023-03-29 오후 8 34 31" src="https://user-images.githubusercontent.com/87372606/228523169-6413bc8a-4105-45f3-bc05-424aeea29905.png">

- `REST API` 는 `HTTP 요청을 통해 통신`함으로써 리소스 내에서 레코드(`CRUD` 라고도 함)의 작성, 읽기, 업데이트 및 삭제 등의 표준 데이터베이스 기능을 수행한다.

  - 예를 들어, REST API는 GET 요청을 사용하여 레코드를 검색하고,

  - POST 요청을 사용하여 레코드를 작성하며,

  - PUT 요청을 사용하여 레코드를 업데이트하고,

  - DELETE 요청을 사용하여 레코드를 삭제한다.

  - 모든 HTTP 메소드는 API 호출에서 사용될 수 있다.

- REST API 설계 시 가장 중요한 항목 2 가지

**_1. `URI 는 정보의 자원을 표현해야 한다.`_**

**_2. `자원에 대한 행위는 HTTP Method (GET, POST, PUT, DELETE) 로 표현한다.`_**

---

### Reference

- Top Down Approach Computer Network

- [REST API 제대로 알고 사용하기](https://meetup.nhncloud.com/posts/92)

- [REST API 란?](https://www.ibm.com/kr-ko/cloud/learn/rest-apis)

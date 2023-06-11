# ConfigMap 실습

- [ConfigMap](#configmap)

- [3 가지 방법](#configmap-배포-3-가지-방식)

  - [ConfigMap Whole data](#configmap-whole-data)

  - [ConfigMap Part of data](#configmap-part-of-data)

  - [ConfigMap Volume Mount](#configmap-volume-mount)

- [ConfigMap 이 수정될 때](#configmap-이-수정될-때)

---

## ConfigMap

<img width="350" alt="스크린샷 2023-06-11 오후 6 04 27" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/c32b581a-0490-4edb-b872-66ea35c4d3fc">

- ConfigMap 안에는 다양한 변수들이 저장되어있다.

- 환경변수를 저장해두는 이유는 컨테이너들이 관리하는 정보를 변경하고자할 때 Pod 안에 일일이 접근해서 변경해줘야하는데, 컨테이너가 많을 때는 관리하기가 복잡하다.

- **Pod 가 직접 관리하지않고 컨테이너들이 관리하는 구성파일이나 환경변수들을 한 군데 모아 관리하도록 하는 것이 `ConfigMap`** 이다.

- `ConfigMap` 은 **_환경변수 또는 구성 파일들을 모아둔 것이기 때문에 환경에 맞게 구성할 수 있고 목적에 맞게 Pod 에 적용할 수 있다._**

---

## ConfigMap 배포 3 가지 방식

- 전체를 기입하는 방법

- 일부 data 만 기입하는 방법

- Volume Mount

---

### ConfigMap Whole data

<img width="587" alt="스크린샷 2023-06-11 오후 6 07 27" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/5384d3b4-3b2d-422a-a686-25bb0fd9a62b">

- `전체를 기입하는 방법`

- configmap yaml 에는 `data` 라는 필드가 있는데 이 영역에는 환경변수로 설정한 key, value 값이 들어가있다.

- deployment yaml 은 양식 그대로지만, container spec 안에 `envFrom` 이라는 필드가 있고 configMap 을 만들 때 어떤 것을 참조할 것인지를 명세해준다.

<img width="540" alt="스크린샷 2023-06-11 오후 6 11 49" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/5f7c25fd-8042-4dec-bcb4-f69408495015">

- 둘 다 apply 하고 pod 안으로 들어가서 설정이 잘 기입됐는지 확인해본다.

```
kubectl exec -it [pod name] bash
echo $MYSQL_ROOT_PASSWORD
echo $MYSQL_DATABASE
```

---

### ConfigMap Part of data

<img width="597" alt="스크린샷 2023-06-11 오후 6 13 19" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/4dd94df1-6ec2-4b06-b498-9ce28c94d3ab">

- `원하는 key 에 대한 value 만 기입하는 방법`

<img width="616" alt="스크린샷 2023-06-11 오후 6 14 42" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/9e5d9c6e-e320-4b0e-a4b3-3ed1c8a9cf19">

---

### ConfigMap Volume Mount

<img width="642" alt="스크린샷 2023-06-11 오후 6 17 24" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/fd32452d-ec38-4e0f-99c5-a2676987b873">

- envFrom 아래 configMapRef 필드가 있다. (전체를 참조할 때 사용했었음)

- 그 아래 `volumeMounts 필드`가 있는데 여기에 `mountPath` 로 경로를 설정한다.

  - mu-test 라는 컨테이너는 volumeMount 옵션을 통해 mointPath 로 설정된 directory 에 mu-config 라는 volume 을 mount 하라는 뜻

  - spec 아래 컨테이너와 동일한 level 의 볼륨이 정의된다. (줄이 맞춰짐)

    - 이 volume 은 container 에 바로 mount 될 수 있다는 것을 의미

<img width="583" alt="스크린샷 2023-06-11 오후 6 22 05" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/409b12e5-9d5b-43df-be8b-99d8bf877800">

- configMap 의 data 키 아래의 환경변수들이 파일 형태로 mountPath 에 저장 된다.

  - key 가 파일 이름이 되고 value 가 data 가 된다.

<img width="445" alt="스크린샷 2023-06-11 오후 6 23 01" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/197ff3c4-5ae4-4383-9e1a-14a6b4e0b539">

---

## ConfigMap 이 수정될 때

- configMap 에 있는 정보를 수정할 경우 container 안에 있는 정보들이 어떻게 변할까..?

- 변경한대로 정상적으로 출력이된다.

<img width="546" alt="스크린샷 2023-06-11 오후 6 26 24" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/078b7deb-f8f8-4b9f-bb91-735f0464a5e2">

---

- Pod 가 직접 관리하지 않고 container 들이 관리하는 구성파일이나 환경변수들을 ConfigMap 이라는 리소스 안에 한군데 모아서 관리가 되는 것을 실습해봤다.

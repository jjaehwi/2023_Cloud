# Secret 실습

- [Secret](#secret)

  - [Creating secret](#creating-secret)

- [Using secret (env1)](#using-secret-env1)

- [Using secret (env2)](#using-secret-env2)

- [Using secret (volume)](#using-secret-volume)

---

## Secret

- `Secret` 은 **ConfigMap 과 비슷한데, 민감한 정보를 Application 에 전달하기 위해 만들어졌다.**

<img width="546" alt="스크린샷 2023-06-11 오후 7 00 16" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/6600ca92-3433-4630-8533-34a45c51ed10">

- etcd 에 secret 을 저장하고, 저장된 secret 을 Pod 에 Volume mount 또는 환경변수로 전달할 것이다.

---

### Creating secret

- configMap 이나 secret 같은 경우는 cli 명령어로 만들어주는 것이 더 간편하다.

```
kubectl create secret [kind] [name] --from-literal=[key]=[value] --from-file=[file name]
```

- `kind`

  - **generic (Opaque)**: user defined secret (일반 사용자 정의 타입)

  - **docker-registry**: private docker registry (docker 에서 private registry 에서 image 를 pull 할 때 필요한 정보를 보내는 것)

  - **tls** : tls information (tls 로 암호화된 통신을 하고자 할 때)

```
kubectl create secret generic test-secret --from-literal=HELLO=WORLD
```

<img width="546" alt="스크린샷 2023-06-11 오후 7 05 26" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/85f59299-15da-4c1a-bd02-0f1682537335">

- 이렇게 **생성된 yaml 파일을 확인**해보면 data 가 `base64 로 인코딩`된 정보로 되어있다.

---

```
kubectl create secret generic test-secret --from-file=test.txt
```

<img width="551" alt="스크린샷 2023-06-11 오후 7 06 44" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/0dafe63a-83b6-4868-95b5-d2311225d5ef">

- **_파일로 만드는 경우 파일 이름이 key 가 되고 파일의 내용이 value 가 된다._**

---

- 실습

<img width="771" alt="스크린샷 2023-06-11 오후 7 09 14" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/ebd6e495-15d7-466f-a3dc-f352f4d83abf">

---

## Using secret (env1)

- 생성한 secret 을 환경변수로 Pod 에 전달하는 실습 (`whole data`)

<img width="566" alt="스크린샷 2023-06-11 오후 7 11 42" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/540dddf9-d111-453d-830a-337fe22fb0c2">

```
// 그림의 yaml 처럼 만드는 cli 명령
kubectl create secret generic db-secret --from-literal=MYSQL_DATABASE=test --from-literal=MYSQL_ROOT_PASSWORD=jangwon --dry-run=client -o yaml > db-secret.yaml
```

- --dry-run : etcd 에 저장한 것 처럼 보이게 함 (실제 배포 x)

- -o yaml : yaml 로 보여줘라

- \> db-secret.yaml : db-secret 에 redirection 할 것

- `envFrom:` 필드에 `secretRef` 로 **방금 만든 secret 을 환경변수로 가져온다.**

- apply 후 확인해본다.

- mysql 에 적용되는 모습 확인

<img width="673" alt="스크린샷 2023-06-11 오후 7 18 29" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/a4615a09-2988-49f7-afff-4fa058b750db">

---

## Using secret (env2)

- 생성한 secret 을 환경변수로 Pod 에 전달하는 실습 (`part of data`)

  - **_원하는 것을 선택하여 보내는 것_**

<img width="579" alt="스크린샷 2023-06-11 오후 7 12 03" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/da5393fc-96d0-47de-b35b-beaf3f802cc5">

- apply 후 확인해보기

```
kubectl exec -it [Pod name] -- bash
printenv | grep MYSQL
```

<img width="697" alt="스크린샷 2023-06-11 오후 7 22 26" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/41745182-95e9-49aa-920a-8512681197d6">

- mysql 에 적용되는 모습 확인

  - 아까랑 달리 test 가 만들어지지 않은 것을 확인할 수 있다. (DATABASE 는 적용안했으니까)

---

## Using secret (volume)

- 생성한 secret 을 volume mount 를 이용해 Pod 에 전달하는 실습

<img width="487" alt="스크린샷 2023-06-11 오후 7 23 37" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/57895f69-edb4-4569-aecd-f96353a52ede">

<img width="522" alt="스크린샷 2023-06-11 오후 7 23 58" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/dc5f85d8-6602-45ad-8d94-e21e974a20c4">

- `envFrom:` 은 **_db 의 경우 ROOT_PASSWORD 가 필요해서 넣어둔 것._** (db 말곤 필요없음)

- `volumeMounts` 필드로 **어디에 volume 을 붙일지** 적어준다.

  - name 필드에는 생성한 볼륨의 이름을 적는다.

- `volumes` 필드에 생성한 `secret 필드` 중 `secretName` 에 생성한 secret 의 이름을 적는다.

  - volumes 의 name 은 volume 의 이름

<img width="782" alt="스크린샷 2023-06-11 오후 7 28 15" src="https://github.com/jjaehwi/2023_Cloud/assets/87372606/e16597b6-f259-4e3d-9986-81b68540f0cf">

- mountPath 를 tmp 의 secret 으로 했기 때문에 tmp 로 가서 secret 에 접근하면 키 이름을 가진 파일 두 개가 생성된 것을 확인할 수 있다.

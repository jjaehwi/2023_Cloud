# Emptydir, Hostpath, PV/PVC/StorageClass

- [Emptydir](#emptydir)

- [hostPath](#hostpath)

- [시나리오](#시나리오)

  - [PersistentVolume](#persistentvolume)

  - [StorageClass](#storageclass)

  - [PersistentVolumeClaim](#persistentvolumeclaim)

  - [Pod 와 연결하여 실습](#pod-와-연결)

---

## Emptydir

- **`Emptydir` 은 Pod 의 컨테이너 사이에서 같은 `임시 스토리지`를 공유할 때 사용하는 저장소**

```
// emptyDir.yaml
apiVersion: v1
kind: Pod
metadata:
    name: test-pod
sepc:
    containers:
        - name: container1
          image: kubetm/init
          volumeMounts:
            - name: test-dir
              mountPath: /test1
        - name: container2
          image: kubetm/init
          volumeMounts:
            - name: test-dir
              mountPath: /test2
        volumes:
            - name: test-dir
              emptyDir: {}
```

- 컨테이너 이름 : container1

  - container1 은 volumeMounts 의 name 으로 보면 test-dir 이라는 volume 을 갖고 있다.

  - test-dir 이라는 volume 을 컨테이너 내의 test1 이라는 곳에 mount 시켰다.

- 컨테이너 이름 : container2

  - container2 은 volumeMounts 의 name 으로 보면 test-dir 이라는 volume 을 갖고 있다.

  - test-dir 이라는 volume 을 컨테이너 내의 test2 이라는 곳에 mount 시켰다.

<img width="438" alt="스크린샷 2023-05-01 오전 1 17 34" src="https://user-images.githubusercontent.com/87372606/235364092-ff9f1394-c1de-4721-8b2a-eafe85b23deb.png">

```
// container1 에 접근
kubectl exec -it test-pod container1 -- bash

// ls 명령어 실행 후 test1 directory 에 접근 (mount 된 것)
ls
cd test1

// 아무 파일이나 생성
touch hello.txt

// 탈출 후 container2 에 접근
exit
kubectl exec -it test-pod container2 -- bash

// test2 directory 에 접근해서 ls 명령어 확인 (mount 된 것)
cd test2
ls
```

- apply 후 컨테이너에 접근해서 ls 명령어로 디렉토리를 확인해본다.

- 같은 Pod 내 두 컨테이너가 임시 emptyDir 로 인해 파일을 공유하는 것을 확인할 수 있다.

- delete 로 생성한 Pod 를 삭제하게 되면 Pod 가 삭제됨가 동시에 임시였던 emptyDir 도 사라지게된다.

---

## hostPath

- **`hostPath` 는 워커노드에 붙는 저장소**이다. (`Pod 의 생명주기를 따라가지 않는다.`)

  - 그러나 노드가 죽을 경우 문제가 생긴다.

```
// hostPath.yaml
apiVersion: v1
kind: Pod
metadata:
    name: test-pod
spec:
    nodeSelector:
        kubernetes.io/hostname: grad3-worker-1
    containers:
        - name: container
          image: kubetm/init
          volumeMounts:
            - name: test-volume
              mountPath: /test1
    volumes:
        - name: test-volume
          hostPath:
            path: /test-volume
            type: DirectoryOrCreate
```

- test-pod 라는 이름의 Pod 를 생성한다.

  - `nodeSelector` 는 Pod 를 kubernetes.io/hostname 라는 label 에 grad3-worker-1 라는 곳에 생성하겠다고 설정한다.

  - 똑같이 container 를 생성 후

    - test-volume 이라는 이름의 **volume 을 /test1 에 mount**

  - **test-volume 이라는 이름의 volume 을 생성** 후

    - path 는 /test-volume

    - `type` 을 설정 (DirectoryOrCreate = test-volume 이라는 디렉토리가 없다면 생성한다.)

<img width="384" alt="스크린샷 2023-05-01 오전 1 30 49" src="https://user-images.githubusercontent.com/87372606/235364687-5760d94f-7f34-49f3-abf0-09b4d24ccc1a.png">

- **grad3-worker-1 에 volume 이 생성되고 container1 의 /test1 에 mount 된다.**

```
// node 의 label 을 확인
kubectl get node --show-labels
```

<img width="674" alt="스크린샷 2023-05-01 오전 1 32 54" src="https://user-images.githubusercontent.com/87372606/235364778-59c54e65-e7d2-4c51-a920-6da06724d1a5.png">

```
// Pod 에 접근해서 test1 이라는 디렉토리 가 생성되었는지 확인하고, 파일을 생성한다.
kubectl exec -it test-pod -- bash
ls
cd test1
touch hi.txt
```

- grad3-worker-1 에 /test-volume 이 생성되어있는 상태에서 컨테이너에 접근해서 /test1 에 hi.txt 를 만든 상황이다.

  - grad3-worker-1 로 넘어가서 /test-volume 에도 정상적으로 hi.txt 가 생성되었는지 확인해본다.

```
ssh root@grad3-worker-1
// 패스워드 입력
```

<img width="576" alt="스크린샷 2023-05-01 오전 1 38 39" src="https://user-images.githubusercontent.com/87372606/235365059-c47ddb5e-c1bc-4ddf-baf4-18080f35b8f3.png">

```
// hostPath2.yaml
apiVersion: v1
kind: Pod
metadata:
    name: test-pod2
spec:
    nodeSelector:
        kubernetes.io/hostname: grad3-worker-2
    containers:
        - name: container2
          image: kubetm/init
          volumeMounts:
            - name: test-volume
              mountPath: /test2
    volumes:
        - name: test-volume
          hostPath:
            path: /test-volume
            type: DirectoryOrCreate
```

- grad3-worker-2 로 지정한 Pod 를 apply 한다.

  - 이 Pod (test-pod2) 로 접근했을 때, mount 된 곳에 들어갔을 때, test-pod 에서 만든 파일이 존재하는지 확인해본다.

  - 없는 것을 확인할 수 있다.

```
// test-pod2 에 접근
kubectl exec -it test-pod2 -- bash

// mount 된 디렉토리에 접근 후 파일 확인
cd test2
ls
```

<img width="237" alt="스크린샷 2023-05-01 오전 1 43 04" src="https://user-images.githubusercontent.com/87372606/235365278-bcab4a04-6d1d-4813-ae33-5e5ab4e9a902.png">

- worker-1 에 Volume 은 hostPath 로 생성됐는데, worker-2 에서 생성된 Pod 에서는 worker-1 에서만 효력이 있는 Volume 에 연결될 수 없다.

---

## 시나리오

<img width="558" alt="스크린샷 2023-05-01 오전 2 05 27" src="https://user-images.githubusercontent.com/87372606/235366397-2cdbecba-3c5e-45a3-97f0-6f1516166164.png">

- local pv 로 실습해볼 것. (Persistance Volume 을 apply)

- pvc 와 storageClass 까지 apply 해서 Pod 가 사라져도 파일이 남아있는지 확인한다.

---

### PersistentVolume

<img width="558" alt="스크린샷 2023-05-01 오전 2 07 55" src="https://user-images.githubusercontent.com/87372606/235366491-97406e6c-844d-4942-99e2-59cad654f2c0.png">

```
// pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: test-pv
spec:
    capacity:
        storage: 5Gi
    accessModes:
        - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    storageClassName: local-storage
    local:
        path: /tmp
    nodeAffinity:
        required:
            nodeSelectorTerms:
                - matchExpressions:
                    - key: kubernetes.io/hostname
                      operator: In
                      values:
                        - grad3-worker-1
```

- test-pv 라는 이름의 `PersistentVolume` 을 생성

  - storage 의 `용량을 설정`한다. (spec: capacity: storage: 5Gi)

  - Pod 하나만 Read/Write 할 수 있다.(`ReadWriteOnce`)

  - `persistentVolumeReclaimPolicy` : pvc 와 pv 가 bound 될 때, pvc 를 지워도 pv 안에 있는 파일의 내용을 유지시킬지 말지에 대해 정하는 필드

    - retain : 유지

  - `storageClassName` : 나중에 정의할 storageClass 이름을 정의

  - local 의 root/tmp 에 pv 를 `mount` 시킨다.

  - `nodeAffinity` : nodeSelector 와 비슷한 역할을 하는데 더 강력함

    - label 역할을 하는 key: kubernetes.io/hostname 를 적고

    - values: - grad3-worker-1 를 적음으로써

    - grad3-worker-1 에 pv 를 생성하겠다는 얘기가 된다.

<img width="558" alt="스크린샷 2023-05-01 오전 2 15 36" src="https://user-images.githubusercontent.com/87372606/235366835-1f59919d-bb61-462e-a84a-e5115d16d03a.png">

---

### StorageClass

<img width="532" alt="스크린샷 2023-05-01 오전 2 17 26" src="https://user-images.githubusercontent.com/87372606/235366936-69e27d0a-915c-4c2f-babd-11b1938b2261.png">

- `StorageClass` 는 사람이 수동으로 provisioning 하는 것이 아니라 자동으로 Volume 을 생성하고 할당하는 (provisioning) 해주는 오브젝트

```
// storageclass.yaml (local pv 상황)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
    name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

- pv 에서 봤던 **storageClassName 으로 metadata: name: 을 설정**

- `volumeBindingMode` : **pvc 를 사용하는 Pod 가 진짜 생성되기 전까지는 pv 에 binding 이랑 provisioning 을 기다린다.** (동적 provisioning 이 안돼서 추가한 것)

<img width="661" alt="스크린샷 2023-05-01 오전 2 20 54" src="https://user-images.githubusercontent.com/87372606/235367055-0a51ca71-086b-4f62-82ce-3dddd710d5ec.png">

---

### PersistentVolumeClaim

<img width="509" alt="스크린샷 2023-05-01 오전 2 22 36" src="https://user-images.githubusercontent.com/87372606/235367128-05025c90-aca1-418d-bca1-13d4e70c9a74.png">

```
// pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: test-pvc
spec:
    storageClassName: local-storage
    volumeName: test-pv
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
            storage: 3Gi
```

- test-pvc 라는 이름의 PersistentVolumeClaim 을 생성

  - `storageClassName` 을 local-storage 로 설정

  - `volumeName` : pv 의 name

  - 요구사항을 적어서 (Claim) 보낸다.

  - STATUS : Bound 면 잘 생성된 것

<img width="490" alt="스크린샷 2023-05-01 오전 2 25 22" src="https://user-images.githubusercontent.com/87372606/235367231-c4526f2c-e8f5-4559-8b73-e90856bdac22.png">

---

### Pod 와 연결

- **PersistentVolume, StorageClass, PersistentVolumeClaim 을 생성한 후** `Pod 에 Claim 에 대한 정보를 넣어야한다.`

```
// test.yaml
apiVersion: v1
kind: Pod
metadata:
    name: test-pod3
spec:
    containers:
        - name: container3
          image: kubetm/init
          volumeMounts:
            - name: test-pvc
              mountPath: /test3
    volumes:
        - name: test-pvc
          persistentVolumeClaim:
            claimName: test-pvc
```

- test-pod3 라는 이름의 Pod 생성

  - test-pvc 라는 이름의 Volume 생성

    - `persistentVolumeClaim` : 의 claimName 을 이전에 만든 pvc 의 이름을 적는다. (test-pvc)

    - volumes 의 name 과 claimName 은 달라도 된다.

  - **volumeMounts 할 때, 이름은 만든 Volume 의 이름이 되어야한다. (volumes: - name: test-pvc)**

    - mount 를 container3 의 test3 에 한다.

- **pv.yaml 에서 nodeAffinity 를 kubernetes.io/hostname: grad3-worker-1** 로 설정했다.

  - pv 는 `grad3-worker-1 에 /tmp 에 mount` 되어 있다.

  - `이 경로가 container3 에 mount 된 경로` (/test3) 이다.

```
// grad3-worker-1 에서 /tmp 에 mount 된 것 확인
ssh root@grad3-worker-1
// 패스워드 입력
ls /tmp

// test-pod3 에 접근 후 test3 에 접근하여 내용 확인
kubectl exec -it test-pod3 -- bash
cd test3
ls
```

<img width="445" alt="스크린샷 2023-05-01 오전 2 36 33" src="https://user-images.githubusercontent.com/87372606/235367732-487a58a3-ff7f-44a1-b380-ce300f9b8b43.png">

<img width="616" alt="스크린샷 2023-05-01 오전 2 36 15" src="https://user-images.githubusercontent.com/87372606/235367717-cafe807d-fb20-4e85-9ff4-7baae3af7346.png">

- `worker-1 의 /tmp 의 내용과 test-pod3 의 /test3 의 내용이 같은 것을 확인`할 수 있다.

<img width="518" alt="스크린샷 2023-05-01 오전 2 40 45" src="https://user-images.githubusercontent.com/87372606/235367958-6e121ad7-77c6-4af2-b84c-a0b6d55e2653.png">

<img width="518" alt="스크린샷 2023-05-01 오전 2 40 57" src="https://user-images.githubusercontent.com/87372606/235367949-b75b2f45-edc9-401b-b1d7-c192037c8e2b.png">

1. `test-pod3 의 /test3 에서 파일을 생성하면 local pv 가 mount 된 /tmp 에서도 똑같이 생성된 것을 확인할 수 있다.` (**같은 storage 에 접근하고 있는 것**)

2. **Pod 를 delete 해도 생성한 파일이 남아있는지 확인**해본다. --> 잘 남아있음을 확인할 수 있다.

3. **또 다시 Pod 를 만들었을 때, 이전에 생성한 파일이 잘 연결될지 확인**해본다. --> 재생성해도 공유가 잘 되는 것을 확인할 수 있다.

```
kubectl apply -f test.yaml
kubectl exec -it test-pod3 -- ls/test3
```

4. **/tmp 에서 파일을 삭제했을 때, test-pod3 의 test3 에서도 파일이 삭제되는 것을 확인**할 수 있다.

```
// grad3-worker-1 에서 명령어 실행
rm /tmp/파일명
// test-pod3 에서 명령어 실행
kubectl exec -it test-pod3 -- ls/test3
```

---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
title: k8s-스토리지
date: 2023-05-22T14:21:21
lastmod: 2023-06-05T14:35:13
---
## 볼륨

- 각 각의 컨테이너끼리 파일시스템 유지되는 공간이 필요함
- 윈도우 밀어도 USB 내용 없어지지 않듯이 따로 파일 시스템이 유지되는 곳을 k8s에서 제공한다
- k8s에서 컨테이너볼륨(디스크)는 POD 컴포넌트로 포드 스펙이 의해 정의
- 볼륨은 k8s의 독립적인 오브젝트가 아며 스스로 생성, 삭제 불가 
- 각 컨테이너의 파일 시스템의 볼륨을 마운트하여 생성




## 볼륨의 종류 

| 임시볼륨 | 로컬볼륨 | 네트워크볼륨  | 네트워크볼륨(클라우드종속적) |     |
| -------- | -------- | ------------- | ---------------------------- | --- |
| emptyDir | hostpath | NFS, cephFS.. | gcePErsistentDisk,awsEBS..                             |     |

- 임시볼륨(emptyDir) : 가장 간단한 볼륨. 임시로 있는 볼륨 컨테이너 파괴시 이것도 파괴됨. 컨테이너끼리 데이터 공유할수 있도록 만들어짐
- 로컬볼륨(hostPath) : 로컬 = 노드, 포드를 노드에 배치시 실행가능한 노드에서 노드와 파드가 파일 시스템 공유함. 단점으로는 노드가 달라지면 데이터가 달라진다. yaml 작성시에 특정포드는 특정 노드의 호스트 패쓰에 저장되도록 한다. 1번 포드가 첫번째 노드에서 포드가 저장해지고 이 포드 사라지고 2번 포드에 생성되면 데이터가 바껴진다. 데이터 유지보단 노드끼리 공유하기 위해 사용된다. 스케일링시 자주 쓰이지 않는다. 
- 네트워크볼륨: 클러스터 외부에 있는 자원과 데이터 공유(NFS, cephFS)
- 네트워크 볼륨(클라우드) : 클러스터 외부에 있는 데이터. NFS, iSCI  이중에서 클라우드 종속적으로 완전히 외부와 떨어짐 gcePersistentDisk, awsEBS, azureFile등이 있음
![](https://i.imgur.com/mvqydVg.png)

- 그외에 PVC : 사용자가 특정 클라우드 환경의 세부 사항을 모른채 GCE persistent Disk 또는 iSCI볼륨과 같은 내구성 스토리지를 claim 할 수 있는 방법
- configMap, Secret ... 

## 1. 임시볼륨

emptyDir

컨테이너 3개 올라와있는 POD하나
- webserver : 이 로그 파악할 수 있는 기능이 없음. 이 웹서버는 컨텐츠 여기서 꺼내, 로그는 누가 접속했는지 기록남김. 컨텐츠 전달
- agent : 서비스가 원활하게 데이터 제공할수있게 컨텐츠 만들어좀. 컨텐츠 생산
- logRotator : 

컨텐츠 에이전트에 있는 컨텐츠를 웹서버와 공유하기위한 empty dir이 필요, 마찬가지로 로그를 공유하는 empty Dir 공유하는것이 필요함 

![](https://i.imgur.com/SiqOpB1.png)


### 임시 볼륨 실습 

- contents generator 생성
 

|                                      |     |
| ------------------------------------ | --- |
| ![](https://i.imgur.com/aRue7Bg.png) |     ![](https://i.imgur.com/EABRxFp.png)|

두 개의 컨테이너가 
- webserver :  사용자가 요청 포트 듣고 있다 응답 돌려줌 htdocs는 html 같은 파일 들어있음
- count : 컨텐츠를 html로 생성함 이렇게 되면 webserver가 count 컨테이너가 만들 컨텐츠를 참조할 수가 없는데 이때 필요한게 `emptyDir`이다

```
# count-httpd.yaml
apiVersion: v1
kind: Pod
metadata:
  name: count
spec:
  containers:
  - image: gasbugs/count
    name: html-generator
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: httpd
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/local/apache2/htdocs
      readOnly: true # volumeMounts가 작성가능
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html
    emptyDir: {}
```

```
k run http-go --image=gasbugs/http-go
k exec -it http-go -- curl 10.0.2.132 (방금생성한 count pod IP )
```



## 2. 로컬볼륨(hostPath)

- 노드 파일 시스템에 있는 특정 파일 , 디렉토리 지정
- 영구 스트리지
- 컨테이너와 노드간의 데이터 공유
- 다른 노드의 포드끼리 데이터 공유 안됨
- 보통 노드에 있는 데이터를 포드에 주고 싶을 때 사용 
- 모니터링, 노드의 로그 정도 수집할 때 자주 사용(프로메테우스등)

## 2. GCE 영구 스토리지 사용

- 데이터 지속
- 모든 포드 공유
- mongo 컨테이너에 마운트해 사용

|     |     |
| --- | --- |
|![](https://i.imgur.com/XvVS4Y3.png)     |   ![](https://i.imgur.com/oUh39ME.png)  |

```
# gce 영구 디스크를 동일한 리전에 생성 (gcp 콘솔 > 컴퓨터엔진 > 디스크)
gcloud compute disks create --size=10GiB --zone=us-west4-b mongodb
# 
```

 - `spec.volumes` 에는 mongodb-data 선언하고 gce 디스크를 사용 정의
 - `spec.volumes.gcePersistentDisk.pdName`  의 명칭은 생성한 디스크 이름과 동일해야함
 - `spec.containers.volumeMounts` 포드의 어느디렉토리를 마운트 할지 선택
```yaml
# gce-mongodb-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - mountPath: /data/db #mongodb dir
      name: mongodb
  volumes:
  - name: mongodb
    # This GCE PD must already exist.
    gcePersistentDisk:
      pdName: mongodb
      fsType: ext4


```

```
$ k exec -it mongodb -- mongo
$ (mongodb) use mystore;
$ (mongodb) db.foo.insert({name:'test',val:'1234'})
$ (mongodb) db.find()
$ k delete pod mongodb 
$ k apply -f gce-mongdb-pod.yaml
$ k exec -it mongodb -- mongo
$ (mongodb) use mystore;
$ (mongodb) db.foo.find()
```


## 4. NFS 네트워크 볼륨 설치하기 실습

기존에 사용하고 있던 GCE 인스턴스 3대 (master, worker1, worker2)

| master     | worker1    | worker2     |
| ---------- | ---------- | ----------- |
| 10.182.0.6 | 10.182.0.9 | 10.182.0.10 |

1. worker2에 nfs 서버 설치하고 `/etc/exports` 수정
```
$apt-get update
$ apt-get install nfs-common nfs-kernel-server portmap
$ mkdir /home/nfs
$ chmod 777 /home/nfs 
$ echo "/home/nfs 10.182.0.6(rw,sync,no_subtree_check) 10.182.0.9(rw,sync,no_subtree_check) 10.182.0.10(rw,sync,no_subtree_check)" >> /etc/exports
$ service nfs-server restart
$ showount -e 127.0.0.1
```

2. master 인스턴스와 worker1 인스턴스에 NFS 라이브러리 설치후 마운트
```
$ apt-get update
$ apt-get install nfs-common nfs-kernel-server portmap 
$ mount -t nfs 10.182.0.10:/home/nfs /mnt 
```

3. 마운트 확인

```
$ (worker2인스턴스에서) echo 'nfs' > /home/nfs/index.html
$ (worker1) cat /mnt/index.html
```

4. 마스터에서 네트워크 스토리지 갖고 있는 팟 실행
```
# nfs-http.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-httpd
spec:
  containers:
  - image: httpd
    name: web
    volumeMounts:
    - mountPath: /usr/local/apache2/htdocs
      name: nfs-volume
      readOnly: true
  volumes:
  - name: nfs-volume
    nfs:
      server: 10.182.0.10
      path: /home/nfs
```


```
$ (마스터) k apply -f nfs-httpd.yaml
$ (마스터) k port-forward nfs-httpd 1234:80
# nfs 반환
$ (마스터) curl 127.0.0.1:1234 
```

>[!qquestion] 
> 그런데 worker1이나  worker2에서 `curl 127.0.0.1:1234`  하면 connection failed 뜨는데 왜 그런거지?



## 3. PV, PVC 

- POD 개발자가 클러스터에 스토리지 사용할 때 인프라를 알아야할까?
	-  ❌ 개발자들이 네트워크 디스크 만들려고 하는건 devops 철학에 어긋남 개발자는 개발만하자는게 k8s철학인데!?
	- ❌ 개발자들이 네트워크 스토리지를 알아야함 
	- ⭕ 어플리케이션 배포하는 개발자가 스토리지 기술 종류 몰라도 배포가능한게 이상적. 
- ❗ 따라서  개발자와 관리자의 역할을 나누어 추상화 해주는게 pv와 pvc역할이라고 보면된다

### PV(persistentVolume), PVC(PersistentVolumeClaim)

|                                      |     |
| ------------------------------------ | --- |
| ![](https://i.imgur.com/e0WZ9jv.png) | ![](https://i.imgur.com/O6HxZ5e.png)    |
|                                      |     |

- PVC가 없었을 때는 개발자가 GCP mongodb디스크를 알았어야한다


|                                      |                                      |
| ------------------------------------ | ------------------------------------ |
| ![](https://i.imgur.com/CvuyjbA.png) | ![](https://i.imgur.com/zjBJ7Wh.png) |

- 필요한것 pV, PVC, pod 이떄 pv와 PVC는 이름이 아니라 내용으로 매핑된다
- `spec.AaccessModes` (보통 readonlymany를 주는편)
	- readwriteonce - 하나의 노드에서만 읽고쓰기
	- readonlymane - 여러개의노드에서 리드온리가능
	- readwritemany.- 여러개의 노드에서 읽고쓰기
- `spec.persistentVolumeReclaimPolicy`
	- retain : PVC 삭제시 여전히 PV 존재(해제상태)
	- delete : 외부 인프라 연관된 스토리지 자산 모두 제거
	- recycle : 새 클레임에 대해 사시사용할 수 있도록함(`rm -rf /thevolume` 

```
k get pv, pvc
```

![](https://i.imgur.com/0WA7dGL.png)


### PV, PVC 실습

- 필요한것 pV, PVC, pod 이떄 pv와 PVC는 이름이 아니라 내용으로 매핑된다
- 일단 `PVC.spec.storageClassName.` 은 비워두기!!
```yaml
# mongo-pv-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - mountPath: /data/db
      name: mongodb
  volumes:
  - name: mongodb
    persistentVolumeClaim:
      claimName: mongo-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: ""
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  gcePersistentDisk:
    pdName: mongodb
    fsType: ext4
```


```
k apply -f mongo-pv-pvc.yaml
k exec -it mongo -- mongo
# 방금전 예제에서 등록한 값이 나오는 것 확인됨
$ (mongo) db.foo.find() 
```

- 리소스 삭제시 pod -> pvc -> pv 순으로 삭제해야함 영구 디스크이므로 

## 4. pv 동적 프로비저닝

> [!question] pv랑 storageClass랑 뭔차이지?
> 


- PV를 동적으로 만들어주는 역할
- PVC가 바운딩 될려면 적합한 PV가 있어야함 바운딩 될려면 적합한 pv가 있어야함 적절한 디스크 찾지 못하면 포드도 뜨지 않게됨 따라서 스토리지 클래스를 선언 (동적으로 pv 만듬) 이때 제한 사항이 있는데 가상화 플랫폼 gcp, openstack등 동적 디스크 생성가능.(제한사항 클라우드에있음)



|     |     |
| --- | --- |
|![](https://i.imgur.com/6zIze0T.png)     | ![](https://i.imgur.com/sryV7FH.png)    |

### PV 동적 프로 비저닝 동작 순서

![](https://i.imgur.com/9gdKs0y.png)


```yaml
# mongo-storage.yaml  
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: storage
---
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - mountPath: /data/db
      name: mongodb
  volumes:
  - name: mongodb
    persistentVolumeClaim:
      claimName: mongo-pvc
```


## 5. 📝 영구 스토리지 연습문제 

- mongo 사용할 수 있도록 pod, pvc, pv 정의하여 수동 프로비저닝 수행
pod, pvc, pv 

- mongo 사용할 수 있도록 pod, pvc, pv 정의하여  동적 프로비저닝 수행

## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스

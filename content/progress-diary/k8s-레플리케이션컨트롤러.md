---
title: k8s-레플리케이션컨트롤러
date: 2023-05-10T16:26:07
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 

categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
lastmod: 2023-05-11T01:43:05
---



## replication controllerf란?

- replicaset의 구버전(k8s 1.8버전에서 사용)
- repplicaset과 동작방식이 거의 동일함
- POD가 항상 실행되도록 유지하는 쿠버네티스 리소스
- 워커노드1의 장애를 5분후에(default 값) replicaset이 인지하고 다른 노드에 새 POD 생성. 곧바로 만들지 않는 이유는 가끔 트래픽이 몰릴 때 네트워크가 끊기는 경우가 있기 때문
- 노드가 클러스터에서 사라지는 경우 POD를 감지하고 대체 POD 생성
- 실행 중 파드 목록 지속적으로 모니터링하고 `유형` 의 실제 POD수가 원하는 수와 일치하는지 확인
![](https://i.imgur.com/H4Rii0N.png)

## replication controller 특징

- pod가 없는 경우 새 pod 항상 실행
- 노드에 장애 발생시 다른 노드에 복제본 생성
- 자동으로 수평 스케일 아웃
- 요소
	- 레이블 셀렉터 : replication controller가 관리하는 pod 범위 결정
	- 복제본 수 : 실행해야하는 Pod 수 결정
	- pod template : 새로운 파드 모양 설명

```yaml
# http-go-rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: http-go
spec:
  replicas: 3
  selector:
    app: http-go ######  label과 동일해야함
  template:
    metadata:
      name: http-go
      labels:
        app: http-go ### selecto 레이블과 동일해야함
    spec:
      containers:
      - name: http-go
        image: gasbugs/http-go
        ports:
        - containerPort: 80
```

POD를 삭제시키면 replication controller가 새  POD 생성
- 레이블을 강제로 변경하는경우?
레블리카 컨트롤러는 label로 조회를하기에 중요함. 이 중 레이블 하나가 삭제되면 조회가 안되므로 강제로 새 POD 생성함



## 만약에 노드하나의 상태가 `Not Ready`  로 바뀐다면 그 안에있던 POD들은 어떻게 도나?
![](https://i.imgur.com/NSrMNBF.png)
![](https://i.imgur.com/qZTCp6p.png)
종료된 node안에 있던 pod들은 여전히 running 상태인데 이는 5분 후에 오류가 감지될 것임

> [!note] 왜 5분을 기다리지?
> 트래픽이 많아지면 통신이 느려지는데 그런경우 컨테이너 빠르게 반응하면 컨테이너가 불필요하게 생성됐다 사라지므로 5분 유예기간을 줌 

![](https://i.imgur.com/9cEJJTN.png)
그리고 다시 살아있는 node에 pod들이 생기고 오류가 발생했던 node에 있던 pod들은 삭제됨

> [!note] 만약에 중지된 node를 다시 살리면?
> 다시 살려도 그 안에있던 pod들이 다시 생성되지 않음




## replication controller 실습

- 실행중 레플리케이션 컨트롤러와 포드 확인
```
kubectl get rc, pod
```


- replication 정보 확인

```
kubectl describe rc rc-http-go
```

- 레플리케이션 컨트롤러 삭제시  (보통은 그 아래있는 POD도 삭제됨)
```
kubectl delete rc http-go
```

- 레플리케이션 컨트롤러 수정(스케일)
```
kubectl scale rc http-go --replicas=5
```
또는

```
kubectl edit rc http-go 
# vim 으로 열리는데 여기에서 replicas=5로 직접 변경도됨(spec아래있는)
```

- 선언적으로 변경하기

 기존에 있던 `http-go-rc` 에서 레플리카스를 10개로 바꾸고 아래의 명령어를 실행하면 k8s가 알아서 바꿔줌 

```
kubectl apply -f http-go-rc.yaml
```




## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
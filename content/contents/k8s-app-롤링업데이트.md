---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
title: k8s-app-롤링업데이트
date: 2023-05-11T16:20:10
lastmod: 2023-05-15T15:01:58
---

## 롤링업데이트란? 

게임 업데이트 할 때 서비스 일시정지함(점검시간 가지고선) 하지만 요즘에는 서비스 다운 되지 않고 (네이버나 구글같은) 일부분 서비스만 중단하고 서비스가 유지된다! 다운 타임이 없이 계속 켜져있다. -> 롤링업데이트라서 가능

![](https://i.imgur.com/xVY1mZQ.png)

기존 서비스스하던 포드들을 삭제하고 새 포드를 배포하는 경우가 있는데 이는 새 포드를 생성하는데 시간이 걸리므로 다운 타임 발생한다. 

롤링업데이트시,  pod v1에 서비스하다가 동시에 로드밸런싱하는게 3개가 있다.
이게 어떻게 가능하지?
포드 버전이  다르더라도 레이블로 스캔하다보니까  pod v1을서비스하면서 pod v2으로도 연결된다.
클라이언트가 기존에 쓰던 앱 v1이고 업데이트 된게 v2인데 이게 v1, v2왔다갔다 할 수 있으므로 v1에서 제공되는 서비스는 v2에서 꼭 제공되어야하고 추후에 v1은 서비스 종료될예정이라고 고객한데 안내해야함. 


1.8버전 부터 deployment API가 생겼는데
그전에 사용하던 사람들은 api가 없어서 kubectl로 scale 명령어로 scale out 정도를 했음. 레플리케이션 컨트롤러 두개 사용해서.
그런데 kubectl이 중단되면 실수가 생길 수도 있음, 하나를 내리고 하나를 새 버전으로 올리고 업데이트 하다가 장애가 생기면 또 이게  자동화가 덜 돼서 힘들다 그래서 deployment API 사용함. 

## 실습계획 
![](https://i.imgur.com/tuSChOR.png)


> [!important] 
> `--record=true` 란 옵션줘서 히스토리가 쌓이도록 어플리케이션 업데이트 할 때마다! 반드시 히스토리 확인가능!

![](https://i.imgur.com/mvzu9ie.png)

![](https://i.imgur.com/2Gb86ve.png)

![](https://i.imgur.com/9wuZOz0.png)

- 리비전 개수는 레플리카셋 하위에 10개 저장가능 
이 업데이트 이력은 `--recore=true`

## 롤링 업데이트 세부전략

recreate에 이런 옵션은 없음 다 없애고 다시 모두 만드는 것이깅..규칙에 따라서 최소 몇개 최대 몇개의 팟 만들개 할건지

![](https://i.imgur.com/bsgNdwW.png)

10개 만들면 최대 +2~3개까지 만들 수 있고 최소 몇개를 운영해야한다~ 인데 최소 7~8개정도 유지되도록 이 범위가 클수록 더 팟을 만들수 있고 더 많이 꺼지므로 시스템 부하
하지만 훨씬 더 빠르게 가능 

- maxSurge 
	- 최대 몇개의 POD?
- maxUnavailable
	- 최소 몇개의 POD?

## 업데이트 실패하는 경우
![](https://i.imgur.com/IFhfrNb.png)

업데이트 실 수 할 수 있으므로 잠시 pause해놓기 그 후에 괜찮으면 resume
![](https://i.imgur.com/NbHqJmM.png)



## 실습

- 모든 자원 삭제
kubectl delete all --all

- deploy 파일
kubectl create -f http-go-deploy-v1.yml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-go
  labels:
    app: http-go
spec:
  replicas: 3
  selector:
    matchLabels:
      app: http-go
  template:
    metadata:
      labels:
        app: http-go
    spec:
      containers:
      - name: http-go
        image: gasbugs/http-go:v1
        ports:
        - containerPort: 8080
```

- kubectl get all 리소스 이름만 가져오기
```sh 
kubectl get pods --no-headers -o custom-columns=":metadata.name"
```

- 롤아웃 확인
```sh
kubectl rollout status deploy/http-go
```

- yaml 파일로 상태 확인 
```
kubectl get deploy http-go -o yaml
```
- 히스토리 확인
```
kubectl rollout history deploy/http-go
```

- patch 
json으로 내부 yaml 건들기 
ready되는데 10초 간격 걸림 이 옵션 없이 실행되면 빠르게 업데이트 돼서 테스트 위해 10초를 줌

```
kubectl patch deploy http-go -p '{"spec":{"minReadySeconds":10}}'
```

또는 `kubectl edit http-go-deploy-v1.yml` 해서 직접 yaml 파일 수정해도 좋음  


- 로드밸런싱 expose
```
kubectl expose deploy http-go
```
서비스 만드는 명령어 expose
Create a Service object that exposes the deployment -> `expose` 명령어 


kubectl get svc 서비스 확인 10.106.23.163에 오픈되어있다
![](https://i.imgur.com/5dXBpx2.png)

클러스터 ip 목록만 보고 싶다면
```
kubectl get svc --no-headers -o custom-headers="spec.clusterIP"
```


- 도커랑 비슷한건데 busy box 배쉬 실행
```
kubectl run -it --rm --image busybox -- bash
```

interactive 모드로 busybox에 배쉬 셸 들어감(이때 종료시 해당 pod삭제됨) 

> [!important] 
kubernetes v1.18 이상은 `run`명령어가 Pod을 만들지만 v1.17 이하는 Deployment 생성됨



배쉬 셸에서 아래 확인
```
wget -O- -q 10.106.23.163:8080
```

클러스터 external IP curl 중 
1초에 한 번씩 컬하는  것 실행
`while true; do wget -O- -q 10.106.23.163:8080; sleep 1; done `

- 새탭에서 다른 이미지로 pod 롤리 업데이트 진행
kubectl set image deploy http-go(deployment이름) http-go(컨테이너이름)=gasbugs/http-go:v2
```
kubectl set image deploy http-go http-go=gasbugs/http-go:v2
```

![](https://i.imgur.com/lIR8pMr.png)

![](https://i.imgur.com/7doVovx.png)

시간이 지나면 .. v2로 모두 바꼈을 것 
하지만 이미지 세팅할 때 --record=true 옵션 주지 않아서 히스토리에 남지 않은데 다시 옵션줘서 보면 히스토리가 잘 남는거 확인 가능
![](https://i.imgur.com/oYpfiy0.png)

기존에 있던 것은 레플리가 desired가 모두 0 
![](https://i.imgur.com/0HlFZn5.png)


- edit 로 바꿔보기 
```
kubectl edit deploy http-go --record=true
```
에서 gasbugs/http-go:v3

- 업데이트 undo 돌아가기
```
kubectl rollout undo deploy http-go
```

리비전이 2로 돌아갈것임
![](https://i.imgur.com/X1xN8AF.png)

- 특정한 리비전으로 들어가고 싶은경우 ? 
리비전 1로 돌아가고 싶을 때 

```
kubectl rollout undo deploy http-go --to-revsion-to=1 (이떄 1은 cli)
```
## 📝 연습문제

- alpine 이미지 사용해 업데이트와 롤백 실행 단 모든 revsion 내용 기록
- alpine:3.4 img사용해 deploy 생성
	- replicas:10
	- maxSurge:50%
	- maxUnavailable:50%
- alpine:3.5 롤링 업데이트 수행
- alpine:3.4 롤백 수행 


## 📝 연습문제 답안

운영체제 이미지라 실행됐다가 곧바로 꺼질 것임
- alpine 이미지 사용해 업데이트와 롤백 실행 단 모든 revsion 내용 기록
- alpine:3.4 img사용해 deploy 생성
	- replicas:10
	- maxSurge:50%
	- maxUnavailable:50%
- alpine:3.5 롤링 업데이트 수행
- alpine:3.4 롤백 수행 

  문법 판단확인

```
kubectl create deploy --image alpine:3.4 alpine-deploy --dry-run=client
```

이 실행되는 것을 yaml 파일로 뽑아줌

```
kubectl create deploy --image alpine:3.4 alpine-deploy --dry-run=client -o yaml > alpine-deploy.yaml
```


```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: alpine-deploy
  name: alpine-deploy
spec:
  replicas: 10
  selector:
    matchLabels:
      app: alpine-deploy
  strategy: 
    type: RollingUpdate
      maxSurge: 50%
      maxUnavailable: 50%
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: alpine-deploy
    spec:
      containers:
      - image: alpine:3.4
        name: alpine
```

실행 하고 
`kubectl apply -f alpine-deploy.yaml`

히스토리 확인
```
kubectl rollout history deploy alpine-deploy
```

- 3.5로 업데이트
```
kubectl set image deploy alpine-deploy alpine-deploy=alpine:3.5 --record=true
```

- 3.4로 undo
```
kubectl rollout undo deploy alpine-deploy --to-revision=1
```

## 📑 Ref
- 인프런 - 지금 당장 AWS!!

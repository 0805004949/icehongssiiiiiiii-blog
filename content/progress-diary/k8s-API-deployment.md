---
title: k8s-API-deployment
date: 2023-05-10T18:05:47
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"]
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
lastmod: 2023-05-11T01:43:18
---

## Deployment

- replicaset을 다를 수 있는 객체이고 다수의 replicaset 관리할 수 있음. 
- 어플리케이션을 다운타임 없이 업데이트 가능하도록 도와주는 리소스 
- 레플리카셋 상위에 배포되는 리소스 
![](https://i.imgur.com/TFbQloP.png)

- 모든 POD를 업데이트 하는 방법?
	- Recreate : POD 를 일괄 삭제하고 다시 생성 잠깐의 다운 타임 발생
	- RollingUpdate : 파드 추가 배포하고 준비되면 오래된 POD 삭제하는 방식


## Deployment - RollingUpdate방식(디폴트)

- 파드 추가 배포하고 준비되면 오래된 POD 삭제하는 방식
- 요청을 처리 할 수 있는 양은 그대로 유지 
- 반드시 이전 버전과 새버전을 동시에 처리 가능하도록 설계한 경우에만 사용

## Deployment - YML 파일 작성

- 포드의 metada부분과 spec부분 그대로 옮김
- deploymnet의 spec, template에는 배포할 POD를 설정
- replicas에서는 이 pod가 몇개를 배포할 것인지 명시
- label은 deployment가 배포한 포드를 관리하는데 사용됨 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

## deployment 스케일링

```
kubectl edit deploy <Deploy Name> 
```

또는

```
kubectl scale deploy <Deploy Name> --replicas=숫자 
```


## 📝  연습문제

- jenkins 디플로이먼트를 deploy-jenkins를 생성하라.  

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-jenkins
  labels:
    app: jenkins
spec:
  replicas: 3
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins:2.60.3
        ports:
        - containerPort: 5000
```
- jenkins 디플로이먼트로 배포되는 앱을 app: jenkins-test로 레이블링하라.  
```
kubectl label deploy deploy-jenkins app=jenkins-test --overwrite
```
- 디플로이먼트로 배포된 파드를 하나 삭제하고 이후 생성되는 파드를 관찰하라.  
```
kubectl describe rs deploy-.... 레플리카셋 확인
```
- 새로 생성된 파드의 레이블을 바꾸어 Deployment의 관리 영역에서 벗어나게 하라. (app태그 빼도 )
-  Scale 명령을 사용해 레플리카 수를 5개로 정의한다.  
- edit 기능을 사용하여 10로 스케일아웃하라.


## 📑 Ref

- 인프런 - devops를 위한 쿠버네티스
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
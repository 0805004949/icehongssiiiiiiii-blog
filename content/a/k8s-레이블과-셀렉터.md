---
title: k8s-레이블과-셀렉터
date: 2023-05-10T14:54:51
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 

categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
lastmod: 2023-05-11T01:42:58
---

## 레이블이란?

- 모든 리소스를 구성하는 매우 간단하면서도 강력한 쿠버네티스 기능
- 리소스에 첨부하는 임의의 키/값 쌍(예 app: test)  
- 레이블 셀렉터를 사용하면 각종 리소스를 필터링하여 선택할 수 있음 ➢ 리소스는 한 개 이상의 레이블을 가질 수 있음  
- 리소스를 만드는 시점에 레이블을 첨부  
- 기존 리소스에도 레이블의 값을 수정 및 추가 가능  
- 모든 사람이 쉽게 이해할 수 있는 체계적인 시스템을 구축 가능
- app레이블: 애플리케이션 구성요소, 마이크로서비스 유형 지정 
- rel레이블: 애플리케이션의 버전 지정


## 레이블을 이용한 포드 구성 예제

![](https://i.imgur.com/BmNLWCw.png)
쇼핑몰 예제
- 열에다가 app이라는 태그를 달아 한 눈에 보기 좋음
- 행에다가 어플리케이션 별 버전 관리

## kubectl을 이용한 레이블 관리

- POD 생성시 레이블 추가 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: http-go-v2
  labels:
    creation_method: manual
    env: prod
spec:
  containers:
  - name: http-go
    image: gasbug/http-go
    ports:
    - containerPort: 8080
    protocol: TCP
```

```
kubectl create -f http-go-pod-v2.yml
```

- 새로운 레이블 추가
```
kubectl label pod http-go-v2 foo=test
```

- 기존 레이블 수정
```
kubectl label pod http-go-v2 foo=test1234 --overwrite
```

- 레이블 삭제
```
kubectl label pod http-go-v2 -foo
```

- 레이블 보여주기
```
kubectl get pod http-go-v2 --show-labels
```

- 특정 레이블 컬럼
```
kubectl get pod http-go-v2 -L env,creation_method
```

- 레이블 필터링
```
kubectl get pod --show-labels -l foo
```

```
kubectl get pod --show-labels -l '!foo'
```

```
kubectl get pod --show-labels -l 'foo!=test'
```


## 레이블 배치 전략 

[k8s-레이블-모범사례](PROJECTS👍/인강-devops를위한-k8s/k8s-레이블-모범사례.md)

![](https://i.imgur.com/5KbDJ2I.png)


## 📝 연습문제

- app=nginx 레이블 가진 포드 생성
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-go
    image: nginx
    ports:
    - containerPort: 80
    protocol: TCP
```

- app=nginx 가진 포드 get
```
kubectl get pod --show-labels -l app=nginx
```

- get된 포드의 레이블의 app 확인

```
kubectl get pod --show-labels -L app
```


- app=nginx레이블 가진 포드에 team=dev1 추가 
```
kubectl label pod http-go-v3 team=dev1
```


## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
---
title: k8s-yaml-POD-작성(descriptor)
date: 2023-05-10T12:16:29
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 

categories: ["TIL"] # <!--"progress-
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
lastmod: 2023-05-11T01:42:41
---

## yml 로 POD 관리 

```
$ mkdir yaml
$ cd yaml
$ vi go-http-pod.yml
$ kubectl create -f go-http-pod.yml 
```

- yml로 작성 
```yaml
# go-http-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: http-go
spec:
  containers:
  - name: http-go
    image: gasbugs/http-go
    ports:
    - containerPort: 8080
```

```sh 
kubectl get pod http-go
```

- 조금 더 자세한 내용
```sh 
kubectl get pod http-go -o wide
```

- yaml이나 json으로 보고 싶다면
```sh 
kubectl get pod http-go -o  yaml (또는 json)
```

- 언제 만들어졌고 이벤트 부분 더 자세히 확인

```sh 
kubectl describe pod http-go
```

- 서비스에 접근하려면 포트 포워딩. 이때 배포한 서버가 있는 곳에서 응답 받을 수 있고 외부에서 접근하면 연결안됨. 따라서 새 터미널 열어서 `curl 127.0.0.1:8080. 하면 잘뜸 또는  `curl 노드 ip:8080`

```sh
kubectl port-forward http-go 8080:8080
```

- POD 리소스 삭제 `kubectl delete -f go-http-pod.yml`
- 로그 확인 (로그 설정 따로 안해서 실습에서는 안뜨는게 확인) `kubectl logs http-go`
- POD에 태그 달기 `kubectl annotate pod http-go key=value`
- 모든 POD 삭제 `kubectl delete all --all`

## 📝 연습문제

1. 모든 리소스 삭제
2. yml 이용해 도커 이미지 젠킨스로 jenkins-maual 포드 생
3. jenkins포드에서 curl 명령어로 로컬호스트:8080 접속
4. jenkins 포트를 8888로 포트포워딩
5. 현재 jenkins-manual 설정 yaml으로 출력

## 📝 연습문제 답안

1. 모든 리소스 삭제
```sh 
kubectl delete --all 
```

2. yml 이용해 도커 이미지 젠킨스로 jenkins-maual 포드 생성
```yaml
# jenkins-mannual.yml
apiVersion: v1
kind: Pod
metadata:
  name: jenkins-mannual
spec:
  containers:
  - name: jenkins-mannual
    image: jenkins:2.60.3
    ports:
    - containerPort: 8080
```

```
kubectel create -f jenkins-mannual.yml
```


3. jenkins포드에서 curl 명령어로 로컬호스트:8080 접속
```
kubectl port-forward jenkins-mannual 8080:8080
```

4. jenkins 포트를 8888로 포트포워딩
```
kubectl port-forward jenkins-mannual 8888:8080
```

5. 현재 jenkins-manual 설정 yaml으로 출력

```
kubectl get pod jenkins-mannual -o  yaml
```

## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
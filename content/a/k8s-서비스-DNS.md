---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
title: k8s-서비스-DNS
date: 2023-05-22T14:36:35
lastmod: 2023-06-05T14:35:27
---

## DNS 서비스
- 이서비스 생성시 대응되는 DNS 엔트리 생성
- 이 엔트리는 `[서비스이름].[네임스페이스이름].svc.cluster.local` 형식을 가짐

## core dns
- 내부에 dns 서버 역할을 하는 POD 존재
- 각 미들웨어를 통해 캐시,로깅,  k8s를 질의하는 등 기능 가짐

![](https://i.imgur.com/yXMop9N.png)


- DNS confimap 저장소를 사용해 설정파일 컨트롤(파일)

## POD에서 subdomain을 사용하면 DNS 서비스를 사용가능
- yml 파일 호스트 이름은 포드의 metadata.name따름
- 필요한 경우 hostname을 따로 선택가능


|                                     |     |
| ----------------------------------- | --- |
|![](https://i.imgur.com/ontGTi1.png) | ![](https://i.imgur.com/McS7Yrr.png)






## 📝 연습문제

 
⚫네임스페이스 blue에 jenkins 이미지를 사용하는 pod-jenkins 디플로이먼트를 생성하고 이를 위한 서비스 srv-jenkins를 생성하라.(8080:8080)

⚫ default 네임스페이스의 http-go 이미지의 curl을 사용하여 pod-jenkis:8080을 요청하라. 
⚫ kubectl exec http-go-77cb5c879-29kld -- curl srv-jenkins.blue:8080



## 📝 연습문제 답안

⚫네임스페이스 blue에 jenkins 이미지를 사용하는 pod-jenkins 디플로이먼트를 생성하고 이를 위한 서비스 srv-jenkins를 생성하라.(8080:8080)

```
$k create ns blue --dry-run=client -o yaml; 
$k create deploy pod-jenkins --image=jenkins/jenkins --dry-run=client -o yaml;
$k create service clusterip srv-jenkins --tcp=8080:8080 --dry-run=client -n blue  -o yaml
```

blue-jenkins-svc-deploy.yaml 에 생성하고
이때 selector는 app=pod-jenkins 로 변경

```
# blue-jenkins-svc-deploy.yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: blue
spec: {}
status: {}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: pod-jenkins
  name: pod-jenkins
  namespace: blue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pod-jenkins
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: pod-jenkins
    spec:
      containers:
      - image: jenkins/jenkins
        name: jenkins
        resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: srv-jenkins
  name: srv-jenkins
  namespace: blue
spec:
  ports:
  - name: 8080-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: pod-jenkins
  type: ClusterIP
status:
  loadBalancer: {}
```

⚫ default 네임스페이스의 http-go 이미지의 curl을 사용하여 pod-jenkis:8080을 요청하라. 

```
k run http-go --image=gasbugs/http-go
k exec http-go -- curl srv-jenkins.blue:8080
```



## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스

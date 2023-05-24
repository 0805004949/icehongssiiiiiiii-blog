---
title: k8s-로드밸런서
date: 2023-05-16T12:03:12
lastmod: 2023-05-16T15:29:12
---

---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---


![](https://i.imgur.com/UAln1JD.png)

vm에서 안되고 클라우드 서비스에서만 가능
L4장비


![](https://i.imgur.com/U1sKGjQ.png)

30533은 임의로 정해진건
로드밸런서는 노드포트에 직접 연결되고 클러스터 IP로 통신함

## 로드밸런서 + GCP

그전에 생성한 리소스 지우고 실행 
```yaml

# http-go-lb.yaml
apiVersion: v1
kind: Service
metadata:
  name: http-go-lb
spec:
  type: LoadBalancer
  selector:
    run: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: http-go
  name: http-go
spec:
  replicas: 1
  selector:
    matchLabels:
      run: http-go
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: http-go
    spec:
      containers:
      - image: gasbugs/http-go
        name: http-go
        ports:
        - containerPort: 8080
        resources: {}
status: {}
```

ip 할당 받을 때 까지 기다려야함📝
이제는 노드외부IP:30001아니라 80포트로 접속됨


## 📝 연습문제

- tomcat 노드포트로 서비스(30002 port)
kubectl create deploy --image=tomcat tomcat --dry-run=client
-o yaml > tomcat-deploy-lb-np.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: tomcat
  name: tomcat
spec:
  replicas: 1
  selector:
    matchLabels:
      run: tomcat
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: tomcat
    spec:
      containers:
      - image: consol/tomcat-7.0
        name: tomcat
        ports:
        - containerPort: 8080
        resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-np
spec:
  type: NodePort
  selector:
    run: tomcat
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30002
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-lb
spec:
  type: LoadBalancer
  selector:
    run: tomcat
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

```

- 노드포트 사용하는경우 30002 풀어야

![](https://i.imgur.com/wLIGbJg.png)


curl 35.202.218.171:80접속가능하고
curl 35.202.218.171:30002접속가능

kubectl edit deploy tomecat 
이떄 이미지를 consol/tomcat-7.0으로 바꿔본다 업데이트가 되는지! container생성하면 잘 들어가진다



## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스

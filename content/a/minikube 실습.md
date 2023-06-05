---
title: minikube 실습
date: 2023-05-09T17:03:34
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
lastmod: 2023-05-15T11:58:42
---
##  macos에서 미니큐브 설치


- https://blog.naver.com/isc0304/221879359568
- 도커 설치
- 미니큐브 설치
- kubectl 설치

m1 애플칩애서는 arm 아키텍처용으로 빌드된 이미지 사용
```sh 
kubectl create deploy hello-minikube --image=k8s.gcr.io/echoserver-arm:1.8
```
https://www.inflearn.com/chats/624565/m1-%EB%A7%A5%EB%B6%81%EC%97%90%EC%84%9C-minikube%EC%97%90-echoserver-%EC%8B%A4%ED%96%89-%EC%8B%A4%ED%8C%A8%EC%8B%9C-echoserver-arm-%EC%82%AC%EC%9A%A9

```sh
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```


- https://velog.io/@pinion7/macOs-m1-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-kubernetes-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0
![](https://i.imgur.com/OhW86zu.png)

도커 인터페이스를 통해

## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
title: k8s-서비스-네트워크
date: 2023-05-22T14:21:17
lastmod: 2023-06-05T14:35:28
---

## k8s 네트워크 모델
- 한 포드에 있는 다수 컨테이너끼리 통신
- 포드끼리 통신
- 포드와 서비스 사이의 통신
- 외부 클라이언트와 서비스 사이의 통신(노드포트, 로드밸런서등..)

```
# 실습전 설치
sudo apt install net-tools
```

## 한 포드에 있는 다수 컨테이너끼리 통신

![](https://i.imgur.com/DaBJSGF.png)

- 컨1, 컨2가 인터페이스 공유해야함(pause명령어로) 한 인터페이스로 동작하게 만들어야한다
- POD가 무너질때 까지 멈춰있는 POD 생성
- 하나의 POD에는 하나의 인터페이스가 생성되어 공유됨



##  포드끼리 통신

- CNI이라는 플러그인이 필요
- ACI< AOS, AWS VPN, CNI, CNI-Genie, flannel, Weave net, Calico등이 있는데 필요에 따라 사용

![](https://i.imgur.com/5VXIffH.png)

calico가 기능을 제일 많이 지원하고  테스트용도로는 weave net이 적절하다. 



- weave net 네트워크 모델
![](https://i.imgur.com/MLvNsoi.png)





## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스

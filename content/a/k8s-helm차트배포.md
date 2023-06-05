---
title: helm소개
date: 2023-06-02T14:55:03
lastmod: 2023-06-05T14:34:15
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---

# helm소개

여러 yaml파일 
- 복잡성관리, 헬름파트 잘 구성하면 추가반복적 작업 
- 업그레이드 단순화
- 롤백 단순화
- 공유
- 버전3공부해둘것
- 차트 레포지토리가 얌구성요소 다운로드받고(mysql repo yaml 인스톨) 클라이언트에 hlem install 하면 쿠버네티스 api서버에 배포됨 
- helm client는 kubectl 설치된곳이어야함
- k8s 설치


## 공개레포지토리


- 대부분 values.yaml -> go template으로 작성할 것

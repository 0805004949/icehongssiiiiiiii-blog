---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
title: k8s-kubesystem개요
date: 2023-05-15T15:21:00
lastmod: 2023-05-15T15:21:37
---

> [!important] 
> 실습 GKE에서 안되므로 가상머신으로 갈것 

## api 서버 

![](https://i.imgur.com/5pxzHB0.png)


![](https://i.imgur.com/WhZ7UvR.png)

- 중앙집중화된, etcd와 유일하게 통신
- etcd=클러스터 설정 창고, 여기에 접근 가능한 사람은 kubeapi서버이다 


## 컨트롤러 매니저
- api는 k8s에서 (참모), 자원들이 어케 움직이면 좋을지를 api서버에 전달
- 이 컨트롤러는api에 의해 받아진 요청 처리하는 역할 


## 큐브 스케줄러
- 실행노드 일반적으로 정해지지 않음
- 다수 포드 배치시 라운드로빈
- 노드 스케줄링

주요 컴포넌트ㅡㄹ은 `kube-system` 네임스페이스에 존재함

## 쿠버네티스 파일 모아져있는곳

![](https://i.imgur.com/Nzl4slE.png)
proxy는 노드마다 한개씩 

## static pod
- 정적 보드
- 큐브릿이 직접 싫ㅇ
- kubectl에 의해 실행되지 않음
- kubelet은 대몬이 실행
- 런타임에 관여
- 포드 삭제시 api 서버통해 삭제되지 않음 계속 띄어짐 노드에 의해 특정
- `정해진 디렉토리에` 올리면 kubelet에 올라감
- 이때 디렉토리는? -> 
![](https://i.imgur.com/tBGYG31.png)

- etcd->포드 
사용자가 지워도 바로 복구되는 것들 삭제는 되는데 금방 올라가짐, 여기 안에 있는 인자를 변경하고 싶다면? 이 yml 파일에 수정하는 즉시 곧바로 반영 되므로 파일을 백업 해놓는게 좋다 

## 실습

마스터 노드

- vim /etc/kubernets/manifests
여기를 변경하고 싶다면? 

kubelet이 실행하므로 

- service kubelet status 대몬 보면  vim /etc/system/kubelet.service.d 에 파일있는거 확인 서비스 실행시켜주는 파일 열어서
kubeconf를 ㅗ면 큐브렛에 대한 멸렁어가 정해졍ㅆ음
큐브렛이 이 인자들 통해 실행됨 

![](https://i.imgur.com/qoxSVl2.png)

kubelet/config.yaml확인 여기 열어보면 설정들이 포함되어있는데 staticpodpath를 수정! kubernetis manifest ㅕㅇ로 변경

manifest경로 아래 stati-pod.yaml 생성
![](https://i.imgur.com/hjKtSqL.png)

저장만해도 실행되어야함
![](https://i.imgur.com/l3nJ56w.png)

지워도 안지워짐

![](https://i.imgur.com/4xoeT93.png)


## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스

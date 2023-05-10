---
title: k8s-레플리카셋
date: 2023-05-10T17:15:45
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 

categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
lastmod: 2023-05-11T01:43:11
---

## 레플리카셋 등장 

- k8s 1.8 중요한 업데이트 진행됐고 중요 API 정식으로 들어왔음! (deployment, daemonset, replicaset, statefulset)
- 초기 k8s에서 replication controller 사용했기에 현장에서는 여전히 이것 사용할 수 있지만 후에 replicaset으로 대체 가능할 듯

- replicaset이나 replication controller는 거의 동일하게 동작
- 다만 replicaset이 더 풍부한 표현식으로 pod 셀렉터 사용가능
- replication controller는 특정 레이블 포함하는 파드가 일지하는지 확인
- 🚸 replicaset은 특정레이블이 없거나 해당 값과 관계없이 특정 레이블 키를 포함하는 파드 매치 확인 🚸

![](https://i.imgur.com/fN3qjAs.png)


## 레플리카셋 실습

```yaml
# nginx-rs.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-nginx
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      app: rs-nginx # 이름이 동일해야함
  template:
    metadata:
      labels:
        app: rs-nginx # 이름이 동일해야함
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

- 스케일링(3가지 방법)
```
kubectl scale rs rs-nginx --replicas=5
```
또는
```
kubectl edit rs rs-nginx
```
또는 (rs-nginx.yml 수정후 단 변경된 yml은 버전관리 추후 해줄것)
```
kubectl apply -f rs-nginx.yml
```



## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
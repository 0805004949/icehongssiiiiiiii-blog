---
title: k8s-네임스페이스
date: 2023-05-15T15:02:02
lastmod: 2023-05-18T21:32:38
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
---

## 네임 스페이스

- 하나의 클러스터 사용해도 격리된 환경처럼 사용가는하게 만든다.
- 개임 서버 여러개 그때 고유한 닉네임을 다른 서버에서도 똑같은 아읻로 쓸 수 있다 네임 스페이스안에서만 유니크한 이름을 쓰면된다
- 네임스페이스 내에서는 유니크한 이름 쓸 수 있음

```sh 
kubectl get ns
```

`kubectl delete all --all` 이떄는 default 네임스페이스만 있는 리소스만 삭제되는데 만약에  전체 네임스페이스에 있는 리소스를 조회하고 싶다면
```
kubectl get pod --all-namespaces
```

## 명령어

- 만들기
```
kubectl create ns office
```

- 또는 dry-run 문법 확인 및 -o yamal 파일
```
kubectl create ns office --dry-run=client -o yaml > office-ns.yaml
```
- 자원할당할려면?
```
kubectl create deploy nginx --image nginx -n office 
```

- 조회
```
kubectl get deploy -n office
```

- 모든 네임스페이스에서 조회
```
kubectl get all --all-namespaces
```
- 리소스 요청할때 무조건 디폴트 네임스페이스에 요청하는걸로 되어있는데 이거 바꾸고 싶으면`~/.kube/config` 수정해서 바꾸면됨 ⚠️ 하지만 추천하지는 않음 ⚠️


## 📝 연습문제 
- 현재 시스템에는 몇 개의 Namespace가 존재하는가?  
```
kubectl get ns --no-headers | wc -l
```

- kube-system에는 몇 개의 파드가 존재하는가?  
```
kubectl get po -n kube-system --no-headers 
```
- ns-jenkins 네임스페이스를 생성하고 jenkins 파드를 배치하라.
```
kubectl create ns ns-jenkins
kubectl run jenkins99 --image=jenkins/jenkins --port=8080 -n ns-jenkins
```
또는 아래 yaml  (두가지 리소스 실행됨 두개반드시 만들어야함)
![](https://i.imgur.com/2q0wrvz.png)


➢ pod image: jenkins ➢ pod name: jenkins

```
kubectl create
```

- coredns는 어느 네임스페이스에 속해있는가?

```sh 
kubectl get pod --all-namespaces | grep coredens 
```





## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
- https://subicura.com/k8s/guide/service.html#service-clusterip-%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5
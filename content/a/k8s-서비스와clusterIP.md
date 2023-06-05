---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
title: k8s-서비스와clusterIP
date: 2023-05-15T15:43:21
lastmod: 2023-05-22T14:17:15
---

## 서비스가 왜 필요한가 
![쿠버네티스-서비스-개념](https://i.imgur.com/u0RAVco.png)
사진출처 : [쿠버네티스 안내서](https://subicura.com/k8s/guide/service.html#service-clusterip-%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5)

- POD는 자체 IP가지고 다른 Pod과 통신 가능하지만 POD는 일시적으로 생성한 컨테이너의 집합이고 스케일링 했을 때 쉽게 사라지고 생성되기에 직접 통신하는 방법은 좋지 않다
- 왜냐면 생성 될 때  IP 주소가 변동되므로 따라서 **별도의 고정된 IP를 가진 서비스**를 만들고 그 서비스를 통해  POD에 접근하는 방식이 권장됨 -> `서비스`
- 서비스의 종류 3가지
	- ClusterIP (내부) - 내부 
	- NodePort(외부) - 서비스를 외부로 노출가능. 노드의 자체 포트를 사용하여 포드로 리다이렉션
	- Loadbalancer(외부) 서비스를 외부로 노출가능. 외부 게이트웨이를 사용해 노드 포트로 리다이렉션


## Service (ClusterIP) 

전문출처 : [쿠버네티스 안내서](https://subicura.com/k8s/guide/service.html#service-clusterip-%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5)

- clusterIP = 외부로 서비스하기 위한게 아니라 내부에 서비스를 공유하기 위한 자원
- 클러스터 내부에 새로운 IP를 할당하고 여러개의 POD를 바라보는 로드밸런서 기능 제공 
- 서비스 이름을 내부 도메인 서버에 등록해 PoD간에 서비스 이름으로 통신 가능
- `replicaset` 과 `deployment`와 동일하게 레이블로 어느 리소스에 서비스를 제공할건지 정할 수 있다 (spec.selector 항목)

- expose로 생성이 가장 쉬움 본적으로 만들어지면 ClusterIP로 만들어지고 외부에 노출 안됨. (디폴트값)

- 또는 yaml파일로 관리

```
# http-go-deploy-v1.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-go
  labels:
    app: http-go
spec:
  replicas: 3
  selector:
    matchLabels:
      app: http-go
  template:
    metadata:
      labels:
        app: http-go
    spec:
      containers:
      - name: http-go
        image: gasbugs/http-go:v1
        ports:
        - containerPort: 8080
```

```
kubectl create -f http-go-deploy-v1.yml
# 서비스 생성
kubectl expose deploy http-go 
```


## Service  세션 고정하기

- http는 stateless. 

로그인 요청했을때 1번 pod에서 로그인 했다는 응답을 받고 다른 서비스 이용할 떄는 2번  pod을 이용하는데  http statless때문에 로그인 한 적이 없다고 뜰 수도 있음 이런 현상 발생할 수 있어서
`spec.sessionAffinity: ClientIP`  옵션을 주면 처음에 들어왔던 클라이언트 IP를 그대로 유지해주는 방법이 필요하다!

처음 붙었던 POD가 계속 그 클라이언트 IP에 세션유지 

## Service (ClusterIP) 실습 

```yaml
# http-go-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: http-go-svc
spec:
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


```
kubectl create -f http-go-svc.yaml
```
생성

- 조회 
```
kubectl describe svc http-go-svc
```

![](https://i.imgur.com/hDh6VuQ.png)

pod 1개 일때 Endpoints IP가 1개 
현재 클러스터  IP `10.99.129.175` 

- replicas 5개로 생성 후 Endpoints IP 확인
```
kubectl scale deploy http-go --replicas=5
```

다시 조회 
![](https://i.imgur.com/JnYuHx3.png)

이때 클러스터 IP이고 `10.99.129.175`
```
kubectl run --it --rm --image=busybox --bash 
$ wget -O- -q 10.99.129.175:80
```

![](https://i.imgur.com/ehcCszO.png)

로드밸런싱이 되어 http 요청 보낼 때마다  다른 POD에 붙는것 확인된다 

- yaml수정 ; sessionAffinity 옵션 추가 
```
kubectl edit http-go-svc
```

spec에서 sessionAffinity: ClientIP추가! 


현재 클러스터  IP `10.99.129.175` 
```
k exec [5개의POD중한개의POD] -- curl 10.99.129.175:80
```
![](https://i.imgur.com/YGd0MGi.png)
이걸로 바꾸면 하나의 POD에 처음 붙으면 그 IP는 그 POD에만 붙는다. 

## Service(ClusterIP)와 같이쓰는 Endpoint

- k8s clusterIP 기능 사용해 외부 서비스와 연결 가능 일반적으로 POD도 외부와 통신가능 근데 특별히 clusterIP이용해서 고정IP로 서비스 제공할 떄 사용가능
- 외부 서비스와 연결 수행시 endpoint 레이블 사용 지정하는게 아니라 외부 IP를 직접 endpoint라는 별도 자원에서 설정


![](https://i.imgur.com/hQwxgVJ.png)
링크 : [레이블이 없는 서비스 사용하는사례](https://kubernetes.io/docs/concepts/services-networking/service/)


![](https://i.imgur.com/eegEvcb.png)

기존에 클러스터 IP를 만들던 SVC에서는 레이블을 추가했던것과는 달리 직접 IP를 넣어준다 
![](https://i.imgur.com/yO1IdG8.png)
레이블 지정하는게 아니라 직접 IP를 넣음

### Endpoint 실습

```
kubectl run my-service --image=gasbugs/http-go 
```

```yaml 
# my-service-endpoints.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

---
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 223.130.195.200
      - ip: 199.201.110.204
    ports:
      - port: 80
```

```
kubectl apply -f my-service-endpoints.yaml
```


POD에서 해당 서비스 curl 해본다 
```
kubectl exec -it http-go -- curl my-service
```

이때 my-service에 있는 내용 보면
`222.130.195.220`에 있는 내용과 `199.201.110.204`에 있는 내용이 번갈아가면서 나오는것 확인 가능 
![](https://i.imgur.com/O7Ym9KB.png)






## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스

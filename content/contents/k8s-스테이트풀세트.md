---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---
## 스테이트풀셋이란?

- 레플리케이션 컨트롤러, 레플리카세트, 디플로이먼트는 모두 상태가 없는 파드들을 관리하는 용도
- 스테이플세트는 그 의미대로 상태가 있는 파드들을 관리하는 컨트롤러이다
- 스테이풀 세트 사용시 볼륨을 사용해 특정 데이터를 저장 한 후에 파드 재시작시 해당 데이터 유지할 수 있다(기존 포드 삭제하고 생성시 유지되지 않음 포드 삭제하고 생성시 새 가상환경과 다름 없음)
- 여러개 파드 사이에 순서ㄹ를 지정해서 실행 할 수 있다

## 장점

- 안정적이고 고유한 네트워크 식별자
- 질서 정역 포드 배치 및 확장(이름이 순차적으로 생성)
- 포드 자동 롤링업데이트 사용원할경우
- 안정적이고 지속적인 스토리지 사용시

## 고려사항

- 스테이트풀셋과 관련된 볼륨 삭제되지 않음(관리 리소스)
- 포드 스토리지는 PV 나 storageclass로 프로비저닝 수행해야함
- 롤링 업데이트 수행시 수동 복구해야할 수 있음 (**롤링업데이트 수행시 기존 스토리지와 충돌로 인해 어플리케이션 오류 발생할 수도 있음**)
- 포드 네트워크 ID 유지하기위해 별도의 헤드레스 서비스가 필요하다 

## 헤드레스 서비스작성 요령

- 기존 서비스에서 clusterIP를 None으로 작성

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```

## 스테이트풀셋 작성요령

디플로이먼트와 굉장히 비슷함


## 실습

```
# nginx-statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx" # 헤드레스 서비스를 지정한다.
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10 # 강제 종료까지 대기하는 시간
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates: # PVC 설정을 저장하는 부분
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi

```

- clusterIP가 없는 헤드레스 서비스, 포드, 스테이트풀셋이 잘 떠있다
- web0~2까지 순차적으로 포드가 네이밍 되어있다. 
![](https://i.imgur.com/TGiW7Sj.png)


![](https://i.imgur.com/EC3PLEU.png)

-   pv와 pvc를 살펴보면 각 포드가 자신만의 pVC와 PV를 할당받은 것을 볼 수 있다
- 스케일 업시 순차적으로, 스케일 다운시 높은 수부터 진행

```
# 스케일업 web3, web4 포드 생성  
k scale statefulset web --recplias 1 
# 스케일업 web1, web2, web3, web4 포드 삭제 
k scale statefulset web --recplias 5
```
이때 web0포드가 하나 남아있는데 여전히 pvc는 살아있다
![](https://i.imgur.com/KQwnfW5.png)
또 statefulset를 삭제하면 포드는 모두 삭제하지만 pvc는 계속 살아있다.

## 📝 연습문제

## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
- 전문출처 https://blog.naver.com/isc0304/221885403537

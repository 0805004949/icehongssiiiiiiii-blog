---
title: k8s-pod-liveness-redainess-소개
date: 2023-05-10T13:37:22
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 

categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
lastmod: 2023-05-11T01:40:15
---

## 1. Liveness Probe
- 컨테이너 살아있는지 판단하고 다시 시작하는 기능
- 컨테이너 상태 스스로 판단해 교착상태 빠진 컨테이너 재시작
- 버그 생겨도 높은 가용성 보임


> [!warning] 잘못 구현하면 연쇄적인 장애가 발생할 수 있습니다. 그 결과 부하가 높을 때 컨테이너가 다시 시작되고, 애플리케이션의 확장성이 떨어지면서 클라이언트 요청이 실패하고, 일부 실패한 파드로 인해 나머지 파드의 워크로드가 증가하게 됩니다


## 2. Readiness Probe
- POD가 준비된 상태에 있는지 확인하고 정상 **서비스** 시작하는 기능
- POD가 적절하게 준비되지 않은 경우 로드밸런싱 하지 않음


## 3. Startup Probe
- 어플리케이션 시작 시기 확인하여 가용성 높이는 기능 (시작시간 보장)
- 🚸 **Liveness와 Readiness 기능 비활성화** 🚸 컨테이너 시작시 과하게 readiness와 liveness가 설정되면 부팅 시간이 오래걸리게 되므로 


## 4. Liveness Probe yml 예제 

healthy파일 만들고 30초 후에 삭제 -> sleep 
이 pod가 살아있는지는 `cat /tmp/healthy` 파일이 있는지 확인할 건데 이게 없다면 오류가 날 것이고 
- 5초마다, 시작한지 5초후 부터 health check 

```yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds:
      periodSeconds: 5
```
- 5초가 지나면 파일이 삭제되므로 pod는 재시작됨 
```
$ kubectl create -f exec-liveness.yaml
$ kubectl get pod liveness-exec
$ kubectl describe pod liveness-exec
```

describe의 결과 
![](https://i.imgur.com/k0279Zi.png)



## 5. Liveness Probe 웹설정(http) yml 예제 

- 서버 응답코드 200~300대이면 컨테이너유지, 그외 서버응답코드(400,500)이라면 컨테이너 재시작 
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

- `registry.k8s.io/liveness`  해당이미지는 10초전에는 넘어가면 500 status code 반환하고 그것보다 작으면 200코드 반환
![](https://i.imgur.com/s2regmW.png)
d


## 6. TCP-liveness-readiness prove yaml 예제

- readiness probe는 8080 포트를 검사, 5초후부터 검사 시작, 10초마다 검사 후 통신되는 경우 TCP 통신 소켓 정상적으로 연결되면 통신 시작
- liveness probe는 8080 포트 검사, 15초 후부터 검사 시작, 20초마다, 컨테이너를 재시작하지 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: registry.k8s.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```


## 7. Start up proble yaml 예제

- 시작할때 까지 검사 수행
- 프로그램 실행 긴 경우, liveness, readiness까지 동작하게 되는 경우 시작시간이 더 오래걸릴 수 밖에 없음 
- startup probe 실행되면 liveness 검사 되지 않음 
- http 요청통해 검사
- 30번 검사하고 10초 간격으로 수행
- 300(30*10)초후에 POD 정상 동작되지 않으면 종료 즉 300초 동안 POD 정상 실행되는 시간 벌어줌 

```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```


## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
- yaml 예제 출처 https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
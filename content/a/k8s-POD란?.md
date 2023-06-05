---
title: k8s-POD란?
date: 2023-05-09T18:55:26
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 

categories: ["TIL"] # <!--"progress-
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
lastmod: 2023-05-11T01:42:33
---



## POD란?

- 쿠버네티스 기본 빌딩 블록
- 쿠버네티스는 컨테이너를 개별적으로 배포하지 않고 컨테이너의 POD를 항상 배포하고 운영
- 일반적으로 POD는 단일 컨테이너만 포함하지만 다수의 컨테이너를 포함 할 수 있음. 다수의 컨테이너 포함해서 실행하면 POD안에서 자원공유 할 수 있도록 만들어짐. 
- POD는 다수의 노드에 생성되지 않고 단일 노드에서만 실행
- 여러 프로세스를 실행하기 위해서는 컨테이너 당 단일 프로세스가 적합
- 다수의 프로세스를 제어하려면? -> 다수의 컨테이너를 다룰 수 있는 그룹이 필요

![](https://i.imgur.com/mmJKQKG.png)

- 다른 노드에 걸쳐서 실행되지는 않음(위에서 한쪽 컨테이너는 다른 노드에, 세모는 다른 노드에 있을 수 없음) 하나의 POD는 하나의 노드에서만

## POD 장점
- 밀접하게 연관된 프로세스를 함께 실행하고 마치 하나의 환경에서 동작하는 것처럼 보임
- 그러나 동일한 환경 제공하면서 어느정도 격리된 상태로 유지

## 동일한 POD 내 컨테이너 사이 부분 격리
- 같은 호스트 이름 , 네트워크 인터페이스 공유(그러나 포트 충돌 가능성있음)

## POD 네트워크 

- POD가 할당 받은 네트워크 대역대가 있음
![](https://i.imgur.com/4gSFhak.png)

- 쿠버네티스 클러스터의 모든 POD는 공유된 단일 플랫, 네트워크 주소 공간에 위치
- 포드 사이에는 NAT 게이트 웨이가 존재하지 않음

## POD 구성 방법
![](https://i.imgur.com/EUEBqKA.png)

- POD 안에는 일반적으로 컨테이너 하나. 스케일 아웃시 불필요한 서비스도 복제 할 수 있기에. 두가지의 컨테이너가 밀접한 실행 필요한 경우에만 두개의 컨테이너를 하나의 POD넣는것을 권장함 
- 일반적으로 POD 하나당 하나의 컨테이너, 하나의 컨테이너 안에 하나의 프로세스 


## YAML으로 POD 디스크립터 만들기
- kubectl 명령어로 작성 가능하지만 저장 편하지 않아서
- 모든 k8s 객체 yaml으로 정의하면 버전 제어 시스템 저장가능


## YAML에서 POD 정의
- 메타데이터, 스펙, status, `apiVersion`, `kind` 로 구성
- `apiVersion` : 쿠버네티스 api version
- `kind` : 어떤 리소스 유형인지 결정(POD replica controller, service 등)
- 메타데이터 : POD와 관련된 이름, 네임스페이스, 라벨, 그밖의 정보 존재
- 스펙 : 컨테이너 볼륨 정보
- 상태 : 포드의 상태, 각 컨테이너의 설명 및 상태, 포드 내부 IP 및 그 밖의 기본 정보 


## 컨테이너 상태 모니터링

전문출처 : [sbicura-쿠버네티스안내서-POD](https://subicura.com/k8s/guide/pod.html#yaml%E1%84%85%E1%85%A9-%E1%84%89%E1%85%A5%E1%86%AF%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%91%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%AFspec-%E1%84%8C%E1%85%A1%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5)

![](https://i.imgur.com/pSTGZhK.png)

`컨테이너 생성`과 실제 `서비스 준비`는 약간의 차이가 있습니다. 서버를 실행하면 바로 접속할 수 없고 짧게는 수초, 길게는 수분~~Java ㅂㄷㅂㄷ~~의 초기화 시간이 필요한데 실제로 접속이 가능할 때 `서비스가 준비되었다`고 말할 수 있습니다.

쿠버네티스는 컨테이너가 생성되고 서비스가 준비되었다는 것을 체크하는 옵션을 제공하여 초기화하는 동안 서비스되는 것을 막을 수 있습니다.

###  [livenessProbe](https://subicura.com/k8s/guide/pod.html#livenessprobe)

컨테이너가 정상적으로 동작하는지 체크하고 정상적으로 동작하지 않는다면 **컨테이너를 재시작**하여 문제를 해결합니다.

`정상`이라는 것은 여러 가지 방식으로 체크할 수 있는데 여기서는 `http get` 요청을 보내 확인하는 방법을 사용합니다.

```
apiVersion: v1
kind: Pod
metadata:
  name: echo-lp
  labels:
    app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
      livenessProbe:
        httpGet:
          path: /not/exist
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 2 # Default 1
        periodSeconds: 5 # Defaults 10
        failureThreshold: 1 # Defaults 3
```


일부러 존재하지 않는 path(/not/exist)와 port(8080)를 입력하였습니다.

**상태 확인**

```
NAME      READY   STATUS             RESTARTS      AGE
echo-lp   0/1     CrashLoopBackOff   4 (15s ago)   60s
```

정상적으로 응답하지 않았기 때문에 Pod이 여러 번 재시작되고 `CrashLoopBackOff` 상태로 변경되었습니다.

상태체크

httpGet 외에 `tcpSocket`, `exec` 방법으로 체크할 수 있습니다.



### [readinessProbe](https://subicura.com/k8s/guide/pod.html#readinessprobe)


컨테이너가 준비되었는지 체크하고 정상적으로 준비되지 않았다면 **Pod으로 들어오는 요청을 제외**합니다.

livenessProbe와 차이점은 문제가 있어도 Pod을 재시작하지 않고 요청만 제외한다는 점입니다.

```
apiVersion: v1
kind: Pod
metadata:
  name: echo-rp
  labels:
    app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
      readinessProbe:
        httpGet:
          path: /not/exist
          port: 8080
        initialDelaySeconds: 5
        timeoutSeconds: 2 # Default 1
        periodSeconds: 5 # Defaults 10
        failureThreshold: 1 # Defaults 3
```



**상태 확인**

```
NAME      READY   STATUS    RESTARTS   AGE
echo-rp   0/1     Running   0          14s
```

READY상태가 `0/1`인 것을 확인할 수 있습니다.

### [livenessProbe + readinessProbe](https://subicura.com/k8s/guide/pod.html#livenessprobe-readinessprobe)

보통 `livenessProbe`와 `readinessProbe`를 같이 적용합니다. 상세한 설정은 애플리케이션 환경에 따라 적절하게 조정합니다.

```
apiVersion: v1
kind: Pod
metadata:
  name: echo-health
  labels:
    app: echo
spec:
  containers:
    - name: app
      image: ghcr.io/subicura/echo:v1
      livenessProbe:
        httpGet:
          path: /
          port: 3000
      readinessProbe:
        httpGet:
          path: /
          port: 3000
```


`3000`번 포트와 `/` 경로는 정상적이기 때문에 Pod이 오류없이 생성된 것을 확인할 수 있습니다.

## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
- https://subicura.com/k8s/guide/pod.html#%E1%84%88%E1%85%A1%E1%84%85%E1%85%B3%E1%84%80%E1%85%A6-pod-%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5
- 
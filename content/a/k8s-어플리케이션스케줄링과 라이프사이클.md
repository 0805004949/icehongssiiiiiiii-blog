---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
title: k8s-어플리케이션스케줄링과 라이프사이클
date: 2023-05-24T15:35:50
lastmod: 2023-06-05T14:34:11
---

## 어플리케이션 변수 관리

1. yaml
2. ConfigMap(k8s 리소스)
3. Secret(k8s 리소스 )


### 1. Yaml

- 포드안에있는 yaml에 환경변수 넣으면 yaml에 있는 환경변수 값을 계속 바꿔줘야함. 이때  pod의 종류가 여러개일텐데  yaml파일 각 각 들어가서 수정해야함. 그래서 외부를 참조하도록 하는데 보통 configmap이나 secret이용함 

- yaml파일 내에 환경변수 주입하는 경우 

```yaml
# envars.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```

```
$ k create -f envars.yaml
$ k exec -it envar-demo -- printenv DEMO_GREETING
```




![](https://i.imgur.com/meI5nwC.png)


![](https://i.imgur.com/eyhNE2S.png)


### 2.  환경변수를 ConfigMap에 저장하는 방법

- configmap은 환경변수 저장 할 수도 있고 스토리지 파일 저장 기능도 있음
- 컨테이너가 

- configmap 생성
```shell
k create configmap <map-name> <data-source>
```

- 예제 
```
// -n옵션 엔터없이 4자리만들어감 
echo -n 1234 > test 

// map-name 이라는 configmap생성 test라는 파일 참조해서
k create configmap map-name --from-file=test

// map-name이라는 configmap파일 출력 
k get configmap map-name -o yaml 
```

```
# configmap.yaml
apiVersion: v1
data:
  test: "1234"
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: map-name
```

![](https://i.imgur.com/WkyXJS2.png)

```yaml
# node-env.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: envar-configmap
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      valueFrom:
              configMapKeyRef:
                      name: map-name
                      key: test
```

```
k exec -it envar-configmap -- printenv DEMO_GREETING
```

### 3. configmap을 활용한 디렉토리 마운트
- cofnigmap내용을 데이터에 저장

```
# cofigmap-multikeys.mal
apiVersion: v1

kind: ConfigMap

metadata:

name: special-config

namespace: default

data:

SPECIAL_LEVEL: very

SPECIAL_TYPE: charm
```

```shell
kubectl create -f https://kubernetes.io/examples/configmap/configmap-multikeys.yaml

k get configmap special-config -o yaml 
```

```
# env-volume.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: volume-configmap
  labels:
    purpose: demonstrate-envars
spec:
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
```

환경변수 출력
![](https://i.imgur.com/Ck4xwv9.png)

pod 재시작할떄 변수들어감 마운팅들어가는경우는 1분마다 들어감..


### 3. secret 활용

- 민감한 Auth, ssh, 비밀번호등을 configMap 과 달리 인코딩으로 저장함 (근데 디코딩도 쉽긴함)
![](https://i.imgur.com/3lRC64C.png)

generic이 가장 일반적인 secret 종류
- secret 생성방법 예제
```
$ echo -n admin > username
$ echo -n 1234qwerty > password
```

seret type중 generic으로 생성
```
$ k create secret generic db-user-pass --from-file=username -- from-file=password
```

 `db-user-pass` 시크릿 내용 출력해서 디코드 해보기
```
$k get secret -o yaml db-user-pass
$echo MTIzNHF3ZXJ0eQ== | base64 --decode
```


```yaml
# secret-pad
apiVersion: v1
kind: Pod
metadata:
  name: envar-secret
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: user
      valueFrom:
              secretKeyRef:
                      name: db-user-pass
                      key: username

    - name: pass
      valueFrom:
              secretKeyRef:
                      name: db-user-pass
                      key: password
```

```
k exec -it envar-secret -- printenv
```

📝 연습문제
-   ⚫  Kube-system에 존재하는 secret의 개수는 몇 개인가?
    k get secret -n kube-system --no-headers | wc -l 
-   ⚫  default-token의 개수는 몇 개인가?
    k get secret -n kube-system --no-headers | grep default
-   ⚫  default-token의 타입은 무엇인가?
    k get secret -n kube-system --no-headers | grep default 에서 나온것 yaml로 확인하기 
-   ⚫  Secret 데이터에는 어떤 secret data가 포함돼 있는가?
    인증서 관련된 정보 
-   ⚫  다음과 같이 Mysql 서버를 지원하는 secret-mysql.yaml을 생성하자. ➢Secret Name: db-secret  
    ➢Secret Data 1: DB_Password=Passw0rd!0

echo -n Passw0rd!0 > DB_PASSWORD
k create secret generic db-secret --from-file=DB_PASSWORD
-   ⚫  Mysql 이미지를 하나 생성하고 앞서 만든 secret을 환경 변수 이름과 연결하자. ➢ Image: mysql:5.6
    
- k run mysql --image=mysql --env="name=MYSQL_ROOT_PASSWORD" --port=3306
- 
    ➢ Port 번호: 3306  
    ➢ 환경 변수 name: MYSQL_ROOT_PASSWORD
    

```
apiVersion: v1
kind: Pod
metadata:
  name: mysql
spec:
  containers:
  - name: mysql
    image: mysql:5.6
    env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
              secretKeyRef:
                      name: db-secret
                      key: DB_PASSWORD
    ports:
            - containerPort: 3306
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
- 
- 
- 
    ➢ 잘 적용됐는지 확인할 수 있는 명령어:  
    ✓ kubectl exec –it mysql –- mysql –u root –p ✓ password: Passw0rd!0

----- 

## 초기 명령어 및 아규먼트 전달

- busybox 이미지 사용하는 busybox 포드 생성
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
    resources: {}
    command: ["sh", "-c", "--"]
    args: ["while true; do sleep 30; done;"]
    # comand : ['sh', '-c','sleep 3600']
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- busybox 포드가 유지되지 않는다면 그 이유?
completed가 됐음(실행) 그리고선 crashloopbackoff 실행되자마자 왜 꺼지냐면 어플리케이션일뿐 올리자마자..
- busybox 장시간 유지하기 위해 장시간 lseep하는 명령어 추가 실행

- busybox게속 유지될수 잇는가?

## 한 포드에 멀티 컨테이너

- 하나의 포드 사용하는 경우 network interpace, ipc 볼륨 공유 지역성
- 호율적으로 통신해 지역성 보장하고 여러개 응용프로그램이 결합된 형태로 하나의 포드 구성
- 리소스 모니터링의경우 호드에 두개의 컨테이너를 사용하는 경우가 많음
- 개발자가 만든 컨테이너를 포드에만들면 이 컨테이너의 리소스상태를 관찰해야하는데 별도의 모니터링하는 컨테이너를 배치해주고 이 모니터링 컨테이너가 별도의 시스템에 전달하기도하고 아니면 노드에 직접 전다 관찰해줘야하는데 안에있는 로그 꺼내올려고하면 보통 리소스 모니터링용으로 포드가 로깅하는 형태로 존재함.
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-container
  name: multi-container
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    ports:
    - containerPort: 80
  - image: redis
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## init 컨테이너
- 초기화 역활하는 컨테이너
- 포드 컨테이너 실행전에 초기화 역할 하는 컨테이너
- 완전히 초기화가 진행된 다음에야 주 컨테이너를 실행
- init 컨테이너가 실패하면 성공할 때 까지 포드를 반복해서 재시작 
- restartPolicy에 Never를 하면 재시작하지 않음

![](https://i.imgur.com/VH4Jnhu.png)

- 이닛컨테이너가 끝나지않으면 주컨테이너가 실행되지 않음.

![](https://i.imgur.com/DLxMl0c.png)
- 첫번째 init 컨테이너 untill 문법 ; ㅈ회가 ㅈ될때까지 계속 시작함. w정상실행될때까지 계속 실행됨 myservice가 어디에 있으면 myapp-container실행됨. myapp-contaier가 의존성이 있을때 init 컨테이너 사용함. 만약에 myapp-container전에 다른 작업이 필요하다면.. 기다려야함. 두 프로세스가 죽어야 myapp-실행됨
![](https://i.imgur.com/4alejs7.png)


⚫ pod-init-container.yaml를 작성하여 리소스를 생성하라.





⚫ my-app-pod 파드에서 주 컨테이너가 실행되지 않는 현상을 관찰하라.  
⚫ 주 컨테이너가 실행되지 않는 이유는 무엇인가?  
⚫ svc-pod-mydb.yaml을 작성 및 실행하고 주 컨테이너의 반응을 관찰하라. ⚫ 주 컨테이너가 정상적으로 실행되었는가? 그렇다면 그 이유는 무엇인가?



## 시스템 리소스 요구사항과 제한설정



⚫ CPU와 메모리는 집합적으로 컴퓨팅 리소스 또는 리소스로 부름  
⚫CPU 및 메모리 는 각각 자원 유형을 지니며 자원 유형에는 기본 단위를 사용

⚫리소스 요청 설정 방법  (최소)
➢ spec.containers[].resources.requests.cpu  
➢ spec.containers[].resources.requests.memory

⚫리소스 제한 설정 방법  (최대)
➢ spec.containers[].resources.limits.cpu  
➢ spec.containers[].resources.limits.memory


| 자원유형 | 단위        |
| -------- | ----------- |
| CPU      | m(millicpu) |
| Memory   | ... Ti, Gi, Mi, T, G, M,K            |

![](https://i.imgur.com/ORIne2t.png)



```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        resources:
          requests:
            memory: "200Mi"
            cpu: "1m"
          limits:
            memory: "400Mi"
            cpu: "2m"
        name: nginx
        ports:
        - containerPort: 80
```

## 시스템 리소스 요구사항 제한설정

- limit range : 네임스페이스에서 포드 또는 컨테이너별로 리소스 제한하는 정책
- 클러스터 다같이 쓰는데 개발자들이 컨테이너로 용량 너무 많이 잡아먹을때  
-            
	- 네임스페이스당 만들 수 있음
	- 포드나 컨테이너 최소 및 최대 컴퓨터 리소스 사용량 제한 
![](https://i.imgur.com/vHTCOex.png)
![](https://i.imgur.com/uV2oX6w.png)

- 옵션활성화
sudo vim /etc/kubernetes/manifests/kube-apiserver.conf

k create ns limitrange-demo

k create -f https://k8s.io/examples/admin/resource/limit-mem-cpu-container.yaml -n limitrange-demo

k describe limitranges -n limitrange-demo
![](https://i.imgur.com/R7RhcvD.png)


k apply -f https://k8s.io/examples/admin/resource/limit-range-pod-1.yaml -n limitrange-demo

k create namespace limitrange-demo

만약 제한된 범위에서 생성한다면?

``` # 넘어가는경우 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
  namespace: limitrange-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        resources:
          requests:
            memory: "200Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1"
        name: nginx
        ports:
        - containerPort: 80
```

디플로인느 생성되지만 레플리카셋에서 오류남

## 네임스페이스별 리소스 총량제한방법 리소스쿼타

이전까지는 네임스페이스 총량이 아니라 정책 이제는 리소스 총량제한
![](https://i.imgur.com/MqGNoxH.png)


```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
  namespace: quota-mem-cpu-example
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

k describe resourcequotas -n quota-mem-cpu-example
kubectl apply -f https://k8s.io/examples/admin/resource/quota-mem-cpu-pod.yaml --namespace=quota-mem-cpu-example 하고 두번째 팟을 띄울려고하면 안됨
kubectl apply -f https://k8s.io/examples/admin/resource/quota-mem-cpu-pod-2.yaml --namespace=quota-mem-cpu-example(작동안됨)

## 데몬셋

- 데몬, 윈도우 백그라운드활동하는 프로세스(서비스)
- k8s 서비스형태의
- 호스트패스와 데몬셋 자주 사용함 왜냐면 데몬셋은 하나의 노드에 하나의 포드만을 구성되므로 노드 모니터링하고자
- 개수를 지정하지 않아도됨 그냥 데몬셋 만들면 노드마다 하나씩 실행됨 대표적으로 kube-proxy
- 마스터에는 app 노드, tolearation 옵션주면 pod올려도된다하는 옵션으로 보면된다
- rsㅘ 거의비슷
- 호스트패스를위해 실행됨


```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: http-go-daemon
spec:
  selector:
    matchLabels:
      app: http-go-daemon
  template:
    metadata:
      labels:
        app: http-go-daemon
    spec:
      tolerations:

      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule      
      # 마스터에도 설치죄도록
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
        operator: Exists

      
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      containers:
      - name: http-go
        image: gasbugs/http-go
```

k get po -o wide를 하면 master에도 노드가 떠여잇다


## 테인트 

- taint : 오점을 남기다, 더럽히다
- 테인트는 노드에 설정해 노드에 어떤 오점을 남김
- 톨러레이션은 용인하다 견인하다
- 노드가 갖고 있는 이 오점을 용인하는 옵션을 포드에 설정 -> 노드에 배치
![](https://i.imgur.com/Ka1yDOI.png)

- 케이스1. 테인트가 있으면 톨로레이션을 적용해서 포드가 뜨게함
- 케이스2. 오점이 없으니 참을게 없음(배포)
- 케이스3. 테인트(오점)이 있으니 참지 않는다
- 톨로레이션은 노드 셀렉터등과 다르게 다른 오점없어도 배치가 될 수 있다
- 만약 오점이 있는 경우 톨로레이션 세팅 필수적으로 필요 

조회
```
k get nodes -o json | jq '.items[].spec.taints'
```

노드에 테인트 추가

```
k taint nodes node01 key=value1:NoExecute
```
- 톨러레이션이 없는포드
	- 톨러레이션이 업는 포드는ㅇ ㅓ떻게 동작하는지 확인
	- 데몬셋 yaml파일 생성하고 클러스터에 배포
	- 테이느와 일치하지 않아서 배ㅗ 실패
	- 따라서 적절한 톨로레이션이 있느 포드를 만들어야함 두 노드에 모두 배포하기 위해 




k taint nodes worker-1 key=value:NoExecute 추가 
k taint nodes worker-w key=value:NoExecute 추가 
k get nodes -o json | jq '.items[].spec.taints'
k taint nodes worker-1 key=value:NoExecute- 삭제
![](https://i.imgur.com/AEpU5F3.png)
포드에서 해당 키가 있으면 exists.. no schedule

- noschedule 옵션 : 포드가 더 이상 스케줄되지않도록함 사용자가 큐브api에  포드를 배치해달라고 하는데 저 테인트가 있으면 노드가 스케줄되지않음 대신에 노스케줄이라는 옵션은 이미 있는 아이들에게는 영향 x
- noexcute 옵션 : 이미 있는 경우 착출됨 기존 영향 o 
- noschedule은 아닌데 가능하면 배치안받고 싶어 오변 prefernoschedule 


```
# no-ds.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: no-ds
spec:
  selector:
    matchLabels:
      app: no-ds
  template:
    metadata:
      labels:
        app: no-ds
    spec:
      containers:
      - name: nx
        image: nginx
```

워커노드 두대에 모두 배치가되지않음 테인트를줘서.

```
# toleration-ds.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: toleration-ds
spec:
  selector:
    matchLabels:
      app: toleration-ds
  template:
    metadata:
      labels:
        app: toleration-ds
    spec:
      containers:
      - name: nx
        image: nginx
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
        effect: "NoSchedule"
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoExecute"
```

## 스태틱포드

/kubernetes/m
anifests/

## 수동 스케줄링

- 원하는 노드에 배치
- gpu, blockchain관련해서 몇개 서버에만 그래픽카드 설치 원하는 노드에 배치(모든서버에 그래픽카드 설치할 수 없으므로)
- pod yaml양식에 노드 이름을 저어주면 된다 또는 셀렉터, 레이블링을 노드에 한다. `kubectl label node <레이블 이름> gpu=true ` , nodeSelector명시
```
apiVersion: v1
kind: Pod
metadata:
  name: http-go
spec:
  nodeName: "worker-2"
  containers:
  - name: http-go
    image: busybox
```


## 멀티플 슼줄러

- 기본 스케줄러가 사용자 필요에 맞지 않으면 사용자 고유의 스케줄러 구현해 사용
- go 로 작성하고 이게 편하다면 작성하면됨


## 오토 스케일링 HPA

- 수동 스케일링을 자동
- 포드 스케일링 두가지방법
	- HAP 포드자체 복제해 처리할수있는 포드 개수 늘리는 ㅂ아법(포드개수)
	- VPA 리소스를 증가시킴(cpu, memory, vertical pod autoscaler)
	- ca : 번외로 클러스터 자체를 늘리는 방법(노드추가)
- HPA : k8s 기본 오토 스케일링 기능 내장
	- cpu 사용률은 모니터링하여 실행된 포드의 개수를 늘리거나 줄임 
- vpa -> glcoud에서 클러스터 자동확장처리 기능 고식 k8s에서 제공안되나 클라우드 서비스에서 제공
- vpa 수직형 pod 자동확장

```
k autoscale deploy my-app --max 6 --min 4 --cpu-eprcent 50

pod의 개수를 가지고있는 rs 설정
최대 6개, 최소4개 
cpu리소스 50%넘어가면 스케일링한다는 뜻 같은 내용으로 
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: default
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: myapp
  targetCPUUtilizationPercentage: 30 

```

좋은 예제
- 사용자가 요청할때마다 cpu사용량이 올라가는 컨테이너
- cpu올라가므로 자동으로 스케일링 진행되는것 확인가능

```
apiVersion: apps/v1

kind: Deployment

metadata:

  name: php-apache

spec:

  selector:

    matchLabels:

      run: php-apache

  template:

    metadata:

      labels:

        run: php-apache

    spec:

      containers:

      - name: php-apache

        image: registry.k8s.io/hpa-example

        ports:

        - containerPort: 80

        resources:

          limits:

            cpu: 500m

          requests:

            cpu: 200m

---

apiVersion: v1

kind: Service

metadata:

  name: php-apache

  labels:

    run: php-apache

spec:

  ports:

  - port: 80

  selector:

    run: php-apache

```

```shell
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```


```shell
kubectl run -i --tty load-1 --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```



---
- ## 📑 Ref

- 인프런 - devops를 위한 쿠버네티스

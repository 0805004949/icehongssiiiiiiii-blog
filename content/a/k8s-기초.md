---
title: k8s-기초
date: 2023-05-05T17:04:02
lastmod: 2023-05-18T16:24:07
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---


## 1. 구글 클라우드 엔진에서 k8s 실행해보기


> [!summary] 
> GKE 엔진 활성화 -> 클라우드 콘솔 클러스터 생성 -> 레플리카 5개 생성 -> 네트워크 노출 -> 확인


GKE 구글 클라우드 엔진 활성화 > 클라우드 콘솔 

```sh 
gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project no1je1233-2023-05-05
```

레플리카 5개 
```sh 
kubectl create deploy --image=consol/tomchat-7.0 --replicas=5
```

```sh
kube get nodes
```



네트워크 외부로 노출해야 외부와 통신가능

```sh
kubectl expose deploy tc --type=LoadBalancer --port=80 --target-port=8080
```



네트워크 노출 확인
```sh
kubectl get pods, svc

```

![](https://i.imgur.com/f48CyVy.png)


- 컨테이너 5개가 떠어있고 서비스가 tc이게 외부로 노출 

- 로드밸런서라는 타입은 클라우드에서 밖에 되지 않음 
- `34.122.28.45` 외부 아이피 접근시 아래와 같이 톰캣 서버로 접속되는데 
이때 트래픽이 5개의 레플리카로 로드밸런싱됨
![](https://i.imgur.com/1JAkASp.png)



![](https://i.imgur.com/aV1zMmc.png)

![](https://i.imgur.com/Pqy4SNH.png)


![](https://i.imgur.com/u0C7CNM.png)



어느 컨테이너에 들어왔는지 확인가능 
(admin/admin이 비밀번호)


컨테이너 가 어디에 배치되어있는지 파드에 대한 옵션은 owide옵션으로 파드에 대해 더 잘알 수 있다
```sh
kubectl get pods -owide
```

내 컨테이너가 어디에 정확히 배포되었는지 알게됨


> [!note] 키워드
> 컨테이너, 노드, 파드, 서비스

현재 노드의 개수 3개




![](https://i.imgur.com/CYLsTjn.png)


hrsx, 5vpl에  노드에 각 각 두개의 톰캣 컨테이너가 있고 hp11 노드에는 톰캣 컨테이너 한개



## 2.우분투 환경에서 클러스터 구성


> [!summary] 
> 1. 마스터 인스턴스 1개, 워커 인스턴스 2개 생성 
> 2. 모든 인스턴스 3개에 swap off 
> 3. 모든 인스턴스에 클러스터 런타임, kubeadm 설치 
> 4. 모든 인스턴스에 넷필터 설정(도커 런타임 환경에서는 필요없지만) 
> 5. 마스터 인스턴스에 노드 초기화 & 토큰 받기(쉘에뜸) -
> 6. 마스터 인스턴스에서 유저설정(쉘에뜸) -
> 7. 워커 인스턴스에서 노드 조인(마스터 인스턴스에서 뜬 토큰으로) -
> 8. 현재상태 ready가 아니므로 네트워크 배포를 위한 adon ciliium설치 -> 노드 상태 확인 
> 9. 톰캣 컨테이너 배포 -> 콤캣 외부로 노출 
> 10. 컨테이너 상태 확인



- GCP 인스턴스 
- 마스터 노트 1대
- 워커노드 2대
- 스왑 : 메모리 사용하는데 자주 사용하지 않는 메모리, 오랫동안 안ㅆ느느 경우 이 메모리에 있는 정보 디스크로 보냄. 메모리는 용량대비 비싸서 메모리를 조금 더 효율적으로 이용하기 위해 디스크로 이동시킴. 이러한 과정을 swap이라고함 -> K8S에서는 비활성화를 권장
- K8S에서는 좋지 않은 기능. 서버 한대에서 스왑은 문제가 되지 않지만 클러스터 입장에서 메모리가 안남아 있는데 메모리가 남아있는 것 처럼 보일 수도 있음. 가용한도가 없는거나 마찬가지인데 메모리에 계속 할당하게 되므로 문제 생길 수 있음. 조금 더 투명하게 

- GKE들어가서 vm UBUNTU 20.0.4/부팅 디스크  변경 표준활용디스크/100GB( 
가상머신은 2CPU, 4GB 메모리 사용 필요)

sudo kueadm reset

### 2-1. containerd 설치하기(컨테이너 런타임구성)

- GKE -> VM생성(ubuntu 20.0.4 x86/64, 100GB, 표준영구디스크) * 3개 인스턴스 생성(master-1, worker-1, worker-2)
- 각 각 인스턴ㅅ느에서 swap기능을 disable
```sh 
sudo swapoff -a # 현재 시스템에 적용(리부팅하면 재설정 필요) 
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab # 리부팅 필수
```


- 컨테이너 런타임 구성 (containerd 설치)

 원래는 도커 엔진을 설치해서 런타임을 구성했지만 호환성 문제가 있기 때문에 직접 containerd를 설치해서 진행 

![](https://i.imgur.com/bntXzlH.png)

```sh 
# Using Docker Repository
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

# containerd 설치
sudo apt update
sudo apt install -y containerd.io
# sudo systemctl status containerd # Ctrl + C를 눌러서 나간다.

# Containerd configuration for Kubernetes
cat <<EOF | sudo tee -a /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
EOF

sudo sed -i 's/^disabled_plugins \=/\#disabled_plugins \=/g' /etc/containerd/config.toml
sudo systemctl restart containerd

# 소켓이 있는지 확인한다.
ls /var/run/containerd/containerd.sock

```



### 2-2. kubeadm, kubelet, kubectl 설치


kubelet = 서비스(데몬)
kubelet이 containerd 컨트롤함

- kubeadm : 클러스터를 부트스트랩하는 명령어. 클러스터 초기화하고 관리함
- kubelet : 클러스터 모든 머신에서 실행되는 파드와 컨테이너 시작과 같은 작업을 수행하는 컴포넌트. 데몬으로 동작하며 컨테이너 관리
- kubectl : 클라이언트로써 마스터노드와 통신하기 위한 커맨드 라인 유틸리티

- 마스터 노드 초기화 
	- 마스터 노드에 깃발 꽂는 작업
	- 마스터 노드 초기화 하면 토큰이 나눠짐. 이 토큰 나눠지면 
- clilim 파드 네트워크 플러그인(에드온) 설치



```sh
cat <<EOF > kube_install.sh
# 1. apt 패키지 색인을 업데이트하고, 쿠버네티스 apt 리포지터리를 사용하는 데 필요한 패키지를 설치한다.
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# 2. 구글 클라우드의 공개 사이닝 키를 다운로드 한다.
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# 3. 쿠버네티스 apt 리포지터리를 추가한다.
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# 4. apt 패키지 색인을 업데이트하고, kubelet, kubeadm, kubectl을 설치하고 해당 버전을 고정한다.
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
EOF

sudo bash kube_install.sh
```

kubeadm이 제대로 깔렸는지 확인
`kubeadm version`


도커엔진으로 설치하면 넷필터 브릿지도 직접 설치되는데 containerd로 런타임을 설정하기 때문에 넷필터 브릿지도 직접
넷필터 브릿지 (=k8s 네트워크 설정)

```sh 
sudo -i (쉘접속)
modprobe br_netfilter 
echo 1 > /proc/sys/net/ipv4/ip_forward 
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables 
exit(쉘나가기)
```


### 2-3. 마스터 노드 초기화
```sh
sudo kubeadm init
```
마스터 인스턴스에서 실행 실행 후에 아래와 같은 명령어가 나오면 따라서 진행

![](https://i.imgur.com/JCmKg1k.png)

이렇게 떴다면 

넷필터 브릿지 설정이 제대로 되어있는지 확인할 것
![](https://i.imgur.com/5JdI0wI.png)


- 이때 마스터 노드 초기화시에 나오는 조인방법은 워커노드에서 실행하면 된다!(3번과정) -> sudo 권한으로 워커 인스턴스에서 실행
- 클러스터를 시작하기 위해서는 유저를 설정해야하므로 아래 1번 과정 진행
```sh
Your Kubernetes control-plane has initialized successfully!

# 1) 유저 설정
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

# 2) 파드 네트워크 설정
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

# 3) 워커 노드 조인 방법
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.142.0.3:6443 --token xcs79e.vnernooln6yyimtv \
        --discovery-token-ca-cert-hash sha256:c9f8642746515eadc28e72c687eface2fa64da93ddca5a30b4ccf931dbcce839
```

이후 `kubectl get nodes. 로 마스터 노드 확인

> [!tip] token 재발급
> ```sh
>   토큰 리스트 확인하기: sudo kubeadm token list
>   토큰 재발급하기: sudo kubeadm token create --print-join-command
> ```






### 2-4. 워커노드와 마스터 노드 join(연결)

> [!error]  init이나 join을 잘못설정 했다면
> `sudo kubeadm reset`

### 2-5. 파드 네트워크 설치 ad-on 

- 보통 callico나 cillium으로 구성
- Cillium은 CNCF프로젝트
- 마스터 노드에서 다음 명령어 실행하면 앞서 구성한 유저설정을 통해 클러스터에 cillium 인스톨

![](https://i.imgur.com/Py85m68.png)


```sh
$ curl -LO https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz 

$ sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin 
$ rm cilium-linux-amd64.tar.gz 

$ cilium install
```

- 마스터 노드에서 인스톨 확인 `cilium status`

![](https://i.imgur.com/4PPjYh6.png)

모든 노드의 상태가 start로 변경됨!

### 2-6. app 배포와 k8s 아키텍처




```sh 
kubectl create deploy tc --image=consol/tomcat-7.0 --replicas=5
```


type = NodePort

```sh 
kubectl expose deploy tc --type=NodePort --port=80 --target-port=8080
```

확인 `kubectl get pod,svc`

external IP는 따로 받지 않고 

https://m.blog.naver.com/onlywin7788/221845944242


저 포트로 오픈한다![](ASSETS/Pasted%20image%2020230508175727.png)

현재 노드의 내부 IP

```sh
curl 10.182.0.2:30627
curl 10.182.0.3:30627
curl 10.182.0.4:30627
```

## 3. 구글 클라우드 엔진에서 k8s 실행 - 설명

GKE 구글 클라우드 엔진 활성화 > 클라우드 콘솔에서 베일에 쌓여있는 마스터노드로 부터 응답을 받았음
![](https://i.imgur.com/5j34qjo.png)

로드밸런서는(클러스터 바깥쪽에 위치) 외부에서 트래픽이 들어오면 로드밸런서에서 부하 분산을 시켜준다. 이 포트 번혼는 3000-32767 

## 4. 우분투 환경에서 클러스터 구성 - 설명

마스터 노드가 베일에 쌓여있지 않고 워커노드가 두배.
kubectl이 마스터 노드에 있음(관리자)
톰캣을 5개 배치.
로드밸런서가 배치 될 수가 없고 외부 포트는 열려있

proxy가 있어서 톰캣으로 부하분산.
마스터에는 컨테이너가 없고 서비스 배치되지 않음

![](https://i.imgur.com/eakxZLo.png)




## 5.  k8s 자격증 준비 관련



![](https://i.imgur.com/L06DU3o.png)
![](https://i.imgur.com/lh365rO.png)
![](https://i.imgur.com/CbYW0Bi.png)

- `--dry-run=client -o yaml` : 실제 실행은 되지 않고 yml파일생성



## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
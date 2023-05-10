
---
title: k8s이해
date: 2023-05-05T15:44:38+09:00
lastMod: 2023-05-09T15:44:38+09:00
tags: ["2023-k8s-bootcamp","k8s","GKE","kr", "blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---


#2023-k8s-bootcamp 

##  GKE 쿠버네티스 들어가기







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

![[Pasted image 20230506032314.png]]

- 컨테이너 5개가 떠어있고 서비스가 tc이게 외부로 노출 

- 로드밸런서라는 타입은 클라우드에서 밖에 되지 않음 
- `34.122.28.45` 외부 아이피 접근시 아래와 같이 톰캣 서버로 접속되는데 
이때 트래픽이 5개의 레플리카로 로드밸런싱됨
![[Pasted image 20230506033149.png]]


![[Pasted image 20230506033306.png]]
![[Pasted image 20230506033413.png]]

![[Pasted image 20230506033442.png]]


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

![[Pasted image 20230506035018.png]]

![[Pasted image 20230506035100.png]]
![[Pasted image 20230506034931.png]]

hrsx, 5vpl에  노드에 각 각 두개의 톰캣 컨테이너가 있고 hp11 노드에는 톰캣 컨테이너 한개

## 우분투 환경에서 클러스터 구성

- GCP 인스턴스 
- 마스터 노트 1대
- 워커노드 2대
- 스왑 : 메모리 사용하는데 자주 사용하지 않는 메모리, 오랫동안 안ㅆ느느 경우 이 메모리에 있는 정보 디스크로 보냄. 메모리는 용량대비 비싸서 메모리를 조금 더 효율적으로 이용하기 위해 디스크로 이동시킴. 이러한 과정을 swap이라고함 -> K8S에서는 비활성화를 권장
- K8S에서는 좋지 않은 기능. 서버 한대에서 스왑은 문제가 되지 않지만 클러스터 입장에서 메모리가 안남아 있는데 메모리가 남아있는 것 처럼 보일 수도 있음. 가용한도가 없는거나 마찬가지인데 메모리에 계속 할당하게 되므로 문제 생길 수 있음. 조금 더 투명하게 

kubelet = 서비스(데몬)
kubelet이 containerd 컨트롤함

- kubeadm : 
- kubelet : 노드에서 데몬으로 마스터 ..
- kubectl : 클라이언트로써 마스터노드오ㅏ 통신
- 넷필터 브릿지 : 컨테이너 네트워크 통신
- 마스터 노드 초기화 
	- 마스터 노드에 깃발 꽂는 작업
	- 마스터 노드 초기화 하면 토큰이 나눠짐. 이 토큰 나눠지면 
- clilim 파드 네트워크 플러그인(에드온) 설치
![](https://i.imgur.com/1vskyko.png)

- GKE vm 20.0.4/표준활용디스크/100GB

sudo kueadm reset

## kubeadm 설치하기 <a name="introduction"></a>

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


도커엔진으로 설치하면 넷필터 브릿지도 직접 설치되는데 containerd로 런타임을 설정하기 때문에 넷필터 브릿지도 직접
넷필터 브릿지 (=k8s 네트워크 설정)

```sh 
sudo -i modprobe br_netfilter echo 1 > /proc/sys/net/ipv4/ip_forward echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables exit
```


## 파드 네트워크 설치 adon 

보통 callico나 cillium으로 구성
Cillium은 CNCF프로젝트




## app 배포와 k8s 아키텍처




```sh 
kubectl create deploy tc --image=consol/tomchat-7.0 --replicas=5
```


type = NodePort

```sh 
kubectl expose deploy tc --type=NodePort --port=80 --target-port=8080
```

external IP는 따로 받지 않고 

https://m.blog.naver.com/onlywin7788/221845944242


저 포트로 오픈한다![](ASSETS/Pasted%20image%2020230508175727.png)

현재 노드의 내부 IP

```sh
curl 10.182.0.2:30627
```



## 📑 Ref
- 인프런 - devops를 위한 쿠버네티스
---
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
title: etcd
date: 2023-05-15T15:21:00
lastmod: 2023-05-15T15:42:35
---


## etcd

 - 오픈소스  키와 밸류 데이터베이스
 - k8s는 이 데이터 사용 
 - 다중키도 사용가능하네
- ![](https://i.imgur.com/Liod6zI.png)

- client가 따로 동작하는게 아니라  kubeapi서버에 client가 따로 포함되어 잇음

## 설치

```sh 
wget https://github.com/etcd-io/etcd/releases/download/v3.5.0/etcd-v3.5.0-linux-amd64.tar.gz
tar -xf etcd-v3.5.0-linux-amd64.tar.gz
cd etcd-v3.5.0-linux-amd64
ls


# 출력
Documentation      README-etcdutl.md  READMEv2-etcdctl.md  etcdctl
README-etcdctl.md  README.md          etcd                 etcdutl
```

환경변수 세팅

```sh 
sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 \ --cacert=/etc/kubernetes/pki/etcd/ca.crt \ --cert=/etc/kubernetes/pki/etcd/server.crt \ --key=/etc/kubernetes/pki/etcd/server.key \ get / --prefix --keys-only
```

키값

```sh 
sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 \
                             --cacert=/etc/kubernetes/pki/etcd/ca.crt \
                             --cert=/etc/kubernetes/pki/etcd/server.crt \
                             --key=/etc/kubernetes/pki/etcd/server.key \
                             put key1 value1
```

조회 
```sh 
sudo ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 \ --cacert=/etc/kubernetes/pki/etcd/ca.crt \ --cert=/etc/kubernetes/pki/etcd/server.crt \ --key=/etc/kubernetes/pki/etcd/server.key \ get key1
```




![](https://i.imgur.com/lAfNhrc.png)



## 📑 Ref



- 인프런 - devops를 위한 쿠버네티스

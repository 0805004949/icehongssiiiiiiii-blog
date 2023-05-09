
---
title: useradd와 adduser차이
date: 2023-04-23
lastmod: 2023-04-23
tags: ["2023-k8s-bootcamp","linux" ,"CS","OS","KR", "blog-published"] 
categories: ["posts"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---

##  🍊 TL;Dr

> adduser는 유저 홈 디렉토리와, 다른 설정들을 쉽게 할 수 있도록 도와주고  기본 쉘이 /bin/bash로 만들어져있다 하지만 몇몇 리눅스 배포판에서 동작하지 않을 수 있다. 백엔드에서는 결국 `useradd` 를 사용하고 있기 때문에 난` user add -m icehongssii -s /bin/bash`로 외워서 쓰고있다



##  🍊 Contents

### 1. useradd

-   useradd 명령어의 gernal syntax는 다음과 같다.

```bash
useradd [OPTINOS] USERNAME
$ user add -m icehongssii -s /bin/bash #홈디렉토리 생성후 bash 쉘 사용
```

-   root 혹은 sudo 권한을 가진 자만이 useradd 명령어로 사용자를 생성할 수 있다.
    -   useradd 를 실행하면 OPTIONS과 /etc/default/useradd 에 설정된 기본값에 따라 사용자 계정을 생성
-   대부분의 리눅스 환경에서 별다른 옵션없이 useradd으로 계정을 생성하면 해당 username의 홈 디렉토리는 생성되지 않는다.
    -   따라서, -m(—create-home) 옵션을 이용하여 계정생성시 /home/username 의 홈 디렉토리를 생성할 수 있게 한다.



### 2. adduser

- 사용자에게 친숙한 UX 제공하고, 비밀번호와 유저 홈 디렉토리까지 생성해준다
- it comes as a soft link or a pearl script so it won’t work with some linux distribution 몇 몇의 리눅스 배포판에서는 동작되지 않을 수 있다.
- 백엔드로는 `useradd` 를 사용한다it utilizes `useradd` in the backend.


## 📑 Ref

[https://superuser.com/questions/547966/whats-the-difference-between-adduser-and-useradd](https://superuser.com/questions/547966/whats-the-difference-between-adduser-and-useradd)

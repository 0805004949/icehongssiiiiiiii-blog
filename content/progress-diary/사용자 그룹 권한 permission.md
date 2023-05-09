---
title: 사용자 그룹 권한 permission
date: 2023-04-14
lastMod: 2023-04-14
tags: ["2023-k8s-bootcamp","linux" ,"CS","OS","KR", "blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---

![](https://i.imgur.com/MSQbDeV.png)

## 🍊 Tl;Dr 
- permission -> 10개 알파벳
- 파일의 소유권 = user
- `chmod` 명령어 사용
- 한편 리눅스에서는 실행권한 = 실행 가능한 파일이어야함, 실행권한은 실행간으한 파일만 준다 윈도우랑 다름(그냥 텍스트파일일 경우도 내가 소유자인데 나에게 실행권한 없을 수도 있음)

![](https://i.imgur.com/oXTAxCg.png)



## 1. 8진표기

![](https://i.imgur.com/hgguuQN.png)


```
chmod 777 test 
```
- test에 777권한(모든사용자그룹 read, write, execute권한 허용)

```
chmod 754 test
```
- user : read, execute, write
- group : read, execute
- 다른 사용자 : read


## 2. 의미표기 

`chodmod u(u,o,g)+(권한을 지우고 싶다면 + x(r,w) 파일명`

```jsx
chomod og+x+w-r *.sh
```
- 해당파일에서  다른유저와 그룹에게 실행권한, 작성권한 추가하고 읽기 권한 제외 

```
chmod go+rx <dir>
```
- 해당 디렉토리를 모두에게 읽게 가능하게 만들고 싶다면 


## 📑 Ref

- 인프런 - 리눅스 입문 - 개념으로 탄탄히!!
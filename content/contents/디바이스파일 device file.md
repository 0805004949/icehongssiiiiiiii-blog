---
title: 디바이스파일 device file
date: 2023-04-12
lastMod: 2023-04-12
tags: ["2023-k8s-bootcamp","linux" ,"CS","OS","KR", "blog-published"] 
categories: ["TIL"] # <!--하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---

## 디바이스 파일

운영체제가 하드웨어 자원관리하고 운영체제위에 어플리케이션 존재한다.

특정하드웨어가 잘 작동하려면 이는 하드웨어를 관리하는 운영체제(리눅스) 연결되어있어야한다. 
이때 운영체제에 하드웨어를 관리하는 하드웨어 소프트웨어가있다(즉, 하드드라이버를 위한 소프트웨어=디바이스 드라이버)가있다. 


- 이 디바이스 드라이버가 하드웨어를 관리한다. 해당 드라이버는 운영체제랑 협업을 한다(운영체제가 명령하면 디바이스가 작동한다든가)

- 디바이스 드라이버는 운영체제 위에서 동작하는 application과 무언가를 주고 받을 수 있는데 이 때 필요한 것이 디바이스 파일이다. 

이 디바이스파일은 디바이스드라이버 연결될때 어플리케이션에서 명령을 내릴 수 있는데, 이 때 자원관리 제한을 위해 커널에서 2가지 모드로 나눔

- 유저모드(application) :  어플리케이션  -> 디바이스드라이버 -> 디바이스로 명령을 전달할 수 있게 한다. 유저가 사용할 수 있는 영역을 제한적으로 둔다 
- 커널모드(os) : 커널과 디바이스드라이버가 동작해서 디바이스 관리. 즉 모든 자원 접근 가능


> 디바이스파일은 통로역할로, 운영체제 안에있는 디바이스드라이버와 애플리케이션연결해주는게 디바이스파일


디파이스파일 ->  디바이스에 특성에 따라 적합한걸 디바이스드라이버가 만들어냄

## 📑 Ref

- 인프런 - 리눅스 입문 - 개념으로 탄탄히!!
- [디바이스 드라이버란?](INBOX/디바이스%20드라이버란?.md)

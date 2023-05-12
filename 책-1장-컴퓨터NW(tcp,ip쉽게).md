---
tags: ["2023-k8s-bootcamp","CS","책-TCP/IP쉽게배우기","blog-published"] 
title: 책-1장-컴퓨터NW(tcp,ip쉽게)
date: 2023-05-12T16:00:33
lastmod: 2023-05-12T17:38:09
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---
## 1. 네트워크란?

## 2. 네트워크 종류
## 3. 네트워크의 역할

- 네트워크가 왜 중요?

## 4. 서버와 클라이언트

## 5. 패킷교환

## 6. 계층모델 

![](https://i.imgur.com/4nMKaOG.png)


- 어플리케이션 계층 : 서비스 제공하는 부분으로 서비스의 내용을 결정  
- 그외의 나머지 3개 계층 : 데이터를 전달하는 통신 기능
![](https://i.imgur.com/evqZW7h.png)

출처: https://study.com/academy/lesson/layers-in-the-tcp-ip-network-stack-function-purpose.html

| 계층              | ㅇ                                                    | 역할                                                                                                                     |
| ----------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| 어플리케이션 계층 | 상대에게 보낼 데이터(이하application data)            |                                                                                                                          |
| 트랜스포트 계층   | application data + transport header                   | application data 전송하기 좋게 작게 쪼갠 뒤 이 데이터 결합순서+ 이 데이터 받을 프로그램 식별할 수 있는 번호데이터 덧붙임 |
| 인터넷 계층       | application data + transport header + Internet header | Internet header   = 목적지 컴퓨터 식별할 수 있는 번호                                                                    |
| 네트워크 인터페이스 계층                  |  application data + transport header + Internet header + network interface header    + network interface trailer                                                 | 유선 LAn에서 데이터 보내는데 필요한 정보                                                                                                                           |


## 7. TCP/IP 4계층

![](https://i.imgur.com/TRfmwc0.png)
출처 : https://www.researchgate.net/figure/Client-Server-Media-For-Data-Transfer-21-TCP-IP-4-has-four-layers-Application_fig1_353222045



## 8. 프로토콜

> [!tip] keywords
> 

- 프로토콜 = 규약
- 프로토콜 = 컴퓨터끼리 커뮤니케이션 할 때 필요한 통신규약으로 어떤 데이터를 어떻게 보낼지 등을 정하는 것을 의미함. 또는 전송이 안되었을 때는 어떻게 처리할지에 대한 것도 포함되어 있음

| TCP/IP계층       | 프로토콜                                                                     |
| ---------------- | ---------------------------------------------------------------------------- |
| 어플리케이션계층 | HTTP라는 프로토콜에 따라 동작하도록 만들어진 웹브라우저                      |
|    트랜스포트, 인터넷계층              | TCP/IP라는 프로토콜따라 동작하도록 만들어진 OS 내장통신 프로그램(윈도우, 맥) |
| 네트워크 인터페이스 계층                 | 이더넷이라는 프로토콜에 따라 동작하도록 만들어진 NW 어댑터용 디바이스 드라이버                                                                              |

- 당연하지만 프로토콜에 맞게 동작하도록 만들어진 프로그램이나 장비들이 존재하고 이들이 서로 약속된 방식으로 잘 동작할 때 통신이 원활함

- 웹페이지 볼떄 프로토콜 조합

각 네트워크 프로토콜은 소속된 계층에 맞게 그 역할이 전문화 되고 세분화 되어있음.  그래서 하나의 통신 전체적으로 성공시키기 위해서는 프로토콜 잘 조합해야함 

| 계층                     | 프로토콜 | 설명                                     | 주고 받는 데이터 구조 |
| ------------------------ | -------- | ---------------------------------------- | --------------------- |
| 어플리케이션 계층        | HTTP     | 웹 페이지 보기 위한 프로토콜             | HTTP데이터            |
| 트랜스포트 계층          | TCP      | 데이털르 확실하게 전달하기 위한 프로토콜 | TCP헤더               |
| 인터넷 계층              | IP       | 일반적인 통신에 사용하는 프로토콜        | IP헤더                      |
| 네트워크 인터페이스 계층 | 이더넷   | 유선 LAN으로 데이터 전송하기 위한 규격   |           이더넷 헤더            |

- 어플리케이션 계층에서 네트워크 인터페이스 계층으로 지나갈 때 마다 목적지 정보와 같은 데이터가 헤더 형태로 덧붙여지고 (캡슐화 ) 유선 LAN을 통해 전송되어 목적지 컴퓨터에서 데이터를 받아 반대로 Network Interface에서   Application 계층으로 지나갈 때 헤더가 하나씩 뗴어진다 (캡슐화의 반대 과정)




- 대표적인 
![](https://i.imgur.com/VixDbZJ.png)


## 9. 인터넷 




## 2. OSI 7계층과 TCP/IP 5계층 차이?

##

## 📑 Ref
- 책 - TCP/IP 쉽게, 더 쉽게
- https://www.guru99.com/tcp-ip-model.html
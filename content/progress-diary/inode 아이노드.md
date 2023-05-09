---
title: inode 아이노드
date: 2023-05-09T15:44:38+09:00
lastMod: 2023-05-09T15:44:38+09:00
tags: ["2023-k8s-bootcamp","linux" ,"CS","OS","KR", "blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---


> inode = 리눅스에서 파일 메타데이터 저장되어있는 데이터 구조

## inode 

- index nodes의 줄임말

- 파일명과 데이터를 제외하고 파일에 관한 정보를 저장하는 파일데이터구조 ([An inode](http://www.linfo.org/inode.html) is a file data structure that stores information about any Linux file except its name and data.)

- Data is stored on your disk in the form of fixed-size blocks. If you save a file that exceeds a standard block, your computer will find the next available segment on which to store the rest of your file. Over time, that can get super confusing. That’s where inodes come in. While they don’t contain any of the file’s actual data, it stores the file’s metadata, including all the storage blocks on which the file’s data can be found.

- 데이터는 고정 크기 블록의 형태로 디스크에 저장됩니다. 표준 블록을 초과하는 파일을 저장하면 컴퓨터는 나머지 파일을 저장할 수 있는 다음 사용 가능한 세그먼트를 찾습니다. 시간이 지나면 매우 혼란스러워질 수 있습니다. 바로 이때 inode가 등장합니다. inode  파일의 실제 데이터는 포함하지 않지만, 파일의 데이터를 찾을 수 있는 모든 저장 블록을 포함한 파일의 메타데이터를 저장합니다. 


![kl0Q9H8.png](https://i.imgur.com/kl0Q9H8.png)

## 📑 Ref

- 인프런 - 리눅스 입문 - 개념으로 탄탄히!!
- http://www.linfo.org/inode.html
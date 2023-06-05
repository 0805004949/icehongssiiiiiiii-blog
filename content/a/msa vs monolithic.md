---
title: msa vs monolithic
date: 2023-05-04T20:32:15
lastmod: 2023-05-15T11:54:33
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 

categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
---

- monolithic에서는 서버 한 대에 세가지 기능이 들어 있지만
- MSA에서는 서버 세 대에 한 개의 기능이 들어가있음

## monolithic architecture


![[Pasted image 20230505161428.png]]

인기있는 서비스는 사용자가 많이 있어서 로드 밸런싱(트래픽 부하분산)
monolithic에서는 서비스를 분리할 수가 없음 기존에는 인기가 있든 없든 복제를 함 그래서 그대로 복제하기에 분산이 ㅇ되지 않음


![[Pasted image 20230505161633.png]]

![[Pasted image 20230505161752.png]]


## MSA

![[Pasted image 20230505161928.png]]



![[Pasted image 20230505162307.png]]

팀당 15분당 1번씩 배포할 수 이쓸 정도로
![[Pasted image 20230505162333.png]]


##  그런데 분산시스템

서버 한대 모든 기능 담당했는데 이제는 MSA 떄문에 서버 하나에 어플리케이션 하나를 만들기 위한 200개의 컨테이너가 들어가 필요하다. 이를 32개의 서버에 띄우면 총 6400개의 컨테이너가 실행되는건데 이 컨테이너 배포, 모니터링을 어떻게 할지? 이런 한계들을 k8s로 돌파해나가보자..


![[Pasted image 20230505162700.png]]
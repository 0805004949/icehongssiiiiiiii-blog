---
title: Load Balancer
date: 2023-04-26T15:44:38+09:00
lastMod: 2023-04-28T15:44:38+09:00
tags: ["2023-k8s-bootcamp","aws","kr", "blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---


-   로드밸런서는 서버의 트래픽을 대상그룹으로 분산 시켜줍니다.
-   여기서는 백엔드에 HTTPS를 적용하는데 사용합니다.

## security group

ec2 > network & security group > security group


- name : saju-alb-sg-prod
- description : saju alb security group production
- vpc : default
- inbound rule 
	- http, 80 for port range, 0.0.0.0/0 for source 
- outbound rul 
	- all trafic 
- tag
	- Name = saju-alb-sg-prod
	- Service = saju-prod


## target group

대상 그룹은 로드밸런서 트래픽을 분산 시켜주는 EC2 그룹입니다.

ec2 > load balancing > target group

- target type : instance
- targey group name : saju-tg-prod
- protocol : http1
- port : 3000 로드밸런서(80포트)에서 EC2(3000포트) 타겟 그룹 전달
- healthcheck : http, 
- healthcheck path : / 
- tag
	- Name = saju-tg-prod
	- Service = saju-prod
- Ports for the selected instances : 3000


## load balancer

application load balancer

ec2 > load balancing > load balancer

- name : saju-alb-prod
- scheme : internet facing
- default vpc
- network mapping : ap-northeast-2a, ap-northeast-2b
- security group : saju-alb-sg-prod
- listener
	- 80, http, saju-tg-prod for default acttion
- tag
	- Name = saju-alb-prod
	- Service = saju-prod


## 백엔드 실행

생성된 application loadbaancer의 DNS로 이동
/var/www/saju.../...
ec2 인스턴에서  ` sudo npm run start pm2 .  

## 프론트엔드 실행

`src/api/index.js.  

production 환경에서 로드밸런서 주소 설정


```js
import axios from "axios";

  

console.log(process.env.NODE_ENV);

export default process.env.NODE_ENV == "production"

? axios.create({

//production 환경 (AWS EC2)

// baseURL: "http://54.180.54.32:3000",

// baseURL: "http://saju-alb-0925-8234669.ap-northeast-2.elb.amazonaws.com",

baseURL: "http://saju-tesst234-1638740354.ap-northeast-2.elb.amazonaws.com/",

})

: axios.create({

//development 환경

baseURL: "http://localhost:3000",

});
```

npm run biild 생성하면서 생긴 dist 파일들 s3업로드

![[Pasted image 20230428183227.png]]


통신완료




## 📑 Ref
- 인프런 - 지금 당장 AWS!!

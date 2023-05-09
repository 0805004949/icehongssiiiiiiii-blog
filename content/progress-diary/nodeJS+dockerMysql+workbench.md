---
title: nodeJS+dockerMysql+workbench
date: 2023-04-25T15:44:38+09:00
lastMod: 2023-04-25T15:44:38+09:00
tags: ["2023-k8s-bootcamp","aws","kr", "blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---




## install nvm(node verion manager)

[[node version manager (NVM)]]

use node 16.3.2


## RUN mysql instance
```sh 
docker run -e MYSQL_ROOT_PASSWORD=1234 --name mysql_saju -d -p 3306:3306  mysql
```

```
$ docker exec -it mysql_saju /bin/bash

$(mysql_saju) mysql -u root -p 

$(mysql_saju)(mysql) create database saju_db_dev; 
```


# Connect the node js server and mysql db

```sh 
git clone https://github.com/vipick/saju-backend-nodejs.git
```

- mysqlworkbench에서 `127.0.0.1:3309` 연결해서 `saju-backend-nodejs/mysql` 에 있는 sql을 `saju_db_dev`  에 import 



```sh
cp .env.template .env 
```

- 이때 `DEV_DB_PORT= 3306` 로 작성 

# RUN 

```
$ cd /saju-backend-nodejs
$ npm install
$ npm run dev 
```


# Postman 



# VUEjs 

```sh 
git clone https://github.com/vipick/saju-frontend-vuejs
```


```sh 
nvm install 16.3.2
nvm use 16.3.2
cd saju-frontend-vuejs
npm install 
npm install serve
```


브라우저에서 `http://localhost:8080.  접속 

# 클라우드 아키텍처 
- 프런트엔드(VueJS)와 백엔드(NodeJS) 소스코드를 AWS에 배포하는 과정
- AWS에서 EC2 (백엔드 API서버), RDS-MySQL(DB 서버), S3(프런트 서버), HTTPS 도메인 적용(Route 53, 로드밸런서, CloudFront)를 사용

![[Pasted image 20230425175050.png]]

## 📑 Ref
- 인프런 - 지금 당장 AWS!!
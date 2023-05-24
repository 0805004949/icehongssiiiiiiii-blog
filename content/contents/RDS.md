
---
title: RDS
date: 2023-04-25T15:44:38+09:00
lastMod: 2023-05-01T15:44:38+09:00
tags: ["2023-k8s-bootcamp","aws","kr", "blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---




## security group 


EC2 > network and security > security groups

```ad-tip
Service 로 태그를 다는 이유는 Resource Group & Tag Editor 에서 한 번에 해당 프로젝트의 리소스를 검색 할 때 필요합니다. 


```


Name 태그는 리소스 마다 이름이 다른 경우가 발생해서 모든 리소스를 한 번에 볼 때 사용할 수 없습니다.

- name : saju-db-sg-prod
- description : saju db security group production
- vpc : default
- inbound rule : `Custom TCP` , `3306` for port range, `Anywhere-IP4` for Source type, `0.0.0.0/0` for Source 
- tag1 : key = `Service`, value = `saju-db-sg-prod` 
- tag2 : key = `Name`, value = `saju-prod` 




## RDS 생성 

- RDS standard - mysql 8.0.28 - free tier  - db.t2.micro
- DB instance identifier : `saju-db-prod` ( //엔드포인트 주소와 관련이 있습니다.)
- 마스터 사용자명 admin / 암호자동생성 
- 스토리지 : 변경 사항 없음
- 가용성 및 내구성 : 변경 사항 없음
- 연결
	- vpc : default 
	- DB 서브넷 그룹 : defaul
	- 퍼블릭 액세스 : 예        //DB서버에 원격 접속을 위해 필요
	- VPC 보안그룹 (방화벽) : 기존 항목 선택
	- 기본 VPC 보안 그룹 : saju-db-sg-prod
	- 가용 영역 : ap-northeast-2a
	- 데이터베이스 포트 : 3306
- (10) 데이터베이스 인증 : 암호 인증
- (11) 모니터링 및 추가 구성 : 변경 사항 없음 
- 생성
- (13) 연결 세부정보 보기   //마스터 암호는 Copy해서 보관해야 합니다.
- (14) DB 생성 완료 후 3가지 정보 확인
	- 마스터 사용자 이름 (PROD_DB_USERNAME) admin
	- 마스터 암호 (PROD_DB_PASSWORD) APfhd1234!
	- 엔드포인트 (PROD_DB_HOST) saju-db-prod.cerfeexwylje.ap-northeast-2.rds.amazonaws.com
- rds 인스턴스 생성 후 태그 추가
	- Name = saju-db-prod
	- Service  = saju-pro


## mysql workbench로 rds 접속

- create database `saju_db_prod`
- import `saju-backend-nodejs /mysql/sqls/saju-db-prod.sql.  inside of `saju-db-prod`. 




## .env 파일 수정  

```.env 
PROD_DB_HOST= saju-db-prod.cerfeexwylje.ap-northeast-2.rds.amazonaws.com
PROD_DB_DATABASE= saju_db_prod
PROD_DB_USERNAME= admin
PROD_DB_PASSWORD= APfhd1234!
```


## run backend server by production version 

`npm run start` 

- 포스트맨에서 signup 후 prod 데이터베이스 확인
post method
`http://127.0.0.1:3000/users/signup`

```json
{

"email" : "test@test.com", 
"password" : "1234",
"nickname" :"test", 
"gender":"MALE", 
"birthdayType" :"SOLAR", 
"birthday": "19870213",
"time":"0710"
}

```

- `use saju_db_prod;`
- `select * from users where nickname='test';`

## 📑 Ref

- 인프런 - 지금 당장 AWS!!
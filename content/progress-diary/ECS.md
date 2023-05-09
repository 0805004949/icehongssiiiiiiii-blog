
---
title: ECS
date: 2023-05-03T15:44:38+09:00
lastMod: 2023-05-09T15:44:38+09:00
tags: ["2023-k8s-bootcamp","aws","kr", "blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---




-   ECS 는 Elastic Container Service로 AWS의 대표적인 도커 컨테이너를 운영하는 서비스로 CI/CD 와 오토스케일링을 쉽게 적용할 수 있어서 서비스 운영에 도움이 됩니다.
-   ECS는 클러스터, 작업 정의, 서비스 3가지로 나뉩니다.







## 대상그룹 생성


```ad-question

target group
ip 주소로 생성할 떄 vpc가 타겟그룹에 들어가지 않음

```


대상 그룹은 로드밸런서 트래픽을 분산 시켜주는 Fargate 그룹입니다.

- target type  : IP주소 
- saju-tg-prod
- http
- 80
- 모니터링(http, /)
- 대상등록
	- -   네트워크 선택 : saju-vpc-prod
	- -   포트 : 80
- 속성편집
	- 등록 취소 지연 : 10초 (기존 300초)

- (6) 상태검사 설정 편집 - 고급 상태 검사 설정
	-  정상 임계값 :  3
		-    간격 : 10초
https://www.inflearn.com/course/lecture?courseSlug=%EC%A7%80%EA%B8%88-%EB%8B%B9%EC%9E%A5-%EB%8D%B0%EB%B8%8C%EC%98%B5%EC%8A%A4-aws&unitId=129058







## 로드밸런서

- application load balancer
-   로드밸런서 이름 : saju-alb-prod
-   체계 : 인터넷 경계
-   IP주소 유형 : IPv4
-   VPC : saju-vpc-prod
-   매핑 : ap-northeast-2a (saju-public-subnet1-prod), ap-northeast-2b (saju-public-subnet2-prod)
- -   보안 그룹 : saju-alb-sg-prod
- 리스너 HTTP:80 대상 saju-tg-prod
- 리스너 HTTPS:443 대상 saju-tg-prod 보안리스너 설정 ACM: `*.icehongssii.xyz`
- tag
	- name=saju-alb-prod
	- service=saju-prod


## 시크릿 매니저

-   Secret Manager 에 .env 환경 변수 등록
- -   .env 는 git으로 관리 및 배포를 하지 않기 때문에 .env 관련 내용은 Secret Manager 에서 관리합니다.
- 예를 들어 .env에 JWT_SECRET = test2022 가 있습니다. 다른 유형의 보안 암호를 선택 후 일반 텍스트 value에  test2022를 입력하고 다음 버튼을 클릭합니다.

|     |     |
| --- | --- |
| ![[Pasted image 20230503213519.png]]    |![[Pasted image 20230503173400.png]]     |



## cluster 생성

- cluster template - noetwroking
- saju-cluster-prod
- vpc 생성 체크하지 않음 saju-vpc-prod 전에 생성해서
- 컨테이너 인사이트 활성화 체크 - 오토스케일링시 필요

## iam 설정

iam 역할 > 역할 만들기 > aws service
- 다른 aws 서비스 사용 사례 > elastic container service > ecs task
- 권한 추가 
	- 권한정책 AmazonEcsTaskExecutionRolePolicy
- 역할 생성
	- 역할 이름 : ecsTaskExecutionWithSecretRoleProd
	- 역할 생성 완료후 인라인 정책 생성 > secret manager 인라인 정책 생성하는 이유는 도커에서 .env 환경변수 사용하기 위해서임 도커에서 .env 파일 포함시키지 않습니다?
-  tag
	- saju-iam-sm-prod
	- service=saju-prod
- 인라인 추가 
	- 인라인 정책에서 아래내용 추가 > 정책명 -   secretsAccessPolicy 
	  ```json
{  
    "Version": "2012-10-17",  
    "Statement": [  
        {  
            "Sid": "VisualEditor0",  
            "Effect": "Allow",  
            "Action": "secretsmanager:*",  
            "Resource": "*"  
        }  
    ]  
}

```

- 시각적 편집기에서 작업, 모든 Secrets Manager 작업 선택
- 정책 검토를 완료하면 secretsAccessPolicy (고객 인라인)이 생성됨을 확인 할 수 있습니다.



## ecs 서비스 생성



```ad-info
ecs는 도커 관리하는데 클러스터, 태스크, 서비스로 나눠짐
- 클러스터는 vpc network 구성과 관련있음. 자동으로 vpc 생성하거나 직접 vpc 구축할 수 있는데 여기서는 직접 구축
- ecr에 저장된 도커 이미지는 작업 정의를 통해 파게이트에 컨테이너 형태로 배포됨. 파게이트는 ec2와 유사한 역할
- 서비스에서 여러개의 컨테이너 관리를 하는데 대표저그로 오토스케일링 잇음 오토스케일링은 특정기준(cpu 사용률) 만족하면 파게이트 증가하거나감소함으로 트래픽 유연하게 처리가능 
```



- 시작유형 파게이트
- 운영체제 패밀리 linux
- 작업정의 saju-task-prod
- 클러스터 saju-cluster-prod
- 서비스이름 saju-service-prod
- 배포 rolling update
- 네트워크 구성
	- 클러스터 vpc saju-vpc-prod
	- saju-public-subnet1-prod (10.0.0.0/24, ap-northeast-2a), saju-public-subnet2-prod (10.0.1.0/24, ap-northeast-2b)
	- 보안그룹 saju-api-sg-prod
	- 자동할당퍼블릭ip : enabled

```ad-info 
- 자동할당 퍼블릭 ip를 enabled로 선택 할 경우 컨테이너에 퍼블릭 ip가 할당됨. 컨테이너는 퍼블릭 서브넷에 위치하고 해당 서브넷은 igw 와 연결되어있어야함
- 자동할당 퍼블릭 ip를 해제햇으면 컨테이너에 퍼블릭 ip 가 할당되지 않음. 컨테이너에서 외부와 통신하려너 컨테이너가 위치한 프라이빗 서브넷이 외부와 통신할 수 있는 nat 게이트웨이와 연결되어야함 
```


- 보안그룹 saju-api-sg-prod
- 로드밸런싱
	- application loadbalancer
	- saju-alb-prod
	- 로드밸런싱할 컨테이너 saju-container-prod:3000
	- 프로덕션 리스너 포트 80 http
	- 대상그룹명 saju-tg-prod(대상유형 ip)




## 태스크 생성




## 서비스 생성


## 📑 Ref
- 인프런 - 지금 당장 AWS!!
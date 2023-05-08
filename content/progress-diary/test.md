---
title: "aws 인프라 구축"
date: 2023-04-19T15:44:38+09:00
draft: false
tags: ["bootcamp-cka-12th", "AWS"]
categories: ["TIL"]
---

LAMP

클라우드 네트워크 환경에 

- Linux 기반의 가상 서버에
- Apache 웹서버
- MySQL 데이터베이스
- Php 어플리케이션

에다가 application load balancer 이용해 이중화된 네트워크 구성 

사용서비스
- aws vpc(vpc subnet, internet gateway, route table, NAT gateway)
- aws ec2, ebs, efs, load balancer (application load balancer)

# 아키텍처 


![[Pasted image 20230417180057.png]]


- seoul region 안에 vpc있고 
- 서로다른 두개의 avavility zone안에 각각 3개의 subnet이 있다
- subnet은 외부 인터넷에서 접근 가능한 public subnet과 제한된 private
- public안에는 ec2는 IG통해 외부 통신. lamp이요하면 웹서버 
- efs는 vpc안에 각 서브넷 마운트 타겟으로 ec2와 연결되고 파일저장및 공유 가능
- 한편 웹서버는 퍼블릭 서브넷에 잇으면 기ㄴ으수행 괜찮지만 외부 인터넷과 연결가능해서 봉나측면에선 별로 적절치 않을 수록 
- 그래서 private subnet에 웹서버 위치하게 함 이렇게 되면 보안적인 측면에서는 개선되지만 외부 와 직접적 통신 제한되어있기에 유지보수 업데이트 작업하기 위해 직접 접속하기는 어려움 그래서 퍼블릭 서브넷에 있는 ec2 통해서 프라비잇 인스턴스에 접속한느데 이러한 중개 서버를 bastion 이라고함 
- 한편 단일 인스턴스 문제없지만 실제 웹서버는 트래픽등 다양한 이슈등 장애가 생길수 있어서 private subnet에 복사 ? 이 인스턴스들을 네트워크 분산서비스 로드밸런서에 연결함 여기서는 application 로밸 사용 이렇게 되면 외부 연결 제한되고 안정적 인프라 구축 가능 

구현순서
1. vpc생성
2. 하위네트워크인 subnet 생성 
3. 외부 연결 가능한 IG 생성
4. 트래픽 이동할 수 잇게 라우트 테이블 생성 및 라우트 설정 

ec2
1. ec2생성(AMI, LAMP웹서버 구성, SG, EBS, 키페어)
2. index.php 파일생성
3. 우베브라우저에 LAMP웹서버 작동테스트


1. custom ami 생성
2. custom ami ㅗ통해 ec2생성
3. 이전 단계 웹브라우저 lamp 웹서버 작동 테스트 


efs
1. efs SG생성
2. EFS생성(avavility, lifecycle, performace/thoughput mode, NW) 
3. ecs-ec2마운트
4. 웹프라우저 통한  efs마운트 테스트

aloadbalancer
어플리케이션 로드 밸런서 이용해 퍼블릭 서브넷에 생성한 두 인스턴스로 이동하는 네트워크를 이중화로 구성

1. 로드밸런서 트래픽 받을 타겟집합인 target group 생성 (target type, protocol/port, target registration)
2. 타겟그룹 대상으로 하는 로드밸런서 구성(scheme, network, SG, listener/Rule등)
3. 웹브라우저 통한 application 텍스트


bastion host와 NAT gatewa통한 private ec2인스턴스 외부 통신구성

외부와 직접통신제한되어잇는 프라이빗 ec2생성하고 이 ec2가 외부와 통신 가능한 환경 구성 

1. private subnet ec2생성 
2. public subnet ec2통해 private subnet ec2접속(키페어생성)
3. NAT Gateway생성
4. route table 설정
5. private subnet ec2외부 통신 테스트

applicaiton load balancer 이중화 네트워크 구성2


1. target group 생성
2. application load balancer 구성
3. 웹서버 브라우저 통한
4. applicaiton load balancer 작동 텟트 


# vpc, subnet 
- create vpc(10.1.0.0/16) seoul 
- Enable DNS hostnames[Info](https://us-east-2.console.aws.amazon.com/vpc/home?region=us-east-2#)
- 하위네트워크 subnet 생성
- 서브넷 CIDR블럭이 10.1.0.0/16 보다 작아야함
- public-subnet-a1 10.1.1.0/24 ap-northeast-2a
- public-subnet-c1 10.1.2.0/24 ap-northeast-2c
- private-subnet-a1 10.1.3.0/24 ap-northeast-2a
- private-subnet-c1 10.1.4.0/24 ap-northeast-2c
- private-subnet-a2 10.1.5.0/24 ap-northeast-2a
- private-subnet-c2 10.1.6.0/24 ap-northeast-2c

여섯개 서브넷생성됏지만 ec2인스턴스가 생성될수 잇는 위치 가 생성됏지만 인프라 작동 ㄴ 
리소스들 사이, 리소스와 외부인터넷 사이에 트래픽이동할수잇는 경로가없기 떄문
이런거 도와주는게 IG와 라우트 테이블


# IG, routetable

- lab-vpc-igw 방금 vpc와  lab-vpc-igw attach 
- 라우트 테이블은 네트워크 통신간에 잇어서 목적지 대상지등으로의 데이터 패킷 이동정보를 구성하는 규칙, 즉 서브넷에 위치한 ec2 리소스같은거를 목적지나 대상지에 따라 어떤 경로를 따라 트래픽을 이동하게 할건지에 대한 테이블 
- 퍼블릭 서브넷에 대한 route 테이블 (편의상 두개의 퍼블릭 서브넷은 같은 라우트 테이블 사용하기로 public-subnet-rt, )
- 라우트 테이블 만들면 어떤 서브넷에 대한 트래픽 경로인지를 정해줘야한다 이를 위해서 이 라우트 테이블을 우리가 만들었던 public subnet a1, c1에 associate해야함 -> subnet association public-subnet-a1 10.1.1.0/24 ap-northeast-2a,  public-subnet-c1 10.1.2.0/24 ap-northeast-2c)
- 방금 만든  퍼블릭 서브넷에 대한 route 테이블에서 public subnet resoucre들이 외부와 통신할수있도록 iGW를 경로로 추가해야한다 -> edit routs에 add route 0.0.0.0/0 추가하고 target은 아까 만든 igw -> 목적지가 외부로향하는 트래픽은 인터넷 게이트웨이 향해서 나가라라는 의미 트래픽이 인터넷 게이트 웨이를 통해 외부로 나간다는의미 
- 10.1.0.0/16이 목적지라면 lab vpc내에서 트래픽 이동 가능하다는 의미
- 그리고 각 private subnet 4개에 대한 라우트 테이블을 각각 생성하고 association함


# ec2
- ec2 생성시체 networksettng lab-vp / public-subnet-a1
- auto-assign-ip -> 인스터느 만들어질떄 자동으로 퍼블릭 ip할당할건지 말지 
- 이따가 elastic ip사용해서 퍼블릭 ip를 부여할건데 이를 사용하는 이뉴는 고정적인 ip부여할거라서 일단 auto assign disable 
- SG는 트래픽제어하는 어디서 오는 트래픽을 허용할것인가? 모든 곳에서 access 허용하면 모든 트래픽 허용 특정 아이피에서 온 트래픽만 허용한다면 custom으로 하고 특정 ip대역 설정가능한데 만약에 10.0.0.0/16으로하면 현재 vpc에서만..
![[Pasted image 20230417124406.png]]
- 웹사이트 https 443, http 80, mysql3306 포트.. 임 일단은 아래 추가 
![[Pasted image 20230417124720.png]]


- advanced details 
- Instance auto-recovery ->default 
- Shutdown behavior -> stop]
- capacity - none
- stop binernate 최대절전모드 disable 
- Termination protection -> disable 비활성화 방지 
- stop protection 갑자기 중지되지 않도록
- Detailed CloudWatch monitoring -> disable
- Credit specification-> 갑자기 상위 인스턴스 ㅡ고싶을떄 근데 이거 비싸니까 standard
- Placement group 서버 배친전략 -> ㄴㄴ
- Capacity reservation 특정 가용영역에서 얼마나 용량사용할건지 특정시점에 인스턴스 용량 미리확보 
- Tenancy -> shared 
- Metadata accessible -> enable, version 1 +2, 
- Metadata response hop limit -> 1 & allow tags in metadata enable
- User data -> root권한으로 실행되는 런치시  sh 

```sh 
#!/bin/bash
yum update -y 
yum install amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
yum install -y httpd mariadb-server
systemctl start httpd
systemctl enable httpd
usermod -a -G apache ec2-user
chown -R ec2-user:apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;
```

- ec2 > elastic ips > allocate ip 생성 -> associate elastic ip -> public ec2 instance
ssh -i "ec2-public-seoul.pem" ec2-user@ec2-3-39-104-130.ap-northeast-2.compute.amazonaws.com

- 여기서 이거 왜 생성 안돼지??<<<<<<<<




# create custom ami

- 기존에 있던거 동일하게 사용
- userdata는 기존에 잇는 인스턴스 에잇던거 상요해서 ㄱㅊ 
- publci subnet만 다르게 
- 커스텀 ami사용해서 인스턴스 또하나 만들면 그전에 만든 인스턴스와 동일한 디렉토리를 사용한다고함 


# efs

 vpc -> sg생성 
 - lab-vpc-efs-sg(security group for lab-vpc) -> lab vpc 선택
 - 인바운드에서는  방금전에 만든 ec2의  sg추가 (인스턴스와 연결하고자 인스턴스sg을 선택 ->  인스턴스 시큐리티 그룹을 통해 나오는 트래픽을 efs에 마운트 타켓이 허용하겠다 라는 뜻 ![[Pasted image 20230417144305.png]]
 - - 아웃바운드든 그대로 
 - efs 생성 lab-vpc-efs
	 - n/w access 
	 - 마운트 타겟은 기본적으로 하나의 마운트 타겟 설정 가능(퍼블릭 서브넷 하나씩, sg는 efs vpc subnet)
	 ![[Pasted image 20230417144933.png]]
	 - 2~3분후에 마운트 
	 - 첫번째 ec2인스턴스 접속 
```
$ df -h // 파일시스템 확인 
aws efs마운트 해커 이용
sudo su 
yum install  amazon-efs-utils -y 
cd /var/www/html 
mkdir efs 
// efs화면으로 들어가서 attach > 
sudo mount -t efs -o tls fs-033b1ca16ee4b592c:/ efs
df -h 마운트 확인 (/var/www/html/efs가 마운트된거확인)
cd efs
// s3에 있던거 efs에 넣기 
wget https://lab-s3-web-hosting-123.s3.amazonaws.com/car.jpg




```


-> ec2 이동 public ip새탭으로 가보면 index확인가능


-> 같은작업 다른 인스턴스에도 진행
-> 이때 같은 efs를 보고있는지 확인하고자 html파일을 바꿔볼것 


# application load balancer
트래픽 분산 시키는 법 

ec2 메뉴 -> 로드밸런서(트래픽을 받으면 리스너를 통해 타겟그룹으로 전송하는데) > 타겟 그룹 생성
-  create target group (instance가 타겟 타입, lab-vpc-alb-public-tg, http80 어플리케이션로드밸런서는 http, https적용 그외는 놉! http만 여기서 적용하는 프로토콜은 alb와 타겟되는 인스턴스사이와 통신, 타겟되는 인스턴스들이 여기서 적용한느 프로토콜만 받는다는뜻 , lab vpc, htt1, http healthcechk / 그대로) 타겟은 인스턴스 두개 이고 등록할것 

타겟그룹으로 트래픽 분산시켜줄 alb 생성
 creat alb
 - lab-vpc-alb-public
 - scheme : 외부 인터넷에서 오는 트래픽을 분산시키기 떄문에 interfet facting(인터넷과 ㅇ녀결) lab-vpc 각 타겟그룹에서 퍼블릭 서브넷 인스턴스를 등록햇기에 public subnet a1, public subnet c1 선택 
 - SG 생성 lab-vpc-alb-public-sg
	 - vpc는 lab vpc 
	 - 인바운드 : http, source 0.0.0.0
	 - 아운바운드 그대로 

- default sg삭제
- 리스너 & 라우팅 ; 리퀘스트 들어오면 타겟으로 가는데, 리스너가 로드밸런서에서 들어온거 받아들이고 어디로 들어올지 ..수행 이 기준 > 룰 > lab-vpc-public-tg 외부와 alb사이 프로토콜  (디폴트는 리스너가 전달받은 트래픽 설정, 리스너 룰 )
- tag는 이름과 동일하게 
- 로드 밸런서 생성되고 나서 listener 보면 타겟그룹 이 룰 통해 특정 타겟으로 보내는 
- description dns이름 복사 -> 새탭에서 붙여넣기하면 동일하게 

# private 영역 ec2 연결

- 같은 ami 
- key pair 새로 생성 (private용 seoul )
- vpc, privatte subnet a1 , 
- 새 sg 생성 ![[Pasted image 20230417151636.png]]
- advanced에서 전과 동일 
- 키페어 
- public ec1 -> private 통해서 ㄱ ㄱ 
- 해당 인스턴스에 대한 키페어가 있어야함 
- 아까 만들었던 키페어 
- 퍼블릭  인스턴스에서 ec2 private-soeul.pem 복붙 
- chmod 400 
- ssh -i ec2-private.pm ec2@프라이빗 인스턴스 주소 
- 근데 프라이빗 서브넷에 잇ㄴ느 인스턴스도 yum 같이 패키지 설치 핑요할수도있음
- vpc -> NAT gateway -> create NAT gateway(이름은 nat-g-a1, public subnet에 위치 )
- ![[Pasted image 20230417152242.png]]
- NAT gateway 생성 -> 라우트 테이블에서 private subnet a1 라우트 테이블 선택 하단에 라우트 선택 -> edit routs add routs -> 0.0.0.00에 타겟에 NAT gateway 방금전에 만든거 선택! 엘라스틱아이피 새로 생성하것 
- private이 트래픽 밖으로 나갈 땐느 publix 이용해서 나감 
- 외부통신다시확인 ping google.com private에서독 ㅏ능!
- private-ec2-c1생성 subnet은 private subnet c1 
- ![[Pasted image 20230417152703.png]]
private subnet c1라우트 테이블에서 다시 라우트 추가 



# alb 2

- private 영역을 타겟으로하는 alb 생성
- 웹서버를 직접 통신가능한 퍼블릭에 두는건 좀 보안측면에 그렇게 좋지 않음 웹서
lab-vpc-alb-private-tg 타겟그룹 생성
- http
- labvpc
- 이전과 동일
- register target은 private에 있는 인스턴스 추가 include pending below 
- 로드 밸런서 lab vpc alb private 
	- network mapping   근데 왜 퍼블릭 서브넷과? 로드밸런스 타겟이 프라이빗인데 매핑은 왜 퍼블릭? ec2같은 리소스는 네트워크 인터페이스 외부인터넷과 통신하려면 인터넷게이트잇는곳에..  그래서 이것들이 퍼블릭통해서..
	- ![[Pasted image 20230417153352.png]]
	-  SG 생성 
	- ![[Pasted image 20230417153602.png]]
	- 디폴트 sg 삭제
	- ![[Pasted image 20230417153624.png]]
	- ![[Pasted image 20230417153651.png]]

- alb dns  ㄱ ㄱ
- 
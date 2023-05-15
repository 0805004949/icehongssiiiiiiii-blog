---
title: 책-2장-NW서비스와-어플리케이션계층
tags: ["2023-k8s-bootcamp","k8s","KR","인강-devops를위한-k8s","blog-published"] 

categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->

date: 2023-05-12T16:00:33
lastmod: 2023-05-15T17:38:13---

---

- [[#1. HTTP|1. HTTP]]
	- [[#1. HTTP#HTTP 요청|HTTP 요청]]
	- [[#1. HTTP#HTTP 응답|HTTP 응답]]
	- [[#1. HTTP#HTTP는 Stateless(무상태)|HTTP는 Stateless(무상태)]]
- [[#2. 웹서비스와 웹 애플리케이션|2. 웹서비스와 웹 애플리케이션]]
	- [[#2. 웹서비스와 웹 애플리케이션#AJAX?|AJAX?]]
- [[#3. 쿠키|3. 쿠키]]
- [[#4. 이메일|4. 이메일]]
	- [[#4. 이메일#SMTP (발신)|SMTP (발신)]]
	- [[#4. 이메일#POP (수신)|POP (수신)]]
- [[#5. PC끼리 공유하기|5. PC끼리 공유하기]]
- [[#6. FTP,  파일 전송 프로토콜|6. FTP,  파일 전송 프로토콜]]
- [[#7. 원격지 컴퓨터제어(SSH, Telnet)|7. 원격지 컴퓨터제어(SSH, Telnet)]]
- [[#8. VoiceOver IP와 영상 스트리밍|8. VoiceOver IP와 영상 스트리밍]]
- [[#📑 Ref|📑 Ref]]




![](https://i.imgur.com/4nMKaOG.png)


- 어플리케이션 계층 : 서비스 제공하는 부분으로 서비스의 내용을 결정  
- 그외의 나머지 3개 계층 : 데이터를 전달하는 통신 기능


![](https://i.imgur.com/VixDbZJ.png)


- HTTP : 웹클라이언트와 웹서버 사이에서 웹데이터 주고 받음
- POP, SMTP, IMAP : 메일 송수신 및 보관
- FTP : 서버를 통해 파일 주고 받음
- Telnet, SSH : 원격에서 서버 제어 

그외에 사용자가 간접적으로 사용하는 프로토콜
- DNS
- SSL/TLS
- DHCP

## 1. HTTP 

- 어플리케이션 계층 프로토콜 중 하나 
- **HTTP는 HTML 문서와 같은 리소스들을 가져올 수 있도록 해주는** [프로토콜](https://developer.mozilla.org/ko/docs/Glossary/Protocol)입니다. 
- HTTP는 웹에서 이루어지는 모든 데이터 교환의 기초이며, 클라이언트-서버 프로토콜이기도 합니다 (= 웹서버와 클라이언트 PC 사이에서 일어나는 정보교환에서 쓰이는 전송 규약



![](https://i.imgur.com/6AxEl00.jpg)

1. 브라우저 창에 URL 입력
2. 서버에 HTTP 요청으로로
3. 웹 서버는 요청에 대한 응답을 HTTP 응답에 실어서 보내고
4. 이에 대한 응답을 받은 클라이언트는 웹 페이지를 표시한다 

이떄 서버와 클라이언트가 주고 받은 정보들은 hTTP메세지로 부르고 아래 두가지로 나뉠 수 있음 

### HTTP 요청 

![](https://i.imgur.com/QffDXkX.png)
-   HTTP [메서드](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods), 보통 클라이언트가 수행하고자 하는 동작을 정의한 [`GET`](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/GET), [`POST`](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/POST) 같은 동사나 [`OPTIONS`](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/OPTIONS)나 [`HEAD`](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/HEAD)와 같은 명사입니다. 일반적으로, 클라이언트는 리소스를 가져오거나(`GET`을 사용하여) [HTML 폼 (en-US)](https://developer.mozilla.org/en-US/docs/Learn/Forms "Currently only available in English (US)")의 데이터를 전송(`POST`를 사용하여)하려고 하지만, 다른 경우에는 다른 동작이 요구될 수도 있습니다.
-   가져오려는 리소스의 경로; 예를 들면 [프로토콜](https://developer.mozilla.org/ko/docs/Glossary/Protocol) (`http://`), [도메인 (en-US)](https://developer.mozilla.org/en-US/docs/Glossary/Domain "Currently only available in English (US)") (여기서는 `developer.mozilla.org`), 또는 TCP [포트 (en-US)](https://developer.mozilla.org/en-US/docs/Glossary/Port "Currently only available in English (US)") (여기서는 `80`)인 요소들을 제거한 리소스의 URL입니다.
-   HTTP 프로토콜의 버전.
-   서버에 대한 추가 정보를 전달하는 선택적 [헤더들](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers).
-   `POST`와 같은 몇 가지 메서드를 위한, 전송된 리소스를 포함하는 응답의 본문과 유사한 본문.

### HTTP 응답 
![](https://i.imgur.com/QawRhTN.png)


-   HTTP 프로토콜의 버전
-   요청의 성공 여부와, 그 이유를 나타내는 [상태 코드](https://developer.mozilla.org/ko/docs/Web/HTTP/Status)
-   아무런 영향력이 없는, 상태 코드의 짧은 설명을 나타내는 상태 메시지
-   요청 헤더와 비슷한, HTTP [헤더들](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers)
-   선택 사항으로 가져온 리소스가 포함되는 본문

### HTTP는 Stateless(무상태)

![](https://i.imgur.com/en1pWO7.png)



>  **HTTP에서 서버가 클라이언트의 상태를 보존하지 않는 무상태 프로토콜이다.  
> 간단한 예시를 들자면 이렇다.  
> 클라가 서버에게 “저녁에 치킨먹자”고 말했고 서버가 “그래”라고 답했다.  
> 저녁이 되어 클라는 서버에게 “먹으러 가자”고 말했고 서버는 대답했다. “뭘?”**

-   서버가 클라이언트 상태를 보존하지 않음
-   장점 : 서버 확장성 높음 (스케일 아웃)
-   단점 : 클라이언트가 추가 데이터 전송

> https://hanamon.kr/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-http-http%EB%9E%80-%ED%8A%B9%EC%A7%95-%EB%AC%B4%EC%83%81%ED%83%9C-%EB%B9%84%EC%97%B0%EA%B2%B0%EC%84%B1/




## 2. 웹서비스와 웹 애플리케이션


![](https://i.imgur.com/PH8lBx2.png)


동작상 웹브라우저가 웹서비스 요청하는거나 웹페이지요청하는 것은 큰차이가 없음
- 단, 웹 페이지 요청시 : 웹서버는 정적파일 응답
- 단, 웹 서비스 요청시 : 웹 서버는 서버 프로그램이 HTMl 데이터 동적으로 응답 

> [!note]  CGI와 서버 사이드 프로그램 
>  - 서버 사이드 프로그램 = 서버측프로그램 = 웹 서버에서 동작하는 프로그램
>  - 서버 사이드 프로그램은 HTTP 메세지 주고 받는 조건만 맞으면 구현 방법 상관없어서 Python, Ruby등 다양한 언어로 개발되었음
>  - 웹 초창기에 웹서버에서 이들 서버 사이드 프로그램 실행하기 위해 CGI라는 방식 사용 했었음 
>  


### AJAX? 

- 사용자가 버튼 눌렀을 때 전체 화면이 바뀌는게 아니라 화면에서 필요한 부분 (예: 검색창, 뉴스)만 바뀌면 페이지 로딩이 더 빠를텐데? -> AJAX 사용하자
- AJAX는 HTTP 메세지로 통신하지만 요청을 브라우저한테 보내는게 아니라 자바스크립트로 작성된 프로그램이 웹서버와 통신한다
- 웹서버 응답을 받은 자바스크립트 프로그램은 특정 부분에만 응답받은 내요이 갱신되도록 처리한다
- 


## 3. 쿠키 

![](https://i.imgur.com/yLePt4j.png)

- HTTP 통신은 stateles 프로토콜로 서버가 클라이언트의 상태를 보존하지 않는다. 따라서 여러건의 요청저리를 동일한 사용자가 보냈는지 아니면 다르 사용자가 보냈는지 판단하지 못한다
- 따라서 이런 경우 여러 처리를 동일한 사용자로 인식할 수 있게 쿠키를 사용

![](https://i.imgur.com/lbAGzog.jpg)



- 단 민감한 정보는 싣어 보내지 않음 


## 4. 이메일

웹서비스에서  HTTP와 달리 이메일에서 사용되는 프로토콜은 메일을 수신할 떄와 발신할 때 사용한 프로토콜이 다르다. 

- SMTP : 메일 발신시에 사용하는 프로토콜
- POP : 메일 수신시에 사용하는 프로토콜 
- IMAP : 클라이언트 PC가 메일 수신하더라도 메일 서버에 수신한 메일을 지우지 않고 보관(클라이언트에 메일을 보관하지 않기에 모바일 기기에 자주 사용됨)
  

### SMTP (발신)

- 아래의 경우에서 많이 사용됨
	1. 클라이언트 PC가 메일 보낼 때 
	2. 발신자 메일 서버에서 수신자의 메일서버로 중계 할 때 
- HTTP 프로토콜과 달리 Stateful! = 서버가 클라이언트의 상태를 보존한다 = 전송 종료 명령을 받기 전까지 통신이 계속 유지 된다 
- POP 프로토콜과 달리 SMTP는 사용자 인증 체계가 없어 종종 스팸 메일로 악용되어 SMTP에  Auth기능이 추가된 확장 프로토콜이 만들어지기도 했음


### POP (수신)
- SMTP로 전송된 메일은 최종으로 수신자 메일 서버 저장 이후 서버에 저장된 메일 확인시 POP 프로토콜 사용
- 아래의 경우에서도 많이 사용됨
	1. 수신한 메일 건수, 용량 
	2. 수신한 메일 삭제


## 5. PC끼리 공유하기

- 파일 공유 : 개인 컴퓨터에 공유 디렉토리 만들고 그 안에 공유 할 파일 저장해 여러 사람이 함께 활용할 수 있도록 만드는 서비스 
- P2P = 파일공유에 참여하는 각 각 컴퓨터가 서로 서버, 클라이언트 둘 다 되는 방식
- 이러한 파일 공유 기능은 OS에 기본적으로 탑재 되어 있고 OS 마다 다르다(윈도우  SMB, 애플 AFP, 리눅스 NFS)


## 6. FTP,  파일 전송 프로토콜

- 주로 인터넷에 연결된 서버에서 파일 전송 할 때 사용된다. (LAN에서는 파일 공유 하면 되니)
- 명령어로 사용해 파일 업로드하거나 다운 로드 하고 파일 삭제등 가능하다
- 접속형태를 두가지로 나눠 파일 전송중에도 명령을 줄 수 있음
	1. 데이터 커넥션 : 파일 주고 받기 위한 접속
	2. 컨트롤 커넥션 : FTP 명령 보내기 위한 접속 


> [!note] 패시브 모드와 액티브 모드 
 FTP 서비스에서 서버 내부에서 외부로 나가는 통신 방화벽이 차단돼서 전송 안되는 경우 많은데, 이때는 패시브 모드를 사용해 클라이언트 쪽에서 서버 쪽으로 역방향 데이터 커넥션을 만들어주면 파일 전송 가능 



## 7. 원격지 컴퓨터제어(SSH, Telnet)

- SSH, Telent : 원격지 컴퓨터 명렁어로 제어하기 위한 프로토콜 
- 서버들은 데이터 센터등 먼곳에 설치 되어 있어 telent, ssh 사용하는 것이 일반적 


## 8. VoiceOver IP와 영상 스트리밍 

- 인터넷 전화(Voice Over IP, VoIP기술) = 카카오톡 전화, 스카이프 =  
- VoIP = IP 네트워크를 활용하여 음성을 데이터 패킷으로 변환하여 통화를 가능하게 하는 통신 서비스 기술 
- 음성, 동영상 데이터는 전송속도가 중요하기에 TCP 아닌 UDP 사용 + 전송시 데이터 압축하되 수신정보 바로 재생 할 수 있는 스트리밍 기술 사용
- 음성이나 동영상 처리하는 프로토콜은 보편화 되지 않기에 일부 네트워크에서 통신 거부되므로 유튜브는 동영상 공유 서비스에서는 동영상 프로토콜에서 사용할 데이터를 HTTP 메세지 안에 채워 넣음 



## 📑 Ref
- 책 - TCP/IP 쉽게, 더 쉽게
- https://www.guru99.com/tcp-ip-model.html
- https://hanamon.kr/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-http-http%EB%9E%80-%ED%8A%B9%EC%A7%95-%EB%AC%B4%EC%83%81%ED%83%9C-%EB%B9%84%EC%97%B0%EA%B2%B0%EC%84%B1/
- https://developer.mozilla.org/ko/docs/Web/HTTP/Overview

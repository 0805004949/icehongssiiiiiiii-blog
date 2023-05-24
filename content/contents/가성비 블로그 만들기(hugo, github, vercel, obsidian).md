---
title: Github+vercel+hugo=blog
date: 2023-05-09 12:58
lastMod: 2023-05-09 12:58
tags: ["blog","KR", "obsidian", "blog-published"] 
categories: ["posts" ] # <!--하나만 선택해서보셈 -->
slug: "how-to-make-static-blog-by-vercel-and-hugo" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---

## 🥨 준비물

- [Install hugo](https://gohugo.io/installation/) (A static website generator)
- [Install hugo-paperMod theme](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation) (A blog theme via HUGO)
- Join github
- Join vercel via github
- obsidian(optional)



## ❓ 왜 HUGO? 

- static framework 
- static site generator장점?(sepdd of end product, easy of use)
- 장점 not like  dyanmic website generator , for example wordpress, connets to db, php, 좀 많이 알아야함, 워드프레스 알아ㅑ앟고 php 도 좀알아야하구 로딩 웹타임. 퍼포먼스는 html, css, js라서 빠르다. 그리고 만들기도 간단하고 maintain도 간단하다 프레젠테이션 웹사이트같은거는 완전 워드프레스 필요없음 
- 지킬이랑 비슷한데 조금 다름 

## ❓ 왜 Vercel+github+obsidian?

- 기존에 노션대신에 사용하던 메모 어플리케이션이 obsidian이고 obsidian 저장소 valut를 github에 올릴 수 있는 플러그인이 있어서 github와 연동이 편함
- Vercel은 호스팅이 무료라서


## local에서 블로그 꾸미기

1. HUGO 인스톨

```sh 
hugo new site demo -f yml 
```

- 블로그명 demo
- 블로그 configuaration file type yml



2. Active the theme

```sh 
git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
```

`demo` 디렉토리 최상단에 아래 `.submodule.git` 생성

```sh
// .submodule.git

[submodule "themes/PaperMod"]
	path = themes/PaperMod
	url = https://github.com/adityatelange/hugo-PaperMod.git

```

그리고 생성된 디렉토리 구조 

```
├── archetypes/
├── assets/
├── config.yml/
├── content/
├── data/
├── resources/
├── static/
└── themes
    └── PaperMod // 
```


>[!important] 
> 이때 theme/PaperMod안에 있는 파일 수정하지말고 
> `layouts` 폴더 새로 생성해서 hugo가 오버라이딩 할 수 있도록함

## 블로그 Vercel에서 배포하기 


> [!important] 
> 환경변수로 HUGO_VERSION 명시해줘야함


![](https://i.imgur.com/42QLzVh.png)


## 도메인 연결하기

https://velog.io/@dy6578ekdbs/Vercel-%EB%8F%84%EB%A9%94%EC%9D%B8%EC%9D%84-Route-53-%EB%8F%84%EB%A9%94%EC%9D%B8%EC%9C%BC%EB%A1%9C-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0


## 📑 Ref

- https://www.youtube.com/watch?v=hjD9jTi_DQ4
- https://velog.io/@dy6578ekdbs/Vercel-%EB%8F%84%EB%A9%94%EC%9D%B8%EC%9D%84-Route-53-%EB%8F%84%EB%A9%94%EC%9D%B8%EC%9C%BC%EB%A1%9C-%EB%B3%80%EA%B2%BD%ED%95%98%EA%B8%B0
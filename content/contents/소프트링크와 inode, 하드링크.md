---
title: 소프트링크와 inode, 하드링크
date: 2023-05-01
lastMod: 2023-05-01
tags: ["2023-k8s-bootcamp","linux" ,"CS","OS","KR", "blog-published"] 
categories: ["TIL"] # <!--"progress-diary", "posts"  , "TIL"하나만 선택해서보셈 -->
slug: "" # <!--영어 slug만 가능 url에서 보일 수 있음-->
aliases: "" # <!--뭔지몰라-->
keywords: [""] # <!--뭔지몰라-->
series: "" # <!--뭔지몰라-->
description: "" # <!--포스트에대한설명 -->
---



![](https://i.imgur.com/lcnwPN8.png)

## 아이노드(inode)

inode - 파일의 실제 데이터가 아니라 파일의 메타데이터가 저장된 곳(파일명, 파일데이터 제외하고 파일크기등)

filedata에 노래가사있다고 쳐보면
이 파일에 연결된 inode에는 해당 파일의 메타데이터가 구성되어잇는데 파일명은 inode에 저장되지않음 



## 소프트링크 (symlink, symboliclink, softlink)

```
$ ln -s [target] [softlinkfile]
```

- 원본파일과 다른 inode를 참조한다

- 윈도우에서 바로가기 만들기와 동일함. 이 파일 만들 때 대상이 누구인지 지정가능 원본이 되는 파일과 소프트링크파일은 같은곳에 있지 않아도 됨 바로가기는 내가 실행하기 편한곳에 두면 된다.

- 하드링크파일이 쉘에서 아무것도 표시가 안돼서 누가 하드링크에 어떤링크가연결되었는지 확인이 좀 어려움 그래서 일반적으로 소프트링크 많이 사용 (성능적인 부분에서는 하드링크가 나을수도, 소프트링크는 중간 단계가 있기에) 
- 원본 파일이 삭제되거나 이동하면 소프트링크파일은 볼 수가 없다 -> 소프트링크는 경로만 본다 

- 심링크파일이동 할때 링크연결이 깨질수도 있고 유지될 수 도 있는데 이는 심링크 생성시 상대경로로 했는지 절대경로로 했는지에 따라 달라짐
![](https://i.imgur.com/nN25fT1.png)

```sh 
$ echo 'hihihi12345' >> test
$ ln -s /test soft_test #절대 경로로 심링크 생성
$ ln -s test soft_rel_test #상대 경로로 심링크 생성

$ mkdir test_dir && mv soft_test soft_rel_test /test_dir/ #심링크파일이동
$ cat /test_dir/soft_test # 출력가능
$ cat /test_dir/soft_rel_test #추력안됨

$ mv
```


## 하드링크 


```
$ ln [target] [hardlinkfile]
```

- 원본파일과 같은 inode를 참조한다
- 원본 파일이 삭제 되어도 하드링크파일에 있는 데이터는 유지된다 
- 원본파일이나 하드링크파일 위치 바뀌어도 동일한 inode 공유하므로 링크 깨지지않음
- 누가 원본이고 복사본인 지 구별 불가

```
$ echo 'test124' >> touch test
$ ln test hard_test
$ ln -s test soft_test
```

![](https://i.imgur.com/wwILTDs.png)
- 하드링크파일은 inode 번호가 같고
- 소프트링크파일은 inode 번호가 다름 -> 직관적으로 소프트링크인 것 확인가능



아이노드 관점에서 보면?
- 원본파일 = 파일이름+아이노드1+파일데이터
- 소프트링크 파일 = 아이노드2 + 링크데이터(원본파일경로)




## 📑 Ref
- 인프런 - 리눅스 입문 - 개념으로 탄탄히!!
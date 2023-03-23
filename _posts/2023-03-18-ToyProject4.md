---
layout: single
title:  "[ToyProject] Domain과 EC2서버 연결 (Gabia)"
categories: ToyProject
tag: [domain]
toc: true   
author_profile: false
sidebar:
    nav: "main"
search:
    true
---

이번 장에서는 가비아에서 Domain을 구입 한 후 EC2 서버와 연동하는 법에 대해 알아보고자 한다.



## Domain 구입

> 가비아를 통해 Domain을 구입을 할 것이다. 
>
> 만약 회원가입이 안되어 있다면 다음 링크에서 회원 가입 후 진행하면 된다.
>
> [가비아 바로가기](https://www.gabia.com)



[홈페이지 화면]![image-20230322233804672]({{site.url}}/images/2023-03-18-ToyProject4/image-20230322233804672.png)

홈페이지 화면에서 빨간색 박스에 내가 원하는 domain을 입력하면 된다. 

![image-20230322234050814]({{site.url}}/images/2023-03-18-ToyProject4/image-20230322234050814.png)

추천 도메인 에서는 내가 원하는 domain 형식을 지정하면 된다. 

domain을 정했다면 `신청하기` 클릭



![image-20230322234616161]({{site.url}}/images/2023-03-18-ToyProject4/image-20230322234616161.png)

**주의**

***등록 기간에 Default 값이 3년으로 되어 있어 많은 금액이 산정 되는데 꼭 1년으로 등록을 해주자***



추가적으로  안전 잠금 신청 선택을 하고 결제를 하면 된다. 

![image-20230323024427315]({{site.url}}/images/2023-03-18-ToyProject4/image-20230323024427315.png)

가비아에서 정상적으로 도메인 등록이 되었다는 메일을 받으면 가비아 홈페이지에서 `서비스관리` 탭을 누르고 생성한 도메인의 우측에 관리 탭 클릭

![image-20230323024804776]({{site.url}}/images/2023-03-18-ToyProject4/image-20230323024804776.png)

화면을 조금 내린 후에 도메인 연결 클릭

![image-20230323024929983]({{site.url}}/images/2023-03-18-ToyProject4/image-20230323024929983.png)

설정할 Domain을 선택 한 후에 DNS 설정 클릭

![image-20230323025122698]({{site.url}}/images/2023-03-18-ToyProject4/image-20230323025122698.png)

타입 : A

호스트 : @

값/위치 : EC2 인스턴스 Public IP V4 주소

ITL : 600 혹은 1800

선택 한 다음 확인 클릭 후 저장



정상적으로 저장이 되었다면 url 입력창에 내가 구매한 http://domain주소:8080/health-check를 입력해보자. 

![image-20230323191613051]({{site.url}}/images/2023-03-18-ToyProject4/image-20230323191613051.png)

ec2 서버에서 정상적으로 WAS가 돌아가고 있고, domain과 ec2 연결이 제대로 되었다면 ok가 뜰 것이다.

만약 health-check api가 없다면 빼고 입력을 해보자.

![image-20230323191439419]({{site.url}}/images/2023-03-18-ToyProject4/image-20230323191439419.png)

다음과 같이 Whitelabel Error Page가 뜨면 제대로 연결이 된 것이다.

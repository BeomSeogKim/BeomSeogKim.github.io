---
layout: single
title:  "[Java] String vs StringBuilder vs StringBuffer"
categories: Java
tag: []
toc: true   
author_profile: true
sidebar:
    nav: "main"
search:
    true
---

이번 장에서는 String , StringBuffer 및 StringBuilder에 대해 알아보려고 한다.

## String

String은 기본적으로 immutable 즉 변경 불가능한 클래스이다.

다음 예시와 함께 알아보자.

```java
{1}
String a = "hello";
String b = "world";
//------ 
{2}
a = a + b;
System.out.println(a);		// "helloworld"
```

{1}의 과정에서는 a는 hello의 주소인 0x100 을 가르키고 b는  world 주소인 0x200을 각각 가르킨다. 

![image-20230326104315656]({{site.url}}/images/2023-03-26-SpringBuffer/image-20230326104315656.png)

{2}의 과정에서는 새로운 문자열 ("helloworld")이 담긴 instance가 생성되고 a는 이전에 가르키고 있던 0x100 주소 대신 

0x300을 가르킨다. 

![image-20230326105805908]({{site.url}}/images/2023-03-26-SpringBuffer/image-20230326105805908.png)

이처럼 덧셈 연산자를 사용해 문자열을 결합할 경우 매 연산마다 새로운 문자열을 가진 String 인스턴스가 생성되어 메모리 공간을 차지한다. 이러한 이유로 **문자열 간의 연산이 (문자열 간 결합, 추출) 많을 경우**에는 String 클래스 대신 StringBuffer 혹은 StringBuilder 클래스를 생성하는 것이 좋다.

## StringBuffer StringBuilder

다음 예시와 함께 알아보자.

```java
{1}
StringBuffer s1 = new StringBuffer("abc");	// StringBuilder 역시 동일합니다.
System.out.println(s1); 	// "abc"
{2}
StringBuffer s2 = s1.append("def");
System.out.println(s1); 	// "abcdef"
System.out.println(s1 == s2); 	// true
```

{1}의 과정에서 s1는 abc의 주소인 0x100을 가르키고 있다.

![image-20230326192027144]({{site.url}}/images/2023-03-26-SpringBuffer/image-20230326192027144.png)

{2}의 과정에서 s1에 새로운 문자열이 추가되고 s2는 기존의 s1 주소인 0x100을 가르키고 있다.

![image-20230326192404472]({{site.url}}/images/2023-03-26-SpringBuffer/image-20230326192404472.png)

그 결과 s1과 s2 모두 동일한 주소를 가르키고 있어 s1==s2가 true가 된다.



## StringBuilder와 StringBuffer의 차이

StringBuffer는 멀티쓰레드에 안전하도록 동기화되어 있다. 동기화로 인해 String Buffer의 성능이 떨어진다. 

따라서 멀티쓰레드로 작성된 프로그램이 아닌 경우 불필요하게 성능만 떨어진다. 

이를 해결하기위해 String Buffer에서 동기화 기능만 빠진 것이 String Builder이다. 

StringBuffer도 충분히 성능이 좋기 때문에 성능향상이 반드시 필요한 경우를 제외하고는 굳이 StringBuilder로 바꿀 필요는 없다.
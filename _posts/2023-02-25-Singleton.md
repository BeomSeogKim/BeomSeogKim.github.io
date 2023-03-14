---
layout: single
title:  "[Spring] Singleton"
categories: Spring
tag: [singleton]
toc: true   
author_profile: true
sidebar:
    nav: "main"
search:
    true
---

#  

> 싱글톤이란 객체지향 프로그래밍에서 사용되는 디자인 패턴 중 하나로, 해당 클래스의 인스턴스가 오직 하나만 생성되도록 보장하는 패턴이다. 



현재 주제에서는 다음과 같은 class가 있다고 가정하고 설명을 해보려 한다. 싱글톤 패턴이 적용되지 않았을 때, 싱글톤 패턴이 적용 되었을 때로 나누어 알아보려고 한다. 

![image-20230225231801317]({{site.url}}/images/2023-02-25-Singleton/image-20230225231801317.png)

AppConfiguration에서 DI(의존성 주입)을 하고 있는 상황. 

AppConfiguration은 다음과 같다.

```java
@Configuration
public class AppConfiguration {
    
    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemberRepositoryImpl();
    }
}
```



보통의 Application은 여러 고객들이 요청을 한다. 

![image-20230225232608839]({{site.url}}/images/2023-02-25-Singleton/image-20230225232608839.png)

현재의 경우 Client가 `MemberService를` 요청을 할 때 마다 객체를 새로 생성해 반환해 주고 있다. 

테스트 코드를 확인해 보면 

```java
public class Singleton {

    @Test
    void pureContainer() {
        AppConfiguration configuration = new AppConfiguration();

        // 1. 조회
        MemberService memberService1 = configuration.memberService();

        // 2. 조회
        MemberService memberService2 = configuration.memberService();

        System.out.println(memberService1);
        System.out.println(memberService2);

        Assertions.assertThat(memberService1).isNotSameAs(memberService2);
    }
}
```

다음과 같은 결과가 출력 된다. 

```ruby
com.example.demo.member.MemberServiceImpl@27216cd
com.example.demo.member.MemberServiceImpl@8576fa0
```



이러한 경우 client 요청이 올 때 마다 새로 생성해야 하므로 메모리 낭비가 심하다는 단점이 있다. 이러한 문제를 해결하기 위해 싱글톤 패턴을 적용한다. 

# 싱글톤 패턴 구현 

스프링을 사용할 때 싱글톤 패턴 구현에는 크게 3가지 방식이 있다. 

1. 싱글톤 패턴 구현
2. 스프링 컨테이너
3. @Configuration 적용

## 싱글톤 패턴 구현

싱글톤 패턴 구현은 다음과 같이 구현을 한다.

```java
public class MemberServiceSingleton {

    // 1
    private static final MemberServiceSingleton instance = new MemberServiceSingleton();
    
    // 2
    public static MemberServiceSingleton getInstance() {
        return instance;
    }
    
    // 3
    private MemberServiceSingleton() {
    }
    
    public void logic() {
        System.out.println("MemberServiceSingleton.logic()");
    }
}
```



1. static 영역에 객체 instance를 미리 하나 생성해 올려두기
2. MemberServiceSingleton 객체가 필요하면 getInstance()메서드를 통해 조회할 수 있다. 
3. 다른 곳에서 MemberServiceSingleton 객체를 생성하지 못하도록 생성자를 private 으로 막기

[테스트 코드]

```java
@Test
    void singletonTest() {
        // 1. 조회
        MemberServiceSingleton memberServiceSingleton1 = MemberServiceSingleton.getInstance();

        // 2. 조회
        MemberServiceSingleton memberServiceSingleton2 = MemberServiceSingleton.getInstance();

        System.out.println("singleton1 = " + memberServiceSingleton1);
        System.out.println("singleton2 = " + memberServiceSingleton2);
        
        Assertions.assertThat(memberServiceSingleton1).isSameAs(memberServiceSingleton2);
    }
```

[테스트 결과] 

```ruby	
singleton1 = com.example.demo.singleton.MemberServiceSingleton@1fc32e4f
singleton2 = com.example.demo.singleton.MemberServiceSingleton@1fc32e4f
```

테스트 결과 같은 class 임을 확인할 수 있다. 

싱글톤 패턴의 문제점은 다음과 같다. 

* 싱글톤 패턴을 구현하는 코드가 많이 들어감
* 의존관계상 클라이언트가 구체 클래스에 의존 => DIP 위반
* 클라이언트가 구체 클래스에 의존하므로 OCP를 지키기 어렵다.

이 외에도 수많은 문제점들을 가지고 있다. 



## 싱글톤 컨테이너

스프링 컨테이너는 앞서 살펴본 싱글톤 패턴의 문제점들을 해결하면서, 객체 인스턴스를 싱글톤으로 관리한다.

 스프링 컨테이너는 객체를 하나만 생성하여 관리한다. 이러한 스프링 컨테이너의 기능 덕분에 싱글톤 패턴의 단점을 해결하면서 객체를 싱글톤으로 유지 할 수 있다. 

싱글톤 컨테이너가 있을 때 client들의 application에 요청하는 로직은 다음과 같이 관리된다.

![image-20230226000602166]({{site.url}}/images/2023-02-25-Singleton/image-20230226000602166.png)

바로 테스트 코드로 확인 해 보려고 한다.

[테스트 코드]

```java
@Test
    void springContainer() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfiguration.class);

        // 1. 조회
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        // 2. 조회
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        Assertions.assertThat(memberService1).isSameAs(memberService2);
    }
```

[테스트 결과]

```ruby	
memberService1 = com.example.demo.member.MemberServiceImpl@1af7f54a
memberService2 = com.example.demo.member.MemberServiceImpl@1af7f54a
```

**싱글톤 방식의 주의점**

싱글톤 방식을 사용할 때에는 다음 주의사항들을 지켜야 한다.

* 무상태(stateless)로 설계할 것
* 특정 클라이언트에 의존적인 필드가 있으면 안된다.
* 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
* 가급적 읽기만 사용할 것 
* 필드 대신 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal등을 사용할 것 

## @Configuration 적용

기존의 AppConfiguration에서 몇가지 더 추가된 Configuration 파일이 있다고 가정하자.

```java
@Configuration
public class AppConfiguration {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }
  	@Bean
  	public PostService postService(){
      return new PostServieImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemberRepositoryImpl();
    }
}
```

위에서의 Configuration 파일에는 memberRepository가 총 세번 사용이 된다.  

이러한 DI 컨테이너를 사용할 경우에 @Configuration 을 붙이지 않으면 `MemberService에서의 memberRepository`, `PostService에서의 memberRepository`, `MemberRepository의 memberRepository` 로 총 세번 호출이 일어나며 각각의 memberRepository는 다른 memberRepository를 나타낸다.  

하지만 @Configuration 을 붙이면 스프링에서 바이트 코드를 조작해 한번만 호출이 되고 3개의 memberRepository 모두 동일한 memberRepository 임을 보장한다. 

**앞으로  DI 컨테이너를 작성하거나 설정 정보들은 @Configuration 을 붙여서 사용하자**
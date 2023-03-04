---
layout: single
title:  "Bean의 LifeCycle"
categories: Spring
tag: []
toc: true   
author_profile: false
sidebar:
    nav: "main"
search:
    true
---

# Bean의 LifeCycle 

> 스프링 빈의 라이프 사이클은 다음과 같다.
>
> 스프링 컨테이너 생성 => 스프링 빈 생성 => 의존관계 주입 => 
>
> 초기화 콜백 => 사용 => 소멸 전 콜백 => 스프링 종료 

예제와 함께 스프링 빈의 라이프 사이클을 알아보려고 한다. 

다음과 같은 Client 가 있다고 하자



[Client Code]

```java
public class Client{
    private String clientName;

    public Client() {
        System.out.println("생성자 호출, clientName = " + clientName);
        init();
        call("Client 초기화 메세지");
    }

    public void setClientName(String clientName) {
        this.clientName = clientName;
    }

    // 시작 시 호출
    public void init() {
        System.out.println("init: " + clientName);
    }

    public void call(String message) {
        System.out.println("clientName: " + clientName + " message: " + message);
    }

    // 종료 시 호출
    public void close() {
        System.out.println("close: " + clientName);
    }
}

```

client에는 clientName이 있고  생성자에 초기화 하는 기능이 있다. 



[Test Code]

```java
public class BeanLifeCycleTest {
    @Test
    public void lifeCycle() {
        ConfigurableApplicationContext applicationContext = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        Client client = applicationContext.getBean(Client.class);
        applicationContext.close();

    }
    @Configuration
    static class LifeCycleConfig {

        @Bean
        public Client client() {
            Client client = new Client();
            client.setClientName("helloClient");
            return client;
        }
    }
}
```



[Test Result]

```ruby
생성자 호출, clientName = null
init: null
clientName: null message: Client 초기화 메세지
```

당연하게도 Client에는 생성자만 있고 clientName을 변경할 부분이 없어 null로 뜨게 된다. 



# Spring의 Bean LifeCycle Callback

스프링 빈은 객체를 생성하고 의존관계 주입이 끝난 후에 데이터를 사용할 수 있는 준비가 완료된다. 그러므로 **초기화 작업은 의존관계 주입이 모두 완료된 후에 호출해야한다.** 

스프링은 **의존관계 주입이 완료**되면 스프링 빈에게 `콜백 메서드`를 통해 초기화 시점을 알려주는 다양한 기능을 제공한다. 추가적으로 스프링은 **스프링 컨테이너가 종료되기 직전에**` 소멸 콜백`을 준다. 

이 부분을 활용해 초기화 및 종료작업을 진행하면 된다. 



스프링은 크게 3가지 방법으로 빈 생명주기 콜백을 지원한다.

* Interface(InitializingBean, DisposableBean)
* 설정 정보에 초기화 메서드, 종료 메서드 지정 
* @PostConstruct, @PreDestroy 어노테이션 



## Interface (InitializingBean, Disposable Bean)

> 적용하려는 class에 InitializingBean, DisposableBean을 구현해 주면 된다. 



[Client Code]

```java
public class Client implements InitializingBean, DisposableBean{
    private String clientName;

    public Client() {
        System.out.println("생성자 호출, clientName = " + clientName);
    }

    public void setClientName(String clientName) {
        this.clientName = clientName;
    }

    // 시작 시 호출
    public void init() {
        System.out.println("init: " + clientName);
    }

    public void call(String message) {
        System.out.println("clientName: " + clientName + " message: " + message);
    }

    // 종료 시 호출
    public void close() {
        System.out.println("close: " + clientName);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        init();
        call("초기화 연결 메세지");
    }

    @Override
    public void destroy() throws Exception {
        close();
    }
}

```



[Test Code]

```java
public class BeanLifeCycleTest {
    @Test
    public void lifeCycle() {
        ConfigurableApplicationContext applicationContext = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        Client client = applicationContext.getBean(Client.class);
        applicationContext.close();

    }
    @Configuration
    static class LifeCycleConfig {

        @Bean
        public Client client() {
            Client client = new Client();
            client.setClientName("helloClient");
            return client;
        }
    }
}
```



[Test Result]

```ruby
생성자 호출, clientName = null
init: helloClient
clientName: helloClient message: 초기화 연결 메세지
15:17:46.532 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@60bd273d, started on Sat Mar 04 15:17:46 KST 2023
close: helloClient
```

test 결과를 확인하면 초기화 메서드가 적절하게 호출된 것을 알 수 있다. 또한 Spring Container 종료가 호출되자 destroy 메서드가 실행된 것을 확인할 수 있다.

인터페이스를 구현하는 방식은 스프링 초창기에 나온 방식으로  다음과 같은 단점을 갖는다. 

* 스프링 전용 인터페이스
  * 해당 코드가 스프링 인터페이스에 의존 
* 초기화, 소멸 메서드의 이름을 변경할 수 없다. 
* 일부 외부 라이브러리의 경우 적용 불가능



## 설정 정보에 초기화 메서드, 종료 메서드 지정 

> 빈 등록 시에 @Bean(initMethod="{method이름}", destroyMethod="{Method이름}")을 넣어주면 된다. 



[Client Code]

```java
public class Client {
    private String clientName;

    public Client() {
        System.out.println("생성자 호출, clientName = " + clientName);
    }

    public void setClientName(String clientName) {
        this.clientName = clientName;
    }

    // 시작 시 호출
    public void init() {
        System.out.println("init: " + clientName);
    }

    public void call(String message) {
        System.out.println("clientName: " + clientName + " message: " + message);
    }

    // 종료 시 호출
    public void close() {
        System.out.println("close: " + clientName);
    }

    public void initBean() {
        System.out.println("Client.initBean");
        init();
        call("초기화 연결 메세지");
    }

    public void closeBean() {
        System.out.println("Client.closeBean");
        close();
    }
}
```

앞선 코드에서 Interface를 제거해주고 Override메서드 대신 initBean(), closeBean() 이라는 메서드를 만들어 준다. 



[Test Code]

```java
public class BeanLifeCycleTest {
    @Test
    public void lifeCycle() {
        ConfigurableApplicationContext applicationContext = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        Client client = applicationContext.getBean(Client.class);
        applicationContext.close();

    }
    @Configuration
    static class LifeCycleConfig {

      
	@Bean(initMethod = "initBean", destroyMethod = "closeBean")
        public Client client() {
            Client client = new Client();
            client.setClientName("helloClient");
            return client;
        }
    }
}
```

Configuration에서 Bean 등록시에 위 코드와 같이 등록을 해주면 된다. 



[Test Result]

```ruby

생성자 호출, clientName = null
Client.initBean
init: helloClient
clientName: helloClient message: 초기화 연결 메세지
15:26:49.097 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@60bd273d, started on Sat Mar 04 15:26:48 KST 2023
Client.closeBean
close: helloClient
```

정상적으로 동작하는 것을 확인 할 수 있다.

설정 정보를 통한 초기화 메서드, 종료 메서드 지정 방식은 다음과 같은 특징을 갖는다.

* 메서드 이름을 자유롭게 줄 수 있다. 
* 스프링 코드에 의존하지 않는다. 
* 외부 라이브러리에도 적용이 가능하다. 

## @PostConstruct, @PreDestroy 어노테이션 

> 초기화 하고 싶은 메서드에 `@PostConstruce`, 종료 하고 싶은 메서드에 `@PreDestroy` 를 붙여주면 된다.



[Client Code]

```java
public class Client {
    private String clientName;

    public Client() {
        System.out.println("생성자 호출, clientName = " + clientName);
    }

    public void setClientName(String clientName) {
        this.clientName = clientName;
    }

    // 시작 시 호출
    public void init() {
        System.out.println("init: " + clientName);
    }

    public void call(String message) {
        System.out.println("clientName: " + clientName + " message: " + message);
    }

    // 종료 시 호출
    public void close() {
        System.out.println("close: " + clientName);
    }
    @PostConstruct
    public void initBean() {
        System.out.println("Client.initBean");
        init();
        call("초기화 연결 메세지");
    }
    @PreDestroy
    public void closeBean() {
        System.out.println("Client.closeBean");
        close();
    }
}
```



[Test Code]

```java
public class BeanLifeCycleTest {
    @Test
    public void lifeCycle() {
        ConfigurableApplicationContext applicationContext = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        Client client = applicationContext.getBean(Client.class);
        applicationContext.close();

    }
    @Configuration
    static class LifeCycleConfig {

        @Bean
        public Client client() {
            Client client = new Client();
            client.setClientName("helloClient");
            return client;
        }
    }
}
```



[Test Result]

```ruby
생성자 호출, clientName = null
Client.initBean
init: helloClient
clientName: helloClient message: 초기화 연결 메세지
15:53:19.136 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@60bd273d, started on Sat Mar 04 15:53:19 KST 2023
Client.closeBean
close: helloClient
```

어노테이션만으로 초기화 종료 메서드가 잘 실행되는 것을 확인할 수 있다. 



특징

* 스프링에서 권장하는 방법(현재)
* javax.annotation.PostConstruct, javax.annotation.PreDestroy 패키지
  * 스프링이 아닌 다른 컨테이너에서도 도악한다.
* 다른 외부 라이브러리에서 적용이 불가능하다.
  * 외부 라이브러리를 초기화, 종료 해야한다면 @Bean(initMethod, destroyMethod) 를 사용하자

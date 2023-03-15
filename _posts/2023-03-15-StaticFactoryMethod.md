---
layout: single
title:  "[Effective Java] 정적 팩터리 메서드"
categories: Java
tag: [EffectiveJava]
toc: true   
author_profile: false
sidebar:
    nav: "main"
search:
    true
---

> Effective Java 시리즈의 경우 Effective Java책과 inflearn의 백기선님 강의를 토대로 공부하고 정리하여 올리는 글임을 밝힙니다.



# Static Factory Method

* [장점1 바로가기](#장점-1-생성자의-signature가-중복되는-경우-static-factory-method를-사용하여-해결을-할-수-있다)
* [장점2 바로가기](#장점-2-객체-생성을-정적-팩터리-내부에서-관리할-수-있다)
* [장점3 바로가기](#장점-3-반환-타입의-하위-타입-객체를-반환할-수-있는-능력이-있다)
* [장점4 바로가기](#장점-4-입력-매개변수에-따라-매번-다른-클래스의-객체를-반환-할-수-있다)
* [장점5 바로가기](#장점-5-정적-팩터리-메서드를-작성하는-시점에는-반환할-객체의-클래스가-존재하지-않아도-된다)
* [단점1 바로가기](#단점-1-정적-팩터리-메서드만-제공하면-하위-클래스를-만들-수-없다)
* [단점2 바로가기](#단점-2-프로그래머가-찾기-어렵다)

#### 장점 1 생성자의 Signature가 중복되는 경우 Static Factory Method를 사용하여 해결을 할 수 있다. 

[Code] - 정적 팩터리 메서드 적용 전

```java
public class Order {
    private boolean prime;
    private boolean urgent;
    private Product product;

    // 긴급하지 않은 주문
    public Order(Product product, boolean prime) {
        this.product = product;
        this.prime = prime;
    }

    // 긴급 주문
    public Order(boolean urgent, Product product) {
        this.product = product;
        this.urgent = urgent;
    }
  
    }
```

만약 주문이 있는데 긴급 주문, 일반 주문이 있다고 가정하자. 

기본적으로 생성자는 class의 이름과 동일하게 return을 해야하므로 Signature를 줄 수 없다는 단점이 있다. 

추가적으로 지금은 parameter의 순서를 바꾸어 compile error가 나지 않도록 설정을 하였지만, 긴급 주문의 parameter 순서를 바꾸게 되면 compile error까지 발생한다. 

이러한 주문 class를 정적 팩터리 메서드를 적용하면 다음과 같이 바뀐다. 

[Code] - 정적 팩터리 메서드 적용 후 

```java
public class Order {
    private boolean prime;
    private boolean urgent;
    private Product product;

    public static Order urgentOrder(Product product){
        Order order = new Order();
        order.urgent = true;
        order.product = product;
        return order;
    }
    public static Order primeOrder(Product product){
        Order order = new Order();
        order.prime = true;
        order.product = product;
        return order;
    }
}

```

정적 팩터리 메서드의 경우 Signature를 줄 수 있다는 장점이 존재한다.

또한 parameter에 의한 compile error 문제점을 해결할 수 있다. 



---

#### 장점 2. 객체 생성을 정적 팩터리 내부에서 관리할 수 있다. 



[Code] - 정적 팩터리 메서드 적용 전

```java
public class Settings {
    private boolean useAutoSteering;
    private boolean useABS;
    private Difficulty difficulty;
  
}
```

```java
public class Product {
    /**
     * 인스턴스가 계속 생성 된다.
     */
    public static void main(String[] args) {
        System.out.println(new Settings());
        System.out.println(new Settings());
        System.out.println(new Settings());
        System.out.println(new Settings());
    }
}
```

[Result]

```ruby
> Task :Product.main()
hello.effectivev1.chapter01.item01.Settings@6b95977
hello.effectivev1.chapter01.item01.Settings@7e9e5f8a
hello.effectivev1.chapter01.item01.Settings@8bcc55f
hello.effectivev1.chapter01.item01.Settings@58644d46
```

Product 에서 Settings를 읽어들여 호출을 하는 구조라고 가정하자.

현재 Settings 에서는 외부에서 생성자에 대한 접근 제한을 두지 않고 있다. 

이로 인해 Product에서 새로운 Settings를 만들어 계속 호출하고 있는 상황이다. (Result를 통해 확인 할 수 있다.)



[Code] - 정적 팩터리 메서드 적용 후 

```java
public class Settings {
    private boolean useAutoSteering;
    private boolean useABS;
    private Difficulty difficulty;

    private Settings() {}

    private static final Settings SETTINGS = new Settings();
    public static Settings newInstance() {
        return SETTINGS;
    }
}
```

```java
public class Product {
    public static void main(String[] args) {
      
        Settings settings1 = Settings.newInstance();
        Settings settings2 = Settings.newInstance();
        System.out.println(settings1);
        System.out.println(settings2);        
    }
}
```

[Result]

```ruby
> Task :Product.main()
hello.effectivev1.chapter01.item01.Settings@6b95977
hello.effectivev1.chapter01.item01.Settings@6b95977
hello.effectivev1.chapter01.item01.Settings@6b95977
hello.effectivev1.chapter01.item01.Settings@6b95977
```

Settings 에서 생성자를 private로 설정하여 외부에서는 함부러 생성하지 못하도록 구현하였다. 

외부에서는 Settings에서 만들어주는 인스턴스를 사용해야 한다.

즉 Settings 에서 객체의 생성을 관리할 수 있다는 장점이 생긴다. (Result를 확인해보면 같은 Settings 임을 확인할 수 있다.)



#### 장점 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

이 능력은 객체의 클래스를 자유롭게 선택할 수 있는 큰 장점을 가진다. `장점 4`와 같이 예시를 들어 설명을 들도록 한다.

#### 장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환 할 수 있다.

Printer라는 Interface 가 존재하고, 이를 구현한 구현체는 KoreanPrinter, EnglishPrinter가 있고, PrinterFactory에서 이를 사용한다고 가정하자.

[Printer - interface]

```java
public interface Printer {
    String print();
    static Printer of(String language) {
        if (language.equals("kor")) {
            return new KoreanPrinter();
        } else {
            return new EnglishPrinter();
        }
    }
}
```

[Korean Printer]

```java
public class KoreanPrinter implements Printer {
    @Override
    public String print() {
        return "한국어로 출력되는 프린터 입니다.";
    }
}
```

[English Printer]

```java
public class EnglishPrinter implements Printer {
    @Override
    public String print() {
        return "This printer is english version";
    }
}
```

[PrinterFactory]

```java
public class PrinterFactory {
    public static void main(String[] args) {
        Printer korPrinter = Printer.of("kor");
        System.out.println(korPrinter.print());
      
        System.out.println();
      
        Printer engPrinter = Printer.of("eng");
        System.out.println(engPrinter.print());
    }
}
```

[출력 결과 ]

```ruby
한국어로 출력되는 프린터 입니다.

This printer is english version
```

정적 팩터리의 반환 값이 인터페이스이다. 

이는 정적 팩터리에서 인터페이스의 구현체를 자유롭게 사용할 수 있는 큰 장점이 된다. 

Printer 인터페이스에는 of 라는 static 메서드가 존재한다. 

of 라는 메서드 덕에 파라미터가 들어왔을 때 정적 팩터리가 조건에 맞는 클래스를 반환 해 줄 수 있다. 

#### 장점 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

> 이 부분은 아직 이해하기가 조금 힘들어 추후에 조금 더 공부를 하고 적도록 하겠습니다.



#### 단점 1. 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

[장점 2](#장점-2-객체-생성을-정적-팩터리-내부에서-관리할-수-있다) 에서 다룬 Settings 에 private 생성자를 사용했다. 

생성자의 접근 범위가 private인 경우 하위 클래스가 상속을 할 수 없다. 

#### 단점 2. 프로그래머가 찾기 어렵다.

생성자처럼 API 설명에 명확히 드러나 있지 않아 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화 할 방법을 알아내야 한다. 

---



기본적으로 개발자들이 사용하는 명명 방식들이 있다. 

* from
  * 매개변수를 하나 받아 해당 타입의 인스턴스를 반환하는 형변환 메서드
* of
  * 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 메서드
* valueOf
  * from 및 of 보다 더 자세한 버전
* instance or getInstance
  * 매개변수를 받을 때 매개변수로 명시한 인스턴스를 반환 
  * 같은 인스턴스임은 보장하지 않는다.
* create or newInstance
  * 새로운 인스턴스를 생성해 반환함을 보장
* getType
  * getInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드르 정의할 때 사용 
* newType
  * newInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드르 정의할 때 사용 
* type
  * getType과 newType의 간결한 버전 

---

***정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다.***

***정적 팩터리 메서드를 사용하는 것이 public 생성자를 사용하는 것 보다 더 유리한 경우가 많으므로 이를 고려하자***


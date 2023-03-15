---
layout: single
title:  "[Java] 함수형 인터페이스"
categories: Java
tag: []
toc: true   
author_profile: false
sidebar:
    nav: "main"
search:
    true
---



함수형 인터페이스의 조건 

* SAM(Single Abstract Method) 인터페이스
  * 즉 추상 메서드를 단 하나만 가지고 있는 인터페이스
* @FunctionalInteface 를 가지고 있는 인터페이스 

예를 들어 Calculator 라는 인터페이스와 AddCalculator라는 클래스가 있다고 가정하자.

[Calculator]

```java
@FunctionalInterface
public interface Calculator {

    int calculate(int a, int b);

    static String description() {
        return "두가지 값에 대해 연산을 해주는 계산기 입니다";
    }

}
```

Calculator에는 calculate 라는 추상 메서드가 존재한다. 

Calculator에 static 메서드가 존재하지만 이는 상관이 없다. 

**중요한 것은 추상 메서드가 하나여야 한다.**

[AddCalculator]

```java
public class AddCalculator {
    public int add(int number1, int number2) {
      	// 람다식으로 구현 
        Calculator calculator = (a, b) -> {
            return a + b;
        };
        return calculator.calculate(number1, number2);
    }

    public static void main(String[] args) {
        AddCalculator addCalculator = new AddCalculator();
        System.out.println(addCalculator.add(10, 20));
        System.out.println(addCalculator.add(10, 20));
        System.out.println(addCalculator.add(10, 20));
        System.out.println(addCalculator.add(10, 20));
    }
}
```

함수형 인터페이스에 대해서는 위의 코드와 같이 람다 표현식으로 표현할 수 있다. 

추가적으로 함수형 프로그래밍은 다음과 같은 특징을 지닌다.

* 함수를 First class object로 사용이 가능하다.
* 순수 함수여야 한다.(Pure Function)
  * 함수 밖에 있는 값을 변경하지 않는다.
  * 함수 밖에 있는 값을 사용하지 않는다.
* 고차 함수
  * 함수가 함수를 매개변수로 받을 수 있고 함수를 리턴 할 수도 있다.
* 불변성
  * 동일한 입력 값에 대해 동일한 결과를 return 해주어야 한다.

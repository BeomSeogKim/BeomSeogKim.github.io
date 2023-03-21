---
layout: single
title:  "[Spring] Junit - Parameterized Test"
categories: Spring
tag: [Test]
toc: true   
author_profile: false
sidebar:
    nav: "main"
search:
    true
---

테스트 코드를 작성할 때 특정 변수 값에 따른 테스트 코드를 짜야할 경우가 있다. 

이러한 경우 모든 변수를 직접 하드코딩 하여 작성해도 되지만 매개변수 테스트를 사용하면

간단한 코드로 효율적으로 실행을 할 수 있다. 

Set 에 이름을 저장할 때 제대로 저장되었는지 확인하기 위해 테스트코드를 짠다고 가정하자.

```java
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

import java.util.HashSet;
import java.util.Set;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;

public class ExampleTest {
    private static Set<String> names;

    @BeforeAll
    static void setup() {
        names = new HashSet<>();
        names.add("Tommy");
        names.add("Bummy");
        names.add("SSU");
        names.add("Lisa");
    }

    @Test
    void version1() {
        assertAll(
                ()-> assertTrue(names.contains("Tommy")),
                ()-> assertTrue(names.contains("Bummy")),
                ()-> assertTrue(names.contains("SSU")),
                ()-> assertTrue(names.contains("Lisa"))
        );
    }

    @ParameterizedTest
    @ValueSource(strings = {"Tommy", "Bummy", "SSU", "Lisa"})
    void version2(String name) {
        assertTrue(names.contains(name));
    }
}
```

모든 예제에서 먼저 만들어 둔 Set을 활용하기 위해 @BeforeAll 어노테이션으로 Set을 채워주었다. 

`Version1`은 Set에 이름이 제대로 저장되어 있는 지 확인하기 위해서는 4줄의 코드를 작성해야 한다. 

코드를 들여다 보면 변수명만 바뀌고 나머지 코드들은 동일한 역할을 한다. 

`Version2`는 `@ParameterizedTest` 및 `@ValueSource`를 사용하여 코드 한줄만 작성된다. 



Version1, Version2 모두 동일한 로직을 가지고 테스트를 진행한다. 

하지만 **가시성 측면** 및 **중복코드의 제거의 효과**를 얻을 수 있기 때문에 Parameterized Test를 알아보려고 한다. 



---

### Simple Value

> `@ValueSource` 을 사용하여 Parameterized Test를  실행 할 수 있다. 

[예시 코드]

```java
public class ExampleTest{
  @ParameterizedTest
  @ValueSource(ints = {1,2,3,4,5})
  void simpleValue(Integer integer) {
      System.out.println("num of value = " + integer);
  }
}
```

[실행 결과]

![image-20230321155534516]({{site.url}}/images/2023-03-20-ParameterizedTest/image-20230321155534516.png)

단순 값일 경우에는 @ValueSource(`내가 원하는 타입` = {`테스트 하고자 하는 변수 값`})을 명시해준 다음

method의 Parameter에 내가 원하는 타입을 명시해 주면 Parameterized Test를 실행 할 수 있다.

@ValueSource에서 지원하는 타입은 다음과 같다. 

* short
* byte
* int
* long
* float
* double
* char
* java.lang.String
* java.lang.Class



---

### Null & Empty Value

> null 일 경우에 `@NullSource`, empty일 경우에 `@EmptySource`, 
>
> Null 및 Empty일 경우에 `@NullAndEmptySource`를 사용한다.

예시 코드에서는 @NullAndEmptySource에 대해서만 알아본다.

[예시 코드]

```java
public class ExampleTest{
	@ParameterizedTest
  @NullAndEmptySource
  void nullValue(String input) {
      System.out.println(input);
  }
}
```

[실행 결과]

![image-20230321160432910]({{site.url}}/images/2023-03-20-ParameterizedTest/image-20230321160432910.png)



---

### Enum

> `@EnumSource`를 이용해 Parameterized Test를 실행 할 수 있다. 

Enum을 생성을 해주어야 하므로 간단한 Fruit enum을 생성하고 테스트를 진행

[예시 코드]

```java
public class ExampleTest{
	private enum Fruit {
        APPLE, BANANA, PEAR
    }

    @ParameterizedTest
    @EnumSource(Fruit.class)
    void enumValue(Fruit fruit) {
        System.out.println("Enum Value = " + fruit.name());
    }
}
```

[실행 결과]

![image-20230321161239724]({{site.url}}/images/2023-03-20-ParameterizedTest/image-20230321161239724.png)



___

### CSV Literal

> `@CsvSource`를 이용해 Parameterized Test를 실행 할 수 있다. 

CSV Literal 의 경우 delimiter를 사용하지 않은 경우, 사용한 경우로 나누어 예시코드를 알아보자

[delimeter 미사용]

delimeter를 사용하지 않을 경우에는 , 를 기준으로 구분한다.

[예시 코드]

```java
public class ExampleTest{
  @ParameterizedTest
  @CsvSource({"1,Kim", "2,Lee", "3,Choi"})
  void csvlimter(Integer index, String name) {
      System.out.println("index = " + index + " name = " + name);
  }
}
```

[실행 결과]

![image-20230321170209504]({{site.url}}/images/2023-03-20-ParameterizedTest/image-20230321170209504.png)



[delimeter 사용]

delimeter를 사용할 경우에는 @CsvSource 선언시에 delimeter 설정을 넣어주면 된다. 

[예시 코드]

```java
public class ExampleTest{
  @ParameterizedTest
  @CsvSource(value = {"1:Kim", "2:Lee", "3:Choi"}, delimiter = ':')
  void csvDelimiter(Integer index, String name) {
      System.out.println("index = " + index + " name = " + name);
  }
}
```

[실행 결과]

![image-20230321170457081]({{site.url}}/images/2023-03-20-ParameterizedTest/image-20230321170457081.png)



이 외에도 많은 설정들이 가능하지만 많이 사용되는 Parameter 위주로 알아보았다. 

더 많은 방법을 참고하고자 하면 아래 참조 링크에서 확인해 보면 될 것 같다.



---

Reference

[Baeldung](https://www.baeldung.com/parameterized-tests-junit-5)








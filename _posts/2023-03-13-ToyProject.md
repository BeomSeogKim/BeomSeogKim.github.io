---
layout: single
title:  "[ToyProject] 프로젝트 생성하기"
categories: ToyProject
tag: []
toc: true   
author_profile: false
sidebar:
    nav: "main"
search:
    true
---

# 
> 이제까지 내가 쌓아왔던 Spring에 대한 지식들을 하나씩 ToyProject에 녹여보고자,  
> 새로 공부하는 것들을 프로젝트에 기능을 추가해 보고자 시작하게 되었다.



## Project 개발 환경

- IDLE : IntelliJ
- SpringBoot : 3.0.4
- java : 17
- dependency : 
    - spring-boot-starter-web
    - lombok
    - h2    

그동안 기존 프로젝트들은 java 8 버전에 SpringBoot 2.X.X 버전을 사용했었으나 SpringBoot 3.X.X 버전이 출시하여 SpringBoot 3.X.X 버전을 사용하려고 한다.   
(참고로 SpringBoot:3.X.X 버전의 경우 java 17이상을 필요로 한다.)    
[SpringBoot 3.X.X 참고 링크](https://spring.io/blog/2022/05/24/preparing-for-spring-boot-3-0)



## 기본 프로젝트 생성 



![image-20230313112531511]({{site.url}}/images/2023-03-13-ToyProject/image-20230313112531511.png)



1. IntelliJ에서 New Project를 누른 뒤 왼쪽 Generators 에서 `Spring Initializar` 클릭
2. Langauge : Java
3. Type : Gradle-Groovy
4. Group : 하고 싶은 그룹명 아무거나 적으면 된다.
5. Package name: 하고싶은 프로젝트 명을 적으면 된다. 
6. JDK : **꼭 Java 17버전을 선택하도록 하자**
7. Packaging : Jar



![image-20230313112828724]({{site.url}}/images/2023-03-13-ToyProject/image-20230313112828724.png)

1. Spring Boot : SNAPSHOT 혹은 M1 이 붙어 있지 않은 `3.X.X` 버전을 눌러주면 된다.

2. Dependencies:

   * spring-boot-starter-web

   * lombok

   * h2    

3. Create 



## HealthCheckController 생성

과연 내가 만든 프로젝트가 정상적으로 동작하는 지 확인하기 위해 HealCheckController를 생성하자.



![image-20230313113315605]({{site.url}}/images/2023-03-13-ToyProject/image-20230313113315605.png)

위 그림과 같이 controller package를 생성 후에 HealthCheckController Class 생성

[HealthCheckController Code]

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HealthCheckController {

    @GetMapping("/health-check")
    public String healthCheck() {
        return "ok";
    }
}
```



## 정상동작 확인



![image-20230313114140971]({{site.url}}/images/2023-03-13-ToyProject/image-20230313114140971.png)

위 사진에서 왼쪽에 있는 재생 표시를 클릭하거나 상단에 ToyProjectApplication 옆에 있는 재생 표시 클릭한 후에 

http://localhost:8080/health-check 를 입력하여 브라우저 창에 ok 가 뜨면 프로젝트를 정상적으로 생성한 것이다.

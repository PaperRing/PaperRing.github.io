---
title: "[Spring Boot] 하나의 Controller는 수 많은 request를 어떻게 처리할까?"
date: 2023-06-09 11:33:00 +0800
categories: [Spring Boot]
tags: [Spring Boot]
pin: true
math: true
mermaid: true
comments: true
toc: true
layout: post

  
---
> 스프링에서 Singletone으로 Controller는 하나로 관리한다. 수많은 HTTP request가 발생하면 스레드는 요청만큼 생길 텐데, 어떻게 Controller 하나로 많은 스레드를 처리할까?
{: .prompt-warning }


<br>
## 1. Spring은 싱글톤으로 Bean을 생성한다.
스프링이 request를 받으면 Tomcat에 의해 Thread를 전달받는다. 당연히 여러 request가 발생하면 여러 Thread가 생성된다. 이 각각의 Thread가 싱글톤으로 생성된 Bean들을 참고하여 일한다.

정말 싱글톤으로 생성되는게 맞는지 실험한 블로그를 참고해보면,\
=> 출처
<https://programmer.group/spring-controller-singletons-and-thread-safe-things.html>

```java
package com.example.controller;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
        Greeting greet =  new Greeting(counter.incrementAndGet(), String.format(template, name));
        System.out.println("id=" + greet.getId() + ", instance=" + this);
        return greet;
    }
}
```
HTTP 벤치마크 툴인 wrk를 사용해서 많은 수의 요청을 터미널에서 테스트 해봤다.
```terminal
wrk -t12 -c400 -d10s http://127.0.0.1:8080/greeting
```
결과
```log
id=162440, instance=com.example.controller.GreetingController@368b1b03
id=162439, instance=com.example.controller.GreetingController@368b1b03
id=162438, instance=com.example.controller.GreetingController@368b1b03
id=162441, instance=com.example.controller.GreetingController@368b1b03
id=162442, instance=com.example.controller.GreetingController@368b1b03
id=162443, instance=com.example.controller.GreetingController@368b1b03
id=162444, instance=com.example.controller.GreetingController@368b1b03
id=162445, instance=com.example.controller.GreetingController@368b1b03
id=162446, instance=com.example.controller.GreetingController@368b1b03
```

<br>
**GrettingController의 인스턴스 로그 주소가 모두 같으므로, 하나의 컨트롤러를 사용하고 있는 것을 알 수 있었다.**

<br>
## 2. JVM메모리 구조에서 Controller는 공유 영역인, 메소드 영역에 저장된다.
![](https://blog.kakaocdn.net/dn/zgFTg/btspd3swhWV/b0ku2aFeKaU6y366T7TX81/img.png)

JVM 구조에서 Method Area는 모든 쓰레드를 공유해서 사용하며 클래스, 인터페이스, 메서드, 필드, static 변수 등의 바이트 코드를 보관한다. 이때 Controller 객체 하나를 생성하면 객체 자체는 Heap Area에 생성되지만, 해당 Class의 정보는 Method Area에 저장된다.

<br>
**메서드 영역은 모든 스레드가 접근 가능하기 때문에, singletone Controller를 참고하여 각각의 스레드가 실행될 수 있는 것!**

<br>
## 3. 그럼 Controller는 동기화 되어야 하는 걸까?

No, 싱글톤으로 관리되는 Controller는 내부적으로 상태(전역변수, 클래스 변수)를 갖는 것이 없으니, 그냥 메서드를 호출하기만 하면 된다. 쓰레드는 그저 처리 로직만 공유해서 사용하는 것이다. 그러므로 동기화 될 필요가 없다. 따라서 쓰레드는 Controller의 메소드를 공유하고 제각각 호출할 수 있기 때문에 들어오는 요청이 1개든 10만 개든 상관이 없다.

<br>
## 4.결론
Spring이 request를 받고, 톰캣에 의해 전달받은 쓰레드를 하나의 Controller가 처리하는 것이 아니다.\
 **`Controller를 공유하고 참고하여 각각의 Thread가 실행되는 것이다.`**

<br>
<br>
<br>
<br>
<br>

-참고-

<https://programmer.group/spring-controller-singletons-and-thread-safe-things.html>

<https://velog.io/@ejung803/Spring-Web-MVC%EC%97%90%EC%84%9C-%EC%9A%94%EC%B2%AD-%EB%A7%88%EB%8B%A4-Thread%EA%B0%80-%EC%83%9D%EC%84%B1%EB%90%98%EC%96%B4-Controller%EB%A5%BC-%ED%86%B5%ED%95%B4-%EC%9A%94%EC%B2%AD%EC%9D%84-%EC%88%98%ED%96%89%ED%95%A0%ED%85%90%EB%8D%B0-%EC%96%B4%EB%96%BB%EA%B2%8C-1%EA%B0%9C%EC%9D%98-Controller%EB%A7%8C-%EC%83%9D%EC%84%B1%EB%90%A0-%EC%88%98-%EC%9E%88%EC%9D%84%EA%B9%8C%EC%9A%94>

<https://darkstart.tistory.com/255>


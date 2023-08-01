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

라는 의문이 들었다. 결론적으로는 나의 접근 방식이 틀렸다. 

<br>

> ### **Controller가 Thread를 처리하는 것이 아니다. Controller를 공유하고 참고하여 각각의 Thread가 실행되는 것이다.**

<br>
Controller를 공유하고 참고한다니, 어떻게?

스프링이 request를 받으면 Tomcat에 의해 Thread를 전달받는다. 당연히 여러 request가 발생하면 여러 Thread가 생성된다. 이 각각의 Thread가 싱글톤으로 생성된 Bean들을 참고하여 일한다.

자세히 알아보려면, 먼저 JVM의 메모리 구조를 알아야한다. (아래의 포스팅 참고)




JVM 구조에서 Method Area는 모든 쓰레드를 공유해서 사용하며 클래스, 인터페이스, 메서드, 필드, static 변수 등의 바이트 코드를 보관한다. 이때 Controller 객체 하나를 생성하면 객체 자체는 Heap Area에 생성되지만, 해당 Class의 정보는 Method Area에 저장된다.

메서드 영역은 모든 스레드가 접근 가능하기 때문에, singletone Controller를 참고하여 각각의 스레드가 실행될 수 있는 것!

그럼 Controller는 동기화 되어야 하는 걸까?

No, 싱글톤으로 관리되는 Controller는 내부적으로 상태(전역변수, 클래스 변수)를 갖는 것이 없으니, 그냥 메서드를 호출하기만 하면 된다. 쓰레드는 그저 처리 로직만 공유해서 사용하는 것이다. 그러므로 동기화 될 필요가 없다. 따라서 쓰레드는 Controller의 메소드를 공유하고 제각각 호출할 수 있기 때문에 들어오는 요청이 1개든 10만 개든 상관이 없다.

<br>
<br>
<br>
<br>
<br>

-참고-

(https://programmer.group/spring-controller-singletons-and-thread-safe-things.html)

https://velog.io/@ejung803/Spring-Web-MVC%EC%97%90%EC%84%9C-%EC%9A%94%EC%B2%AD-%EB%A7%88%EB%8B%A4-Thread%EA%B0%80-%EC%83%9D%EC%84%B1%EB%90%98%EC%96%B4-Controller%EB%A5%BC-%ED%86%B5%ED%95%B4-%EC%9A%94%EC%B2%AD%EC%9D%84-%EC%88%98%ED%96%89%ED%95%A0%ED%85%90%EB%8D%B0-%EC%96%B4%EB%96%BB%EA%B2%8C-1%EA%B0%9C%EC%9D%98-Controller%EB%A7%8C-%EC%83%9D%EC%84%B1%EB%90%A0-%EC%88%98-%EC%9E%88%EC%9D%84%EA%B9%8C%EC%9A%94

https://darkstart.tistory.com/255

https://darkstart.tistory.com/255

---
title: "[Spring Boot]  Dispatcher Servlet의 동작원리"
date: 2023-04-15 11:33:00 +0800
categories: [Spring Boot]
tags: [Spring Boot]
pin: true
math: true
mermaid: true
comments: true
toc: true
layout: post

  
---
## **Dispatcher Servlet**
---
Spring MVC (Model-View-Controller) 디자인 패턴에서 핵심요소는  Dispatcher Servlet이다. **모든 request 요청을 받아서 필요한 클래스에 넘겨주는 최초 앞단이며, HTTP 요청을 모든 응용 프로그램의 다른 핸들러 및 컨트롤러로 보낸다.** Spring IoC Container와 통합되며 스프링의 모든 기능을 지원한다.

SpringBoot가 등장하기 이전에 DispatcherServlet은 web.xml 파일로 선언되었으나, 스프링부트 등장 이후에는 spring-boot-starter-web 라이브러리를 제공함으로써 DispatcherServlet을 자동으로 등록하고 구성한다. 따라서 더 이상 수동으로 등록 할 필요가 없다.

**디스패처 서블릿의 동작 방식**

![](https://blog.kakaocdn.net/dn/dWVTGV/btso8yxQ9RL/RbGIe20sKD5fqkdbXDUKs1/img.png)

이미지 출처 : https://mangkyu.tistory.com/18

위 그림을 통해 디스패처 서블릿의 동작 방식을 알아보자.

그림에는 나와있지 않지만, 먼저 톰캣이 실행되면 Filter가 web.xml을 읽고 본인이 해야될 일을한다. 그 후 DB에 관련된 애들을 메모리(spring container)에 띄워놓는다.

그 후에,

1) 클라이언트의 request를 디스패처 서블릿이 받는다.

2) 요청 정보를 통해 FrontController패턴으로 분배한다.

3) 요청을 컨트롤러로 위임할 Handler Adapter를 찾아서 넘겨준다.

4) Handler Adapter가 RestController로 요청을 위임한다.

5) 비즈니스 로직을 처리한다.

6) Controller가 response값을 반환한다.

7)  Handler Adapter가 response 처리한다.

8) 서버의 응답을 클라이언트로 반환한다.

**쉽게말해, 디스패처 서블릿을 통해서 요청을 처리할 컨드롤러를 찾아서 넘겨주고 그 결과를 다시 받아온다.**

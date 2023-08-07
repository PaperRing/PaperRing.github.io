---
title: "[Spring Boot] Filter vs Interceptor vs AOP 언제 사용하고 뭐가 다를까?"
date: 2022-08-18 11:33:00 +0800
categories: [Spring Boot]
tags: [Spring Boot]
pin: true
math: true
mermaid: true
comments: true
toc: true
layout: post

  
---
실제 비즈니스 로직이 호출되기 전후로 logging, 인증, 인코딩 변환과 같은 공통적으로 처리해야 할 기능들이 있다.  
이런 공통적인 기능을 하는 코드를 모든 모듈에 붙여준다면!

1) 코드의 중복이 발생하게 되고

2) MSA 기반에서는 각 모듈마다 다른 코드가 작성되어 관리가 어려워질 수 있다.

이 때  **공통 기능을 한번에 관리해서 사용**할 수 있는 방법은  **Filter, Interceptor, AOP** 3가지가 있다.

언제 사용하는지, 차이점은 뭔지 알아보자.

###전체적인 Filter, Interceptor, AOP의 흐름

![](https://blog.kakaocdn.net/dn/xgzhe/btso7RysP91/tCE9BI1NkB3m60Y0lcGVK1/img.png)


<br>
## **1. 개념**
---
### **1) Filter**

필터는 웹서버의 서블릿 컨테이너의 일부이다. 스프링의 바깥쪽에 위치하고 있으며, 필터는 request를 DispatcherServlet에 도달하기 전에 차단하거나 클라이언트에게 도달하기 전에 수정할 수 있다.

> DispatcherServlet이란? 모든 request 요청을 받아서 필요한 클래스에 넘겨주는 최초 앞단이며, HTTP 요청을 모든 응용 프로그램의 다른 핸들러 및 컨트롤러로 보내는 역할을 한다.

일반적으로 스프링 컨텍스트 외부에 존재하며, 스프링과 무관하게 전역적으로 처리해야하는 작업들을 처리할때 사용한다.

- 일반적으로 web.xml에 설정한다.

- WAS 구동 시 FilterMap이라는 배열에 등록되고, 실행 시 Filter chain을 구성하여 순차적으로 실행한다.

- Spring Context 외부에 존재하며 Spring과 무관한 자원에 대해 동작한다.

- Dispatcher Servlet 전, 후 ServletRequest/ServletResponse 객체 변경 및 조작 수행 가능하다.

**Filter에 적합한 처리**

- 인코딩 변환처리

- XSS 방어

**실행 메서드**

- Init() : 필터 인스턴스 초기화

- doFilter() : 전/후 처리

- destroy() : 필터 인스턴스 종료

<br>
### **2) Interceptor**

Interceptor란 Filter와 매우 유사한 형태로 존재한다.

그러나 스프링 외부에 있는  Filter와 달리,

Interceptor는  Dispatcher Servlet이 컨트롤러를 호출하기 전, 후로 끼어들기 때문에 스프링 컨텍스트 내부에서 Controller에 관한 요청과 응답에 대해 처리한다.

- Interceptor는 Dispatcher Servlet에 N개 등록될 수 있다.

- Spring Context 내부에서 컨트롤러의 요청과 응답에 관하여 모든 Bean 객체에 접근할 수 있다.

- 일반적으로 servlet-context.xml에 설정한다.

- 예외 발생 시@ControllerAdvice 에서 @ExceptionHandler를 사용해 예외를 처리한다.

**Interceptor에 적합한 처리**

- 세부적인 보안 및 인증/인가 공통작업 (로그인, 권한, 인증단계)

- API 호출에 대한 로깅

- Controller로 넘겨주는 데이터의 가공

**실행 메서드**

-  preHandle() : 컨트롤러 메소드가 실행되기 전

-  postHandle() : 컨트롤러 메소드 실행 직후 view 페이지 렌더링 되기전

- afterCompletion() : view 페이지가 렌더링 되고 난 후

<br>
### **3) AOP**

AOP는 OOP(Object Oriented Programming)를 보완하기 위해서 나온 개념이다. 객체지향의 프로그래밍을 했을 때 중복을 줄일 수 없는 부분을 줄이기 위해 종단면에서 바라보고 처리한다.

Filter와 Interceptor는 URL으로 전후 처리를 하는 반면에, AOP는 URL, 파라미터, 어노테이션 등 PointCut이 지원하는 다양한 방법으로 대상을 지정할 수 있다. 그러므로 비즈니스단의 메서드에서 좀 더 세밀하게 조정할 수 있다.

Filter와 Interceptor와는 달리 메서드 전후의 지점에 자유롭게 설정이 가능하다.

**AOP에 적합한 처리**

- 로깅

- 트랜잭션

- 에러처리

**포인트컷**

- @Before : 대상 메소드의 수행전

- @After : 대상 메소드의 수행 후

- @After-returning : 대상 메소드의 정상적인 수행 후

- @After-throwing : 예외 발생 후

- @Around : 대상 메소드의 수행 전,후

<br>
## **2. 차이점**
---
**1) 적용 시점이 다름**

Filter -> Interceptor -> AOP

**2) 적용 방식이 다름**

- Filter : web.xml

- Interceptor : servlet-context.xml

**3) 실행 위치가 다름**

- Interceptor, Filter : Servlet 단위에서 실행

- AOP : 메소드 앞에 Proxy 패턴의 형태로 실행

<br>
## **3. Interceptor와 AOP중에서 고민 중 이라면?**

[스프링 MVC의 컨드롤러]

1. 타입이 하나로 정해져 있지 않다.

2. 실행 메소드 또한 제각각이기 때문에 적용할 메소드를 선별하는 포인트컷 작성도 쉽지 않다.

3. 파라미터나 리턴값 또한 일정치 않다.

이에 따라, 컨트롤러 AOP를 사용할 경우 많은 수고가 필요하다.

하지만 스프링 MVC는 모든 종류의 컨트롤러에게 동일한 핸들러 인터셉터를 적용할 수 있게 해준다.

따라서 컨트롤러에 공통적으로 적용할 부가기능이라면 핸들러 인터셉터를 이용하는 편이 낫다.

(토비의 스프링에서 발췌)


<br>
<br>
<br>
<br>
<br>

-참고-

[https://medium.com/javarevisited/spring-framework-filter-vs-dispatcher-servlet-vs-interceptor-vs-controller-745aa34b08d8](https://medium.com/javarevisited/spring-framework-filter-vs-dispatcher-servlet-vs-interceptor-vs-controller-745aa34b08d8)

[https://imnotabear.tistory.com/576](https://imnotabear.tistory.com/576)
https://veneas.tistory.com/entry/Spring-Boot-Filter-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-Response-Body-%ED%95%B8%EB%93%A4%EB%A7%81-HttpServletResponseWrapper

---
title: "[Spring Boot] Spring Container"
date: 2022-05-24 11:33:00 +0800
categories: [Spring Boot]
tags: [Spring Boot]
pin: true
math: true
mermaid: true
comments: true
toc: true
layout: post

  
---

## **1. Container 란?**
---
Container는 보통 객체의 생명주기를 관리, 생성된 인스턴스들에게 추가적인 기능을 제공하도록 하는것이다.

스프링 프레임워크도 이런 역할들 하는 컨테이너가 있는데, **IoC Container  또는  DI Container**  라고 불린다.

**인스턴스의 생성부터 소멸까지의 인스턴스 생명주기 관리를 개발자가 아닌 컨테이너가 대신 해준다.**

**객체관리 주체가 프레임워크(Container)가 되기 때문에 개발자는 로직에 집중할 수 있는 장점이 있다.**

스프링 컨테이너는 XML, 어노테이션 기반의 자바 설정 클래스로 만들 수 있다.

스프링부트를 사용하기 이전에는 xml을 통해 직접적으로 설정해 주어야 했지만, 스프링 부트가 등장하면서 대부분 사용하지 않게 되었다.

<br>
## **2. Spring Container의 종류**
---
스프링 컨테이너는 Beanfactory와 ApplicationContext 두 종류의 인터페이스로 구현되어 있다.

**빈팩토리는 빈의 생성과 관계설정 같은 제어를 담당하는 IoC오브젝트이고, 빈팩토리를 좀 더 확장 한 것이 애플리케이션 컨텍스트이다.**

주로 사용되는 컨테이너는  **ApplicationContext**이다.

![](https://blog.kakaocdn.net/dn/bpvJJS/btsoSl1rnjR/AItXr2RciunLtosrL3Peok/img.png)

**1) Beanfactory**

빈 팩토리는 스프링 컨테이너의 최상위 인터페이스이다. 빈을 등록, 생성, 조회 등의 관리하는 역할을 하며, getBean() 메서드를 통해 빈을 인스턴스화 할 수 있다. @Bean 어노테이션이 붙은 메서드의 이름을 스프링 빈의 이름으로 사용하여 빈 등록을 한다.

ApplicationContext와 다른 점은 Bean Factory에 로드되는 객체들은 미리 로드되지 않고 필요할 때 호출하여 로드하기 때문에 lazy-loading이 된다는 점이다.

<br>
**2) ApplicationContext**

애플리케이션 컨텍스트는 빈 팩토리의 기능을 상속받아 제공한다. 따라서, 빈을 관리하고 검색하는 기능을 BeanFactory가 제공하고, 그 외의 부가 기능은 ApplicationContext에서 제공한다.

수 많은 객체들이 ApplicationContext에 등록된다. 이것을 IoC라고 한다. IoC란 제어의 역전을 의미한다. 개발자가 직접 new를 통해 생성하게 된다면 해당 객체를 가르키는 레퍼런스 변수를 관리하기 어렵다. 그래서 스프링이 직접 해당 객체를 관리한다. 이때 우리는 주소를 몰라도 된다. 왜냐하면 필요할때 DI하면 되기 때문이다. DI는 의존성 주입이라고 한다. 필요한 곳에서 ApplicationContext에 접근하여 필요한 객체를 가져올 수 있다. ApplicationContext는 싱글톤으로 관리되기 때문에 어디에서 접근하든 동일한 객체라는 것을 보장해준다.

@Controller, @Service 등과 같은 어노테이션을 붙여주면 ApplicationContext 알아서 찾아간다.

<br>
## **3. 사용하는 이유**
---
객체를 생성하기 위해서는 new 생성자를 사용해야한다. 그로 인해 애플리케이션에서는 수많은 객체가 존재하고 서로를 참조하게 된다.

객체 간의 참조가 많을수록 의존성이 높아지게 된다.

따라서, 객체 간의 의존성을 낮추어(느슨한 결합) **`결합도는 낮추고`**,  **`높은 캡슐화`**를 위해 스프링 컨테이너가 사용된다.

구현 클레스에 있는 의존성을 제거하고 인터페이스에만 의존하도록 설계할 수 있는 장점이 있다.
<br>
<br>
<br>
<br>
-참고-

[https://chobopark.tistory.com/200](https://chobopark.tistory.com/200)

[https://ittrue.tistory.com/220](https://ittrue.tistory.com/220)

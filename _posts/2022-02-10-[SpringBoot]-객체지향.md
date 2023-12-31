---
title: "[Spring Boot] 객체지향 프로그래밍과 SOLID원칙이란?"
date: 2022-02-10 11:33:00 +0800
categories: [Spring Boot]
tags: [Spring Boot]
pin: true
math: true
mermaid: true
comments: true
toc: true
layout: post

  
---




## **1. 객체지향 프로그래밍(Object-Oriented Programming, OOP)이란?**

---

**객체란?** \
객체는 프로그램에서 사용 되는 데이터 또는 식별자에 의해 참조 되는 공간을 의미하며, 값을 저장 할 변수와 작업을 수행 할 메소드를 서로 연관된 것들끼리 묶어서 만든 것을 객체라고 할 수 있다.

객체지향 프로그래밍은 컴퓨터 프로그램이 각각의 명령어를 순차적으로 수행하는 절차지향을 벗어나, 객체 하나하나가 서로 메세지를 주고 받으며 로직을 수행하는 협력 형태라고 볼 수 있다.

이는 프로그램을 **`유연`**하고 **`변경이 용이`**하게 만든다.\
유연하고 변경이 용이하다는 의미란 예를들어, 레고 블럭으로 성을 만든다고 가정 해보자.\
성의 벽돌 하나를 바꾸려고 할때 다른 블럭들과 결합만 제대로 된다면 쉽게 교체할 수 있다. 이는 변경이 용이하고 유연하다고 할 수 있다.\
이처럼 컴포넌트를 쉽게 변경할 수 있는 방법이 객체 지향 프로그래밍이다.

유지 보수의 편리함, 코드의 재사용 용이, 업무의 분담이 편리, 대규모 소프트웨어 개발에 적합하다는 장점이 있다.


<br>
## **2. 객체지향 프로그래밍의 특징**

---
객체 지향 프로그래밍은 크게 4가지 특성을 가진다.

**1) 추상화**

- 객체에서 공통된 속성과 행위를 추출하는것
- 공통의 속성과 행위를 찾아서 타입을 정의하는 과정
- 추상화는 불필요한 정보는 숨기고 중요한 정보만을 표현함으로써 프로그램을 간단하게 만드는 것

ex) 아우디, 니싼, 볼보는 모두 `자동차`에 해당된다. 자동차라는 추상화 집합을 만들어두고 자동차들이 가진 공통적인 특징을 만들어서 활용한다.

**추상화가 왜 필요할까?**

`자동차` class를 구현해 놓으면 `현대`와 같은 다른 자동차 브랜드를 추가할때, 다른 코드를 수정할 필요없이 object를 추가로 생성해주면 된다.

**2) 캡슐화**

- 데이터 구조와 데이터를 다루는 방법들을 결합 시켜 묶는 것 (변수와 함수를 하나로 묶는 것을 뜻함)
- 낮은 결합도를 유지할 수 있도록 설계하는 것

속성과 기능을 정의하는 변수나 메소드를 클래스라는 캡슐에 넣어서 분류하는 것으로 재활용이 원활하다는 장점이 있고 캡슐화를 통해서 정보은닉을 활용할 수도 있다. (접근제어자의 활용)

**3) 상속**

- 클래스의 속성과 행위를 하위 클래스에 물려주거나 하위 클래스가 상위 클래스의 속성과 행위를 물려받는 것
- 새로운 클래스가 기존의 클래스의 데이터와 연산을 이용할 수 있게 하는 기능

장점)
- 재사용으로 인한 코드가 줄어든다.
- 범용적인 사용이 가능하다.
- 자료와 메서드의 자유로운 사용 및 추가가 가능하다.

단점)
- 상위 클래스의 변경이 어려워진다.
- 불필요한 클래스가 증가할 수 있다.
- 상속이 잘못 사용될 수 있다.

**4) 다형성**

- 하나의 변수명, 함수명이 상황에 따라 다른 의미로 해석 될 수 있는 것
- 어떠한 요소에 여러 개념을 넣어 놓는 것

객체 지향 프로그래밍은 하나의 클래스 내부에 같은 이름의 행위를 여러개 정의하거나(오버로딩), 상위 클래스의 행위를 하위 클래스에서 재정의하여 사용(오버라이딩)할 수 있기때문에 다형성이라는 특징을 가지게된다.

 
<br>
## **3. SOLID (객체 지향 설계 원칙)**

---

객체 지향적으로 설계하기 위해 SOLID라 불리는 다섯가지 원칙이 있다.

**1) SRP (Single Responsibility Principle, 단일 책임 원칙)**\
하나의 클래스는 단 하나의 책임만 가져야 한다.\
ex) UI변경, 객체의 생성과 사용을 분리

**2) OCP (Open-Closed Principle, 개방-폐쇠 원칙)**\
기존의 코드를 변경하지 않으면서 기능을 추가할 수 있도록 설계가 되어야한다.(다형성)

**3) LSP (Liskov Substitution Principle, 리스코프 치환 원칙)**\
프로그램 객체는 프로그램의 정확성을 깨지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야한다.\
상위 타입의 객체를 하위 타입의 객체로 치환해도, 상위 타입을 사용하는 프로그램은 정상적으로 동작해야한다.

**4) ISP (Interface Segregation Principle, 인터페이스 분리 원칙)**\
범용 인터페이스 하나 보다 클라이언트를 위한 여러개의 인터페이스로 구성하는 것이 좋다.

**5) DIP (Dependency Inversion Principle, 의존관계 역전 원칙)**\
추상화에 의존해야지 구체화에 의존하면 안된다.


<br>
<br>
<br>
<br>
-참고-

https://catsbi.oopy.io/8e469ffc-8446-458e-af9d-8b81d5d62d21

https://catsbi.notion.site/SOLID-df957786ea1f49299e8fc74453b12695

https://catsbi.oopy.io/8e469ffc-8446-458e-af9d-8b81d5d62d21

---
title: "[Spring Boot] IoC, DI란?"
date: 2022-01-20 11:33:00 +0800
categories: [Spring Boot]
tags: [Spring Boot]
pin: true
math: true
mermaid: true
comments: true
toc: true
layout: post

  
---



## **1. IoC (Inversion of Control)**
***

**IoC(Inversion Of Control, 제어의 역전)**이란, 간단히 프로그램의 제어 흐름 구조가 뒤바뀌는 것이다.

IoC가 적용된 대표적은 기술은 프레임워크이며, Spring Framework를 예로 들자면 Controller, Service 같은 객체들의 동작(인스턴스 생명주기 관리)을 개발자가 아닌 Spring 컨테이너가 제어한다.

쉽게말해,  객체는 개발자가 구현하지만 어느 시점에 호출 될 지는 Spring이 알아서 한다는 것이다.

스프링 프레임워크가 요구하는대로 객체를 생성하면, 해당 객체들을 가져다 생성하고 호출하고 소멸시킨다.

이런 과정이 개발자 제어권의 역전이다.

**[IoC의 장점]**

-   객체 간 낮은 결합도
-   유연한 코드 작성 가능
-   가독성 증가
-   코드 중복 방지
-   유지보수 용이

**[IoC의 분류]**

![](https://blog.kakaocdn.net/dn/dxTboP/btsowYrJVsZ/NEWQ7K1QYdcUjNoovktrpk/img.png)

출처 : [https://dog-developers.tistory.com/12](https://dog-developers.tistory.com/12)

**1) DL(Dependency Lookup)**

저장소에 저장되어 있는 Bean에 접근하기 위해 컨테이너가 제공하는 API를 이용하여 Bean을 Lookup하는 것.

**2) DI(Dependency Injection)**

각 클래스 간의 의존관계를 빈 설정(Bean Definition) 정보를 바탕으로 컨테이너가 자동으로 연결해 주는 것.

DL 사용시 컨테이너 종속이 증가하기 때문에 주로 DI를 사용한다.

## **2. DI (Dependency Injection)**
***
의존성 주입에는 3가지 방법이 있다.

**1) 필드 주입**

```
@Service
public class MemberService {
    
    @Autowired
    private MemberRepository memberRepository;
}
```

필드에 @Autowired 어노테이션만 붙여주면 자동으로 의존성 주입된다.

[단점]

-   코드가 간결하지만, 외부에서 변경하기 힘들다.
-   프레임워크에 의존적이고 객체지향적으로 좋지않다.

**2) Setter 주입**

```
@Service
public class MemberService {
    
    private MemberRepository memberRepository;
    
    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

Setter 메소드에 @Autowired 어노테이션을 붙이는 방법이다.

[단점]

-   수정자 주입을 사용하면 setXXX 메서드를 public으로 열어두어야 하기 때문에 언제 어디서든 변경이 가능하다.

**3) 생성자 주입 (권장)**

```
@Service
public class MemberService {
    
    private final MemberRepository memberRepository;
    
    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

클래스의 생성자가 하나이고, 그 생성자로 주입받을 객체가 빈으로 등록되어 있다면 @Autowired를 생략할 수 있다.


## **왜 생성자 주입을 권장하는 걸까?**


1.  **순환 참조를 방지할 수 있다.**  
      
    개발을 하다보면 여러 컴포넌트 간에 의존성이 생기는데, A 클래스와 B 클래스와 서로를 순환 참조하는 코드가 있다고 가정해보자.  
      
    필드 주입과 setter주입은 빈이 생성된 후에 참조를 하기 때문에 어플리케이션이 아무런 오류, 경고 없이 구동된다. 그리고 실제 코드가 호출 될 때까지 문제를 알 수 없다.  
      
    반면, 생성자를 통해 주입하고 실행하면  BeanCurrentlyInCreationException이 발생하게 된다.  
    순환 참조 뿐만아니라 더 나아가 의존 관계에 대한 내용을 외부로 노출시킴으로써  **어플리케이션을 실행하는 시점에서 오류를 체크**할 수 있다.  
      
      
    
2.  **불변성(Immutability)**  
      
    생성자로 의존성을 주입할 때  **final로 선언**할 수 있고, 이로 인해 런타임에서 의존성을 주입 받는 객체가 변할 일이 없어지게된다.  
      
    setter주입이나 필드주입을 사용하면 수정의 가능성이 있고, 이는 OOP의 5가지 원칙중 OCP(Open-Closed Principal, 개방-폐쇠의 원칙)을 위반하게 된다.  
      
    **그러므로 생성자 주입을 통해 변경의 가능성을 배제하고 불변성을 보장하는 것이 좋다.**  
      
    또한 fianl로 선언했기 때문에, NullPointerException의 발생을 막는다.  
      
      
    
3.  **테스트에 용이하다.**
      
    독립적으로 인스턴스화가 가능한 POJO(Plain Old Java Object)를 사용하면, DI 컨테이너 없이도 의존성을 주입하여 사용할 수 있다.  
      
    이를 통해 코드 가독성이 높아지며, 유지보수가 용이하고 테스트의 격리성과 예측 가능성을 높일 수 있다는 장점이 있다.

-참고-

<https://dev-coco.tistory.com/80>

<https://mimah.tistory.com/entry/Spring-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85-Dependency-Injection-DI-%EC%98%88%EC%A0%9C>

<https://devmango.tistory.com/174>

<https://dev-coco.tistory.com/70>

---
title: "JVM 메모리구조"
date: 2023-11-12 11:33:00 +0800
categories: [Language]
tags: [SpringBoot, Kotlin, JAVA]
pin: true
math: true
mermaid: true
comments: true
toc: true
layout: post

  
---
<br>
JVM(Java Virtual Machine) 메모리 구조를 쉽게 알아보자. 👊

## **1. JVM(Java Virtual Machine) 이란?**
---
자바 가상 머신(JVM)은 운영체제에 구애받지 않고 자바가 프로그램을 실행할 수 있도록 도와주는 역할을 한다. 자바로 개발한 프로그램을 컴파일하여 만들어지는 바이트코드를 실행하기 위한 가상머신이다.

여기서 궁금증, 자바 가상 머신이니까 Java에서만 동작하는걸까? 🧐

정답은  **No**!

이름에 Java가 들어 있지만 JVM은 자신이 무슨 언어를 실행하는지 전혀 관심이 없다! 자바 바이트코드로 변환이 가능하다면 자바가 아니라 자바 바이트코드를 만드는 다른 언어도 얼마든지 실행할 수 있다.

JVM 기반 언어 : Python, Ruby, JavaScript, Kotilin, Groovy 등

(출처 : 나무위키 JVM)

아래는 자바 프로그램의 실행 단계이다.

![](https://blog.kakaocdn.net/dn/miAQU/btspdXTtGAc/4OBjDKvwNn8Cju2n6eKSwK/img.png)

먼저, 자바 컴파일러가 자바 코드(.java)를 바이트 코드(.class)로 변환시켜준다. 그리고 바이트 코드를 JVM이 읽은 후, 어떤 운영체제이든 프로그램을 실행할 수 있도록 만들어 준다.

<br>
## **2. JVM 메모리 구조**
---

이제 JVM이 내부에서 어떻게 동작하고, 구조는 어떻게 되어있는지 알아보자.JVM의 구조는 크게 4가지로 나눌 수 있다.

![](https://blog.kakaocdn.net/dn/bC5wv1/btspd3FWDzC/6p7EltICOtEPQrd3alXkL0/img.png)

<br>
**1) Class Loader**

Class Loader가 런타임시에 동적으로 JVM 내로 클래스 파일을 로드한다. .class 파일을 묶어서 JVM이 운영체제로부터 할당받은 메모리 영역인 Runtime Data Area로 적재한다.

<br>
**2) Execution Engine**

Class Loader가 적재한 Runtime Data Area에 배치된 .class파일(바이트 코드)을 명령어 단위로 읽어서 실행한다.

<br>
**3) Garbage Collector**

Garbage Collector(GC)는 런타임 데이터 에리어 중 Heap Area에 생성된 객체들 중에서 참조되지 않은 객체들을 탐색 후에 제거하는 역할을 한다. GC가 역할을 하는 시간은 언제인지 정확히 알 수 없다.

**4) Runtime Data Area**

![](https://blog.kakaocdn.net/dn/zgFTg/btspd3swhWV/b0ku2aFeKaU6y366T7TX81/img.png)

JVM의 메모리 영역으로  자바 애플리케이션을 실행할 때 사용되는 데이터를 적재하는 영역이다. 크게 5가지로 나뉜다.

Method Area,  Heap Area는  모든 쓰레드를 공유해서 사용하고, GC의 대상이다.

Stack Area,  PC Register,  Native Method Stack은  쓰레드 마다 하나씩 생성된다.

**(1) Method Area**

모든 쓰레드를 공유해서 사용하며 클래스, 인터페이스, 메소드, 필드, static 변수 등의 바이트 코드를 보관한다.

**(2) Heap Area**

모든 쓰레드를 공유해서 사용하며 new 키워드로 생성된 객체와 배열이 생성된다. Method Area에 로드된 클래스만 생성이 가능하고, GC가 참조되지 않는 메모리를 확인하고 제거하는 영역이다.

**(3) Stack Area**

지역변수, 파라미터, 리턴 값, 연산에 사용되는 임시 값 등이 생성되는 영역이다.

**(4) PC Register**

프로그램 카운터, 즉 현재 쓰레드가 실행되는 부분의 주소와 명령을 저장하고 있는 영역이다.

**(5) Native Method Stack**

자바 외 언어로 작성된 네이티브 코드를 위한 메모리 영역이다.

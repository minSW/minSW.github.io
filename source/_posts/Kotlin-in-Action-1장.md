---
title: '&#039;Kotlin in Action&#039; 1장 - Kotlin (코틀린) 은 뭘까'
date: 2019-07-02 23:07:55
categories: kotlin
tags:
	- kotlin
	- study
	- book
	- summarization
	- mwolkka
---

<br/>

#### 'Kotlin in Action'
공부 겸 프로젝트 준비 겸 Kotlin 책을 하나 샀다.
***'Kotlin in Action'*** 은 Kotlin 언어를 개발한 JetBrains 개발자들이 직접 쓴 책으로, <u>Kotlin 다운 Kotlin 개발</u>을 하기 위해 첫 단추로 택했다.
내가 책 읽는 속도는 빠른데 머리에서 휘발되는 속도도 빠른 편이라(...) 시간 날 때 마다 읽은 부분은 차근차근 정리 해두려고 한다. 
> **Kotlin**은 *최신 멀티플랫폼 애플리케이션을 위한 정적 타입 언어* 로,
> 2017 Google I/O에서 안드로이드 공식언어로 선정되었고 현재 1.3 버전까지 릴리즈 되어있다.
> 
> Kotlin 공식 페이지 https://kotlinlang.org/


<center><h2>_ _ _</h2></center>

<br/>

---

## Overview
#### Java 를 대신할 언어에 대한 Requirement 3가지
1. 정적 타입 지정 (static typing)
2. 기존 자바 코드와의 완전한 호환성
3. 해당 언어를 위한 도구 개발 복잡성 x

⇒ 배우고 이해하기 쉬우며 대규모 개발/유지보수성/기존 자바와의 호환성에 적합한 강력한 언어, <u>**'Kotlin'**</u>

<br/>

# 1장. 코틀린이란, 무엇이며 왜 필요한가?

Kotlin은 자바 플랫폼에서 돌아가는 새로운 프로그래밍 언어다.
**간결하고 실용적**이며 **Java 코드와의 상호운용성** (interoperability)을 중시한다.

## 특성
#### 1. 대상 플랫폼 : 자바가 실행되는 Everywhere

일부가 아닌 개발 과정에서 수행해야하는 모든 과업에 있어 폭넓게 생산성을 향상 시킨다.
구체적인 영역 or 특정 프로그램 패러다임을 지원하는 여러 라이브러리와의 융합성 ↑

<br/>

#### 2. 정적 타입 지정 언어

Kotlin은 **정적 타입 지정 언어** 이면서, <u>type inference</u> (타입추론) 과 <u>nullable type</u>을 지원한다.
⇒ 프로그래머의 불편함 해소 & 컴파일 시점에 NPE 여부 검사 가능


* **정적 타입** (statically typed) 지정 언어 : Java, Kotlin ..

	> 모든 프로그램 구성 요소 타입을 컴파일 시점에 알 수 있고, 객체의 필드나 메소드를 사용할 때마다 컴파일러가 타입을 검증한다.

	장점 : 성능 / 신뢰성 / 유지 보수성 / 도구 지원 _ `p.36`
	
	<br/>

* **동적 타입** (dynamically typed) 지정 언어 : Groovy, JRuby ...

	> 타입과 관계없이 모든 값을 변수에 넣을 수 있고, 필드나 메소드 접근에 대한 검증이 실행 시점에 일어난다. 
	
	동적이 더 유연하고 코드도 짧아지지만, 오류를 사전에 거르지 못하고 Runtime Error 발생 가능성 존재

<br/>

#### 3. 함수형 / 객체지향 프로그래밍

Kotlin으로 코드를 작성 할 땐, Java와 같은 <u>객체지향 프로그래밍 (OOP)</u> 과 <u>함수형 프로그래밍</u> 접근 방법을 조합해서 문제에 가장 적합한 도구를 사용하면 된다.

> **함수형 프로그래밍** 의 핵심 개념
> 
> * first-class function (일급 함수)
> * immutability
> * no side effect _ pure function
> 

<br/>

#### 4. 무료 오픈소스

Kotlin 언어와 이와 관련된 모든 도구는 오픈소스이다.
(https://github.com/jetbrains/kotlin - Apache2 License.)


<br/>

## 응용
#### 코틀린 서버 프로그래밍
> **서버 프로그래밍** 의 범위
> 
> * 브라우저에 HTML 페이지를 반환하는 웹 애플리케이션
> * 모바일 애플리케이션에게 HTTP를 통해 JSON API를  
> * RPC (Remote Procedure Call) 프로토콜을 통해 서로 통신하는 마이크로 서비스
> 

Kotlin은 이러한 애플리케이션 개발에 도움을 주는 기존의 자바 프레임워크나 기술과 매끄럽게 상호운용 가능하다.

\+ 새로운 기술도 적용 가능 (ex. Kotlin의 Builder Pattern, Persistence Framework ...)
  ⇒ `7.5절` & `11장` 에서 좀 더 자세히

<br/>

#### 코틀린 안드로이드 프로그래밍

모바일 애플리케이션은 전형적인 엔터프라이즈 애플리케이션보다 더 작고 기존과 신규 코드 통합 필요성도 더 적고, 다양한 디바이스에 대한 서비스 신뢰성 보장과 빠른 개발&배포가 필요하다.

Kotlin 언어의 특성과 특별한 컴파일러 플러그인 지원을 조합하면 개발 생산성을 더 높일 수 있다.
뿐만 아니라 애플리케이션 신뢰성 향상, 자바6와 완전한 호환, 성능 손실 x 과 같은 장점도 취할 수 있다.

\+ 참조 _ 안드로이드 API에 대한 Kotlin Adaptor를 제공하고 있는 **Anko Library** https://github.com/kotlin/anko

<br/>

## 철학
> 대개 Kotlin은 Java와의 ***상호운용성*** 에 초점을 맞춘 ***실용적*** 이고 ***간결*** 하며 ***안전한*** 언어로 표현된다.
> 

#### 실용성

- 연구를 위한 언어가 아닌, 실제 문제를 해결하기 위해 만들어진 실용적인 언어이다.
- 특정 프로그래밍 스타일이나 패러다임 사용을 강제하지 않는다.
- 도구를 강조한다. (IDE 지원)

<br/>

#### 간결성

- 기존 코드 이해가 더 쉬워진다. (→ 생산성과 개발 속도 향상)
- 부수적인 요소등을 묵시적으로 제공하여 코드가 깔끔하다.
- 람다를 지원한다.
- 그러나 소스코드를 가능한 짧게 만드는 것이 코틀린의 설계 목표는 아니다.

<br/>

#### 안전성

  프로그램의 안전성과 생산성 사이에는 trade-off 존재

- JVM에서 실행한다. (메모리 안전성과 버퍼 플로우 방지등 기본적으로 높은 안전성 확보)
- 정적 타입 지정 언어으로, 애플리케이션의 타입 안전성을 보장한다.
- 실행 시점이 아닌 컴파일 시점에 검사를 통해 더 많은 오류를 방지해준다. (ex. `NullPointerException`, `ClassCastException`)

<br/>

#### 상호운용성

- Java의 기존 라이브러리를 그대로 사용 가능하고, 최대한 활용하고 있다.
- Java ←→ Kotlin 호출에 따로 노력이 필요하지 않다.
- 다중 언어 프로젝트를 완전히 지원한다.

<br/>

## 코틀린 도구 사용

Kotlin도 Java와 마찬가지로 컴파일 언어이다.
![kotlin-runtime-diagram](https://user-images.githubusercontent.com/26691216/60521297-f6b43e80-9d21-11e9-956b-4f827ed75f1b.jpg)
<center><i>Kotlin Build Process</i></center>

<br/>


#### 자바-코틀린 변환
Intellij IDEA에서는 자바 코드 조각을 코틀린 파일(.kt)에 붙여넣기
자바 파일 자체를 변환하려면 `Code > Convert Java File to Kotlin File` 

도구에 대해서는 필요에 따라 쓰면 되는거라 나머지 내용 생략.

<br/>

### 1장 요약 `p.57`
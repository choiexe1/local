---
title:
tags:
  - java
  - programming
publish: true
date: 2024-09-21
---

## String 클래스

`String` 클래스는 문자로 이루어진 배열을 쉽게 다루기 위해 제공하는 클래스다.

소문자로 시작하는 기본형 타입과 다르게, `String` 클래스는 대문자로 시작한다, 즉 참조형 타입이라는 것이다.

그러나 일반적인 참조형 타입과 다르게, `String` 클래스는 문자열을 두 가지 방법으로 생성할 수 있다.

```java
public static void main(String[] args) {
	String str1 = "Hello";
	String str2 = new String("Hello");
}
```

생성자를 이용해 생성하는 방법은 익숙한 방법이다. 그러나 문자열만 넣어서 생성하는 첫번째 방법은 익숙하지 않다.

문자열은 매우 자주 사용되기 때문에, 편의상 쌍따옴표로 문자열을 감싸면 자바가 `new String("Hello")`와 같이 변경해준다.

**문자열의 비밀**

앞서 살펴본 `String` 클래스는 사실 속성으로 문자의 배열을 가지는 클래스다. C언어에서 문자열을 출력하려면 `char` 타입의 문자 배열을 만들어서 사용했던 것과 유사하다.

자바의 버전에 따라 `byte[]`를 속성으로 가지거나, `char[]`를 속성으로 가지고 이 속성에 문자열을 저장한다.

**참조형**

`String` 클래스는 참조형이다. 따라서 `String` 타입의 변수엔 계산할 수 있는 값이 아닌 참조값이 들어있다. 그러므로 원칙적으로 `+` 같은 연산자를 사용할 수 없다.

하지만 문자열은 너무 자주 다루어지기 때문에 자바 언어에서 편의상 특별히 `+` 연산을 제공한다.

**String 비교는 항상 equals()를 사용하라**

대부분의 상황에서 문자열의 비교는 그 값을 비교하지, 참조가 같은지 확인하진 않을 것이다.

어쨌든 `String` 객체를 비교할 때는 `==` 연산자가 아니라, 항상 `equals()`를 사용하여 비교해야한다. `String` 타입은 참조형이기 때문에 해당 유형의 변수는 참조값을 가지기 때문이다.

위 내용은 [[Object 클래스#동일성(Identity)과 동등성(Equality)|Object 클래스 > 동일성과 동등성]]을 참고할 수 있다.

```java
public static void main(String[] args) {
  String str1 = "hello";
  String str2 = "hello";

  System.out.println("리터럴 == 비교: " + (str1 == str2));
  System.out.println("리터럴 equals 비교: " + (str1.equals(str2)));
}

// 출력 결과
// 리터럴 == 비교: true
// 리터럴 equals 비교: true
```

그러나 위 코드는 이런 추측대로 작동하지 않는다. 자바에서 리터럴 문자열을 생성하면, 자동으로 `new Strnig("hello")`로 변환한다.

이 경우 자바는 메모리 효율성과 성능 최적화를 위해 [[풀|문자열 풀]]을 사용한다. 자바 실행 시점에 클래스에 문자열 리터럴이 있으면 문자열 풀에 `String` 인스턴스를 미리 만들어둔다.

이때 같은 문자열이 있으면 만들지 않는다. 즉, 위의 예시 코드에서 `str1`과 `str2`는 문자열 풀에 담겨져 있는 같은 객체의 참조값을 사용하여 비교하게 된다.

따라서 `==` 연산을 사용해도 `true`가 출력되는 것이다.

**불변 객체**

`String` 클래스의 내부 구조를 살펴보면, 문자 배열의 속성에 `final` 키워드를 사용하여 불변 객체로 만들어져 있다.

따라서 문자열의 변경이 필요한 경우 사용하는 메서드들을 사용할 때, 반환값을 사용해야 하는 것을 잊지 말자.

**String이 불변으로 설계된 이유**

앞서 문자열 풀이 잠깐 언급 되었다.

만약 문자열 풀에 있는 `String` 인스턴스의 값이 중간에 변경되면 같은 문자열을 참조하고 있는 변수들의 값도 함께 변경되는 [[사이드 이펙트]]를 방지한다.

> [!note] > `String` 클래스는 문자열을 쉽게 다룰 수 있게 자바에서 특별 대우를 해주는 클래스다.
>
> 불변 객체이다.
>
> 리터럴 문자열은 메모리 효율성과 성능 최적화를 위해 문자열 풀에 보관된 채로 여러 곳에 사용된다.
>
> 문자열 간 비교 연산시 `equals()` 메소드만 사용하여야 한다.

## StringBuilder

`String` 클래스는 불변 객체이다. 따라서 문자를 더하거나 변경할 때 마다 새로운 객체를 생성해야 한다. 문자를 자주 변경해야 하는 상황이라면 더 많은 `String` 객체를 만들고 가비지 컬렉팅 한다.

결과적으로 컴퓨터의 리소스(CPU, 메모리)를 더 많이 사용하게 된다. 그리고 문자열의 크기가 클수록, 문자열을 더 자주 변경할수록 시스템의 자원을 더 많이 소모한다.

`StringBuilder` 클래스는 위 문제를 해결하기 위해 존재한다. 불변 객체가 아니라 가변 객체인것이다.

물론 가변 객체인만큼 사이드 이펙트가 발생하지 않게 주의 해야한다.

따라서 `StringBuilder`는 보통 문자열을 변경하는 동안만 사용하다가, 변경이 끝나면 안전한 불변 객체인 `String` 객체로 변환 하는 것이 좋다.

## String 최적화

**문자열 리터럴 최적화**

```java
// Before compile
String helloWorld = "Hello, " + "World!";

// After compile
String helloWorld = "Hello, World!";
```

자바 컴파일러는 위와 같이 문자열 리터럴 더하기 연산 코드를 자동으로 합친다. 따라서 런타임에 별도의 문자열 결합 연산을 수행하지 않기 때문에 성능이 향상된다.

**String 변수 최적화**

```java
// Before
String result = str1 + str2;

// After
String result = new StringBuilder().append(str1).append(str2).toString();
```

문자열 변수의 경우, 그 안에 어떤 값이 들어있는지 컴파일 시점에는 알 수 없기 때문에 단순하게 합칠 수 없다.

이런 경우 예를 들면 위와 같이 최적화를 수행 한다. 다만 최적화 방식은 자바 버전에 따라 달라진다.

**String 최적화가 어려운 경우**

```java
public static void main(String[] args) {
  long startTime = System.currentTimeMillis();
  String result = "";

  for (int i = 0; i < 100_000; i++) {
    result += "Hello, Java ";
  }

  long endTime = System.currentTimeMillis();

  System.out.println("result = "+ result);
  System.out.println("time = " + (endTime - startTime) + "ms");
}
```

위와 같이 루프 안에서 문자열을 더하는 경우에는 최적화가 이루어지지 않는다.

대략 다음과 같이 최적화가 된다.

```java
String result = "";

for (int i = 0; i < 100_000; i++) {
	result = new StringBuilder().append(result).append("Hello Java ").toString();
}
```

반복문의 내부에서는 최적화가 되는 것 같아 보이지만 반복 횟수만큼 객체를 생성해야 한다.

반복문 내에서 문자열 연결은 런타임에 연결할 문자열의 개수와 내용이 결정된다.

왜냐하면 컴파일러 입장에서는 얼마나 많은 반복이 일어날지, 각 반복에서 문자열이 어떻게 변할지 예측할 수 없다. 따라서 이런 상황에서는 최적화가 어렵다.

따라서 `StringBuilder` 객체에 `append()` 메소드로 문자열을 연결하는 연산, 그것을 다시 `String` 객체로 변환하는 작업을 10만번 한 것이다.

이럴 때는 직접 `StringBuilder`를 사용하면 된다.

내 게임용 윈도우 데스크탑 기준, `String` 사용 시 `6440ms`가 걸렸지만 `StringBuilder`를 사용했을땐 `3ms`밖에 걸리지 않았다.

문자열을 합칠 때 대부분의 경우 최적화 되므로 `+` 연산을 사용하면 된다.

그러나 `StringBuilder`를 사용하는 것이 더 좋은 경우도 있다.

- 반복문에서 반복해서 문자를 연결할 때
- 조건문을 통해 동적으로 문자열을 조합할 때
- 복잡한 문자열의 특정 부분을 변경해야 할 때
- 매우 긴 대용량 문자열을 다룰 때

> [!note] > `StringBuilder` 클래스는 문자열을 가변으로 다룰 수 있게 해주는 클래스다.
>
> 문자열을 동적으로 다뤄야 하는 상황에서 사용한다.
>
> 특정 상황들을 제외하고는 컴파일러가 최적화하므로 일반적으로 `+` 연산을 사용하면 된다.

---

References: 김영한의 실전 자바 - 중급 1편

Links to this page: [[노드와 연결, 연결 리스트]]

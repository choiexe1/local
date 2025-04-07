---
title:
tags:
  - java
  - programming
publish: true
date: 2024-09-22
---

자바는 객체 지향 언어이다. 그런데 객체가 아닌 `int`, `double` 같은 기본형이 존재한다. 기본형은 객체가 아니기 때문에 한계가 있다.

**기본형의 한계**

1. 객체는 유용한 메서드를 제공할 수 있는데, 기본형은 객체가 아니므로 메서드를 제공할 수 없다.
2. 기본형은 `null` 값을 가질 수 없다. 때로는 데이터가 `없음`이라는 상태를 나타내야 할 필요가 있는데 기본형은 항상 값을 가지기 때문에 이런 표현을 할 수 없다.

```java
public static void main(String[] args) {
  int value = 10;
  int i1 = compareTo(value, 5);
  int i2 = compareTo(value, 10);
  int i3 = compareTo(value, 20);

  System.out.println(i1);
  System.out.println(i2);
  System.out.println(i3);

}

public static int compareTo(int value, int target) {
  if (value < target) {
    return -1;
  } else if (value > target) {
    return 1;
  } else {
    return 0;
  }
}
```

위의 예시 코드에서는 `value` 값을 단순히 비교하고 결과에 따라 `-1`, `0`, `1`을 반환하는 메서드가 존재한다.

위와 같은 경우, 만약 `value`가 객체였다면 `value` 객체 스스로 자기 자신의 값과 다른 값을 비교하는 메서드를 제공하는 것이 더 유용할 것이다.

> [!note]
> 기본형은 객체가 아니기 때문에, 객체 지향 언어 관점에서 메서드를 제공하거나 `null` 값을 표현하지 못하는 제약이 존재한다.
>
> 이 문제는 기본형을 한번 감싸는 래퍼 클래스를 만듦으로써 해결할 수 있다.

## 래퍼 클래스

```java
public class MyInteger {
  private final int value;

  public MyInteger(int value) {
    this.value = value;
  }

  public int getValue() {
    return value;
  }

  public int compareTo(int target) {
    if (value < target) {
      return -1;
    } else if (value > target) {
      return 1;
    } else {
      return 0;
    }
  }

  @Override
  public String toString() {
    return String.valueOf(value);
  }
}
```

`int` 값을 가지는 래퍼 클래스 `MyInteger` 클래스다. 캡슐화 되어 있고, 불변 객체이다.

자기 자신의 값과 다른 정수 값을 비교하는 `compare()` 메서드를 가지고, `toString()`을 오버라이딩하여 `value` 속성의 값을 문자열로 반환한다.

이런 형태로 기본형을 내부에 품고, 메서드를 통해 다양한 기능을 추가할 수 있다. 이를 [[래퍼 클래스]]라고 한다.

쉽게 말해, 래퍼 클래스는 기본형의 객체 버전이다.

## 기본 제공 래퍼 클래스

자바는 기본형에 대응하는 래퍼 클래스를 기본으로 제공한다.

- `byte` > `Byte`
- `short` > `Short`
- `int` > `Integer`
- `long` > `Long`
- `float` > `Float`
- `double` > `Double`
- `char` > `Character`
- `boolean` > `Boolean`

이런 기본 래퍼 클래스는, 다음과 같은 특징을 가진다.

- 불변 객체이다
- `equals`로 비교해야 한다.

> [!note] > `Integer.valueOf()` 메서드를 사용하면 `-127`부터 `128`까지 캐싱 해놓고 반환한다. 즉, 문자열 풀처럼 재사용 되는 것이다.
>
> 만약 `129`의 값을 가지는 `Integer`를 생성하려고 하면, `new Integer()`를 호출하여 반환한다.

기본형을 래퍼 클래스로 변경하는 것을 마치 박스에 물건을 넣은 것 같다고 해서 **박싱(Boxing)**이라 한다.

래퍼 클래스에 박싱 되어 있는 값을 꺼내는 것은 **언박싱(Unboxing)**이라고 한다.

## 오토 박싱 (Auto-Boxing), 오토 언박싱 (Auto-Unboxing)

개발자들이 오랜 기간 개발을 하다 보니 기본형을 래퍼 클래스로 변환하거나 또는 반대의 경우로 래퍼 클래스를 기본형으로 변환 하는 일이 자주 발생 했다.

많은 개발자들이 이로 인해 불편함을 호소 했다. 자바는 이런 문제를 해결하기 위해 자바 5부터 오토 박싱, 오토 언박싱(Auto-Unboxing)을 지원한다.

**오토 박싱**

단순히 기본형의 데이터를 래퍼 클래스 타입의 변수에 할당하면 된다.

**오토 언박싱**

반대로, 래퍼 클래스 타입의 변수를 기본형에 할당하면 된다.

```java
// Primitive -> Wrapper
int value = 7;
Integer boxedValue = value;

// Wrapper -> Primitive
int unboxedValue = boxedValue;
```

오토 박싱과 오토 언박싱은 컴파일러가 개발자 대신 래퍼 클래스의 `valueOf()`, `xxxValue()` 등의 메서드로 코드를 변환해주는 것이다.

## 래퍼 클래스와 성능

래퍼 클래스는 객체이기 때문에, 기본형보다 다양한 기능을 제공한다. 그렇다면 더 좋은 래퍼 클래스만 제공하면 되지 기본형을 제공하는 이유는 무엇일까?

```java
public static void main(String[] args) {
  // 기본형 long
  int iterations = 1_000_000_000; // 반복 횟수, 10억
  long startTime, endTime;

  long sumPrimitive = 0;
  startTime = System.currentTimeMillis();
  for (int i = 0; i < iterations; i++) {
    sumPrimitive += i;
  }

  endTime = System.currentTimeMillis();
  System.out.println("sumPrimitive = " + sumPrimitive);
  System.out.println((endTime - startTime) + "ms");

  // 래퍼 클래스 Long  Long sumWrapper = 0L;
  startTime = System.currentTimeMillis();
  for (int i = 0; i < iterations; i++) {
    sumWrapper += i;
  }

  endTime = System.currentTimeMillis();
  System.out.println("sumWrapper = " + sumWrapper);
  System.out.println((endTime - startTime) + "ms");
}

// 출력 결과
// sumPrimitive = 499999999500000000
// 229ms
// sumWrapper = 499999999500000000
// 2159ms
```

내 컴퓨터 기준, 기본형의 연산이 래퍼 클래스 연산보다 대략 10배 정도 빠르다.

기본형은 메모리에서 단순히 그 크기만큼의 공간을 차지한다. 예를 들어 `int`는 보통 4바이트의 메모리를 사용한다.

래퍼 클래스의 인스턴스는 내부에 필드를 가지고 있는 기본형의 값 뿐만 아니라 자바에서 객체를 다루는데 필요한 객체 메타데이터를 포함하므로 더 많은 메모리를 사용한다. 대략 8~16바이트의 메모리를 추가로 사용한다.

**기본형, 래퍼 클래스 어떤 것을 사용?**

- 기본형이든 래퍼 클래스든 반복문에 있는 연산을 1회로 환산하면 둘 다 매우 빠른 연산이다.
- 일반적인 어플리케이션을 만드는 관점에서 보면 이런 부분을 최적화해도 사막의 모래알 하나 정도의 차이가 날 뿐이다.
- CPU 연산을 아주 많이 수행하는 특수한 경우이거나, 수만-수십만 이상 연속해서 연산을 수행해야 하는 경우라면 기본형을 사용해서 최적화를 고려하자.
- 그렇지 않은 경우라면 코드를 유지보수하기 더 나은 것을 선택하면 된다.

> [!note]
>
> 기본형과 래퍼 클래스는 메모리 공간을 얼만큼 사용하냐에 차이가 있다.
>
> CPU 연산을 아주 많이 수행하는 경우나, 수만-수십만 이상 연속해서 연산을 수행해야 할 때는 기본형을 사용하여 최적화를 고려해야 한다.

**유지보수 vs 최적화**

유지보수냐, 최적화냐를 고려해야 하는 상황이라면 유지보수하기 좋은 코드를 먼저 고민해야 한다.
특히 최신 컴퓨터는 매우 빠르기 때문에 메모리 상에서 발생하는 연산을 몇 번 줄인다고해도 실질적인 도움이 되지 않는 경우가 많다.

- 성능 최적화는 단순함보다 복잡함을 요구하고, 더 많은 코드를 추가로 만들어야 한다. 최적화를 위해 유지보수 해야 하는 코드가 늘어나는 것이다. 그런데 진짜 문제는 최적화를 한다고 했지만 전체 어플리케이션의 성능 관점에서 보면 불필요한 최적화를 할 가능성이 있다.
- 특히 웹 어플리케이션의 경우 메모리 안에서 발생하는 연산 하나보다 네트워크 호출 한 번이 많게는 수십만배 더 오래 걸린다.
- 자바 메모리 내부에서 발생하는 연산을 수천번에서 한 번으로 줄이는 것보다, 네트워크 호출 한 번을 더 줄이는 것이 더 효과적인 경우가 많다.
- 권장하는 방법은 개발 이후에 성능 테스트를 해보고 정말 문제가 되는 부분을 찾아서 최적화 하는 것이다.

---

References: 김영한의 실전 자바 - 중급 1편

Links: [[래퍼 클래스]], [[스프링 MVC - 기본 기능]]

---
title:
tags:
  - java
  - programming
publish: true
date: 2024-09-23
---

## System 클래스

`System` 클래스는 시스템과 관련된 기본 기능들을 제공한다.

**표준 입출력, 오류 스트림**

`System.in`, `System.out`, `System.err` 는 각각 표준 입력, 표준 출력, 표준 오류 스트림을 나타낸다.

**시간 측정**

`System.currentTimeMillis()`와 `System.nanoTime()`은 현재 시간을 밀리초 또는 나노초 단위로 제공한다.

**환경 변수**

`System.getenv()` 메서드를 사용하여 OS에서 설정한 환경 변수의 값을 얻을 수 있다.

**시스템 속성**

`System.getProperties()`를 사용해 현재 시스템 속성을 얻거나 `System.getProperty(String key)`로 특정 속성을 얻을 수 있다. 시스템 속성은 자바에서 사용하는 설정 값이다.

**시스템 종료**

`System.exit(int status)` 메서드는 프로그램을 종료하고, OS에 프로그램 종료의 상태 코드를 전달한다.

- 상태 코드 0 : 정상 종료
- 상태 코드 0이 아님 : 오류나 예외적인 종료

**배열 고속 복사**

`System.arraycopy()`는 시스템 레벨에서 최적화된 메모리 복사 연산을 사용한다. 직접 반복문을 사용해서 배열을 복사할 때 보다 수 배 이상 빠른 성능을 제공한다.

```java
package lang.system;

import java.util.Arrays;

public class SystemMain {

  public static void main(String[] args) {
    // 현재 시간 (밀리초)
    long currentTimeMillis = System.currentTimeMillis();
    System.out.println(currentTimeMillis);

    // 현재 시간 (나노초)
    long currentTimeNano = System.nanoTime();
    System.out.println(currentTimeNano);

    // 환경 변수 읽기
    System.out.println(System.getenv());

    // 시스템 속성 읽기
    System.out.println("properties = " + System.getProperties());
    System.out.println("Java version: " + System.getProperty("java.version"));

    // 배열을 고속으로 복사한다.
    // loop로 돌려서 사용하는 것보다 최소 2배에서 5배정도 빠르다.
    // 운영체제에 넘겨서 고속으로 복사한다.
    char[] originalArray = {'h', 'e', 'l', 'l', 'o'};
    char [] copiedArray = new char[5];
    System.arraycopy(originalArray, 0, copiedArray, 0, originalArray.length);

    // 배열 출력
    System.out.println("copiedArray = " + copiedArray);
    System.out.println("Arrays.toString() = " + Arrays.toString(copiedArray));

    // 프로그램 종료
    System.exit(0);
    System.out.println("Hello");
  }
}
```

---

References: 김영한의 실전 자바 - 중급 1편

Links:

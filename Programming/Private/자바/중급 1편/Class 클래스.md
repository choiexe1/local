---
title:
tags:
  - java
  - programming
publish: true
date: 2024-09-23
---

## Class 클래스

어감이 이상하지만 자바에서 `Class` 클래스는 클래스의 정보(메타데이터)를 다루는데 사용된다.

`Class` 클래스를 통해 개발자는 실행 중인 자바 어플리케이션 내에서 필요한 클래스의 속성과 메소드에 대한 정보를 조회하고 조작할 수 있다.

`Class` 클래스의 주요 기능은 아래와 같다.

- 타입 정보 조회: 클래스 이름, 슈퍼 클래스, 인터페이스, 접근 제한자등과 같은 정보 조회
- 리플렉션: 클래스에 정의된 메소드, 필드, 생성자 등을 조회하고, 이들을 통해 객체 인스턴스를 생성하거나 메소드를 호출하는 등의 작업을 할 수 있다.
- 동적 로딩과 생성: `Class.forName()` 메서드를 사용하여 클래스를 동적으로 로드하고, `newInstance()` 메서드를 통해 새로운 인스턴스를 생성할 수 있다.
- 어노테이션 처리: 클래스에 적용된 어노테이션(Annotation)을 조회하고 처리하는 기능을 제공한다.

예를 들어, `String.class`는 `String`에 대한 `Class` 객체를 나타내며, 이를 통해 `String` 클래스에 대한 메타데이터를 조회하거나 조작할 수 있다.

```java
package lang.clazz;

import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ClassMetaMain {

  public static void main(String[] args) throws Exception {
    // Class 조회
    Class clazz = String.class; // 1. 클래스에서 조회
    Class clazz1 = new String().getClass(); // 2. 인스턴스에서 조회
    Class clazz2 = Class.forName("java.lang.String"); // 3. 문자열로 조회

    // 모든 필드 출력
    Field[] fields = clazz.getDeclaredFields();
    for (Field field : fields) {
      System.out.println(field.getType() + " " + field.getName());
    }

    Method[] declaredMethods = clazz.getDeclaredMethods();
    for (Method declaredMethod : declaredMethods) {
      System.out.println(declaredMethod);
    }

    System.out.println("SuperClass = " + clazz.getSuperclass().getName());
    Class[] interfaces = clazz.getInterfaces();
    for (Class i : interfaces) {
      System.out.println("interface = "+ i.getName());
    }
  }
}
```

위의 예제를 실행해보면, `Class` 클래스로 메타데이터를 모두 조회할 수 있다는 것을 알 수 있다.

아래 예제를 확인하면 객체를 생성하는 것도 가능하다는 것을 알 수 있다.

```java
public class ClassCreateMain {

  public static void main(String[] args) throws Exception {
    //Class helloClass = Hello.class;
    Class helloClass = Class.forName("lang.clazz.Hello");


    Hello hello = (Hello) helloClass.getDeclaredConstructor().newInstance();
    String result = hello.hello();

    System.out.println(result);
  }
}
```

이런 작업을 [[리플렉션]]이라고 한다.

> [!note] > `Class` 클래스는 클래스의 정보를 조회하고 조작하는 등의 리플렉션 기능을 제공한다.

---

References: 김영한의 실전 자바 - 중급 1편

Links: [[리플렉션]]

---
tags:
  - java
  - programming
date: 2024-08-21
publish: true
---

## 생성자 오버로딩과 this 키워드

먼저 [[메서드 오버로딩]]이란 클래스 내에 같은 이름의 메서드를 여러 개 정의하는 것을 의미한다.

### 오버로딩의 조건

- 메서드 이름이 같아야 한다.
- 매개변수의 개수 또는 타입이 달라야 한다.

> 첫번째 사례 - 메서드 이름과 매개변수의 갯수가 같으나 매개변수의 이름만 다른 상황

```java
int add(int a, int b) {
	return a + b;
}

int add(int x, int y) {
	return x + y;
}
```

매개변수의 이름이 다르다고 해도, 매개변수의 갯수가 동일하고 타입이 동일하기 때문에 프로그램 실행 시 컴퓨터 입장에서는 어떤 메서드를 호출했는지 결정할 수 없어서 오류가 발생한다.

> 두번째 사례 - 메서드의 반환 타입만 다른 경우

```java
int add(int a, int b) {
	return a + b;
}

long add(int a, int b) {
	return a + b;
}
```

메서드의 반환 타입만 다른 경우에도 `첫번째 사례`와 마찬가지로 어떤 메서드가 호출 되었는지 결정할 수 없기 때문에 오류가 발생한다.

### 생성자 오버로딩의 중복 제거

```java
public class Constructor {
    String name;
    int age;
    int grade;

    Constructor(String name, int age, int grade) {
        this.name = name;
        this.age = age;
        this.grade = grade;
    }

    Constructor(String name, int age) {
        this(name, age, 50);
    }
}
```

생성자 오버로딩 시, this 키워드를 사용하여 객체 자신에게 값을 할당하게 되는데 위 코드의 두 가지의 생성자 중 두번째 생성자 메소드처럼 this 키워드를 함수처럼 사용하게 되면 첫번째 생성자 함수가 호출되어 grade 값을 입력하지 않더라도 기본 값으로 50이라는 정수를 입력하여 객체를 생성하게 된다.

즉, `this()` 를 사용하면, 생성자 내부에서 자신의 생성자를 호출할 수 있게 된다.

### this() 사용 시 제약 조건

- `this()`는 생성자 메소드의 스코프 내에서 첫줄에만 작성할 수 있다.

### 정리

- 생성자는 반드시 호출되어야 한다.
- 생성자가 없으면 자동으로 기본 생성자가 생성된다.
- 만약 생성자가 하나라도 있으면, 기본 생성자가 제공되지 않는다.

---

References: [[메서드 오버로딩]]

Links: [[Programming/Private/김영한의 자바/기본/생성자]]

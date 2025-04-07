---
title:
tags:
  - java
  - programming
date: 2024-09-17
publish: true
---

자바에서 모든 클래스의 최상위 부모 클래스는 항상 Object 클래스이다.

> [!Note]
> 클래스 생성 시, 상속 관계에 상위 부모가 없다면 암묵적으로 Object 클래스를 상속 받는다.

## 자바에서 Object 클래스가 최상위 부모 클래스인 이유

- 공통 기능 제공
- 다형성의 기본 구현

**공통 기능 제공**

객체가 정보를 제공하고, 객체가 다른 객체와 같은지 비교하고, 객체가 어떤 클래스로 만들어졌는지 확인하는 기능은 모든 객체에게 필요한 기본 기능이다. 이런 기능을 객체를 만들 때마다 항상 새로운 메서드를 정의해서 만들어야 한다면 상당히 번거로울 것이다.

막상 만든다고 하더라도 개발자마다 서로 다른 이름의 메서드를 만들어서 일관성이 없을 것이다.

따라서 `Object` 클래스는 모든 객체의 최상위 부모 클래스이기 때문에 모든 객체는 공통 기능을 편리하게 제공 받는다.

**다형성의 기본 구현**

자바의 모든 오브젝트들은 `Object` 클래스를 상속 받으므로 모든 객체에 [[다형적 참조]]가 가능하다.
이처럼 다양한 타입의 객체를 통합적으로 처리할 수 있게 해준다.

![[java-mid-polymorphism-00.png]]

위 사진과 같은 경우 `Object` 클래스를 이용하면 다형적 참조는 가능하지만, 구조적으로 메서드 오버라이딩이 불가능하다. 왜냐하면 `Object` 클래스에는 `sound()`나 `move()` 메서드가 존재하지 않기 때문이다.

따라서 사용하고자 하는 메서드 내부에서 [[다운 캐스팅]]을 통하여 변환한 뒤에나 `Dog` 클래스의 `sound()`나 `Car` 클래스의 `move()` 메서드를 사용할 수 있게 된다.

> [!note] > `Object` 클래스를 이용하면 [[다형적 참조]]는 가능하지만, 기존에 `Object` 클래스에 정의되지 않은 메서드는 [[메서드 오버라이딩]]이 불가능하므로 다형성에 한계가 있다.

## .toString()

`Object` 클래스에서 제공하는 메서드 중 하나이며 객체의 정보를 출력해준다. 클래스 정보와 참조값을 제공하지만 이 정보만으로는 객체의 상태를 적절히 나타내지 못한다. 그래서 보통 `toString()`을 재정의해서 보다 유용한 정보를 제공하는 것이 일반적이다.

```java
public class Dog {
  private String dogName;
  private int age;

  public Dog(String dogName, int age) {
    this.dogName = dogName;
    this.age = age;
  }

  @Override
  public String toString() {
    return "Dog{" +
        "dogName='" + dogName + '\'' +
        ", age=" + age +
        '}';
  }
}

// 출력
// Dog{dogName='멍멍이1', age=2}
// Dog{dogName='멍멍이2', age=5}

```

> [!tip] > `System.out.println()` 메서드는 내부에서 `Object` 클래스의 `toString()` 메서드를 호출한다.

## 동일성(Identity)과 동등성(Equality)

> [!note] > **자바는 두 객체가 같다라는 표현을 두가지로 분리해서 제공한다.**
>
> 동일성
> `==` 연산자를 사용해서 두 객체의 참조가 동일한 객체를 가리키고 있는지 확인
>
> 동등성
> `equals()` 메서드를 사용하여 두 객체가 논리적으로 동등한지 확인

쉽게 말해, 동일성이란 물리적으로 같은 메모리에 있는 객체 인스턴스인지 참조값을 확인하는 것이다.

반면 동등성은 논리적으로 두 객체가 같은 것인지를 확인한다.

```java
User user1 = new UserV1("id-100");
User user2 = new UserV1("id-100");
```

위 예시 코드에서, `User` 객체인 `user1`과 `user2`는 참조값이 같지 않으므로 동일하지 않다. 그러나 논리적으로 보았을 때는 두 객체가 모두 `User` 객체이므로 동등하다.

그러나 실제로 아래 코드를 실행해보면 예상과는 다른 결과가 출력된다.

```java
public class EqualsMainV1 {

  public static void main(String[] args) {
    UserV1 user1 = new UserV1("id-100");
    UserV1 user2 = new UserV1("id-100");

    System.out.println("identity = " + (user1 == user2));
    System.out.println("equality = " + (user1.equals(user2)));
  }
}

// 출력 결과
// identity = false
// equality = false
```

이유는, `Object` 클래스에서 기본으로 제공하는 `equals()`의 내부에서 `==` 연산자를 사용한 결과를 출력하기 때문이다.

동등성이라는 개념은 각각의 클래스마다 다르다. 어떤 클래스는 주민등록번호를 기반으로 동등성을 처리할 수 있고 어떤 클래스는 고객의 연락처를 기반으로 동등성을 처리할 수 있다.

따라서 동등성 비교를 사용하고 싶으면 `equals()`를 재정의 해야한다. 그렇지 않으면 `Object`는 동일성 비교를 기본으로 제공한다.

```java
public class UserV2 {
  private String id;

  public UserV2(String id) {
    this.id = id;
  }

  @Override
  public boolean equals(Object obj) {
    UserV2 user = (UserV2) obj;

    return id.equals(user.id);
  }
}
```

위와 같이 `equals()`를 재정의하여 사용하게 되면, `User` 객체의 `id` 속성을 기준으로 동등성을 비교하게 된다. 그러나 실제 프로젝트에서 사용하기엔 매우 문제가 많은 코드다.

원래라면 인자로 들어온 객체가 비교를 위한 동등할 수 있는 조건을 갖춘 객체인지 여러 조건들을 체크해주어야 한다.

다행히도 실무에선 일반적으론 인텔리제이 IDE가 해당 코드를 생성하는 기능을 사용한다고 한다.

---

References: 김영한의 실전 자바 - 중급 1편

Links:

---
tags:
  - java
  - programming
date: 2024-08-21
publish: true
---

## static 키워드

### 정적 변수

클래스 변수, 스태틱 변수, 정적 변수라고도 부른다.
static 키워드가 붙은 멤버 변수는 인스턴스 영역에 생성되지 않고, 메서드 영역에서 관리한다.

```java
package static1;

public class Data {
    String name;
    static int count;

    Data3(String name) {
        this.name = name;
        count++;
    }
}
```

```java
//정적 변수 접근 시
package static1;

public class DataCountMain3 {
    public static void main(String[] args) {
        Data3 a = new Data3("A");
        System.out.println(Data3.count);  // 출력: 1

        Data3 b = new Data3("B");
        System.out.println(Data3.count);  // 출력: 2
		System.out.println(b.count);
    }
}
```

따라서 생성된 객체(인스턴스)가 아닌 정적 변수

특정 클래스의 인스턴스간에 값을 공유해야 하는 상황에서 사용할 수 있다. 인스턴스를 통하거나 클래스에 직접 접근을 통하여 사용할 수 있다.

하지만 인스턴스에서 정적 변수를 접근하는 것은 권장되지 않는다. 코드를 읽을 때 마치 인스턴스 변수에 접근하는 것 처럼 오해할 수 있기 때문이다.

### 정적 메서드

클래스 메서드에 static을 사용하면, static 변수와 마찬가지로 메서드 영역에서 관리된다. 즉 인스턴스를 생성하지 않고도 해당 클래스의 메서드를 사용할 수 있게 된다.

### 정적 메서드 사용법

- 정적 메서드는 같은 정적 메서드만 사용할 수 있다.
  - 클래스 내부의 기능을 사용할 때, 정적 메서드는 static이 붙은 정적 메서드나 정적 변수만 사용할 수 있다.
  - 클래스 내부의 기능을 사용할 때, 정적 메서드는 인스턴스 변수나 인스턴스 메서드를 사용할 수 없다.
- 반대로 모든 곳에서 static을 호출할 수 있다.
  - 정적 메서드는 공용 기능이다. 따라서 접근 제어자만 허락한다면 클래스를 통해 모든 곳에서 static을 호출 할 수 있다.

이런 규칙이 있는 이유는, 당연히 인스턴스 메서드는 인스턴스가 생성되고 나서야 힙 영역에 생성이 되기 때문이다. 생성이 되었는지의 여부나 참조값 자체도 모르기 때문에 정적 메서드는 인스턴스 메서드나 변수를 사용 할 수 없다.

물론 객체의 참조값을 직접 정적 메소드의 매개변수로 전달하면 정적 메서드도 인스턴스 변수나 메서드를 사용할 수 있게 된다.

### static import

```java
// static import 전
DecoData.staticCall();
DecoData.staticCall();
DecoData.staticCall();

// static import 후
import static static2.DecoData.*;
staticCall();
staticCall();
staticCall();
```

특정 클래스의 정적 메서드를 자주 호출해야 한다면, static import를 통하여 위처럼 사용할 수 있다.

---

References:

Links: [[final]] [[자바의 메모리 구조]]

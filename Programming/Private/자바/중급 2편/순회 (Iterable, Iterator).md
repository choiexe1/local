---
title:
tags:
  - java
  - programming
  - algorithm
  - data-structure
  - collection-framework
  - iterable
  - iterator
  - design-pattern
publish: true
date: 2024-11-10
---

## 순회 1

순회라는 단어는 여러 곳을 돌아다닌다는 뜻이다. 자료구조에 순회는 자료구조에 들어있는 데이터를 차례대로 접근해서 처리하는 것을 순회라고 한다.

그런데 다양한 자료구조가 있고, 각각의 자료구조마다 데이터를 접근하는 방법이 모두 다르다.

![[순회-01.png]]

예를 들어서 배열 리스트는 `index`를 `size`까지 차례로 증가하면서 순회해야 하고, 연결 리스트는 `node.next`를 사용해서 `node`의 끝이 `null`일 때 까지 순회해야 한다. 이렇듯 각 자료구조의 순회 방법이 서로 다르다.

배열 리스트, 연결 리스트, 해시 셋, 연결 해시 셋, 트리 셋 등등 다양한 자료구조가 있다. 각각의 자료구조마다 순회하는 방법이 서로 다르기 때문에, 각 자료구조의 순회 방법을 배워야 한다.

그리고 순회 방법을 배우려면 자료구조의 내부 구조도 알아야한다. 결과적으로 너무 많은 내용을 알아야 하는 것이다. 하지만 자료구조를 사용하는 개발자 입장에서 보면 단순히 자료구조에 들어있는 모든 데이터에 순서대로 접근해서 출력하거나 계산하고 싶을 뿐이다.

자료구조의 구현과 관계 없이 모든 자료구조를 동일한 방법으로 순회할 수 있는 일관성 있는 방법이 있다면, 자료구조를 사용하는 개발자 입장에서 매우 편리할 것이다.

자바는 이런 문제를 해결하기 위해 `Iterable`과 `Iterator` 인터페이스를 제공한다.

## Iterable, Iterator 인터페이스

- **Iterable**: 순회 가능한
- **Iterator**: 반복자

```java title="Iterable 인터페이스의 주요 메서드"
public interface Iterable<T> {
	Iterator<T> iterator();
}
```

- 단순히 `Iterator` 반복자를 반환한다.

```java title="Iterator 인터페이스의 주요 메서드"
public interface Iterator<E> {
	boolean hasNext();
	E next();
}
```

- `hasNext()`: 다음 요소가 있는지 확인한다. 다음 요소가 없으면 `false`를 반환한다.
- `next()`: 다음 요소를 반환한다. 내부에 있는 위치를 다음으로 이동한다.

자료구조에 들어있는 데이터를 처음부터 끝까지 순회하는 방법은 단순하다. 다음 요소가 있는지 확인하고, 있으면 다음 요소를 꺼내는 과정을 반복하면 된다. 다음 요소가 없다면 종료하면 된다.

`Iterable`, `Iterator`를 사용하는 자료구조를 하나 만들어본다. 둘 다 인터페이스여서 구현체가 필요하다.

```java title="MyArrayIterator.java"
public class MyArrayIterator implements Iterator<Integer> {
    private int currentIndex = -1;
    private int[] targetArr;

    public MyArrayIterator(int[] targetArr) {
        this.targetArr = targetArr;
    }

    @Override
    public boolean hasNext() {
        return currentIndex < targetArr.length - 1;
    }

    @Override
    public Integer next() {
        return targetArr[++currentIndex];
    }
}
```

- 생성자를 통해 반복자가 사용할 배열을 참조한다. 여기서 참조한 배열을 순회한다.
- `currentIndex`: 현재 인덱스, `next()` 호출 시 하나씩 증가한다.
- `hasNext()`: 다음 항목이 있는지 검사한다. 배열의 끝에 다다르면 순회가 끝났으므로 `false`를 반환한다.
- `next()`: 다음 항목을 반환한다.
  - `currentIndex`를 하나 증가하고 항목을 반환한다.
  - 인덱스는 `0`부터 시작하기 때문에 `currentIndex`는 처음엔 `-1`을 가진다. 이렇게 하면 다음 항목을 조회할 때 `0`이 된다. 따라서 처음 `next()`를 조회하면 `0`번 인덱스를 가리킨다.

`Iterator`는 단독으로 사용할 수 없다. 순회의 대상이 되는 자료구조를 만들어본다. 여기선 매우 간단한 자료구조를 하나 만든다.

```java title="MyArray.java"
import java.util.Iterator;

public class MyArray implements Iterable<Integer> {
    private int[] numbers;

    public MyArray(int[] numbers) {
        this.numbers = numbers;
    }

    @Override
    public Iterator<Integer> iterator() {
        return new MyArrayIterator(numbers);
    }
}
```

- 배열을 가지는 단순한 자료구조이다.
- `Iterable` 인터페이스를 구현한다.
  - 이 인터페이스는 이 자료구조에 사용할 반복자(`Iterator`)를 반환하면 된다.
  - 앞서 만든 반복자인 `MyArrayIterator`를 반환한다.
  - 이때 `MyArrayIterator`는 생성자를 통해 `MyArray`의 내부 배열인 `numbers`를 참조한다.

```java
public static void main(String[] args) {
    MyArray myArray = new MyArray(new int[]{1, 2, 3, 4});
    Iterator<Integer> iterator = myArray.iterator();

    while (iterator.hasNext()) {
        System.out.print(iterator.next() + " ");
    }
}
```

- `MyArray`는 `Iterable` 인터페이스를 구현한다. 따라서 `MyArray`는 반복할 수 있다는 의미가 된다.
- `Iterable` 인터페이스를 구현하면 `Iterator()`를 구현해야 한다. 이 메서드는 `Iterator` 인터페이스를 구현한 반복자를 반환한다. 여기서는 `MyArrayIterator`를 생성해서 반환했다.

> [!note] 핵심
> 전제조건만 맞는다면 `Iterable` 인터페이스를 사용하여 어떤 자료구조든 순회할 수 있다.
>
> 예시로 이 점을 이용하면 다형성을 통해 어떤 자료구조던 하나의 함수로 순회할 수 있도록 만들 수 있다.
>
> - `Iterator` 인터페이스를 구현한 `Iterator` 클래스를 만든다.
> - `Iterable` 인터페이스를 구현한 자료구조를 만든다.
> - `Iterable` 인터페이스의 구현체는 `Iterator()`를 구현해야 한다. 따라서 만들었던 `Iterator`를 반환하도록 구현한다.

## 순회 2 - 향상된 for문(for-each)의 비밀

`Iterable`과 `Iterator`를 사용하면 또 하나의 큰 장점을 얻을 수 있다.

```java title="for-each"
for (int value : myArray) {
	System.out.println("value = " + value);
}
```

위 코드는 `for-each` 문법이다. `for-each`는 자료 구조를 순회하는 것이 목적이다. 자바는 `Iterable` 인터페이스를 구현한 객체에 대해서 `for-each`문을 사용할 수 있게 해준다.

위 코드는 컴파일 타임에 다음과 같이 코드를 변경한다.

```java title="compile time의 for-each"
while (Iterator.hasNext()) {
	Integer value = iterator.next();
	System.out.println("value = " + value);
}
```

모든 데이터를 순회해야하는 경우라면 깔끔한 `for-each`문을 사용하는 것이 좋다.

> [!note] 만드는 사람이 수고로우면 쓰는 사람이 편하고, 만드는 사람이 편하면 쓰는 사람이 수고롭다.
>
> 특정 자료 구조가 `Iterable`, `Iterator`를 구현한다면 해당 자료구조를 사용하는 개발자는 단순히 `hasNext()`, `next()` 또는 `for-each`문을 사용해서 순회할 수 있다.
>
> 자료구조가 아무리 복잡해도 해당 자료구조를 사용하는 개발자는 동일한 방법으로 매우 쉽게 자료구조를 순회할 수 있다. 이것이 인터페이스가 주는 큰 장점이다.
>
> 물론 자료구조를 만드는 개발자 입장에서는 `Iterable`, `Iterator`를 구현해야 하니 수고롭겠지만, 해당 자료구조를 사용하는 개발자 입장에서는 매우 편리하다.

## 순회 3 - 자바가 제공하는 Iterable, Iterator

![[java-provides-data-structure.png]]

- 자바 컬렉션 프레임워크는 배열 리스트, 연결 리스트, 해시 셋, 연결 해시 셋, 트리 셋 등등 다양한 자료구조를 제공한다.
- 자바는 컬렉션 프레임워크를 사용하는 개발자가 편리하고 일관된 방법으로 자료구조를 순회할 수 있도록 `Iterable` 클래스를 제공하고, 이미 각각의 구현체에 맞는 `Iterator`도 다 구현해두었다.
- 자바 `Collection` 인터페이스의 상위에 `Iterable`이 있다는 것은 모든 컬렉션을 `Iterable`과 `Iterator`를 사용해서 순회할 수 있다는 뜻이다.
- `Map`의 경우 `Key`뿐만 아니라 `Value`까지 있기 때문에 바로 순회할 수는 없다. 대신에 `Key`나 `Value`를 정해서 순회할 수 있는데, `keySet()`, `values()`를 호출하면 `Set`, `Collection`을 반환하기 때문에 `Key`나 `Value`를 정해서 순회할 수 있다. 물론 `Entry`를 `Set` 구조로 반환하는 `entrySet()`도 가능하다.

```java
import java.util.ArrayList;
import java.util.HashSet;
import java.util.Iterator;

public class JavaIterableMain {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        HashSet<Integer> set = new HashSet<>();

        list.add(1);
        list.add(2);
        list.add(3);
        set.add(1);
        set.add(2);
        set.add(3);

        Iterator<Integer> listIterator = list.iterator();
        Iterator<Integer> setIterator = set.iterator();

        System.out.println("== list iterator ==");
        run(listIterator);

        System.out.println("== set iterator ==");
        run(setIterator);
    }

    public static void run(Iterator<Integer> iterator) {
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

- `run(Iterator<Integer> Iterator)`는 `Integer`를 다루는 어떤 `Iterator`던 재사용할 수 있게 된다. 다형성을 통해 범용적인 메서드로 만든 것이다.

```title="실행 결과"
== list iterator ==
1
2
3
== set iterator ==
1
2
3
```

- 핵심은 각각의 자료 구조별로 어떤 `Iterator` 구현체를 반환한다는 것이다.

> [!tip] 참고
> `Iterator` 디자인 패턴은 객체 지향 프로그래밍에서 컬렉션의 요소들을 순회할 때 사용되는 디자인 패턴이다.
>
> 이 패턴은 컬렉션의 내부 표현 방식을 노출시키지 않으면서도 그 안의 각 요소에 순차적으로 접근할 수 있게 해준다.
>
> `Iterator` 패턴은 컬렉션의 구현과는 독립적으로 요소들을 탐색할 수 있는 방법을 제공하며, 이로 인해 코드의 복잡성을 줄이고 재사용성을 높일 수 있다.

---

References: 김영한의 실전 자바 - 중급 2편

Links to this page:

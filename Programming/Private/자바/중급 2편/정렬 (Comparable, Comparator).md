---
title:
tags:
  - java
  - programming
  - algorithm
  - data-structure
  - collection-framework
  - comparable
  - comparator
publish: true
date: 2024-11-11
---

## 정렬 1 - Comparable, Comparator

먼저 예제를 통해서 배열에 들어있는 데이터를 정렬한다.

```java
public static void main(String[] args) {
    Integer[] array = {3, 2, 1};
    System.out.println("array = " + Arrays.toString(array));

    System.out.println("기본 정렬 후");
    Arrays.sort(array);
    System.out.println("array = " + Arrays.toString(array));
}
```

```title="실행 결과"
array = [3, 2, 1]
기본 정렬 후
array = [1, 2, 3]
```

`Arrays.sort()`를 사용하면 배열에 들어있는 데이터를 순서대로 정렬할 수 있다. 하지만 정렬을 하려면 어떤 순서로 정렬할 것인지에 대한 기준이 있어야한다.

### 정렬 알고리즘

정렬은 대략 다음과 같은 방식으로 이루어진다.

![[sort-1.png]]

- 먼저 가장 왼쪽의 데이터와 그 다음 데이터를 비교한다.
- 3과 2를 비교했을 때, 3이 더 크기 때문에 둘을 교환한다.

![[sort-2.png]]

- 다음 차례의 둘을 비교한다.
- 3과 1을 비교했을 때 3이 더 크기 때문에 둘을 교환한다.
- 이렇게 처음부터 끝까지 비교하면 마지막 항목은 가장 큰 값이 된다. 여기서는 3이다.

![[sort-3.png]]

- 처음으로 돌아와서 다시 비교를 시작한다.
- 2와 1을 비교했을 때 2가 더 크기 때문에 둘을 교환한다.
- 최종적으로 1, 2, 3으로 정렬된다.

위 예시의 정렬 알고리즘은 가장 단순한 정렬이다. 실제로는 정렬 성능을 높이기 위한 다양한 알고리즘이 존재한다.

자바는 초기에는 퀵 소트(QuickSort)를 사용했다가 지금은 데이터가 작을 때(32개 이하)는 듀얼 피벗 퀵 소트(Dual-Pivot QuickSort)를 사용하고 데이터가 많을 때는 팀소트(TimSort)를 사용한다.

이런 알고리즘은 평균 `O(n log n)`의 성능을 제공한다.

## 비교자 Comparator

그런데 정렬을 할 때 1, 2, 3의 순서가 아니라 3, 2, 1로 정렬하고 싶다면 어떻게 해야할까?

이때는 비교자를 사용하면 된다. 이름 그대로 두 값을 비교할 때 비교 기준을 직접 제공할 수 있다.

```java
public interface Comparator<T> {
	int compare(T o1, T o2);
}
```

- 두 인수를 비교해서 결과 값을 반환하면 된다.
  - 첫 번째 인수가 더 작으면 음수, 예(`-1`)
  - 두 값이 같으면 `0`
  - 첫 번째 인수가 더 크면 양수, 예(`1`)

백문이 불여일타라고, 비교자를 직접 구현해본다.

```java
public static void main(String[] args) {
    Integer[] array = {3, 2, 1};
    System.out.println("array = " + Arrays.toString(array));
    System.out.println("Comparator 비교");
    Arrays.sort(array, new AscComparator());

    System.out.println("AScComparator = " + Arrays.toString(array));

    Arrays.sort(array, new DescComparator());
    System.out.println("DescComparator = " + Arrays.toString(array));

    Arrays.sort(array, new AscComparator().reversed());
    System.out.println("AscComparator.reversed() = " + Arrays.toString(array));
}

static class AscComparator implements Comparator<Integer> {
    @Override
    public int compare(Integer o1, Integer o2) {
        return (o1 < o2) ? -1 : ((o1 == o2) ? 0 : 1);
    }
}

static class DescComparator implements Comparator<Integer> {
    @Override
    public int compare(Integer o1, Integer o2) {
        return (o1 < o2) ? 1 : ((o1 == o2) ? 0 : -1);
    }
}
```

- `Arrays.sort()`를 사용할 때, 두 번째 인자로 비교자를 넘겨주면 내부적으로 어떤 값이 더 큰지 두 값을 비교할 때 전달한 비교자를 사용한다.
- `Comparator`는 기본적으로 `reversed()`를 제공한다. 계산의 결과를 반대로 만들어주는 유용한 메서드이다.

이렇게 비교자를 사용하면 정렬의 기준을 자유롭게 변경할 수 있다.

## 정렬 2 - Comparable, Comparator

자바가 기본으로 제공하는 `Integer`, `String` 같은 객체를 제외하고 `MyUser`와 같이 직접 만든 객체를 정렬하려면 어떻게 해야 할까?

내가 만든 객체이기 때문에 정렬을 할 때 내가 만든 두 객체 중에 어떤 객체가 더 큰지 알려줄 방법이 있어야 한다.

이 때 `Comparable` 인터페이스를 구현하면 된다. 이 인터페이스는 이름 그대로 비교 가능한, 비교할 수 있는 이라는 뜻으로 객체에 비교 기능을 추가해준다.

```java
public interface Comparable<T> {
	public int compareTo(T o);
}
```

- 자기 자신과 인수로 넘어온 객체를 비교해서 반환하면 된다.
  - 현재 객체가 인수로 주어진 객체보다 더 작으면 음수, 예(`-1`)
  - 두 객체가 같으면 `0`
  - 현재 객체가 인수로 주어진 객체보다 더 크면 양수, 예(`1`)

### MyUser 구현

```java title="MyUser.java"
public class MyUser implements Comparable<MyUser> {
    private String id;
    private int age;

    public MyUser(int age, String id) {
        this.age = age;
        this.id = id;
    }


    public int getAge() {
        return age;
    }

    public String getId() {
        return id;
    }

    @Override
    public String toString() {
        return "MyUser{" +
                "age=" + age +
                ", id='" + id + '\'' +
                '}';
    }

    @Override
    public int compareTo(MyUser o) {
        return this.age < o.age ? -1 : (this.age == o.age ? 0 : 1);
    }
}
```

- `MyUser`는 `Comparable` 인터페이스를 구현한다.
- `compareTo()` 구현을 보면 여기서는 정렬의 기준을 나이인 `age`로 정했다.
- `MyUser` 클래스의 기본 정렬 방식을 나이 오름차순으로 정한 것이다.
- `Comparable`을 통해 구현한 순서를 자연 순서(Natural Ordering)이라 한다.

```java
public static void main(String[] args) {
    MyUser user1 = new MyUser("A", 30);
    MyUser user2 = new MyUser("B", 20);
    MyUser user3 = new MyUser("C", 10);

    MyUser[] array = {user1, user2, user3};
    System.out.println("초기 데이터 = " + Arrays.toString(array));

    Arrays.sort(array);
    System.out.println("Comparable 기본 정렬 = " + Arrays.toString(array));
}
```

```title="실행 결과"
초기 데이터 = [MyUser{age=30, id='A'}, MyUser{age=20, id='B'}, MyUser{age=10, id='C'}]
Comparable 기본 정렬 = [MyUser{age=10, id='C'}, MyUser{age=20, id='B'}, MyUser{age=30, id='A'}]
```

### 다른 방식으로 정렬

만약 객체가 가지고 있는 `Comparable` 기본 정렬이 아니라, 다른 정렬을 사용하고 싶다면 어떻게 해야할까?

나이가 아니라 `id`로 비교하는 예제를 추가로 만들어본다.

```java
public class IdComparator implements Comparator<MyUser> {

    @Override
    public int compare(MyUser o1, MyUser o2) {
        return o1.getId().compareTo(o2.getId());
    }
}
```

- `Comparator`를 구현하는 `IdComparator`를 만든다. 이 후에 `Array.sort(array, Comparator)`에 인수로 사용하면 된다.

```java
Arrays.sort(array, new IdComparator());
System.out.println("IdComparator 정렬" + Arrays.toString(array));

// 실행 결과
// IdComparator 정렬 = [MyUser{age=30, id='A'}, MyUser{age=20, id='B'}, MyUser{age=10, id='C'}]
```

참고로 비교자를 `Array.sort()`에 전달하면 객체가 기본으로 가지고 있는 `Comparable`을 무시하고 별도로 전달한 비교자를 사용해서 정렬한다.

참고로 앞서 사용했던 `reversed()`는 비교자 구현체에서만 제공된다.

> [!warning]
> 만약 `Comparable`도 구현하지 않고, `Comparator`도 제공하지 않으면 다음과 같은 런타임 오류가 발생한다.
>
> `java.lang.ClassCastException: class collection.compare.MyUser cannot be cast to class java.lang.Comparable`

> [!note] 정리
> 객체의 기본 정렬 방법은 객체에 `Comparable`을 구현해서 정의한다. 이렇게 하면 객체는 이름 그대로 비교할 수 있는 객체가 되고 기본 정렬 방법을 가진다.
>
> 그런데 기본 정렬 외에 다른 정렬 방법을 사용하고 싶은 경우 `Comparator`를 별도로 구현해서 정렬 메서드에 전달하면 된다. 이 경우 전달한 `Comparator`가 항상 우선권을 가진다.
>
> 자바가 제공하는 `Integer`, `String`같은 기본 객체들은 대부분 `Comparable`을 구현해두었다.

## 정렬 3 - Comparable, Comparator

정렬은 배열 뿐만 아니라 순서가 있는 `List`같은 자료구조에도 사용할 수 있다.

```java
public static void main(String[] args) {
    MyUser user1 = new MyUser("A", 30);
    MyUser user2 = new MyUser("B", 20);
    MyUser user3 = new MyUser("C", 10);

    LinkedList<MyUser> list = new LinkedList<>();
    list.add(user1);
    list.add(user2);
    list.add(user3);

    System.out.println("기본 데이터 = " + list);

    list.sort(null);
    System.out.println("Comparable 기본 정렬 = " + list);

    list.sort(new IdComparator());
    System.out.println("IdComparator 정렬 = " + list);

	Collections.sort(list);
	System.out.println("Collections.sort() 정렬 = " + list);
}
```

**Collections.sort(list)**

- 리스트는 순서가 있는 컬렉션이므로 정렬할 수 있다.
- 이 메서드를 사용하면 기본 정렬이 적용된다.
- 하지만 이 방식보다는 객체 스스로 정렬 메서드를 가지고 있는 `list.sort()` 사용을 더 권장한다. 참고로 둘의 결과는 같다.
- 비교할 컬렉션 다음 인자로 비교자를 넘기면 전달한 비교자로 비교할 수 있다.

**list.sort(null)**

- 별도의 비교자가 없으므로 `Comparable`로 비교해서 정렬한다.
- 자연스러운 순서(Natural Ordering)로 정렬한다.
- `null`이 아닌 비교자를 전달하면 전달한 비교자로 비교할 수 있다.

### Tree 구조와 정렬

[[자바의 셋 자료구조#자바가 제공하는 Set 2 - TreeSet|TreeSet]]과 같은 이진 탐색 트리 구조는 데이터를 보관할 때, 데이터를 정렬하면서 보관한다. 따라서 정렬 기준을 제공하는 것이 필수다.

![[tree-structure.png]]

이진 탐색 트리는 데이터를 저장할 때 왼쪽 노드에 저장해야 할 지, 오른쪽 노드에 저장해야 할 지 비교가 필요하다. 따라서 `TreeSet`, `TreeMap`은 `Comparable` 또는 `Comparator`가 필수다.

```java
MyUser user1 = new MyUser("A", 30);
MyUser user2 = new MyUser("B", 20);
MyUser user3 = new MyUser("C", 10);

TreeSet<MyUser> treeSet1 = new TreeSet<>();
treeSet1.add(user1);
treeSet1.add(user2);
treeSet1.add(user3);

System.out.println("Comparable 기본 정렬 = " + treeSet1);

TreeSet<MyUser> treeSet2 = new TreeSet<>(new IdComparator());
treeSet2.add(user1);
treeSet2.add(user2);
treeSet2.add(user3);

System.out.println("IdComparator 정렬 = " + treeSet2);
```

- `TreeSet` 인스턴스 생성 시 별도의 비교자를 제공하지 않으면 객체가 구현한 `Comparable`을 사용한다.
- `TreeSet`을 생성할 때 별도의 비교자를 제공하면 `Comparable` 대신 `Comparator`를 사용해서 정렬한다.

> [!warning] 주의
> 만약 `Comparable`도 구현하지 않고, `Comparator`도 제공하지 않으면 다음과 같은 런타임 오류가 발생한다.
>
> `Exception in thread "main" java.lang.ClassCastException: class collection.compare.MyUser cannot be cast to class java.lang.Comparable (collection.compare.MyUser is in unnamed module of loader 'app'; java.lang.Comparable is in module java.base of loader 'bootstrap')`

> [!note] 정리
> 자바의 정렬 알고리즘은 매우 복잡하고, 또 거의 완성형에 가깝다. 만약 여기서 더 성능을 높일 수 있다면 아마도 큰 상을 받을 것이다.
>
> 자바는 개발자가 복잡한 정렬 알고리즘은 신경 쓰지 않으면서 정렬의 기준만 간단히 변경할 수 있도록, 정렬의 기준을 `Comparable`, `Comparator` 인터페이스를 통해 추상화해 두었다.
>
> 객체의 정렬이 필요한 경우 `Comparable`을 통해 기본 자연 순서를 제공하자. 자연 순서 외에 다른 정렬 기준이 추가로 필요하면 `Comparator`를 제공하자.

---

References: 김영한의 실전 자바 - 중급 2편

Links to this page: [[자바의 셋 자료구조]]

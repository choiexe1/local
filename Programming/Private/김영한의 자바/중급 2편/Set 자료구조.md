---
title:
tags:
  - java
  - programming
  - collection-framework
  - data-structure
  - algorithm
  - set
  - hash
publish: true
date: 2024-10-26
---

## List와 Set 자료구조

앞서 학습한 리스트 자료구조를 앞으로 학습할 셋 자료구조와 비교한다.

**리스트 자료구조**

- 요소의 순서를 보장한다.
- 요소의 중복을 허용한다.
- 인덱스를 통해 요소에 접근한다.

**셋 자료구조**

- 요소의 순서를 보장하지 않는다.
- 요소의 중복을 허용하지 않는다.
- 요소의 유무를 빠르게 확인할 수 있도록 최적화되어 있다. 이는 데이터의 중복을 방지하고 빠른 조회를 가능하게 한다.

> [!tip] 실제 사용 예시
> **List**: 장바구니 목록, 순서가 중요한 일련의 이벤트 목록
> **Set**: 회원 ID 집합, 고유한 항목의 집합

## 직접 구현하는 Set 0

셋을 구현하는 것은 아주 단순하다. 인덱스가 없기 때문에 데이터를 넣고, 데이터가 있는지 확인하고, 중복 유무를 확인하고, 데이터를 삭제하는 정도면 충분하다.

- `add(value)`: 셋에 값을 추가한다. 중복 데이터는 저장하지 않는다.
- `contains(value)`: 셋에 값이 있는지 확인한다.
- `remove(value)`: 셋에 있는 값을 제거한다.

예제에선 최대한 단순하게 셋을 구현한다.

```java title="MyHashSetV0.java"
import java.util.Arrays;

public class MyHashSetV0 {
    private int[] elements = new int[10];
    private int size;

    public boolean add(int value) {
        if (contains(value)) {
            return false;
        }
        elements[size] = value;
        size++;

        return true;
    }

    public boolean contains(int value) {
        for (int e : elements) {
            if (value == e) {
                return true;
            }
        }
        return false;
    }

    public int size() {
        return size;
    }

    @Override
    public String toString() {
        return "MyHashSetV0{" +
                "elements=" + Arrays.toString(Arrays.copyOf(elements, size)) +
                ", size=" + size +
                '}';
    }
}
```

- `add()`로 데이터를 추가할 때, 첫 번째 데이터 입력 연산은 `O(1)`이 된다. 그러나 두 번째 데이터부터는 입력 연산이 `O(n)`이 된다. `contains()` 메서드로 해당 셋의 모든 데이터를 확인하여 데이터의 중복 유무를 체크하기 때문이다.

> [!note] 정리
>
> 우리가 만든 셋은 구조는 단순하지만, 데이터 추가, 검색 모두 `O(n)`으로 성능이 좋지 않다. 특히 데이터가 많을 수록 효율은 매우 떨어진다.
>
> 검색의 경우 이전에 보았던 `ArrayList`, `LinkedList`도 `O(n)`이어서 어느정도 받아들 수 있지만, 데이터의 추가가 특히 문제이다.
>
> 데이터를 추가할 때마다 중복 데이터가 있는지 체크하기 위해 셋의 전체 데이터를 확인해야 한다. 이때 O(n)으로 성능이 떨어진다.
>
> 데이터를 추가할 때마다 이렇게 성능이 느린 자료 구조는 사용하기 어렵다. 어떻게 개선 할 수 있을까?

다음 내용은 [[해시 알고리즘]] 문서에 작성한다.

---

References: 김영한의 실전 자바 - 중급 2편

Links to this page: [[해시 알고리즘]], [[해시 알고리즘 2]], [[JDBC 이해]]

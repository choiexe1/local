---
title:
tags:
publish: true
date: 2024-09-23
---

## Math 클래스

`Math` 클래스는 수 많은 수학 문제를 해결해주는 클래스이다.

```java
// 기본 연산 메소드
System.out.println("max(10, 20) = " + Math.max(10, 20)); // 최대값
System.out.println("min(10, 20) = " + Math.min(10, 20)); // 최소값
System.out.println("abs(-10) = " + Math.abs(-10)); // 절대값

// 반올림 및 정밀도 메서드
System.out.println("ceil(2.1) = " + Math.ceil(2.1)); // 올림
System.out.println("floor(2.1) = " + Math.floor(2.1)); // 내림
System.out.println("round(2.1) = " + Math.round(2.1)); // 반올림

// 기타 유용한 메서드
System.out.println("sqrt(4) = " + Math.sqrt(4)); // 제곱근
System.out.println("random() = " + Math.random()); // 0.0 ~ 1.0 사이의 double 값
```

이 외에도 다양한 메소드들이 존재하므로, 필요할 때 검색해서 API 문서를 찾아보아야 한다.

## Random 클래스

랜덤의 경우 `Math.random()`을 사용해도 되지만 `Random` 클래스를 사용하면 더욱 다양한 임의의 값을 구할 수 있다.

- `Math.random()`도 내부에서 `Random` 클래스를 사용한다.
- `Random` 클래스는 `java.util` 패키지 소속이다.

실행 결과는 항상 다르다.

```java
package lang.math;

import java.util.Random;

public class RandomMain {

  public static void main(String[] args) {
    Random random = new Random();

    int randomInt = random.nextInt();
    System.out.println(randomInt);

    double randomDouble = random.nextDouble();
    System.out.println(randomDouble);

    boolean randomBoolean = random.nextBoolean();
    System.out.println(randomBoolean);

    // 범위 조회
    int randomRange1 = random.nextInt(10); // 0 ~ 9까지 출력
    System.out.println(randomRange1);
    int randomRange2 = random.nextInt(10) + 1; // 1 ~ 10까지 출력
    System.out.println(randomRange2);
  }
}
```

랜덤은 내부에서 시드(Seed) 값을 사용해서 랜덤 값을 구한다. 그런데 이 시드의 값이 같으면 항상 같은 결과가 출력된다.

`new Random(int seed)` 생성자를 사용하여 시드 값을 직접 전달할 수 있다. 그러나 시드 값이 같으면 항상 결과가 같아지기 때문에, 테스트 코드 같은 곳에서 사용 된다.

## 문제와 풀이 (로또 번호 생성기)

**요구사항**

- 로또 번호는 1~45 사이의 숫자를 6개 뽑아야 한다.
- 각 숫자는 중복되면 안된다.
- 실행할 때 마다 결과가 달라야 한다.

**`LottoGenerator` 클래스 정의**

```java
public class LottoGenerator {
  private Random random = new Random();
  private int[] lottoNumbers;
  private int count;

  public int[] generate() {
    lottoNumbers = new int[6];

    while (count < 6) {
      int n = random.nextInt(45) + 1;

      if (!isExist(n)) {
        lottoNumbers[count] = n;
        count++;
      }
    }

    return lottoNumbers;
  }

  private boolean isExist(int n) {
    for (int lottoNumber : lottoNumbers) {
      if (n == lottoNumber) {
        return true;
      }
    }

    return false;
  }
}
```

**실행 결과**

```java
import java.util.Arrays;

public class RandomMain {
  public static void main(String[] args) {
    LottoGenerator lotto = new LottoGenerator();
    int[] numbers = lotto.generate();

    System.out.println(Arrays.toString(numbers));
  }
}

// 출력 결과
// [2, 22, 24, 4, 13, 14]
```

---

References: 김영한의 실전 자바 - 중급 1편

Links:

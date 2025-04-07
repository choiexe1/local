---
tags:
  - java
  - programming
date: 2024-08-21
publish: true
---

## instanceof

A 클래스의 인스턴스인지 B 클래스의 인스턴스인지 확인할 수 있는 키워드다.
즉 어떤 변수가 어떤 클래스의 인스턴스인지 확인할 때 사용한다.

`parent instanceof Parent`의 형태로 작성한다. 주로 IF 조건문에 사용 된다.

### instanceof와 함께 변수 선언

자바 16부터 지원하는 기능이다. 이런 기능이 있는 것만 알아두자.

```java
private static void call(Parent parent) {
	parent.parentMethod();
	//Child 인스턴스인 경우 childMethod() 실행
	if (parent instanceof Child child) {
		System.out.println("Child 인스턴스 맞음");
		child.childMethod(); // child 인스턴스일 경우, 조건문의 우측에 child 변수를 선언하고 해당 변수를 조건문 스코프 내부에서 사용한다.
	}
}
```

---

References:

Links: [[캐스팅과 그 종류들]]

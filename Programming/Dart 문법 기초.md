#dart #syntax 

## Data Type
- num
	- int (extends num)
	- double (extends num)
- String
- bool

## Type Inference (타입 추론)
- `var` 키워드를 사용해서 컴파일 타임에 타입 추론을 사용할 수 있다.

```dart
var i = 10; // int
var d = 19.2 // double
var str = 'hello'; // String
```

## dynamic 키워드
- `dynamic` 키워드는 컴파일 타임에 타입을 추론할 수 없으므로 사용하지 않는다.

## 상수
- `final` 키워드는 런타임에 사용하는 상수 키워드이다.
- 불변 객체 또는 필드를 만들 때에 사용하게 될 것 같다.
	```dart
	final String name = "Jay";
	```

- `const` 키워드는 컴파일 타임에 사용하는 상수 키워드이다.
	```dart
	final String name = "Compile";
	```


## Null Safety
- 타입 뒤에 물음표 기호를 붙여서 `nullable`을 표현할 수 있다.
	```dart
	String? nullableVariable;
	```
- 만약 변수가 `null`이 아닌데 타입 추론 기능이 잘 작동하지 않으면 느낌표 기호를 붙여 `null`이 아님을 선언할 수 있다.
	```dart
	String? name;
	String result = name!;
	```
- 그러나 예외가 발생할 수 있으므로 권장되는 방법은 아니다.


## Cascade Notation
- 캐스케이드 표기법은 특정 객체의 속성에 접근할 때 사용하는 표기법이다.
- 코드로 살펴보면 쉽게 이해할 수 있다.

```dart
// 일반적인 객체 접근
var paint = Paint();
paint.color = 'black';
paint.strokeCap = 1;

// Cascade Notation을 활용한 접근
var paint = Paint()
..color = 'black'
..strokeCap = 1;
```

## Type Casting
- `as` 키워드를 통해 타입 캐스팅을 할 수 있다.
```dart
num i = 10;
int ii = i as int;

num d = 10.5;
double dd = d as double;
```


## Named Parameter
- 이름 지정 파라미터는 다음과 같이 함수 인자 부분을 중괄호로 감싼다.
- 이 때 반드시 파라미터의 기본 값을 지정하거나 `required` 키워드를 통해 입력 받도록 지정해야 한다.
```dart
void main() {
  h(a: 10, b: 20);
}

void h({int a = 0, required int b}) {
  print(a + b);
}
```

## Lambda
- 화살표를 통해 람다식으로 함수를 간결히 작성할 수 있다.

```dart
int sum(int a, int b) => a + b
```

## Collections
- List
- Map<type, type>

## Class

```dart
void main() {
  User user = User(1, "Jay", false);
  print(user);
}

class User {
  final int id;
  final String username;
  bool isAdmin;
  
  User(this.id, this.username, this.isAdmin);
}
```

### private
- `dart`에서 `private`는 따로 없고, 언더바를 통해 다음과 같이 사용한다.

```dart
class User {
	final int _id;
	
	User(this._id);
}
```

### static
- `static` 키워드를 사용하면 정적 변수나 함수를 정의할 수 있다.

```dart
class StaticClass {
	static String name = "Jay";
	static void test() {
		print("ABC");
	};
}
```

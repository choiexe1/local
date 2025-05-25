#flutter

### OCP(개방-폐쇄 원칙)란 무엇인가?

개방-폐쇄 원칙은 "소프트웨어 개체(클래스, 모듈, 함수 등)는 확장에 대해서는 개방되어야 하지만, 수정에 대해서는 폐쇄되어야 한다"는 원칙

쉽게 말해, 새로운 기능을 추가할 때는 기존 코드를 변경하지 않고 확장할 수 있어야 한다는 의미

**이점:**
- **유지보수성 향상:** 기능 추가 시 기존 코드에 대한 변경이 적어 버그 발생 가능성이 줄어든다.
- **재사용성 증가:** 잘 정의된 추상화를 통해 여러 곳에서 재사용 가능한 모듈을 만들 수 있다.
- **유연성 증대:** 시스템의 변경에 유연하게 대응할 수 있다.


그런데 이 앞서 설명한 OCP 원칙을 지키는 건 DI 없이 불가능하다. 

### DI 없이 OCP를 지킬 수 없는 이유
플러터에서 `PaymentProcessor` 클래스가 있다고 가정해 봅시다. 이 클래스는 다양한 결제 방법을 처리해야 합니다. 초기에는 `CreditCardPayment`만 지원한다고 가정해 봅시다.

#### 1. DI 없이 직접 구현체에 의존하는 경우 (OCP 위반)
```dart
class CreditCardPayment {
  void processPayment(double amount) {
    print('$amount원 신용카드로 결제 처리 중...');
  }
}

class PaymentProcessor {
  final CreditCardPayment _creditCardPayment = CreditCardPayment();

  void processOrderPayment(double amount) {
    _creditCardPayment.processPayment(amount);
  }
}

void main() {
  final processor = PaymentProcessor();
  processor.processOrderPayment(100.0);
}

```

**이 코드의 문제:**

- 만약 새로운 결제 수단이 추가된다면?
- `PaymentProcessor` 클래스 내부에서 새로운 `if/else` 또는 `switch` 문을 추가하거나, 아니면 `CreditCardPayment` 대신 새로운 결제 클래스의 인스턴스를 직접 생성하도록 **`PaymentProcessor` 자체를 수정**해야 합니다.
- 이는 OCP가 "수정에 대해서는 폐쇄되어야 한다"는 원칙을 명백히 위반한다. 새로운 기능(새로운 결제 수단)이 추가될 때마다 기존 코드를 수정해야 하는 불편함과 위험이 발생하기 때문이다.


#### 2. 추상화는 도입했지만 DI가 없는 경우
그렇다면 추상화를 도입하면 OCP를 지킬 수 있을까요? 예를 들어, `IPaymentMethod` 인터페이스를 만들고 `CreditCardPayment`가 이를 구현하게 해봅시다.

```dart
abstract interface class Payment {
  void processPayment(double amount);
}

// 신용카드 결제 구현체
class CreditCardPayment implements IPaymentMethod {
  @override
  void processPayment(double amount) {
    print('$amount원 신용카드로 결제 처리 중...');
  }
}

// 새로운 카카오페이 결제 구현체
class KakaoPayPayment implements IPaymentMethod {
  @override
  void processPayment(double amount) {
    print('$amount원 카카오페이로 결제 처리 중...');
  }
}

// 결제 처리를 담당하는 클래스 (문제 발생 지점)
class PaymentProcessorWithAbstraction {
  // 인터페이스에 의존하는 것은 맞지만...
  late IPaymentMethod _paymentMethod;

  // 생성자 내부에서 구체적인 구현체를 '직접' 선택하고 생성
  PaymentProcessorWithAbstraction(String methodType) {
    if (methodType == 'creditCard') {
      _paymentMethod = CreditCardPayment(); // 여기서 구체적인 구현체를 선택
    } else if (methodType == 'kakaoPay') {
      _paymentMethod = KakaoPayPayment(); // 여기서 구체적인 구현체를 선택
    } else {
      throw Exception('지원하지 않는 결제 방식입니다.');
    }
  }

  void processOrderPayment(double amount) {
    _paymentMethod.processPayment(amount);
  }
}

void main() {
  final creditCardProcessor = PaymentProcessorWithAbstraction('creditCard');
  creditCardProcessor.processOrderPayment(100.0);

  final kakaoPayProcessor = PaymentProcessorWithAbstraction('kakaoPay');
  kakaoPayProcessor.processOrderPayment(250.0);

  // 새로운 결제 수단 (예: ApplePay) 추가 시, PaymentProcessorWithAbstraction 클래스 '수정' 필요
  // final applePayProcessor = PaymentProcessorWithAbstraction('applePay'); // 이 시점에 PaymentProcessorWithAbstraction를 수정해야 함
}

```

**여전히 OCP 위반:**

- `ApplePayPayment`와 같은 새로운 결제 수단이 추가되면, `PaymentProcessorWithAbstraction`의 생성자 로직(`if/else` 문)을 **직접 수정**해야 합니다.
- 결과적으로, `PaymentProcessorWithAbstraction`는 새로운 기능(새로운 결제 방식) 확장에 대해 폐쇄되어 있지 않고, 수정이 불가피합니다. 추상화를 도입했음에도 불구하고 DI가 없으면 OCP를 완전히 지킬 수 없는 이유입니다.


## 결론
OCP는 소프트웨어의 유지보수성과 유연성을 극대화하기 위한 중요한 원칙입니다. 하지만 단순히 추상화(`interface` 또는 `abstract class`)를 도입하는 것만으로는 OCP를 완전히 지킬 수 없습니다. 클래스 내부에서 구체적인 구현체를 직접 생성하거나 선택하는 방식은 여전히 OCP를 위반하게 됩니다.

**의존성 주입(DI)**은 클래스가 자신이 의존하는 객체의 구체적인 구현을 직접 알 필요 없이, 외부에서 필요한 의존성을 주입받도록 함으로써 **추상화에만 의존할 수 있게 만들어 줍니다.** 이로 인해 시스템의 구성 요소들이 **느슨하게 결합(Loose Coupling)**되고, 새로운 기능이 추가되더라도 기존 코드를 수정할 필요 없이 확장할 수 있게 됩니다.

결국, **OCP를 실질적으로 구현하고 유지하기 위해서는 의존성 주입이 선택이 아닌 필수적인 설계 원칙**이 되는 것입니다.
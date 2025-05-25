#flutter

OCP(Open-Closed Principle), 즉 개방-폐쇄 원칙에 초점을 맞추고, 이 원칙을 지키기 위해 **의존성 주입(Dependency Injection, DI)**이 왜 필수적인지 플러터(Flutter) 코드의 예시를 통해 알아보겠습니다.

### OCP(개방-폐쇄 원칙)란 무엇인가?

개방-폐쇄 원칙은 "소프트웨어 개체(클래스, 모듈, 함수 등)는 확장에 대해서는 개방되어야 하지만, 수정에 대해서는 폐쇄되어야 한다"는 원칙입니다. 쉽게 말해, 새로운 기능을 추가할 때는 기존 코드를 변경하지 않고 확장할 수 있어야 한다는 의미입니다.

**이점:**
- **유지보수성 향상:** 기능 추가 시 기존 코드에 대한 변경이 적어 버그 발생 가능성이 줄어듭니다.
- **재사용성 증가:** 잘 정의된 추상화를 통해 여러 곳에서 재사용 가능한 모듈을 만들 수 있습니다.
- **유연성 증대:** 시스템의 변경에 유연하게 대응할 수 있습니다.


그런데 이 앞서 설명한 OCP 원칙을 지키는 건 DI 없이 불가능 합니다. 예시를 보겠습니다.

그런데 이 앞서 설명한 OCP 원칙을 지키는 건 DI 없이 불가능합니다. 예시를 보겠습니다.

### DI 없이 OCP를 지킬 수 없는 이유
플러터에서 `PaymentProcessor` 클래스가 있다고 가정해 봅시다. 이 클래스는 다양한 결제 방법을 처리해야 합니다. 초기에는 `CreditCardPayment`만 지원한다고 가정해 봅시다.

#### 1. DI 없이 직접 구현체에 의존하는 경우 (OCP 위반)
```dart
// 구체적인 신용카드 결제 로직
class CreditCardPayment {
  void processPayment(double amount) {
    print('$amount원 신용카드로 결제 처리 중...');
  }
}

// 결제 처리를 담당하는 클래스
class PaymentProcessor {
  // PaymentProcessor가 CreditCardPayment에 직접 의존
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

이 코드는 `PaymentProcessor`가 `CreditCardPayment`라는 **구체적인 구현체에 직접 의존**하고 있습니다.

**문제점:**

- 만약 `KakaoPayPayment`나 `ApplePayPayment`와 같은 새로운 결제 수단을 추가해야 한다면 어떻게 될까요?
- `PaymentProcessor` 클래스 내부에 새로운 `if/else` 또는 `switch` 문을 추가하거나, 아니면 `CreditCardPayment` 대신 새로운 결제 클래스의 인스턴스를 직접 생성하도록 **`PaymentProcessor` 자체를 수정**해야 합니다.
- 이는 OCP가 "수정에 대해서는 폐쇄되어야 한다"는 원칙을 명백히 위반합니다. 새로운 기능(새로운 결제 수단)이 추가될 때마다 기존 코드를 수정해야 하는 불편함과 위험이 발생합니다.


#### 2. 추상화는 도입했지만 di가 없는 경우
그렇다면 추상화를 도입하면 OCP를 지킬 수 있을까요? 예를 들어, `IPaymentMethod` 인터페이스를 만들고 `CreditCardPayment`가 이를 구현하게 해봅시다.

```dart
// 결제 방식에 대한 추상화 (인터페이스)
abstract class IPaymentMethod {
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
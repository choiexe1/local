#dart #package 

## Freezed
- 모델 클래스 코드 제너레이트 패키지

### 단순 정의

```dart
part 'instructor.freezed.dart';

@freezed
abstract class Instructor with _$Instructor {
  const factory Instructor({
    required int id,
    required String name,
    required List<Schedule> schedules,
  }) = _Instructor;
```

### 메서드 정의
- 메서드 정의 시 private 기본 생성자가 필요

```dart
@freezed
abstract class Instructor with _$Instructor {
  const Instructor._();

  const factory Instructor({
    required int id,
    required String name,
    required List<Schedule> schedules,
  }) = _Instructor;

  Instructor book(Schedule schedule) {
    return copyWith(schedules: [...schedules, schedule]);
  }

  Instructor cancel(Schedule schedule) {
    return copyWith(schedules: schedules.where((s) => s != schedule).toList());
  }
}

```
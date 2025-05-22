#flutter #mounted 




| 구분    | mounted                     | context.mounted                      |
| ----- | --------------------------- | ------------------------------------ |
| 위치    | StatefulWidget의 State 객체 속성 | 모든 BuildContext에서 사용 가능              |
| 주요 용도 | State가 위젯 트리에 존재하는지 확인      | BuildContext가 가리키는 위젯 트리가 유효한지 확인    |
| 사용 대상 | 주로 상태 관리에 사용                | 비동기 작업 후 UI 작업 시 안전성 체크              |
| 도입 시기 | Flutter 초창기부터 존재            | Flutter 3.7 버전부터 추가                  |
| 예시 코드 | if (mounted) setState(...)  | if (context.mounted) Navigator.pop() |

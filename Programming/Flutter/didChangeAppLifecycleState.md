#flutter #lifecycle

## didChangeAppLifecycleState
이 라이프 사이클은 `WidgetsBindingObserver` 믹스인을 `StatefulWidget`의 `State` 클래스에 추가해야 사용할 수 있다.

`didChangeAppLifecycleState`는 `AppLifecycleState`의 열거형 값을 인자로 입력 받는데, 이 `AppLifecycleState`는 앱의 현재 상태를 나타낸다.


| 값        | 상태                                                                                                    |
| -------- | ----------------------------------------------------------------------------------------------------- |
| resumed  | 앱이 화면에 보이고 사용자 입력을 받을 수 있는 상태 (포그라운드)                                                                 |
| inactive | 앱이 비활성 상태, 예를 들면 전화가 오거나 IOS에서 앱 전환기 (App Switcher)를 열었을 때처럼 포커스를 잃은 상태를 의미. 앱이 화면에는 보이지만 이벤트를 받지는 못함 |
| paused   | 앱이 사용자에게 보이지 않는 상태 (백그라운드), 이 상태에서는 리소스를 많이 소모하는 작업을 중지하는 것이 좋음                                       |
| detached | Flutter 엔진은 아직 실행 중이지만, 어떤 View에도 연결되지 않은 상태, 앱이 처음 시작될 때나 View가 파괴된 후에 발생할 수 있음                      |
| hidden   | 최신 버전에 추가된 상태이며, 앱이 숨겨진 상태로 `paused`와 유사하지만 Flutter 엔진이 UI 렌더링을 중지하지 않을 수 있음<br>                      |

### 예시

```dart
// WidgetsBindingObserver 믹스인을 추가
class _MyLifecycleObserverState extends State<MyLifecycleObserver>
    with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    // Observer를 등록합니다.
    WidgetsBinding.instance.addObserver(this);
    print("initState: Observer registered.");
  }

  @override
  void dispose() {
    // Observer를 제거하여 메모리 누수를 방지
    WidgetsBinding.instance.removeObserver(this);
    print("dispose: Observer removed.");
    super.dispose();
  }

  // 이 메서드가 앱 상태 변경 시 호출됨
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    super.didChangeAppLifecycleState(state);
    switch (state) {
      case AppLifecycleState.resumed:
        print("App is in resumed state.");
        break;
      case AppLifecycleState.inactive:
        print("App is in inactive state.");
        break;
      case AppLifecycleState.paused:
        print("App is in paused state.");
        break;
      case AppLifecycleState.detached:
        print("App is in detached state.");
        break;
      case AppLifecycleState.hidden:
        print("App is in hidden state.");
        break;
    }
  }

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(child: Text('버튼을 눌러 앱을 백그라운드로')),
    );
  }
}
```
#firebase #emulator

비용 문제 없이 `Firebase` 가상 개발 서버를 만들어서 개발 시에 사용할 수 있다.

https://firebase.google.com/docs/emulator-suite?hl=ko


```bash
firebase emulators:start --import=./saved-data --export-on-exit=./saved-data
```

예시 코드가 `Javascript`로 되어 있는데, [pubdev/cloud_firestore](https://pub.dev/packages/cloud_firestore/example)를 살펴보면 `Dart`로 작성된 에뮬레이터 실행 코드가 있다.

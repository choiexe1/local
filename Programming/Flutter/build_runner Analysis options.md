#freezed #analyzer #build_runner #ci/cd

build_runner를 통해서 소스 파일을 생성하고 github action에서 `flutter analysis`를 돌릴 경우 생성된 소스 파일을 레포지토리에 업로드하지 않기 때문에 `error`로 인해 `action`이 통과되지 않는다.

다음과 같이 `analysis_options.yaml`에서 `analyzer`의 옵션을 지정하거나, 파일 내에서 직접 해당 문제를 무시하도록 해서 해결할 수 있다.

```dart
analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
  errors:
    uri_has_not_been_generated: ignore
    uri_does_not_exist: ignore
```
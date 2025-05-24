#flutter 

## 개요
`Flavor`는 하나의 프로젝트에서 각기 다른 환경을 손쉽게 구성할 수 있게 한다.

- [Flutter 공식 문서 - Flavor](https://docs.flutter.dev/deployment/flavors)

## 핵심 개념
- `Flavor`는 `build.gradle.kts` 파일로 지정할 수 있다.
- `flavorDimensions`를 통해 `Flavor`를 그룹화 할 수 있다.


## 예시
- app/src/build.gradle.kts
```java


    flavorDimensions += "environment"

    productFlavors {
        create("dev") {
            dimension = "environment"
            applicationIdSuffix = ".dev"
            resValue("string", "app_name", "mulmuger - dev")
        }
        create("production") {
            dimension = "environment"
            applicationIdSuffix = ".production"
        }
    }
```

## resValue
- app/src/main/res/string.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">물먹어</string>
</resources>

```
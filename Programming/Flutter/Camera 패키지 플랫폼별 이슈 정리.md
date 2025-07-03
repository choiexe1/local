#flutter #camera
## Android
1. Android 6.0(API 23) 이상에서는 앱 실행 중 사용자에게 카메라 접근 권한을 요청해야한다. Camera 플러그인이 이 부분을 내부적으로 처리하지만 사용자가 권한을 거부하면 CameraException이 발생한다.

## IOS
1. IOS Simulator는 카메라 기능을 지원하지 않는다. 따라서 실 기기로 테스트 해야한다.
2. IOS에서는 앱 실행 중 사용자에게 카메라 접근 권한을 요청해야한다. Camera 플러그인이 이 부분을 내부적으로 처리하지만, 사용자가 권한을 거부하면 CameraException이 발생한다.
#flutter 

## AndroidManifest.xml
`app/src/main/AndroidManifest.xml`에서의 설정 부분
### Permission

```
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<uses-permission android:name="android.permission.VIBRATE"/>
<uses-permission android:name="android.permission.ACCESS_NOTIFICATION_POLICY" />
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
<uses-permission android:name="android.permission.USE_EXACT_ALARM" />
```
- `POST_NOTIFICATIONS`: Android 13 이상 앱에서 알림을 표시하도록 허용
- `RECEIVE_BOOT_COMPLETED`: 기기가 재시작된 후 앱이 코드를 실행하도록 허용
- `VIBRATE`: 앱이 기기를 진동하도록 설정
- `ACCESS_NOTIFICATION_POLICY`: 앱이 방해 금지 설정을 읽거나 변경하도록 설정
- `SCHEDULE_EXACT_ALARM`: 앱이 정확한 시간(exact timing)에 알림 또는 작업을 예약할 수 있도록 허용, 이 권한은 앱이 사용자에게 직접 알람 권한을 요청할 때 필요함
- `USE_EXACT_ALARM`: 앱이 정확한 시간(exact timing)에 알림 또는 작업을 예약할 수 있도록 허용합니다. 이 권한은 주로 앱이 Android 13 (API 레벨 33) 이상을 타겟팅할 때 사용, 사용자에게 별도의 권한 요청 없이 작동할 수 있지만, 앱 스토어 제출 시 추가적인 검토 및 승인이 필요할 수 있음

### Receiver

```
<receiver  
android:exported="false"  
android:name="com.dexterous.flutterlocalnotifications.ScheduledNotificationReceiver" />
<receiver  
android:exported="false"  
android:name="com.dexterous.flutterlocalnotifications.ScheduledNotificationBootReceiver">  
<intent-filter>  
<action android:name="android.intent.action.BOOT_COMPLETED"/>  
<action android:name="android.intent.action.MY_PACKAGE_REPLACED"/>  
<action android:name="android.intent.action.QUICKBOOT_POWERON" />  
<action android:name="com.htc.intent.action.QUICKBOOT_POWERON"/>  
</intent-filter>  
</receiver>
<receiver  
android:exported="false"  
android:name="com.dexterous.flutterlocalnotifications.FlutterLocalNotificationsReceiver"/>
```
- `ScheduledNotificationReceiver`: 예약된 알림의 트리거를 처리
- `ScheduledNotificationBootReceiver`: 재부팅/앱 재설치/업데이트 및 특정 제조업체별 빠른 부팅 후 알림 일정 재조정`
- `FlutterLocalNotificationsReceiver`: 알림 즉시 표시 기능


## build.gradle.kts
`android/app/build.gradle.kts`에서의 설정 부분

```kotlin
android {
    compileOptions {
        isCoreLibraryDesugaringEnabled = true
    }
}

dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.1.4")
}
```

## LocalNotifier

`flutter_local_notifications`를 추상화 한 클래스

```dart
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:timezone/timezone.dart' as tz;
import 'package:timezone/data/latest.dart' as tz;
import 'package:timezone/timezone.dart' show TZDateTime;

class LocalNotifier {
  static const String notificationIcon = '@mipmap/ic_launcher';
  final notificationPlugin = FlutterLocalNotificationsPlugin();

  bool _isInitialized = false;

  bool get isInitialized => _isInitialized;

  Future<void> initNotification() async {
    if (_isInitialized) return;

    tz.initializeTimeZones();

    tz.setLocalLocation(tz.getLocation('Asia/Seoul'));

    const androidSettings = AndroidInitializationSettings(notificationIcon);

    const initializeSetting = InitializationSettings(android: androidSettings);

    await notificationPlugin.initialize(initializeSetting);
    _isInitialized = true;
  }

  NotificationDetails notificationDetails() {
    return const NotificationDetails(
      android: AndroidNotificationDetails(
        'daily_channel_id',
        'Daily Notifications',
        channelDescription: 'Daily Notification Channel',
        importance: Importance.max,
        priority: Priority.high,
      ),
    );
  }

  Future<void> showNotification({
    int id = 0,
    String? title,
    String? body,
  }) async {
    return notificationPlugin.show(id, title, body, notificationDetails());
  }

  Future<void> scheduleNotification({
    required String title,
    required String body,
    required int hour,
    required int minute,
    int id = 1,
  }) async {
    final now = tz.TZDateTime.now(tz.local);

    final scheduledDate = tz.TZDateTime(
      tz.local,
      now.year,
      now.month,
      now.day,
      hour,
      minute,
    );

    await notificationPlugin.zonedSchedule(
      id,
      title,
      body,
      scheduledDate,
      const NotificationDetails(),
      androidScheduleMode: AndroidScheduleMode.inexactAllowWhileIdle,
      // 반복 수행
      // matchDateTimeComponents: DateTimeComponents.time,
    );
  }

  Future<void> cancelAllNotifications() async {
    await notificationPlugin.cancelAll();
  }
}

```

## main

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final localNotifier = LocalNotifier();
  await localNotifier.initNotification();

  final appConfig = AppConfig.initialize(appName: '물머거', flavor: Flavor.dev);

  runApp(App(appConfig: appConfig));
}

```

## 일회성
```dart
ElevatedButton(
	onPressed: () {
		LocalNotifier().showNotification(
			title: '물 먹을 시간',
			body: '물 먹드세요',
		);
	},
	child: const Text('Send Noti'),
),
```
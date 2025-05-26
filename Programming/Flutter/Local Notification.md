#flutter 

## Permission
`app/src/main/AndroidManifest.xml`

```
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <uses-permission android:name="android.permission.VIBRATE"/>
    <uses-permission android:name="android.permission.ACCESS_NOTIFICATION_POLICY" />
```
- `POST_NOTIFICATIONS`: Android 13 이상 앱에서 알림을 표시하도록 허용
- `RECEIVE_BOOT_COMPLETED`: 기기가 재시작된 후 앱이 코드를 실행하도록 허용
- `VIBRATE`: 앱이 기기를 진동하도록 설정
- `ACCESS_NOTIFICATION_POLICY`: 앱이 방해 금지 설정을 읽거나 변경하도록 설정

## Receiver

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
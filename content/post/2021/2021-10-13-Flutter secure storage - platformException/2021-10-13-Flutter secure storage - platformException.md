---
title: Flutter secure storage - platformException
description: 
date: 2021-10-13
image: 
categories:
- Flutter
tags:
weight: 1
---

FlutterSecure Storage에서 read를 할때 platformEception이 발생했다. 테스트 기기는 LG폰이였는데, 구글링 해보니까 삼성에서도 간혹 있는 것 같다. 해결 방법은 2가지이다.

1. AnroidMnifest.xml
    
    ```xml
    <application
    ...
    android:allowBackup="false"
    android:fullBackupContent="false">
    ```
2. `.deleteAll()` 호출하기
    
    저장되어 있는걸 불러는 것에 문제가 있는 것 같으니까, 예외처리로 `.deleteAll()`을 실행하고 다시 read하면 되지 않을까 싶다.
    
**참고**
> 
> [flutter_secure_storage | Flutter Package](https://pub.dev/packages/flutter_secure_storage)
> [FlutterSecureStorage.read broken on Android 9.0 Samsung only · Issue #43 · mogol/flutter_secure_storage](https://github.com/mogol/flutter_secure_storage/issues/43)
>
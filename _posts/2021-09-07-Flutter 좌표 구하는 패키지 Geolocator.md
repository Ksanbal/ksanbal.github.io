---
layout: post
---
## What?

[geolocator | Flutter Package](https://pub.dev/packages/geolocator)
- 사용자의 위치를 가져오는 패키지

## Use

- 위치서비스 사용가능 여부
  - `bool serviceEnabled = await Geolocator.isLocationServiceEnabled();`
- 위치권한
  - 권한 체크
    `LocationPermission permission = await Geolocator.checkPermission();`
  - 권한 요청
    `await Geolocator.requestPermission();`
- 좌표 구하기
    `var currentPosition = await Geolocator.getCurrentPosition( desiredAccuracy: LocationAccuracy.lowest);`

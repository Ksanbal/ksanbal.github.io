---
title: Flutter 상태관리와 라우트관리 패키지 GetX
description: 
date: 2021-09-07
image: 
categories:
- Flutter
tags:
weight: 1
---

## What?

---

[get | Flutter Package](https://pub.dev/packages/get) 
- `MaterialApp → GetMaterialApp`

## Navigation

```dart
// Setting
return GetMaterialApp(
	title: "GetX!!",
	home: HomeView(),
	getPages: [
		// page 등록
		GetPage(name: "/first", page: () => FirstView()),
		// page에 웹처럼 param을 받을 수 있음
		GetPage(name: "/second/:param", page: () => SecondView()),
		// 전화효과를 지정
		GetPage(
			name: "/third/", 
			transition: Transition.cupertino,
			page: () => ThirdView(),
		),
	],
```

```dart
// Using
Get.to(nextPage());
Get.toNamed('/next');

// Param으로 값을 보냈을 때
// Send
Get.toNamed('/next/test?name=Hello&message=Getx');
// Take
Get.parameters['param'];
Get.parameters['name'];
Get.parameters['message'];

// 이전 화면을 없애면서 이동
Get.off(nextPage());
// 이전의 모든 화면을 없애고 이동
Get.offAll(nextPage());

// 뒤로가기, 뒤로갈 페이지가 없으면 오류없이 동작x
Get.back();
```

## SnackBar

```dart
// Type 1 - GetX style
Get.snackbar(
	"Title",
  "Subtitle?",
  snackPosition: SnackPosition.TOP
);

// Type 2 - Flutter style
Get.showSnackbar(
  GetBar(
    title: "SnackBar type2",
    message: "This is SnackBar message!",
    duration: Duration(seconds: 3),
    snackPosition: SnackPosition.BOTTOM,
  ),
);
```

## Dialog

```dart
// Type 1 - GetX style
Get.defaultDialog(
  title: "Hello Dialog",
  middleText: "How\ncan\ni\nwrite\ntext\nin\nhere",
  textConfirm: "Confirm",
  onConfirm: () {
    print("confirm");
  },
  confirm: TextButton(
    child: Text("yes"),
    onPressed: () {
      Get.back();
    },
  ),
);

// Type 2 - Flutter style
Get.dialog(
  Dialog(
    child: Container(
      height: 100,
      child: Center(
        child: Text("hi"),
      ),
    ),
  ),
);
```

## BottomSheet

```dart
Get.bottomSheet(
  Container(
    height: 500,
    color: Colors.indigo,
  ),
);
```
---
title: Flutter 문법 Tip
description: 
date: 2021-09-29
image: 
categories:
    - Flutter
tags:
weight: 1
---

`...` : List의 요소를 하나씩 꺼내오기

```dart
renderKeyboard() {
	return (...).toList();
...
children: [...renderKeyboard()],
```

- 이런 문법을 사용하면 ListView에 Header나 마지막에 뭔가 Widget을 추가하고 싶을때, .builder를 사용하지 않고 할 수 있다.
  ```dart
  ListView(
	...
	children: [
		HeaderWidget(),
		...renderKeyboard(),
		LastWidget(),
	],
)
  ```
---
title: Flutter Drawer Programmatic하게 다루기
description: 
date: 2021-09-30
image: 
categories:
    - Flutter
tags:
weight: 1
---

Flutter에는 Drawer라는 사이드 메뉴바가 존재한다. 기본적으로 Scaffold에 추가해주면 자동으로 AppBar에 메뉴 아이콘이 생성되면서 이용할 수 있다.

그런데 아이콘을 바꾸려고 하거나, 다른 버튼을 통해서 Drawer를 열려면? `.openDrawer()`를 사용할 수 있다!

일단 Drawer를 등록해놓고

```dart
return Scaffold(
    appBar: AppBar(...), // AppBar가 없으면 Drawer 버튼이 생성되지 않는다!
    drawer: Drawer(...), // 왼쪽에서 나타나는 Drawer, 버튼도 왼쪽에 생성
    endDrawer: Drawer(...), // 오른쪽에서 나타나는 Drawer, 버튼이 오른쪽에 생성
)
```

이렇게 작성하면 좌우로 아이콘 버튼이 생긴 것을 볼 수 있다.

이제 AppBar에서 메뉴 아이콘을 다른걸로 바꾸고 `Scaffold.of(context).openEndDrawer()`으로 프로그램매틱하게 Drawer를 열어보자!

그런데 이걸 그냥 `() => Scaffold.of(context).openEndDrawer()` 이렇게 해서 실행되면 참 좋을텐데...아래와 같은 오류를 낸다.

```bash
════════ Exception caught by gesture ═══════════════════════════════════════════
Scaffold.of() called with a context that does not contain a Scaffold.
════════════════════════════════════════════════════════════════════════════════
```

그래서 이걸 사용할때는 아래와 같은 방식으로 Widget을 Builder로 한번 씌워서 작성해야한다.

```dart
Builder(
    builder: (context) => IconButton()
)
```

결론

```dart
return Scaffold(
    appBar: AppBar(
        actions : [
            Builder(
                builder: (context) => IconButton(
                    icon: Icon(Icons.add),
                    onPressed: Scaffold.of(context).openDrawer, // drawer 열기
                )
            ),
            Builder(
                builder: (context) => IconButton(
                    icon: Icon(Icons.plus_one),
                    onPressed: Scaffold.of(context).openEndDrawer, // endDrawer 열기
                )
            ),						
        ]
    ),
    drawer: Drawer(...),
    endDrawer: Drawer(...),
)
```

참고로 drawer를 끌때는 `Navigator.pop(context)`으로 끌 수 있다. GetX를 사용한다면 `Get.back()`를 이용해도 괜찮다!
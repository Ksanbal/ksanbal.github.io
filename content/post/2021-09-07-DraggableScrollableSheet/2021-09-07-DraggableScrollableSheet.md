---
title: DraggableScrollableSheet
description: 
date: 2021-09-07
image: 
categories:
- Flutter
tags:
weight: 1
---

```dart
Stack(
  children: [
    Container(color: Colors.lightBlue),
    DraggableScrollableSheet(
      initialChildSize: 0.2,
      minChildSize: 0.13,
      maxChildSize: 1.0,
      builder: (context, scrollController) {
        return Padding(
          padding: EdgeInsets.all(0),
          child: Container(
            decoration: BoxDecoration(
              color: Colors.white,
              borderRadius: BorderRadius.only(
                topLeft: Radius.circular(15),
                topRight: Radius.circular(15),
              ),
            ),
            child: ListView.builder(
              itemCount: 20,
              controller: scrollController,
              itemBuilder: (context, index) {
                return ListTile(title: Text("Test item $index"));
              },
            ),
          ),
        );
      },
    ),
  ],
),
```
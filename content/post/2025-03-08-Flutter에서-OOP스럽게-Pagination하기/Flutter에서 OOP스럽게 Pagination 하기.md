---
title: Flutter에서 OOP스럽게 Pagination 하기
description:
date: 2025-03-08 00:00:00+0000
categories:
tags:
weight: 1
---

Client에서 일반적으로 가장 많이 구현하는 기능이 뭘까? 난 단연 리스트 페이지라고 생각한다. 쿠팡이라면 수많은 상품을 리스팅해야하고, 인스타그램이라면 쏟아지는 피드를 보여줘야한다. 그리고 수많은 데이터를 빠르게 스크롤하면서 탐색하고 끊김없이 불러오는게 중요하다. 리스트를 불러올때는 다양한 상황이 있다. 처음으로 불러올때, 추가로 불러올때, 새로고침, 에러 등 다양한 상황에 대처해야한다. 그리고 이 페이지네이션 기능은 앱 전체에서 이뤄지기 때문에 코드의 재활용성이 중요하다.

이만큼 자주 구현하는 기능인데 그동안은 모든 페이지의 ViewContrller에 반복적으로 구현으로 해왔는데, OOP를 이용한 일반화를 통해 재활용할 수 있는 방법을 알게되어서 소개한다.

## 구현시나리오

배달앱에서 가게 목록을 불러오는 페이지를 만든다고 해보자. 한번에 20개의 가게를 불러올거고, 끝에 다다르면 다음 20개의 가게를 불러올거다. 그리고 맨 위에서는 아래로 당겨 새로고침을 구현한다고 해보자.

## Pagination

DB의 수많은 데이터를 한번에 불러와서 보여주는건 DB에게도, API에게도, 클라이언트에게도 부담이다. 인스타의 초단위로 쏟아지는 피드를 어떻게 다 불러와서 보여주겠나. 그래서 개수로 나눠서 조금씩 불러와서 보여주면서 사용자는 마치 끊임없이 보고있다고 느끼게 해야한다.
Pagination을 구현하는 방법은 2가지로, Page based와 cursor based가 있다. Page based는 전통적인 방식으로

### Page based

전통적인 방식으로 데이터 개수로 페이지를 나누고 페이지를 옮겨다니면서 조회하는 방식이다. 책에서 페이지를 넘기면서 보는 것에서 유래하지 않았을까 싶다. 중간에 페이지를 건너뛸수도 있어서, 빠르게 탐색이 가능하다. 하지만 조회 중간에 데이터 생성이나 삭제가 일어나면 누락되거나 한번 더 보여주는 일이 발생할 수 있다.

### Cursor based

모바일 서비스가 많아지면서 스크롤 마지막에 다다르면 다음 데이터를 불러오는 방식이다. 페이지 베이스처럼 중간에 건너뛸 수는 없지만 이어져서 데이터를 보면서 마치 모든 데이터를 보고있다고 느낄 수 있다. 클라이언트에서 마지막으로 본 데이터 다음부터 불러오기 때문에 중간에 생성이나 삭제가 이뤄져도 반영된 결과를 볼 수 있다. 더 끊김없는 로딩을 위해 스크롤이 끝까지 가기전에 미리 불러오는 등으로 사용자 경험을 향상시킬 수 있다.

## OOP란 무엇일까?

OOP는 객체지향 프로그래밍을 의미한다. 갑자기 페이지네이션 얘기하다 왠 OOP인가 싶겠지만, 페이지네이션에서는 상태에 따른 UI 변경이 필요하다. 상태에 대한 처리와 재활용성을 높히기 위해 필요하다.

OOP에서 가장 재밌는건 상속이 가능하다는 점이다. 그리고 자식 Class를 다시 상속할 수 있다. 이게 필요한 이유는 최종 부모 Class (조상 Class...?)가 같기 떄문에 파생된 다양한 Class들의 뿌리가 같은지를 `is` 명령어를 통해 검사할 수 있다.

```dart
abstract class KimFamily {}

class Fater extends KimFamily {}
class Son extends Fater {}
class Daughter extends Fater {}

final fater = Fater();
final son = Son();
final daughter = Daughter();

fater is KimFamily // true
son is KimFamily // true
son is Fater // true
daughter is KimFamily // true
daughter is Fater // true
```

## Generic을 통한 일반화

제네릭은 클래스 내부에서 사용할 데이터의 타입을 외부에서 지정하는 기법이다. Class를 정의할때 내부에 객체의 타입이 String일지 int일지 상황에 따라 다를 수 있다. 상품 페이지에서는 상품 목록을 불러와야하고, 가게 페이지에서는 가게 목록을 불러와야한다. 이렇게 세부 타입이 달라도 Class를 재활용하기 위해서 사용할것이다.

```dart
class MyList<T> {
  final int page;
  final T data;
}

class Product {}
class Item {}

final product = MyList<Product>();
final item = MyList<Item>();
```

## Pagination 적용하기

실제 구현에 앞서 API는 어떻게 응답을 해줄까? Cursor Pagination을 기준으로 작성해봤다.

```json
{
  "meta": {
    "count": 20,
    "hasMore": true,
  },
  "data": [
    {
      "id": 532,
      "name": "김밥천국",
    },
    {
      "id": 531,
      "name": "맥도날드",
    },
    ...
  ]
}
```

자 이제 실제 구현을 해보자. 리스트 페이지에서 처리할 상황은 크게 5가지이다.

> 1. 아무것도 없다가 불러오는 상황 (로딩 상황)
> 2. 데이터를 불러온 일반 상황
> 3. 기존에 데이터가 있는데 더 불러오는 상황
> 4. 새로고침을 하려는 상황
> 5. 에러가 발생한 상황

이 모든 상황들이 결국엔 리스트 페이지를 구성하기 위해서 필요한 요소들이다. 그 이야기는 모두 하나의 Class로 연결지어서 사용할 수 있다는 뜻이다. 위의 상황에 맞는 Class를 하나씩 만들어 나갈 것인데, 먼저 각 상황에 따른 Class를 연결해주기 위한 최상위 추상화 Class를 생성할 것이다. 그리고 페이지네이션을 위한 Meta class 까지만 만들어보자.

```dart
abstract class PaginationBase {}

class PaginationMeta {
  final int count;
  final bool hasMore;
}
```

### 1. 로딩상황

데이터를 불러오는 중에는 안에 데이터도 없고 행동도 없다. 그대로 상속해서 만들어주자.

```dart
class PaginationLoading extends PaginationBase {}
```

### 2. 데이터를 불러온 일반 상황

View에서 데이터를 보여주는 가장 일반적인 상황이다. 데이터를 불러왔다면 데이터와 meta를 가지고 있는 class.

```dart
class Pagination<T> extends PaginationBase {
  final PaginationMeta meta;
  final List<T> data;
}
```

### 3. 기존에 데이터가 있는데 더 불러오는 상황

첫 로딩 후 데이터를 가지고 있는데, 더보기로 데이터를 불러오는 상황이다. 로딩중인건 맞지만, 기존 데이터를 가지고 덧붙일 예정이니까 일반 상황을 상속하자.

```dart
class PaginationFetchingMore<T> extends Pagination<T> {}
```

### 4. 새로고침을 하려는 상황

기존 데이터가 있는데 아예 새로고침을 실행할때 상황이다. 새로고침을 하는 동안 로딩창을 보여줘도 좋지만, 기존 데이터가 있으면 같이 보여주다 바꿔치는게 좋으니까 마찬가지로 같은 데이터를 가지도록 해보자.

```dart
class PaginationReFetching<T> extends Pagination<T> {}
```

### 5. 에러가 발생한 상황

위의 1~4 상황에서 에러가 발생한 상황이다. 기존 데이터를 보관할 필요는 없고 에러 메시지만 가지고 있도록 해보자.

```dart
class PaginationError extends PaginationBase {
  final String message;
}
```

## 데이터를 불러오는 공통 함수 작성하기

위에서 정의한 Class를 이용해서

```dart
class PaginationProvider<T extends IModelWithId, U extends IBasePaginationRepository<T>>
    extends StateNotifier<CursorPaginationBase> {

  // repository를 제네릭을 받아와서 모든 repository에서 공통적으로 사용할 수 있게 처리
  final U repository;

  PaginationProvider({
    required this.repository,
  }) : super(CursorPaginationLoading()) {
    // 이 Class가 생성될때 데이터를 불러오도록 메소드 실행
    paginate();
  }

  Future<void> paginate({
    int fetchCount = 20, // 한번에 불러올 데이터의 개수
    // 추가로 데이터 더 가져오기
    // true - 추가로 데이터 더 가져옴
    // false - 새로고침 (현재 상태를 덮어씌움)
    bool fetchMore = false,
    // 강제로 새로고침
    bool forceRefetch = false,
  }) async {
    try {
      // 이미 데이터가 있고, 강제 새로고침이 아닌 경우
      if (state is CursorPagination<T> && forceRefetch == false) {
        final pState = state as CursorPagination<T>;

        // 추가적인 데이터가 없는 경우 중지
        if (pState.meta.hasMore == false) {
          return;
        }
      }

      final isLoading = state is CursorPaginationLoading;
      final isRefetching = state is CursorPaginationRefetching<T>;
      final isFetchingMore = state is CursorPaginationFetchingMore<T>;

      // 더 불러오라고 했는데 지금 상황이 로딩, 새로고침, 더보기 상태면 중지
      if (fetchMore && (isLoading || isRefetching || isFetchingMore)) {
        return;
      }

      // PaginationParams 생성
      PaginationParams paginationParams = PaginationParams(
        count: fetchCount,
      );

      // fetchMore - 데이터를 추가로 더 가져오는 상황
      if (fetchMore) {
        final pState = state as CursorPagination<T>;

        // 다음꺼를 불러오는 상태로 변경
        state = CursorPaginationFetchingMore(
          meta: pState.meta,
          data: pState.data,
        );

        paginationParams = paginationParams.copyWith(
          after: pState.data.last.id,
        );
      } else {
        // 데이터를 처음부터 가져오는 상황
        // 만약에 데이터가 있는 상황이라면 기존 데이터를 가지고 Fetch 진행
        if (state is CursorPagination<T> && forceRefetch == false) {
          final pState = state as CursorPagination<T>;

          state = CursorPaginationRefetching<T>(
            meta: pState.meta,
            data: pState.data,
          );
        } else {
          state = CursorPaginationLoading();
        }
      }

      final res = await repository.paginate(
        paginationParams: paginationParams,
      );

      if (state is CursorPaginationFetchingMore<T>) {
        final pState = state as CursorPaginationFetchingMore<T>;

        // 기존 데이터 + 새로운 데이터
        state = res.copyWith(
          data: [
            ...pState.data,
            ...res.data,
          ],
        );
      } else {
        state = res;
      }
    } catch (e, stack) {
      print(e);
      print(stack);
      state = CursorPaginationError(message: '데이터를 가져오지 못했습니다');
    }
  }
}
```

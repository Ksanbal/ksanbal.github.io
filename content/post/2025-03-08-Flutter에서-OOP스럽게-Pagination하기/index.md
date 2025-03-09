---
title: Flutter에서 OOP스럽게 Pagination 하기
description:
image: Gemini_Generated_Image_l8u653l8u653l8u6.jpg
date: 2025-03-09 00:00:00+0000
categories:
  - Flutter
tags:
  - Flutter
  - OOP
  - Generic
  - Pagination
weight: 1
---

Client에서 일반적으로 가장 많이 구현하는 기능이 뭘까? 난 단연 리스트 페이지라고 생각한다.

쿠팡이라면 수많은 상품을 리스팅 해야하고, 인스타그램이라면 쏟아지는 피드를 보여줘야 한다. 그리고 수많은 데이터를 빠르게 스크롤하면서 탐색하고 끊임없이 불러오는 게 중요하다.

리스트를 불러올 때는 다양한 상황이 있다. 처음으로 불러올 때, 추가로 불러올 때, 새로고침, 에러 등 다양한 상황에 따라 UI를 업데이트해야 한다. 그리고 이 페이지네이션 기능은 앱 전체에서 이뤄지기 때문에 코드의 **재활용성**이 중요하다.

이만큼 자주 구현하는 기능인데 그동안은 모든 페이지의 ViewContrller에 반복적으로 구현으로 해왔는데, **OOP와 Generic을 이용한 일반화를 통해 재활용**할 수 있는 방법을 알게 되어서 소개한다.

## 구현시나리오

배달앱에서 가게 목록을 불러오는 페이지를 만든다고 해보자. 한 번에 20개씩의 가게를 불러올 거고, 끝에 다다르면 다음 20개의 가게를 불러올 거다. 그리고 맨 위에서는 아래로 당겨 새로고침을 구현한다고 해보자.

## Pagination

DB의 수많은 데이터를 한 번에 불러와서 보여주는 건 DB에게도, API에게도, 클라이언트에게도 부담이다.

인스타의 초 단위로 쏟아지는 피드를 어떻게 다 불러와서 보여주겠나. 그래서 개수로 나눠서 조금씩 불러와서 보여주면서 사용자는 마치 끊임없이 보고 있다고 느끼게 해야 한다.

Pagination을 구현하는 방법은 2가지로, Page based와 Cursor based가 있다.

### Page based

전통적인 방식으로 **데이터 개수로 페이지를 나누고 페이지를 옮겨 다니면서 조회하는 방식**이다. 책에서 페이지를 넘기면서 보는 것에서 유래하지 않았을까 싶다.

구현도 간단하고 중간에 페이지를 건너뛸 수도 있어서, 빠르게 탐색이 가능하다. **하지만 조회 중간에 데이터 생성이나 삭제가 일어나면 누락되거나 한번더 보여주는 일이 발생할 수 있다.**

### Cursor based

모바일 서비스가 많아지면서 스크롤 마지막에 다다르면 다음 데이터를 불러오는 방식이다. Page based처럼 중간에 건너뛸 수는 없지만 이어져서 데이터를 보면서 사용자는 마치 모든 데이터를 보고 있다고 느낄 수 있다.

클라이언트에서 마지막으로 본 데이터 다음부터 불러오기 때문에 **중간에 생성이나 삭제가 이뤄져도 반영된 결과를 볼 수 있는 장점이 있다.** 더 끊김 없는 로딩을 위해 스크롤이 끝까지 가기 전에 미리 불러오는 등으로 사용자 경험을 향상 시킬 수 있다.

## 구현 컨셉

Flutter 앱에서 구현할 예정이니 **Cursor based**로 구현 해보자.
API의 Response는 아래와 같다.

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

### 이전의 구현 방식

Generic 정도는 사용했으니까 PaginationClass 같은 Class를 만들어서 공통으로 사용은 했다.

하지만 로딩 상태, 더 불러오기 등의 기능을 ViewController의 변수와 함수로 작성해서 사용했고, View에서는 각 상황에 따라 함수를 호출했다.

이렇게 되면 모든 리스트 페이지의 ViewController에서 로딩 상태, 초기 불러오기 함수, 더 불러오기 함수, 새로고침 함수를 반복적으로 작성해야 한다. View에서도 로딩 여부만 가지고 UI를 그리면서 화면 전체에 로딩 창을 보여주는 정도로만 구현이 가능했다.

### OOP와 Generic으로 개선

API 응답에는 `meta`에 불러온 개수인 `count`와 이 이상 데이터가 있는지 여부인 `hasMore`가 있다. 모든 리스트 페이지에서 저렇게 리스팅에 대한 정보가 **반복적**으로 담겨있을 거다. **Class로 일반화**해서 반복적으로 사용하게 할 수 있을 것 같다.

`data` 부분에 실제로 우리가 보여줄 정보가 있다. 이번엔 가게 정보를 담았지만, 다른 페이지에서 상품 목록, 평점 목록 등 다양한 정보가 있을 거다. 각 페이지에서 data 부분에 type을 유동적으로 변경해 주기 위해 **Generic**을 사용할 거다.

리스트 페이지에는 로딩 중, 로딩 완료, 에러, 더 보기 등 데이터를 불러오는 상태가 존재한다. 이 상태를 타입을 비교하는 `is` 명령어를 통해 확인할 수 있게 하나의 공통 class를 상속 할거다.

마지막으로, 이 상태에 따른 페이지네이션을 하나의 함수로 일반화해서 사용 해보자.

> 참고
>
> 코드 전체가 상태관리툴인 Riverpod과 Repository 패턴으로 구현되어있어 익숙치 않은 부분이 있을 수 있지만, OOP를 이용하는 컨셉과 과정을 참고해주길 바란다.

<details>
<summary> Generic을 통한 일반화</summary>

Generic은 클래스 내부에서 사용할 데이터의 타입을 외부에서 지정하는 기법이다.

Class를 정의할 때 객체의 타입이 String일지 int일지 상황에 따라 다를 수 있다. 상품 페이지에서는 상품 목록을 불러와야 하고, 가게 페이지에서는 가게 목록을 불러올 수 있게 각

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

</details>

## Pagination 적용하기

자 이제 실제 구현을 해보자. 리스트 페이지에서 처리할 상황은 크게 5가지이다.

1. 아무것도 없다가 불러오는 상황 (초기 로딩 상황)
2. 데이터가 있는 상황
3. 기존에 데이터가 있는데 더 불러오는 상황
4. 새로고침을 하려는 상황
5. 에러가 발생한 상황

먼저 모든 상황을 하나의 Class를 상속해서 사용할 수 있도록 추상 Class로 `Base Class`를 생성 해주고, 같이 사용되는 `Meta Class`를 생성했다.

```dart
abstract class PaginationBase {}

class PaginationMeta {
  final int count;
  final bool hasMore;
}
```

### 1. 아무것도 없다가 불러오는 상황 (초기 로딩 상황)

데이터를 불러오는 중에는 안에 데이터도 없고 행동도 없다. 그대로 상속해서 만들어주자.

```dart
class PaginationLoading extends PaginationBase {}
```

### 2. 데이터가 있는 상황

View에서 데이터를 보여주는 가장 일반적인 상황이다.

meta 정보를 객체로 가지고 데이터의 타입을 Generic으로 받아서 다양한 Class를 사용할 수 있게 하자.

```dart
class Pagination<T> extends PaginationBase {
  final PaginationMeta meta;
  final List<T> data;
}
```

### 3. 기존에 데이터가 있는데 더 불러오는 상황

첫 로딩 후 데이터를 가지고 있는데, 더보기로 데이터를 불러오는 상황이다.

로딩 중인건 맞지만, 기존 데이터를 가지고 덧붙일 예정이니까 `Pagination<T>`을 상속 해주자.

```dart
class PaginationFetchingMore<T> extends Pagination<T> {}
```

### 4. 새로고침을 하려는 상황

기존 데이터가 있는데 아예 새로고침을 실행할 때 상황이다.

새로고침을 하는 동안 로딩 창을 보여줘도 좋지만, 기존 데이터가 있으면 같이 보여주다 바꿔치는 게 좋으니까, 마찬가지로 `Pagination<T>`을 상속 해주자.

```dart
class PaginationReFetching<T> extends Pagination<T> {}
```

### 5. 에러가 발생한 상황

위의 1~4 상황에서 에러가 발생한 상황이다.

기존 데이터를 보관할 필요는 없고 에러 메시지만 가지고 있도록 해보자.

에러가 발생해도 데이터를 보여주고 싶다면 `Pagination<T>`를 상속하고 `message`를 추가해서 사용할 수도 있겠다.

```dart
class PaginationError extends PaginationBase {
  final String message;
}
```

### 데이터를 불러오는 공통 함수 작성하기

위에서 정의한 Class를 이용해서 현재 상태를 확인하고 데이터를 로딩하는 함수를 작성 해보자.

이미 데이터를 불러오는 중인데, 중복으로 API 호출이 발생하지 않도록 상태 확인을 통해 함수를 중단시키는 로직을 추가 할거다.

```dart
class PaginationProvider<T extends IModelWithId, U extends IBasePaginationRepository<T>>
    extends StateNotifier<CursorPaginationBase> {

  // repository를 제네릭을 받아와서 모든 repository에서 공통으로 사용할 수 있게 처리
  final U repository;

  PaginationProvider({
    required this.repository,
  }) : super(CursorPaginationLoading()) {
    // 이 Class가 생성될 때 데이터를 불러오도록 메소드 실행
    paginate();
  }

  Future<void> paginate({
    int fetchCount = 20, // 한 번에 불러올 데이터의 개수
    // 추가로 데이터 더 가져오기
    // - true : 추가로 데이터 더 가져옴
    // - false : 새로고침 (현재 상태를 덮어씌움)
    bool fetchMore = false,
    // 강제로 새로고침 여부
    bool forceRefetch = false,
  }) async {
    try {
      // state는 현재 요청하고자 하는 데이터의 인스턴스다. 가게 목록 class의 인스턴스라고 생각 해주길 바란다.

      // 이미 데이터가 있고, 강제 새로고침이 아닌 경우
      if (state is CursorPagination<T> && forceRefetch == false) {
        final pState = state as CursorPagination<T>;

        // 추가적인 데이터가 없는 경우 중지
        if (pState.meta.hasMore == false) {
          return;
        }
      }

      // 로딩 상태 여부
      final isLoading = state is CursorPaginationLoading;
      // 새로고침 여부
      final isRefetching = state is CursorPaginationRefetching<T>;
      // 추가 요청 여부
      final isFetchingMore = state is CursorPaginationFetchingMore<T>;

      // 더 불러오라고 했는데 지금 상황이 로딩, 새로고침, 더 보기 상태면 중지
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

        // 현재 상태를 다음 거를 불러오는 상태로 변경
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

          // 현재 상태를 새로고침 상태로 변경
          state = CursorPaginationRefetching<T>(
            meta: pState.meta,
            data: pState.data,
          );
        } else {
          // 초기 로딩 상태로 변경
          state = CursorPaginationLoading();
        }
      }

      // API 호출을 통해 데이터를 받아온다.
      final res = await repository.paginate(
        paginationParams: paginationParams,
      );

      // 데이터를 추가로 불러오는 상태라면 기존 데이터 + 새로운 데이터로 처리한다.
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
        // 초기 호출이거나 새로고침이니까 응답받은 데이터로 덮어씌운다.
        state = res;
      }
    } catch (e, stack) {
      // 에러가 발생한 상황으로 변경
      // message는 API에서 반환해줄 수도 있지만 일단 이렇게 처리 해보자.
      state = CursorPaginationError(message: '데이터를 가져오지 못했습니다');
    }
  }
}
```

### View에서 상태에 따른 UI 구현

이제 위 함수를 View에서 호출해서 사용 해보자.

로딩상황일 때는 로딩 창을, 에러일 때는 에러 창을 보여주고 데이터를 더 불러올 때는 마지막에 로딩 UI를 표시 해주자.

```dart
class RestaurantScreen extends ConsumerStatefulWidget {
  const RestaurantScreen({super.key});

  @override
  ConsumerState<RestaurantScreen> createState() => _RestaurantScreenState();
}

class _RestaurantScreenState extends ConsumerState<RestaurantScreen> {
  // 스크롤 상태를 확인하기 위한 Controller
  final ScrollController controller = ScrollController();

  @override
  void initState() {
    super.initState();

    // 더 불러오기 구현을 위해 Controller의 Listener를 등록 해준다.
    controller.addListener(scrollListener);
  }

  void scrollListener() {
    // 현재 위치가 최대 길이보다 조금 덜 되는 위치까지 왔다면 새로운 데이터 추가 요청
    if (controller.offset > controller.position.maxScrollExtent - 300) {
      ref.read(restaurantProvider.notifier).paginate(
        fetchMore: true,
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    final data = ref.watch(restaurantProvider);

    // 완전 첫 로딩
    if (data is PaginationLoading) {
      return Center(
        child: CircularProgressIndicator(), // 로딩 UI
      );
    }

    // 에러 발생
    if (data is PaginationError) {
      return Center(
        child: Text(data.message),
      );
    }

    // CursorPaginationModel
    // CursorPaginationFetchingMore
    // CursorPaginationRefetching
    final cp = data as CursorPagination;

    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 16.0),
      child: ListView.separated(
        controller: controller,
        itemCount: cp.data.length + 1,
        itemBuilder: (context, index) {
          // 추가로 불러오는 로딩 UI와 마지막 데이터 UI를 보여주기 위한 조건
          if (index == cp.data.length) {
            return Padding(
              padding: const EdgeInsets.symmetric(
                horizontal: 16.0,
                vertical: 8.0,
              ),
              child: Center(
                child: data is PaginationFetchingMore
                    ? CircularProgressIndicator() // 로딩 UI
                    : Text("마지막 데이터입니다 ㅠㅠ"),
              ),
            );
          }

          final item = cp.data[index];

          return GestureDetector(
            onTap: () {
              // 클릭했을 때 상세페이지로 이동
              Navigator.of(context).push(
                MaterialPageRoute(
                  builder: (context) => RestaurantDetailScreen(
                    id: item.id,
                  ),
                ),
              );
            },
            // Component화 해놓은 UI로 모델을 전달해서 UI 생성
            child: RestaurantCard.fromModel(model: item),
          );
        },
        // 아이템 간격 추가
        separatorBuilder: (context, index) => Gap(16.0),
      ),
    );
  }
}
```

## 마치며

기능 구현을 위해 작성한 Class도 많고 상태를 체크하는 로직이 조금 복잡할 수도 있지만, 앱 전체에서 재활용한다고 생각하면 이 정도는 고생할 수 있다고 생각한다.

이전이라면 ViewController에서 모든 상태를 변수로 관리해서, **구현도 복잡하고 반복적으로 작성**해야 했을 거다. 거기에 디버깅할 때 변수값을 확인해야 해서 오류를 찾는 과정도 길어진다. 만약에 응답 스펙이나 UI 수정이라도 들어오면 모든 페이지를 뒤지면서 코드를 수정해야 할 거다. ~~야근각~~

이제는 새로운 리스트 페이지를 만들 때 Model Class만 만들어서 넣으면 로직을 다시 작성할 필요도 없고, 수정이 생겨도 한 번에 적용할 수 있다.

이번에는 페이지네이션에 대해서만 이야기했지만, 앱 전체의 반복적인 기능에 대해서 공통으로 적용할 수 있는 부분인 것 같아서, 또 적용할 수 있는 부분이 있을지 찾아봐야겠다.

> 역시 개발자의 최고 덕목은 반복 코드를 줄이고 구조화해서 **딸깍**할 수 있는 코드를 짜는 것 같다.

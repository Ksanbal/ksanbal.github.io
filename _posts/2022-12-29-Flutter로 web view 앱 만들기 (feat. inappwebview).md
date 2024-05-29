---
layout: post
---
Flutter 내장의 Webview가 있지만 나는 [flutter_inappwebview](https://pub.dev/packages/flutter_inappwebview)를 선택했다.

내장 웹뷰보다 기능이 많고, js interface와 같은 기능이 필요했기 때문에 선택하게 되었다.

## ⚒️ 기본설정

### Android

android > app > build.gradle 에서 `minSdkVersion 17` 이상

### Pub get

```bash
flutter pub add flutter_inappwebview
```

## 🧑‍💻 Code

### main()

main 부분에 os에 따른 컨트롤러 설정이 필요하다.

```dart
Future main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // WebView
  if (Platform.isAndroid) {
    await AndroidInAppWebViewController.setWebContentsDebuggingEnabled(true);
  }

  runApp(MyApp());
}
```

### InAppWebViewGroupOptions

웹뷰에 사용될 option을 지정해준다.

```dart
InAppWebViewGroupOptions _options = InAppWebViewGroupOptions(
  android: AndroidInAppWebViewOptions(
    useHybridComposition: true,
  ),
  ios: IOSInAppWebViewOptions(
    allowsInlineMediaPlayback: true,
  ),
);
```

- Webview의 userAgent 변경 또는 뒤에 추가하기
    
    웹뷰에서의 userAgent 값을 완전히 변경하거나, 뒤에 원하는 텍스트를 추가할 수 있다.
    
    ```dart
    InAppWebViewGroupOptions _options = InAppWebViewGroupOptions(
      android: AndroidInAppWebViewOptions(
        useHybridComposition: true,
      ),
      ios: IOSInAppWebViewOptions(
        allowsInlineMediaPlayback: true,
      ),
      crossPlatform: InAppWebViewOptions(
    		userAgent: "APP_FLUTTER", // 완전 변경
    		applicationNameForUserAgent: "APP_FLUTTER", // 맨뒤에 추가
      ),
    );
    ```
    
    - 2021.12.29
        - Flutter version : 2.8.1
        flutter_inappwebview : 5.3.2
        Android에서는 `applicationNameForUserAgent`가 정상적으로 돌아가는데, iOS에서는 안돌아가더라;;; github issues에 등록된 이슈가 있고, 수정되었다고 하는데 난 왜 안될까...

### InAppWebView

Webview Widget

```dart
InAppWebViewController? webViewController;

@override
Widget build(BuildContext context) {
	return Scaffold(
	  backgroundColor: Colors.white, // 배경색
	  body: SafeArea(
	    bottom: Platform.isIOS,
	    child: InAppWebView(
	      initialUrlRequest: URLRequest(url: Uri.parse("https://m.naver.com")), // 주소는 꼭 https부터 쓰자!
	      initialOptions: _options, // 미리 작성했던 옵션
	      onWebViewCreated: (controller) async {
					// WebView 첫 생성시 실행
					// initState() 같은 느낌으로 사용
					// JS Interface 핸들러를 여기에 작성
	        webViewController = controller;
	      },
	      onConsoleMessage: (controller, consoleMessage) {
					// Console 메세지 출력
	        print("console : $consoleMessage");
	      },
	      androidOnPermissionRequest: (controller, origin, resources) async {
	        return PermissionRequestResponse(
	          resources: resources,
	          action: PermissionRequestResponseAction.GRANT,
	        );
	      },
	    ),
	  ),
	);
}
```

기본적인 세팅은 여기까지! 이제 각종 옵션을 추가해보자

## Plus➕

### Android 뒤로가기 버튼을 web내의 뒤로가기로 작동하도록 변경

andrid는 하단에 뒤로가기 버튼이 있는데, 따로 설정해주지 않으면 바로 앱을 꺼버린다. 웹에서 뒷페이지로 가도록 추가해주자.

- 뒤로가기 기능 작성

```dart
Future<bool> _goBack(BuildContext context) async {
  if (webViewController == null) {
    return Future.value(true);
  }

  if (await webViewController!.canGoBack()) {
    webViewController!.goBack();
    return Future.value(false);
  }
  return Future.value(true);
}
```

- WillPopScope로 감싸기
    - WillPopScope는 뒤로가기 기능을 제어할 수 있는 Widget이다. WillPopScope로 감싸면 Appbar에 자동생성되는 뒤로가기가 작동하지 않는다;;;;; 필요하다면 leading으로 IconButton을 추가하여 사용하자.

```dart
@override
Widget build(BuildContext context) {
	return WillPopScope(
		onWillPop: () => _goBack(context),
		child: Scaffold(
				...
		),
	);
}
```

### 새로고침 적용하기

`아래로 끌어서 새로고침` 기능을 추가해보자

- refesh 했을때의 동작을 작성한 컨트롤러

```dart
_pullToRefreshController = PullToRefreshController(
  options: PullToRefreshOptions(
    color: Colors.green,
  ),
  onRefresh: () async {
    if (Platform.isAndroid) {
      webViewController?.reload();
    } else if (Platform.isIOS) {
      webViewController?.loadUrl(
        urlRequest: URLRequest(
          url: await webViewController?.getUrl(),
        ),
      );
    }
  },
);
```

- InAppWebView에 적용

```dart
@override
Widget build(BuildContext context) {
	return Scaffold(
		...
	  body: SafeArea(
	    bottom: Platform.isIOS,
	    child: InAppWebView(
				...
	      pullToRefreshController: _pullToRefreshController,
	      onProgressChanged: (controller, progress) {
					// 내려서 refresh를 사용했을때 완료되면 종료되도록
	        if (progress == 100) {
	          _pullToRefreshController.endRefreshing();
	        }
	      },
				...
	    ),
	  ),
	);
}
```

### JavaScript Handler(JS Interface, JS Message Handler) 적용하기

android에는 WebView JavaScript Interface가 있고, ios에는 WKWebview JavaScript Message Handler가 있다. WebView안의 JS와 데이터를 주고받을때 사용할 수 있다. 

- Example html
    - 아래 코드는 Webview가 아닌 상황에서는 작동하지 않는다. pc나 모바일 웹 브라우저에서 접속할때는 오류가 발생할 수 있으니까, userAgent를 체크해서 작성해주는게 좋다!

```html
<!DOCTYPE html>
<html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
  </head>
  <script>
    function call() {
      window.flutter_inappwebview.callHandler('callHandler').then(function(result) {
					console.log("JS Handler!");
          console.log(JSON.stringify(result));
      });
    }
    function callWithArgs() {
			const args = [1, true, ['bar', 5], {'key':'value'}]
      window.flutter_inappwebview.callHandler('argsHandler', ...args).then(function(result) {
					console.log("JS Handler with args!");
          console.log(JSON.stringify(result));
      });
    }
  </script>
  <body>
    <h1>JavaScript Handlers</h1>
    <FORM>
      <INPUT type="BUTTON", style="WIDTH:100pt; HEIGHT:50pt", value="call", onclick='call()'>
      <INPUT type="BUTTON", style="WIDTH:100pt; HEIGHT:50pt", value="callWithArgs", onclick='callWithArgs()'>
      <br>
    </FORM>
  </body>
</html>
```

- Flutter에 callback 작성하기

```dart
InAppWebView(
	...
	initialData : InAppWebViewInitialData(data: """
		<!DOCTYPE html>
		<html>
		...
		</html>
	"""),
	onWebViewCreated: (controller) async {
		webViewController = controller;

		// call
		webViewController?.addJavaScriptHandler(
			handlerName: 'callHandler',
			callback: (args) async {
				return {
					'this is': 'return value!',
				};
			},
		);

		// call with args
		webViewController?.addJavaScriptHandler(
			handlerName: 'argsHandler',
			callback: (args) async {
				print(args);
				// -> [1, true, ['bar', 5], {'key':'value'}]
				return {
					'this is': 'return value!',
				};
			},
		);
	},
	...
),
```

위 방법 외에도 Web Message Channels & Listeners도 있으니까, 한번쯤 확인해봐도 좋을 것 같다.

### 자잘한 기능

- url 이동하기 : `webViewController?.loadUrl(urlRequest: URLRequest(url: Uri.parse(”https://m.naver.com”)));`
  - FCM Push랑 같이 쓰기
    - Push를 클릭해서 들어왔을때 loadURL을 쓰면 열릴때도 있고, 안열릴때도 있다. 웹뷰가 생성되기 전에 push handler가 실행되서 무시되거나, 웹뷰가 생성되자마자 실행되서 첫 url이 무시되는 경우가 있다. 첫 url이 무시되면 뒤로가기 history에 남지 않기 때문에, 뒤로가기를 누르면 앱에 꺼진다. 
    그래서 timer로 1.5~2초 뒤에 실행되도록 작성해봤다.
    
    ```dart
    void _handleMessage(RemoteMessage message) {
      print("닫았다가 실행!");
      if (message.data['url'] != null) {
        Timer? _timer;
    
        _timer = Timer.periodic(Duration(milliseconds: 500), (timer) {
          if (timer.tick > 3) {
            webViewController?.loadUrl(
              urlRequest: URLRequest(url: Uri.parse(message.data['url'])),
            );
            _timer?.cancel();
          }
        });
      }
    }
    ```

## 참고

[flutter_inappwebview | Flutter Package](https://pub.dev/packages/flutter_inappwebview)
[Flutter InAppWebView Documentation](https://inappwebview.dev/docs/)
[Flutter WebView JavaScript Communication - InAppWebView 5](https://medium.com/flutter-community/flutter-webview-javascript-communication-inappwebview-5-403088610949)
애플 로그인은 처음 시도할때만 id, email, name을 전달해줌
2번째 시도부터는 id만 전달해줌
회원가입 페이지에 email 값을 채워넣어야하는데 필요했음
identityToken은 jwt로 되어있는데, 그 payload에 email 정보가 있음
```dart
final jwt = result.identityToken!.split('.');
String payload = jwt[1];
payload = base64.normalize(payload);

final List<int> jsonData = base64.decode(payload);
final userInfo = jsonDecode(utf8.decode(jsonData));
email = userInfo['email'];
```
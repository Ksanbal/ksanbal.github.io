---
title: Flutter & Firebase Auth(Google, Apple, Email Login) 사용하기
description: 
date: 2021-09-07
image: 
categories:
- Flutter
tags:
weight: 1
---

## Setting

### Google Login

- Firebase에서 안드로이드 앱을 등록할때 반드시 **SHA 인증서 지문**을 입력해줘야한다.
    ```
    keytool -list -v \
    -alias androiddebugkey -keystore ~/.android/debug.keystore
    ```
    
    - 비밀번호는 반드시 **android**로 한다
    
    ```dart
    // flutter pub add google_sign_in.dart
    
    import 'package:google_sign_in/google_sign_in.dart';
    
    Future<UserCredential> signInWithGoogle() async {
    	try {
    	  // Trigger the authentication flow
    	  final GoogleSignInAccount? googleUser = await GoogleSignIn().signIn();
    	
    	  // Obtain the auth details from the request
    	  final GoogleSignInAuthentication googleAuth =
    	      await googleUser!.authentication;
    	
    	  // Create a new credential
    	  final credential = GoogleAuthProvider.credential(
    	    accessToken: googleAuth.accessToken,
    	    idToken: googleAuth.idToken,
    	  );
    	
    	  // Once signed in, return the UserCredential
    	  return await FirebaseAuth.instance.signInWithCredential(credential);
    	} catch (e) {
    		print(e);
    	}
    }
    ```
    
- 구글 로그인의 경우 예외처리를 해줘도 PlatformExption이 발생하는데, reales 버전에서는 문제 없다고 한다...

### Apple Login

- 사전준비
    1. Firebase에서 앱을 추가할때 Apple Developer에서 발급받은 앱 ID가 필요
        - Apple Developer → Certificates → identifiers → 생성
        - App IDs → App → Sign In with Apple 체크 후 등록
    2. Xcode Runner → Signing & Capabilities → + Capability → Sign in with Apple
    3. GoogleService-Info.plist에서 REVERSED_CLIENT_ID 값을 Runner → Info → URL Types 추가 후 URL Schemes에 입력
- 필요조건
    1. iOS 13 이상 : Signing with Apple
        
        ```dart
        // flutter pub add sign_in_with_apple
        
        import 'package:sign_in_with_apple/sign_in_with_apple.dart';
        import 'dart:math';
        import 'dart:convert';
        
        String generateNonce([int length = 32]) {
          final charset =
              '0123456789ABCDEFGHIJKLMNOPQRSTUVXYZabcdefghijklmnopqrstuvwxyz-._';
          final random = Random.secure();
          return List.generate(length, (_) => charset[random.nextInt(charset.length)])
              .join();
        }
        
        /// Returns the sha256 hash of [input] in hex notation.
        String sha256ofString(String input) {
          final bytes = utf8.encode(input);
          final digest = sha256.convert(bytes);
          return digest.toString();
        }
        
        Future<UserCredential> signInWithApple() async {
        	try {
        	  // Request credential for the currently signed in Apple account.
        	  final appleCredential = await SignInWithApple.getAppleIDCredential(
        	    scopes: [
        	      AppleIDAuthorizationScopes.email,
        	      AppleIDAuthorizationScopes.fullName,
        	    ],
        	  );
        	
        	  // Create an `OAuthCredential` from the credential returned by Apple.
        	  final oauthCredential = OAuthProvider("apple.com").credential(
        	    idToken: appleCredential.identityToken,
        	    accessToken: appleCredential.authorizationCode,
        	  );
        	
        	  // Sign in the user with Firebase. If the nonce we generated earlier does
        	  // not match the nonce in `appleCredential.identityToken`, sign in will fail.
        	  return await FirebaseAuth.instance.signInWithCredential(oauthCredential);
        	} catch (e) {
        		// 로그인을 취소했을 경우
        		print(e);
        	}
        }
        ```
        

### Email Login

```dart
void _emailSignUp() async {
    try {
      UserCredential userCredential = await FirebaseAuth.instance
          .createUserWithEmailAndPassword(
              email: "myemail@google.com", password: "SuperSecretPassword!");
    } on FirebaseAuthException catch (e) {
      if (e.code == 'weak-password') {
        print('The password provided is too weak.');
      } else if (e.code == 'email-already-in-use') {
        print('The account already exists for that email.');
      }
    } catch (e) {
      print(e);
    }
  }

  void _emailSignIn() async {
    try {
      // UserCredential userCredential =
      await FirebaseAuth.instance.signInWithEmailAndPassword(
          email: "myemail@google.com", password: "SuperSecretPassword!");
    } on FirebaseAuthException catch (e) {
      if (e.code == 'user-not-found') {
        print('No user found for that email.');
      } else if (e.code == 'wrong-password') {
        print('Wrong password provided for that user.');
      }
    }
  }
```

## Use

### StreamBuilder로 상태 가져오기

```dart
class Home extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (BuildContext context, AsyncSnapshot<User?> user) {
        if (!user.hasData) {
          // 로그인한적이 없는 경우
          return SignIn();
        }

        // 로그인한 경우
        return MyPage();
      },
    );
  }
}
```

### 정보가져오기

- `FirebaseAuth.instance.currentUser!.`
    - photoURL
    - displayName : google에서 따로 지정하지 않으면 null
    - email
    - emailVerified
    - metadata
        - creationTime
        - lastSignInTime
    - uid

### 동작하기

- 이메일 인증 메일 보내기
    `FirebaseAuth.instance.currentUser!.sendEmailVerification();`
- 로그아웃
    `FirebaseAuth.instance.signOut();`
- 유저 삭제
    `FirebaseAuth.instance.currentUser!.delete();`
- 비밀번호 재설정
    `FirebaseAuth.instance.sendPasswordResetEmail(email: "email@example.com")`

---
title: Flutter CocoaPods could not find compatible versions for pod "FirebaseMessaging" on M1 해결법
description: 
date: 2021-09-24
image: 
categories:
    - Flutter
tags:
weight: 1
---

 Flutter로 개발하다보면 Android를 먼저 적용하고, 나중에 ios를 적용하는게 뭔가 편하게 느껴질 때가 있다. ios에서 FCM이나 Apple 로그인을 적용해주려면 Developer 계정이 필요한데, 늦게 결정되는 경우가 있어서 Android를 먼저 적용하게된다. 하지만 Flutter는 ios에서 이슈가 더 많이 생기는 것 같다...ㅎ

 오늘의 이슈는 Firebase 적용해서 ios 빌드 중 발생하는 문제이다.

 Firebase에서 알려주는대로 Podfile을 작성하면

```yaml
platform :ios, ‘10.0’

# Add the Firebase pod for Google Analytics
pod 'Firebase/Analytics'

# For Analytics without IDFA collection capability, use this pod instead
# pod ‘Firebase/AnalyticsWithoutAdIdSupport’

# Add the pods for any other Firebase products you want to use in your app
# For example, to use Firebase Authentication and Cloud Firestore
pod 'Firebase/Auth'
pod 'Firebase/Firestore'
```

이런 식이여서 메세지 추가하고, 하는 방식으로 진행해도 된다.

 그런데 나는  Blufi_plugin이라는 패키지를 쓰는데, 이게 이렇게 하면 적용이 안된다;;;

그래서 해결방법은 ios/Podfile을 아래와 같이 작성해주면, 자동으로 패키지들을 가져오게된다.

```yaml
# Uncomment this line to define a global platform for your project
platform :ios, '10.0'

# CocoaPods analytics sends network stats synchronously affecting flutter build latency.
ENV['COCOAPODS_DISABLE_STATS'] = 'true'

project 'Runner', {
  'Debug' => :debug,
  'Profile' => :release,
  'Release' => :release,
}

def flutter_root
  generated_xcode_build_settings_path = File.expand_path(File.join('..', 'Flutter', 'Generated.xcconfig'), __FILE__)
  unless File.exist?(generated_xcode_build_settings_path)
    raise "#{generated_xcode_build_settings_path} must exist. If you're running pod install manually, make sure flutter pub get is executed first"
  end

  File.foreach(generated_xcode_build_settings_path) do |line|
    matches = line.match(/FLUTTER_ROOT\=(.*)/)
    return matches[1].strip if matches
  end
  raise "FLUTTER_ROOT not found in #{generated_xcode_build_settings_path}. Try deleting Generated.xcconfig, then run flutter pub get"
end

require File.expand_path(File.join('packages', 'flutter_tools', 'bin', 'podhelper'), flutter_root)

flutter_ios_podfile_setup

target 'Runner' do
  use_frameworks!
  use_modular_headers!

  flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
  end
end
```

그럼 이제 적용해야하니까

```bash
# 일단 마음 편하게 Clean부터
pod clean

pod install
```

하면 `CocoaPods could not find compatible versions for pod "Firebase/Messaging"`를 뱉는 경우가 있는데 구글링 해보면 `pod install —repo-update`를 이야기한다. 이게 또 m1에서는 잘 안돌아간다;;;

그래서 

```bash
arch -x86_64 pod install --repo-update
```

해주면 돌아간다!

 한참 돌다가 해결했는데, `coreNotInitialized()`가 생기더라...이것도 해결해야지 뭐...

~~→ 는 내가 google 파일을 잘못넣었더라 데헿~~
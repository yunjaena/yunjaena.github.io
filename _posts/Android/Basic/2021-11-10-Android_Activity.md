---
title: Android Activity
author: Yun-Jae Na
date: 2021-11-10 01:00:00 +0900
categories: [Android, Basic]
tags: [Android, Activity]
---

## Activity 란?

- An activity is the entry point for interacting with the user. It represents a single screen with a user interface

- 사용자와 상호작용하는 위한 진입점이다. 사용자 인터페이스를 포함한 화면 하나를 나타낸다.

- The Activity class is a crucial component of an Android app, and the way activities are launched and put together is a fundamental part of the platform's application model.

- Activity 클래스는 Android 앱의 중요한 구성요소로 Activity(활동)이 실행되고 결합되는 방식은 애플리케이션 모델의 기본 요소이다.

- Unlike programming paradigms in which apps are launched with a main() method, the Android system initiates code in an Activity instance by invoking specific callback methods that correspond to specific stages of its lifecycle.

- main() 메서드를 사용하여 앱을 실행하는 프로그래밍 패러다임과 달리 Android 시스템은 수명 주기의 특정 단계에 해당하는 특정 콜백 메서드를 호출하여 Activity 인스턴스 코드를 시작한다.

## Activity 개념

- 모바일 앱 환경은 사용자의 앱의 상호작용이 항상 동일한 위치에서 시작되는 것이 아니라는 점에서 데스크톱 앱 환경과 다르다. 홈 화면에서 이메일 앱을 열면 이메일 목록이 표시될 수 있다. 반대로 소셜 미디어 앱을 사용하고 있는 상태에서 이메일을 작성하기 위한 이메일 앱 화면으로 바로 이동할 수 있다. 즉 `비결정론적`으로 시작할 수 있다.

- 한 앱이 다른 앱을 호출할 때 호출 앱은 다른 앱을 전체적으로 호출하는 것이 아니라 다른 앱의 활동을 호출한다. 이런 방식으로 Activity는 앱과 사용자의 상호작용을 위한 진입점 역할을 한다.

- Activity는 Activity 클래스의 서브클래스로 구현된다.

- Activity는 앱이 UI를 그리는 창을 제공한다. 이 창은 일반적으로 화면을 채우지만 화면보다 작고 다른 창위에 떠 있을 수 있다.

- 대부분의 앱에는 여러 화면이 포함되어 있으며 여러 Activity로 구성된다. 일반적으로 앱에서 하나의 활동이 기본 활동으로 지정되며 이 기본 Activity는 사용자가 앱을 실행할 때 표시되는 첫 번째 화면이다.

- 각 Activity는 다양한 Activity를 실행하기 위해 또 다른 Activity를 실행할 수 있다. 예를들어 간단한 이메일 앱의 기본 Activity는 이메일 받은편지함을 표시하는 화면을 제공할 수 있다. 여기에서 기본 Activity는 이메일 작성 및 개별 이메일 열기와 같은 작업을 위한 화면을 제공하는 다른 Activity를 실행할 수 있다.

- Activity가 앱의 일관된 사용자 환경을 형성하기 위해 함께 작동하지만 각 Activity는 다른 Activity와 느슨하게 결합된다. 일반적으로 앱의 Activity 간에는 최소한의 종속성만 있다.

- Activity는 흔하게 다른 앱에 속하는 Activity를 실행한다. 예를 들어 브라우저 앱은 소셜 미디어 앱의 공유 활동을 실행할 수 있다.

## manifest 구성

### Activity 선언

- Activity을 선언하려면 manifest 파일을 열고 `<activity>` 요소를 `<application>` 요소의 하위 요소로 추가해야 한다.

```xml
<manifest ... >
  <application ... >
      <activity android:name=".ExampleActivity" />
      ...
  </application ... >
  ...
</manifest >
```

- Activity 클래스 이름을 지정하는 `android:name`는 필수 속성이다. 또한 라벨, 아이콘 또는 UI 테마와 같은 Activity 특성을 정의하는 속성도 추가할 수 있다.


### 인텐트 필터 선언

- 인텐트 필터는 명시적 요청뿐만 아니라 암시적 요청을 기반으로도 Activity를 실행하는 기능을 제공한다.

- 명시적 요청을 예를 들면 'Gmail 앱에서 이메일 보내기 Activity를 실행하기' 기능을 제공하는 것이다.

- 암시적 요청은 '작업을 할 수 있는 Activity로 이메일 보내기 화면을 시작하기' 기능을 제공하도록 시스템에 지시하는 것이다.

- `<activity>` 요소에서 `<intent-filter>` 속성을 선언함으로써 이 기능을 활용할 수 있다. 이 요소의 정의에는 `<action>` 요소와 선택적으로 `<category>` 요소 또는 `<data>` 요소가 포함된다.

```xml
<activity android:name=".ExampleActivity" android:icon="@drawable/app_icon">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

- 위의 코드는 텍스트 데이터를 전송하고 다른 Activity들의 요청을 수신하는 Activity를 구성하는 방법을 나타낸다. `<action>` 에서는 Activity가 데이터를 전송하도록 지정되어있다.  `<category>` 요소를 `DEFAULT`로 선언하면 Activity 실행 요청을 수신할 수 있다. `<data>` 요소는 Activity가 전송할 수 있는 데이터 유형을 지정한다.

- 아래와 같은 코드를 호출하면 위에 설명된 Activity를 호출할 수 있다.

```kotlin
val sendIntent = Intent().apply {
    action = Intent.ACTION_SEND
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, textMessage)
}
startActivity(sendIntent)
```

## Activity lifecycle (액티비티 생명 주기) 관리

### onCreate()

- 시스템이 Activity를 생성할 떄 실행되는 이 콜백을 구현해야 한다. 구현 시 Activity의 필수 구성요소를 초기화해야한다. 앱은 뷰를 생성하고 목록에 데이터를 결합해야 한다.

- 이 콜백에서 `setContentView()`를 호출하여 Activity의 사용자 인터페이스를 위한 레이아웃을 정의해야 한다.

- onCreate()가 완료되면 다음 콜백은 항상 onStart()이다.

### onStart()

-  onCreate()가 종료되면 Activity는 '시작됨' 상태로 전환되고 Activity가 사용자에게 표시된다. 이 콜백에는 Activity가 포그라운드로 나와서 대화형이 되기 위한 최종 준비에 준하는 작업이 포함된다.

### onResume()

- Activity가 사용자와 상호작용을 시작하기 직전에 시스템은 이 콜백을 호출한다. 이 시점에서는 Activity는 Activity 스택의 맨 위에 있으며 모든 사용자의 입력을 캡쳐한다. 앱의 핵심 기능은 대부분 onResume() 메서드로 구현된다.

- onPause() 콜백은 항상 onResume() 뒤에 온다.

### onPause()

- Activity가 포커스를 잃고 '일시중지됨' 상태로 전환될 때 시스템은 onPause()를 호출한다.

- 예를 들어 이 상태는 사용자가 뒤로 또는 최근 버튼을 탭할 떄 발생한다.

- 엄밀히 말하면 Activity가 여전히 부분적으로 표시되지만 대체로 사용자가 Activity를 떠나고 있고 조만간 '중지됨' 또는 '다시 시작됨' 상태로 전환됨을 나타낸다.

### onStop()

- Activity가 사용자에게 더 이상 표시되지 않을 때 시스템은 onStop()을 호출한다.

- Activity가 제거 중이거나 새 Activity 시작 중이거나 기존 Activity가 'onResume()' 상태로 전환 중이고 중지된 Activity를 다루고 있기 때문에 발생할 수 있다. 이 모든 상황에서는 중지된 Activity는 더 이상 표시되지 않는다.

- 시스템이 호출하는 다음 콜백은 활동이 사용자와 상호작용하기 위해 다시 시작되면 `onRestart()`이며 Activity가 완전히 종료되면 `onDestroy()`이다.

### onRestart()

- '중지됨' 상태의 Activity가 다시 시작되려고 할 때 시스템은 이 콜백을 호출한다.

- onRestart()는 활동이 중지된 시간부터 활동 상태를 복원한다.

- 이 콜백 뒤에 항상 onStart()가 온다.

### onDestroy()

- 시스템이 Activity가 제거되기 전에 이 콜백을 호출한다.

- 이 콜백은 Activity가 수신하는 마지막 콜백이다.

- 일반적으로 Activity 또는 Activity가 포함된 프로세스가 제거될 때 Activity의 모든 리소스를 해제하도록 구현된다.

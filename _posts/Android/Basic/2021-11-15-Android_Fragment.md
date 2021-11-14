---
title: Android Fragment
author: Yun-Jae Na
date: 2021-11-15 12:54:00 +0900
categories: [Android, Basic]
tags: [Android, Fragment]
---

## Fragment 란?

- `Fragment`는 앱 UI의 재사용 가능한 부분을 나타낸다. 프래그먼트는 자체 레이아웃을 정의 및 관리하고 자체 수명 주기를 보유하며 자체 입력 이벤트를 처리할 수 있다.

- 프레그먼트는 독립적으로 존재할 수 없고 Activity나 다른 프래그먼트에서 호스팅되어야 한다.

- 프르그먼트의 뷰 계층 구조는 호스트 뷰 계층 구조의 일부가 되거나 여기에 연결된다.

## 모듈성

- 프래그먼트는 UI를 개별 청크로 분할할 수 있도록 하여 Acitivty의 UI에 모듈성 재사용성을 도입한다.

- Activity는 앱의 사용자 인터페이스 (예: 탐색 창) 주위에 전역 요소를 배치하기 적합하다. 반대로 프래그먼트는 단일 화면이나 화면 일부의 UI를 정의하고 관리하는데 적합하다.

- UI를 프래그먼트로 나누면 런타임 시 Activity의 모양을 더 쉽게 수정할 수 있다. Acitivity가 `STARTED` 수명 주기 상태 이상에 있는 동안 프래그먼트를 추가하거나 교체, 삭제 할 수 있다. 이러한 변경사항 기록을 Activity에서 관리하는 백 스택에 보관할 수 있으므로 변경사항을 취소할 수 있다.

- 동일한 Activity 내에서, 여러 Activity 또는 다른 Fragment의 하위 요소로도 동일한 프래그먼트 클래스의 여러 인스턴스를 사용할 수 있다. 이 점에 유의하여 자체 관리에 필요한 로직만 프래그먼트에 제공해야 한다.

- 한 프래그먼트에 의존하거나 다른 프래그먼트에서 프래그먼트를 조작하지 않아야 한다.

## 프래그먼트 생성

### 환경 설정

- Fragment는 AndroidX Fragment 라이브러리를 참조한다. Google Maven repository를 프로젝트단 build.gradle 파일에 추가해야한다.

```groovy
buildscript {
    ...

    repositories {
        google()
        ...
    }
}

allprojects {
    repositories {
        google()
        ...
    }
}
```

- App build.gradle 파일에 아래와 같은 라이브러리를 추가한다.

```groovy
dependencies {
    val fragment_version = "1.3.6"

    // Java language implementation
    implementation("androidx.fragment:fragment:$fragment_version")
    // Kotlin
    implementation("androidx.fragment:fragment-ktx:$fragment_version")
}
```

## Fragment 클래스 생성

- Fragment를 생성하기위해, AndroidX 프레크먼트 클래스를 상속해야한다. 그리고 Fragment 메소드를 override 해서 앱의 로직을 구현해야한다.

- 레이아웃만 정의하는 최소한 Fragment 생성자에 레이아웃 리소스만 제공하면 된다.

```kotlin
class ExampleFragment : Fragment(R.layout.example_fragment)
```

- Fragment는 아래와 같은 기본 클래스를 제공한다.

    - [DialogFragment](https://developer.android.com/reference/androidx/fragment/app/DialogFragment) : 위에 떠있는 다이얼로그를 보여준다.

    - [PreferenceFragment](https://developer.android.com/reference/androidx/preference/PreferenceFragmentCompat) : 세팅화면을 생성한다.

## Activity에 Fragment 추가하기

- 일반적으로, Fragment는 Activity의 레이아웃 UI에 일부가 되기 위해서 AndroidX `FragmentActivity`에 내장되어야한다.

- `FragmentActivity`는 `AppCompatActivity`의 부모 클래스 이므로 `AppCompactActivity`를 상속받고 있으면 액티비티의 부모 클래스를 변경할 필요가 없다.

- 액티비티의 레이아웃 파일에서 프래그먼트를 정의하거나 액티비티의 레이아웃 파일에서 프래그먼트 컨테이너를 정의한 다음 액티비티 내에서 프로그래밍 방식으로 프래그먼트를 추가하여 액티비티의 뷰 계층에 프래그먼트를 추가할 수 있다.

- 위의 두 가지 경우 Fragment를 배치해야 하는 위치를 정의하는 `FragmentContainerView`를 추가해야한다.

- `FragmentContainerView`에서는 `FrameLayout`과 같은 다른 View group에서 제공하지 않는 Fragment의 동작을 조정할 수 있는 추가 기능이 있으므로 `FragmentContainerView`를 사용하는것이 좋다.

## XML를 통해서 Fragment를 추가하기

- xml 레이아웃을 통해서 Fragement를 추가하기 위해서 `FragmentContainerView`를 사용한다.

```xml
<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:name="com.example.ExampleFragment" />
```
- `android:name` 속성은 인스턴화를 하기위해서 `Fragment` 클래스 이름을 명시한다.

- `onInflate()`가 새로 Fragment가 인스턴스화가되면 불린다. 또한 `FragmentTransaction`이 `FragmentManager`에 추가되기 위해 생성된다.

## 프로그래밍 방식으로 Fragment 추가하기

- 프로그래밍 방식으로 Activity 레이아웃에 추가하기위해서 `FragmentContainerView`를 fragment container로 사용하기위해 추가해야한다.

```xml
<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

- XML를 통해서 Fragment를 추가하는 방법과 다르게 `android:name` 속성을 사용하지 않았으므로, 아무런 fragment가 자동적으로 인스턴화가 되지 않을것이다. 대신에, `FragmentTransaction`를 통해 fragment를 인스턴화해서 activity 레이아웃애 추가를한다.

- Activity가 동작하고 있을때 fragment를 추가하고, 제거하고 대체하는 fragment 트랜잭션을 할 수 있다.

- FragmnetActivity에서 `FragmentManager`의 인스턴스를 가져올 수 있으며, `FragmentTransaction`을 생성하는데 사용할 수 있다.

- Activity가 `onCreate()` 상태일때 `FragmentTransaction.add()` 메서드를 사용해서 fragment를 인스턴화 할 수 있다. 레이아웃에 있는 컨테이너의 `ViewGroup` ID 와 추가하고 싶은 Fragment 클래스를 넘긴다음 트랜잭션을 commit 하면된다.

```kotlin
class ExampleActivity : AppCompatActivity(R.layout.example_activity) {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if (savedInstanceState == null) {
            supportFragmentManager.commit {
                setReorderingAllowed(true)
                add<ExampleFragment>(R.id.fragment_container_view)
            }
        }
    }
}
```

- `FragmentTransaction`를 사용할때 항상 `setReorderingAllowed(true)`를 사용해야한다.

- 위의 예제에서 fragment 트랜잭션은 `savedInstaceState` 가 `null` 일때 생성된다. 이것은 fragment가 한번 추가되기 위함이다. configuration change 나 Activity가 재생성 되었을떄 `savedInstanceState`가 더이상 `null`이 아니기 떄문이다. 그리고 fragment는 `savedInstanceState`에서 자동으로 복구하기 때문에 두번 추가될 필요가 없다.

- fragment가 추가적으로 초기화 데이터가 필요할때 `Bundle`을 통해서 fragment에 데이터를 보낼 수 있다.

```kotlin
class ExampleActivity : AppCompatActivity(R.layout.example_activity) {
      override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if (savedInstanceState == null) {
            val bundle = bundleOf("some_int" to 0)
            supportFragmentManager.commit {
                setReorderingAllowed(true)
                add<ExampleFragment>(R.id.fragment_container_view, args = bundle)
            }
        }
    }
}
```

- `Bundle` 인자들은 fragment에서 `requireArguments()`를 통해서받을 수 있다.

```kotlin
class ExampleFragment : Fragment(R.layout.example_fragment) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        val someInt = requireArguments().getInt("some_int")
        ...
    }
}
```

## 프래그먼트 관리자 (Fragment manager)

- `FragmentManager`는 앱 프래그먼트에서 작업을 추가, 삭제 또는 교체하고 백 스택에 추가하는 등의 작업을 실행하는 클래스이다.

## 프래그먼트 관리자에 액세스

### Activity에서 액세스

- 모든 `FragmentActivity` 및 서브 클래스(예: AppCompatActivity)는 `getSupportFragmentManager()` 메서드를 통해 FragmentManager에 액세스할 수 있다.

### 프래그먼트에서 액세스

- 프래그먼트는 하위 프래그먼트를 하나 이상 호스팅할 수도 있다.

- 프래그먼트 내에서 `getChildFragmentManager()`를 통해 프래그먼트의 하위 요소를 관리하는 `FragmentManager` 참조를 가져올 수 있다.

- 호스트 `FragmentManager`에 액세스해야 한다면 `getParentFragmentManager()`를 사용하면 된다.

![fragment manager](https://drive.google.com/uc?export=view&id=1h5_PWiKAtwX5tfdsLvGhhEemv8wiIioe)

## 프래그먼트 관리자 사용

- `FragmentManager`는 프래그먼트 백 스택을 관리한다. 런타임 시 `FragmentManager`는 사용자 상호작용에 응답하여 프래그먼트를 추가하거나 삭제하는 등 백 스택 작업을 실행할 수 있다.

- 각 변경사항 집합은 `FragmentTransaction`이라는 단일 단위로 함께 커밋된다.

- 사용자가 기기에서 뒤로버튼을 누르는 경우 또는 개발자가 `FragmentManager.popBackStack()`을 호출하는 경우 최상위 프래그먼트 트랜잭션이 스택에서 사라진다. 즉, 트랜잭션이 취소된다. 스택에 더 이상 프래그먼트 트랜잭션이 없고 개발자가 하위 프래그먼트를 사용하지 않는 경우 뒤로 이벤트가 Activity까지 채워진다.

- 트랜잭션에서 `addToBackStack()`을 호출하면 여러 프래그먼트 추가, 여러 컨테이너의 프래그먼트 교체 등 많은 작업의 트랜잭션에 포함될 수 있다. 백 스택이 표시되면 이러한 모든 작업이 단일 원자 작업으로 취소된다.

- `popBackStack()` 호출 전에 추가 트랜잭션을 커밋한 경우 그리고 트랜잭션에 `addToBackStack()`을 사용하지 않은 경우 이러한 작업은 취소되지 않는다.

## 프래그먼트 생명주기

- Fragment 인스턴스는 각각의 생명주기가 있다. 사용자가 앱을 탐색하고 상호 작용할 때 프래그먼트는 추가, 제거, 화면 진입 또는 종료에 따라 생명 주기의 다양한 상태로 전환된다.

- Lifecycle을 관리하기 위해, Fragment는 LifecycleOwner를 구현하며 `getLifecycle()`를 통해 Lifecycle 객체를 접근할 수 있다.

- 각 `Lifecycle` 상태는 `Lifecycle.State` enum으로 표현된다.

  - INITIALIZED

  - CREATED

  - STARTED

  - RESUMED

  - DESTROYED

- `LifecycleObserver`를 사용하는 것에 대한 대안으로 Fragment class에 있는 콜백 메소드를 사용하면된다.

  - onCreate()

  - onStart()

  - onResume()

  - onPause()

  - onStop()

  - onDestroy()

- Fragment의 뷰는 분리된 `Lifecycle`이 Fragment 생명주기에 의해 독립적으로 관리된다.

- Fragment들은 `LifecycleOwner`를 관리하고 `getViewLifecycleOwner()` 또는 `getViewLifecycleOwnerLiveData()`를 통해서 접근할 수 있다.

## 프래그먼트와 프래그먼트 관리자

- Fragment가 인스턴스화 가 되면 `INITALIZED`상태로 시작한다. 프래그먼트가 나머지 생명 주기를 통해 전환하려면 `FragmentManager`에 더해져야한다.

- `FragmentManager`는 fragment가 어떤 상태인지를 결정하고 해당 상태로 이동 시킨다.

- `FragmentManager`는 또한 fragment를 그들의 호스트 Activity 에 연결하고 분리하는 역할도 한다.

- `onAttach()` 콜백은 `FragmentManager`에 더해졌고 호스트 Activitiy에 연결되었을때 호출된다. `FragmentManager`에서 `findFragmentById()`가 해당 fragment를 반환한다.

- `onDetach()` 콜백은 `FragmentManager`에 호스트 Activity에서 프래그먼트가 제거되었을때 불린다. `FragmentManager`에서 `findFragmentById()`가 해당 fragment를 반환하지 않는다.

## 프래그먼트 생명 주기 상태 및 콜백

- Fragment의 생명 주기를 결정할때, `FragmentManager`는 아래와 같은 것을 고려한다.

  - `FragementManager`에 의해서 fragment의 최대 상태가 결정된다. fragment는 `FragmentManager` 상태 이상으로 진행할 수 없다.

  - `FragmentTransaction`의 사용해서 `setMaxLifecycle()`를 통해 최대 fragment 생명 주기를 결정할 수 있다.

  - `Fragment`의 생명 주기는 상위보다 클 수 없다. 예를 들어 상위 Fragment 또는 Activity는 하위 Fragment보다 먼저 시작되어야 한다. 마찬가지로 자식 프레그먼트는 부모 프래그먼트 또는 Activity 보다 먼저 정지되어야 한다.

![fragment lifecycle](https://drive.google.com/uc?export=view&id=1iHHLDnn-FBsDul2ALZ5W4KgzhfhNYoGM){: width="500"}

- Fragment는 Fragment의 생명주기에 따라서 진행된다. Fragment는 위와 아래로 상태가 이동된다. 예를 들어, Fragment가 back stack의 위에 추가가되었을때 `CREATE`에서 `STARTED` 에서 `RESUMED` 상태로 이동된다. Fragment가 back stack에서 제거되었을때, `RESUMED` 에서 `STARTED` 에서 `CREATED` 로 이동되고 최종적으로 `DESTROYED`로 이동된다.

## 상향 상태 전환

- 생명 주기 상태를 통해 위쪽으로 이동할때 Fragment는 새 상태에 대해 연결된 생명 주기 콜백을 호출한다. 이 콜백이 끝나면, `Lifecycle.Event`가 Fragment의 Lifecycle에 의해 Observers에게 내보내지고, 그 다음에 인스턴스화가 되었을 경우 Fragment 뷰의 Lifecycle이 발생한다.

### Fragment CREATED

- Fragment가 `CREATED` 상태가 되었을때, 프래그먼트는 `FragmentManager`에 추가되어있고, `onAttach()` 메소드가 이미 호출되어있다.

- 여기서 프래그먼트의 `SavedStateRegistery`를 통해서 저장된 상태를 복구해야한다. 프래그먼트의 view가 아직 생성되지 않았으므로 fragment view는 생성된 다음에 복구되어야한다.

- 이 transaction에서는 `onCreate()` 콜백을 호출한다. 이 콜백은 또 한 `savedInstanceState`를 받는다. `Bundle` 인자는 이전에 `onSavedInstanceState()`에 저장되고 처음 생성되었을때는 null 값을 가진다.

### Fragment CREATED 그리고 View INITIALIZED

- 프래그먼트의 view의 `Lifecycle`는 `Fragment`의 View가 유효한 상태를 제공했을때 생성된다.

- `Fragment` 생성자로 `@LayoutId`를 넘길 경우 자동적으로 적절한 타이밍에 view를 inflate 시킨다. `onCreateView()`를 override(재정의) 해서 프로그래밍으로 fragment 의 view를 inflate를 할 수 있다.

- 만약 fragment의 view가 null 이 아닌 상태로 인스턴화 되면, fragment에 view가 세팅이 되고 getView()를 통해서 사용할 수 있다.

- `getViewLifecycleOwnerLiveData()`는 fragment view의 LifecycleOwner에서 `INITIALIZED`로 새로 업데이트 된다.

- `onViewCreated()` 생명 주기 콜백이 부르기도 한다.

- 이상태에서 view의 초기 상태를 세팅을 해야 하고, `LiveData` 인스턴스를 통해 observing 하기 시작해야한다. 또한 RecyclerView와 ViewPager2의 어댑터를 설정해줘야한다.


### Fragment 그리고 View CREATED

- Fragment의 view가 생성이된 다음에 이전 view의 상태가 복구가 되면, `Lifecycle`은 `CREATE` 상태로 변경된다.

- view 의 lifecycle owner는 observers 한테 `ON_CREATE` 이벤트를 내보낸다.

### Fragment 그리고 View STARTED

- Fragment의 view가 사용할 수 있음을 보장한다.

- 자식 fragment의 `FragmentManager`에서 FragmentTransaction을 실행하는데 안전하다.

- Fragment의 view가 null이 아닐때 Fragment의 생명주기가 `STARTED`로 바뀐 후에 바로 Fragment view의 Lifecycler가 `STARTED` 상태로 이동된다.

- Fragment가 `STARTED` 상태가되면 `onStart()` 콜백이 불린다.


### Fragment 그리고 View RESUMED

- Fragment가 뵝ㄹ때 `Animator`과 `Transition` 효과가 끝난다. 그리고 fragment는 사용자와 상호작용할 준비가 끝났다.

- Fragment의 생명주기는 `RESUME` 상태로 이동하고 `onResume()` 콜백이 불린다.

## 하향 상태 전환

- 인스턴스화된 경우 Fragment가 더 낮은 수명 주기 상태가 아래로 이동하면 관련된 `Lifecylce.Event`가 Observers에게 내보내지고 Fragment 생명 주기가 뒤따른다.

- Fragment의 생명 주기 이벤트가 발생한 후 Fragment는 연결된 생명 주기 콜백을 호출한다.

## Fragment 그리고 View STARTED

- 사용자가 Fragment에서 떠나려고 할때 Fragment가 아직 보여질때, Fragment의 `Lifecycle` 가 `STARTED`상태로 다시 이동되고 Observers에게 `ON_PAUSE`이벤트를 보낸다.

- Fragment에서 `onPause()` 콜백이 호출된다.


### Fragment 그리고 View CREATED

- Fragment가 더이상 보여지지 않을때 프래그먼트의 `Lifecycle`가 `CREATED`상태로 이동되고 Observers에게 `ON_STOP` 이벤트를 보낸다.

- 이 상태는 전환은 부모 Activity에서만 트리거되는것이 아니라 부모 Activity 또는 Fragment에서 상태를 저장할때 불리기도한다.

- `ON_STOP` 이벤트는 fragmenet의 상태가 저장되기 전에 불리는것을 보장한다. 이렇게 하면 `ON_STOP` 이벤트가 자식 FragmentManager에서 FragmentTransaction을 수행하는 것이 안전한 마지막 지점이 된다.

- API 28 전에서는 `onSaveInstanceState()`가 `onStop()`전에 불렸지만 28 이후에서는 반대로 불린다.


### Fragment CREATED 그리고 View DESTROYED

- 끝내기위한 애니메이션과 전환이 완료된 상태이다. 그리고 view가 윈도우로 부터 제거되었다.

- Fragment의 view의 `Lifecycle`은 `DESTROYED` 상태로 변경되었고 Observers에게 `ON_DESTROY`상태를 배출한다. 그다음 fragment는 `onDestroyView()` 콜백을 호출한다.

- 이 시점에 fragment의 view는 생명주기는 끝에 도달했고 `getViewLifecycleOwnerLiveData()`는 null을 반환한다.

- 이 시점에 모든 fragment의 view는 제거되어야하고, fragment의 뷰는 Garbage collected 되어야한다.

## Fragment DESTROYED

- Fragment가 제거되었거나, `FragmentManager`가 destroyed 되었을때, fragment의 `Lifecycle`은 `DESTROYED` 상태로 이동되고 Observers에게 `ON_DESTROY`상태를 알려준다.

- Framgment는 `onDestroy()`를 호출하고 이 시점에 fragment는 생명주기의 끝에 도달했다.

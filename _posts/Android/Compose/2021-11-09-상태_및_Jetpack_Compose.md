---
title: Compose - 상태 및 Jetpack Compose
author: Yun-Jae Na
date: 2021-11-09 00:31:00 +0900
categories: [Android, Compose]
tags: [Android, Compose]
image : https://drive.google.com/uc?export=view&id=1FiyACihKUpfl4_8uCh8LVRx5DfQsIv7m
---

> [https://developer.android.com/jetpack/compose](https://developer.android.com/jetpack/compose) 공식 문서를 보고 정리한 글입니다.

# 상태 및 Jetpack Compose

## 상태 및 컴포지션

- Compose는 선언적이므로 Compose를 업데이트하는 유일한 방법은 새 인수로 동일한 컴포저블을 호출하는 것이다. 이러한 인수는 UI상태를 표현한다.

- 상태가 업데이트될 때마다 재구성이 실행된다. 따라서 `TextField`와 같은 항목은 명령형 XML기반 뷰에서 처럼 자동으로 업데이트가 되지 않으므로 새 상태에 따라 업데이트 되려면 새 상태를 명시적으로 알려야 한다.

```kotlin
@Composable
fun HelloContent() {
   Column(modifier = Modifier.padding(16.dp)) {
       Text(
           text = "Hello!",
           modifier = Modifier.padding(bottom = 8.dp),
           style = MaterialTheme.typography.h5
       )
       OutlinedTextField(
           value = "",
           onValueChange = { },
           label = { Text("Name") }
       )
   }
}
```

- 위의 코드를 실행하면 아무 일도 일어나지 않는다. `TextField`가 자체적으로 업데이트되지 않기 때문이다.

- Compose에서의 컴포지션 및 리컴포지션 작동 방식 때문에 `value` 매개변수가 변경될 떄 업데이트가된다.

- 용어

  - 컴포지션 : Jetpack Compose가 컴포저블을 실행할 때 빌드한 UI에 관한 설명

  - 초기 컴포지션: 처음 컴포저블을 실행하여 컴포지션을 만든다.

  - 리컴포지션: 데이터가 변경될 때 컴포지션을 업데이트하기 위해 컴포저블을 다시 실행하는 것을 말한다.

## 컴포저블의 상태

- 구성 가능한 함수는 `remember` 컴포저블을 사용하여 메모리에 단일 객체를 저장할 수 있다.

- `remember`에 의해 계산된 값은 초기 컴포지션 중에 컴포지션에 저장되고 저장된 값을 리컴포지션 중에 반환된다.

- `remember`는 변경 가능한 객체뿐만 아니라 변경할 수 없는 객체를 저장하는 데 사용할 수 있다.

- `remember`는 객체를 컴포지션에 저장하고, `remember`는 호출한 컴포저블이 컴포지션에서 삭제되면 그 객체를 잊는다.

- `mutableStateOf`는 관찰 가능한 `MutableState<T>`를 생성하고, 런타임 시 Compose에 통합되는 관찰 가능한 유형이다.

```kotlin
interface MutableState<T> : State<T> {
    override var value: T
}
```

- `value`가 변경되면 `value`를 읽는 구성 가능한 함수의 리컴포지션이 예약이된다.

- 컴포저블에서 `MutableState`객체를 선언하는 데는 세 가지 방법이 있다.

  - val mutableState = remember { mutableStateOf(default) }

  - var value by remember { mutableStateOf(default) }

  - val (value, setValue) = remember { mutableStateOf(default) }

- by 위임을 구문에는 아래와 같이 import를 해야된다.

```kotlin
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
```

- 기억된 값은 다른 컴포저블의 매개변수로 사용하거나 구문의 로직으로 사용하여 표시할 컴포저블을 변경할 수 있다.

```kotlin
@Composable
fun HelloContent() {
   Column(modifier = Modifier.padding(16.dp)) {
       var name by remember { mutableStateOf("") }
       if (name.isNotEmpty()) {
           Text(
               text = "Hello, $name!",
               modifier = Modifier.padding(bottom = 8.dp),
               style = MaterialTheme.typography.h5
           )
       }
       OutlinedTextField(
           value = name,
           onValueChange = { name = it },
           label = { Text("Name") }
       )
   }
}
```

- `remember`가 재구성 과정 전체에서 상태를 유지하는 데 도움은 되지만 구성 변경 전반에서는 상태가 유지되지 않는다. 이 경우에는 `rememberSaveable`을 사용해야 한다.

- `rememberSaveable`은 `Bundle`에 저장할 수 있는 모든 값을 자동으로 저장한다. 다른 값의 경우에는 맞춤 `Saver`객체를 전달할 수 있다.

## 지원되는 기타 상태 유형

- Jetpack Compose에서는 상태를 보존하기 위해 `MutableState<T>` 뿐만 아니라 관찰 가능한 다른 유형도 제공을 한다.

- Jetpack Compoose에서 관찰 가능한 다른 유형을 읽으려면 상태를 `State<T>`로 변환해야 한다.

- 지원 목록

  - [LiveData](https://developer.android.com/reference/kotlin/androidx/compose/runtime/livedata/package-summary)

  - [Flow](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#(kotlinx.coroutines.flow.StateFlow).collectAsState(kotlin.coroutines.CoroutineContext))

  - [RxJava2](https://developer.android.com/reference/kotlin/androidx/compose/runtime/rxjava2/package-summary)

## 스테이트풀(Stateful)과 스테이트리스(Stateless)

- `remember`를 사용하여 객체를 저장하는 컴포저블은 내부 상태를 생성하여 컴포저블을 스테이트풀(Stateful)로 만든다.

- 내부 상태를 갖는 컴포저블은 재사용 가능성이 저곡 테스트하기가 더 어려운 경향이 있다.

- 재사용 가능한 컴포저블을 개발할 때는 동일한 컴포저블의 스테이트풀(Stateful) 버전과 스테이트리스(Stateless) 버전을 모두 노출해야 하는 경우가 있다. 스테이트풀(Stateful) 버전은 상태를 염두에 두지 않는 호출자에 편리하며, 스테이트리스(Stateless) 버전은 상태를 제어하거나 끌어올려야 하는 호출자에 필요하다.


## 상태 호이스팅

- Compose에서 상태 끌어올리기는 컴포저블을 스테이트리스(Stateless)로 만들기 위해 상태를 컴포저블의 호출자로 옮기는 패턴이다.

- Jetpack Composeㅇ에서 상태를 끌어올리기 위한 일반적 패턴은 상태 변수를 다음 두 개의 매개변수로 바꾸는 것이다.

 - `value: T`: 표시할 현재 값

 - `onValueChange: (T) -> Unit : T`가 제안된 새 값인 경우 값을 변경하도록 요청하는 이벤트

- 위와 같은 경우 뿐만이 아니라 컴포저블의 더 특정한 이벤트가 어울리는 경우 ex) `ExpandingCard`가 `onExpand`와 `onCollapse`를 정의할 때와 같이 람다를 사용하여 그 이벤트를 정의해야 한다.

- 끓어올린(호이스팅) 상태에는 중요한 속성이 몇 가지 있다.

  - *단일 소스 저장* : 상태를 복제하는 대신 옮겼기 때문에 소스 저장소가 하나만 있다. 버그 방지에 도움이 된다.

  - *캡슐화됨*: 스테이트풀(Stateful) 컴포저블만 상태를 수정할 수 있다. 철저히 내부적 속성이다.

  - *공유 가능함*: 호이스팅한 상태를 여러 컴포저블과 공유할 수 있다. 다른 컴포저블에서 name을 사용하려는 경우 호이스팅을 통해 가능하다.

  - *분리됨*: 스테이트리스(Stateless) `ExpandingCard`의 상태는 어디에나 저장할 수 있다. 예를 들어 `name`을 `ViewModel`로 옮길 수 있다.

```kotlin
/**
* HelloContent 에서 name과 onValueChange를 추출한 다음, 이러한 항목을 트리 상단을 거쳐 HelloContent를 호출하는 HelloScreen 컴포저블로 옮긴다.
*/
@Composable
fun HelloScreen() {
    var name by rememberSaveable { mutableStateOf("") }

    HelloContent(name = name, onNameChange = { name = it })
}

@Composable
fun HelloContent(name: String, onNameChange: (String) -> Unit) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "Hello, $name",
            modifier = Modifier.padding(bottom = 8.dp),
            style = MaterialTheme.typography.h5
        )
        OutlinedTextField(
            value = name,
            onValueChange = onNameChange,
            label = { Text("Name") }
        )
    }
}
```

- 위에 코드와 같이 `HelloContent`에서 상태를 끌어오리면 더 쉽게 컴포저블을 추론하고 여러 상황에서 재사용하며 테스트 할 수 있다.

- `HelloContent`는 상태의 저장 방식과 분리된다. 분리된다는 것은 `HelloScreen`을 수정하거나 교체할 경우 `HelloContent`의 구현 방식을 변경할 필요가 없다는 의미다.

- 상택3ㅏ 내려가고 이벤트가 올라가는 패턴을 `단방향 데이터 흐름이라고 한다.` 이 경우 상태는 `HelloScreen`에서 `HelloContent`로 내려가고 이벤트는 `HelloContent`애서 `HelloScreen`으로 올라간다.

- 단뱡향 데이터 흐름을 따르면 UI에 상태를 표시하는 컴포저블과 상태를 저장하고 변경하는 앱 부분을 서로 분리할 수 있다.


## Compose에서 상태 복원

- 활동 또는 프로세스가 다시 생성된 이후 `rememberSaveable`을 사용하여 UI 상태를 복원한다.

- `rememberSaveable`은 재구성 과정 전체에서 상태를 유지한다.

- `rememberSaveable`은 Activity 및 프로세스 재생성 전반에 걸쳐 상태를 유지한다.

### 상태를 저장하는 방법

- `Bundle`에 추가되는 모든 데이터 유형은 자동으로 저장된다.

- `Bundle`에 추가할 수 없는 항목을 저장하려는 경우 몇 가지 옵션이 있다.

#### Parcelize

- 가장 간단한 방법은 객체에 `@Parcelize` 주석을 추가하는 것이다. 그러면 객체가 `parcelable`이 되며 번들로 제공될 수 있다.

```kotlin
@Parcelize
data class City(val name: String, val country: String) : Parcelable

@Composable
fun CityScreen() {
    var selectedCity = rememberSaveable {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```

#### MapSaver

어떤 이유로 `@Parcelize`가 적합하지 않은 경우 `mapSaver`를 사용하여 시스템이 `Bundle`에 저장할 수 있는 값 집합으로 객체를 변환하는 고유한 규칙을 정의할 수 있다.

```kotlin
data class City(val name: String, val country: String)

val CitySaver = run {
    val nameKey = "Name"
    val countryKey = "Country"
    mapSaver(
        save = { mapOf(nameKey to it.name, countryKey to it.country) },
        restore = { City(it[nameKey] as String, it[countryKey] as String) }
    )
}

@Composable
fun CityScreen() {
    var selectedCity = rememberSaveable(stateSaver = CitySaver) {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```

#### ListSaver

- `listSaver`를 사용하고 색인을 키로 사용하면 맵의 키를 정의할 필요가 없다.

```kotlin
data class City(val name: String, val country: String)

val CitySaver = listSaver<City, Any>(
    save = { listOf(it.name, it.country) },
    restore = { City(it[0] as String, it[1] as String) }
)

@Composable
fun CityScreen() {
    var selectedCity = rememberSaveable(stateSaver = CitySaver) {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```

## Compose에서 상태 관리

- 간단한 상태 끌어올리기는 구성 가능한 함수 자체에서 관리 가능하지만 추적할 상태의 양이 늘어나는 경우나 구성 가능한 함수에서 실행할 로직이 발생하는 경우 로직과 상태 책임을 다른 클래스, 즉 `상태 홀더`에 위임하는 것이 좋다.

- 용어

  - `Composable(컴포저블)` : 간단한 UI 요소 상태 관리 목적.

  - `State holders(상태 홀더)` : 복잡한 UI 요소 상태 관리 목적, 상태 홀더는 UI 요소의 상태와 UI 로직을 소유한다.

  - `Architecture Components ViewModels(아키텍처 구성요소 ViewModel)` - 비즈니스 로직 및 화면 상태나 UI 상태에 대한 액세스 권한을 제공하는 특수한 유형의 상태 홀더이다.

- 상태 홀더는 하단 앱 바와 같은 단일 위젯을 포함해 전체 화면 등 관리하는 관련 UI 요소의 범위에 따라 다양한 크기로 제공된며, `상태 홀더는 혼합 가능`하다. 즉, 상태 홀더는 상태를 집계할 때 다른 상태 홀더에 통합할 수 있다.

- 컴포저블은 복잡성에 따라 0개 이상의 상태 홀더(일반 객체 또는 ViewModel이거나 둘 다일 수 있음)를 사용할 수 있다.

- 일반 상태 홀더는 비즈니스 로직이나 화면 상태에 액세스해야 하는 경우 ViewModel을 사용할 수 있다.

- ViewModel은 비즈니스 레이어 또는 데이터 영역을 사용한다.


### 상태 및 로직 유형

Android 앱에는 고려할 여러 유형의 상태가 있다.

- `Ui elemenet state(UI 요소 상태)` : UI 요소를 호이스팅한 상태이다. 예를들어 `ScaffoldState`는 `Scaffold` 컴포저블의 상태를 처리한다.

- `Screen or UI state(화면 상태 또는 UI 상태)` : 화면에 표시되어야 하는 요소이다. 예를들어, 장바구니 항목, 사용자에게 표시할 메시지 또는 로딩 플래그를 포함하는 `CartUiState`가 있다. 이 상태는 애플리케이션 데이터를 포함하므로 대개 계층 구조의 다른 레이어에 연결된다.

- `UI behavior logic or UI logic(UI 동작 로직 또는 UI 로직)` : 화면에 상태 변경을 표시하는 방법과 관련되어있다. 예를 들어 탐색 로직은 다음에 표시할 화면을 결정하고, UI 로직은 스낵바 또는 토스트 메시지를 사용할 수 있는 화면에 사용자 메시지를 표시하는 방법을 결정한다. UI 동작 로직은 항상 컴포지션 내에 있어야 한다.

- `Business logic(비즈니스 로직)` : 상태 변경에 따라 진행해야 할 작업이다. 결제 또는 사용자 환경설정 저장 등이 에이다. 이 로직은 대개 비즈니스 레이어나 데이터 영역에 배치되고 UI 레이어에는 배치되지 않는다.

### 정보 소스로서의 컴포저블

- 상태와 로직이 간단하면 컴포저블에 UI 로직과 UI 요소 상태를 사용하는 것이 좋다. 예를 들어 `ScaffoldState` 및 `CoroutineScope`를 처리하는 `MyApp` 컴포저블이다.

```kotlin
@Composable
fun MyApp() {
    MyTheme {
        val scaffoldState = rememberScaffoldState()
        val coroutineScope = rememberCoroutineScope()

        Scaffold(scaffoldState = scaffoldState) {
            MyContent(
                showSnackbar = { message ->
                    coroutineScope.launch {
                        scaffoldState.snackbarHostState.showSnackbar(message)
                    }
                }
            )
        }
    }
}
```

- `ScaffoldState`에 변경 가능한 속성이 포함되므로 이 컴포저블과의 모든 상호작용은 `MyApp` 컴포저블에서 발생해야한다.

- 다른 컴포저블에 전달되면 그 컴포저블이 상태를 변경할 수 있으므로 단일 정보 소스 원칙에 위배되고 버그 추적이 어려워진다.

### 정보 소스로서의 상태 홀더

- 여러 UI 요소의 상태가 관련되어 있는 복잡한 UI 로직이 포함된 컴포저블은 책임을 상태 홀더에 위임해야 한다. 그러면 로직을 더 쉽게 격리하여 테스트할 수 있으며 컴포저블의 복잡성이 줄어든다.

- 이러한 접근 방식은 관심사 분리 원칙을 따르다. 즉, `컴포저블이 UI 요소를 방출하고 상태 홀더가 UI로직과 UI 요소의 상태를 포함한다.`

- 상태 홀더는 컴포지션에서 생성되고 기억되는 일반 클래스이다. `컴포저블 수명 주기`를 따르므로 Compose 종속 항목을 사용할 수 있다.

- `정보 소스로서의 컴포저블` 색션에서 `MyApp` 컴포저블의 책임이 증가하면 `MyAppState` 상태 홀더를 만들어 복잡성을 관리할 수 있다.

```kotlin
// Plain class that manages App's UI logic and UI elements' state
class MyAppState(
    val scaffoldState: ScaffoldState,
    val navController: NavHostController,
    private val resources: Resources,
    /* ... */
) {
    val bottomBarTabs = /* State */

    // Logic to decide when to show the bottom bar
    val shouldShowBottomBar: Boolean
        @Composable get() = /* ... */

    // Navigation logic, which is a type of UI logic
    fun navigateToBottomBarRoute(route: String) { /* ... */ }

    // Show snackbar using Resources
    fun showSnackbar(message: String) { /* ... */ }
}

@Composable
fun rememberMyAppState(
    scaffoldState: ScaffoldState = rememberScaffoldState(),
    navController: NavHostController = rememberNavController(),
    resources: Resources = LocalContext.current.resources,
    /* ... */
) = remember(scaffoldState, navController, resources, /* ... */) {
    MyAppState(scaffoldState, navController, resources, /* ... */)
}
```

- `MyAppState`가 종속 항목을 취하므로 컴포지션 내 `MyAppState` 인스턴스를 기억하는 메서드를 제공하는 것이 좋다. 위의 경우에는 `rememberMyAppState` 함수이다.

- `MyApp`은 UI 요소를 방출하는 데 집중하고 모든 UI 로직과 UI 요소의 상태를 MyAppState에 위임한다.

```kotlin
@Composable
fun MyApp() {
    MyTheme {
        val myAppState = rememberMyAppState()
        Scaffold(
            scaffoldState = myAppState.scaffoldState,
            bottomBar = {
                if (myAppState.shouldShowBottomBar) {
                    BottomBar(
                        tabs = myAppState.bottomBarTabs,
                        navigateToRoute = {
                            myAppState.navigateToBottomBarRoute(it)
                        }
                    )
                }
            }
        ) {
            NavHost(navController = myAppState.navController, "initial") { /* ... */ }
        }
    }
}
```

- `컴포저블의 책임을 늘리면 상태 홀더의 필요성이 증가`한다.

- 책임은 UI로직이거나, 단순히 추적할 상태의 양일 수 있다.

### 정보 소스로서의 ViewModel

- 일반 상태 홀더 클래스가 UI로직과 UI 요소의 상태를 담당하는 경우 `ViewModel은 특별한 유형의 상태 홀더`로서 다음 작업을 진행한다.

  - 비즈니스 레이어와 데이터 영역 같이 대개 계층 구조의 다른 레이어에 배치되는 애플리케이션의 비즈니스 로직에 대한 액세스 권한 제공

  - 특정 화면에 표시하기 위한 애플리케이션 데이터 준비(화면 상태 또는 UI 상태가 됨)

- `ViewModel은 컴포지션보다 수명이 더 길다.` 구성 변경 후에도 그대로 유지되기 떄문이다.

- ViewModel은 Compose 콘텐츠(Activity 또는 Fragment)의 호스트 생명 주기나 대상 새명 주기 또는 탐색 그래프의 생명 주기를 타를 수 있다.

- ViewModel은 수명이 길기 때문에 컴포지션의 생명주기에 바인딩된 상태에 관한 장기 지속적인 참조를하면 안된다. 메모리 누수가 발생할 수 있다.

- `화면 수준 컴포저블`에서 ViewModel을 사용하여 UI 상태의 정보 소스로 비즈니스 로직에 대한 액세스 권한을 제공하는 것이 좋다.

```kotlin
data class ExampleUiState(
    dataToDisplayOnScreen: List<Example> = emptyList(),
    userMessages: List<Message> = emptyList(),
    loading: Boolean = false
)

class ExampleViewModel(
    private val repository: MyRepository,
    private val savedState: SavedStateHandle
) : ViewModel() {

    var uiState by mutableStateOf<ExampleUiState>(...)
        private set

    // Business logic
    fun somethingRelatedToBusinessLogic() { ... }
}

@Composable
fun ExampleScreen(viewModel: ExampleViewModel = viewModel()) {

    val uiState = viewModel.uiState
    ...

    Button(onClick = { viewModel.somethingRelatedToBusinessLogic() }) {
        Text("Do something")
    }
}
```

### ViewModel 및 상태 홀더

- Android 개발에서 `ViewModel이 가진 이점` 덕분에, 비즈니스 로직에 대한 액세스 권한을 제공하고 화면에 표시하기 위한 애플리케이션 데이터를 준비하는데 ViewModel이 적합하다.

  - ViewModel에 의해 트리거된 작업이 구성 변경에도 그대로 유지된다.

  - Navigation과의 통합

    - 화면이 백 스택에 있는 동안 Navigation이 ViewModel을 캐싱한다.

    - 개발자가 대상으로 돌아갈 때 이전에 로드한 데이터를 즉시 사용할 수 있도록 하는 것이 중요하다. 이 작업은 컴포저블 화면의 수명 주기를 따르는 상태 홀더를 사용할 경우 어려워진다.

    - 백 스택에서 사라질 때 ViewModel도 삭제되기 때문에 상태가 자동으로 정리된다. 구성 변경으로 인한 새 화면으로의 이동 등 여러 이유로 발생할 수 있는 컴포저블 폐기에 대한 수신 대기와는 다르다.

    - Hilt와 같은 다른 Jetpack 라이브러리와의 통합을 할 수 있다.

- 상태 홀더가 혼합 가능하고 ViewModel과 일반 상태 홀더가 서로 다른 책임을 담당하므로, 비즈니스 로직에 대한 액세스 권한을 제공하는 ViewModel과 UI 로직과 UI요소의 상태를 관리하는 상태 홀더를 `화면 수준의 컴포저블에 모두 포함`할 수 있다.

- ViewModel이 상태 홀더보다 수명이 길기 때문에, 필요한 경우 상태 홀더는 ViewModel을 종속 항목으로 사용할 수 있다.

```kotlin
// ExampleScreen에서 함께 작동하는 ViewModel과 일반 상태 홀더
private class ExampleState(
    val lazyListState: LazyListState,
    private val resources: Resources,
    private val expandedItems: List<Item> = emptyList()
) { ... }

@Composable
private fun rememberExampleState(...) { ... }

@Composable
fun ExampleScreen(viewModel: ExampleViewModel = viewModel()) {

    val uiState = viewModel.uiState
    val exampleState = rememberExampleState()

    LazyColumn(state = exampleState.lazyListState) {
        items(uiState.dataToDisplayOnScreen) { item ->
            if (exampleState.isExpandedItem(item) {
                ...
            }
            ...
        }
    }
}
```

---
title: Compose - Compose의 이해
author: Yun-Jae Na
date: 2021-11-06 01:14:00 +0900
categories: [Android, Compose]
tags: [Android, Compose]
image : https://drive.google.com/uc?export=view&id=1FiyACihKUpfl4_8uCh8LVRx5DfQsIv7m
---

> [https://developer.android.com/jetpack/compose](https://developer.android.com/jetpack/compose) 공식 문서를 보고 정리한 글입니다.

## Compose 이해

- Jetpack Compose는 Android를 위한 현대적인 선언형 UI 도구 키트이다.
- Compose는 프런트엔드 뷰를 명령형으로 변형하지 않고도 앱 UI를 렌더링할 수 있게 하는 선형형 API를 제공하여 앱 UI를 더 쉽게 작성하고 유지관리할 수 있도록 지원한다.

## 선언형 프로그래밍 패러다임

### 기존 방식

- 기존 Android 뷰 계층 구조는 UI 위젯의 트리 형태
- 사용자 상호작용 등의 이유로 인해 앱의 상태가 변경되면, 현재 데이터를 표시하기 위해 UI 계층 구조를 업데이트해야 한다. (findViewById() 로 탐색, setText, addChild 등)
- 뷰를 수동으로 조작하면 오류가 발생할 가능성이 커진다. 데이터를 여러 위치에서 렌더링한다면 데이터를 표시하는 뷰 중 하나를 업데이트 하는 것을 잊기 쉽다. 또한 두 업데이트가 예기치 않은 방식으로 충돌할 경우 잘못된 상태를 야기하기도 쉽다.

### 최근 방식

- 업계 전반에서 선언형 UI 모델로 전환하기 시작했으며, 이에 따라 사용자 인터페이스 빌드 및 업데이트와 관련된 엔지니어링이 크게 간소화됨
- 처음부터 화면 전체를 개념적으로 재생성한 후 필요한 변경사항만 적용하는 방식으로 작동함.
- Stateful 뷰 계층 구조를 수동으로 업데이트할 떄의 복잡성을 방지할 수 있다.
- Compose는 선언형 UI 프레임워크이며 특정 시점에 UI의 어떤 부분을 다시 그려야 하는지를 지능적으로 선택한다. (재구성)

## 간단한 구성 가능한 함수

- Compose를 사용하면 데이터를 받아서 UI 요소를 내보내는 구성 가능한 함수 집합을 정의하여 사용자 인터페이스를 빌드 할 수 있다.

```kotlin
// 데이터를 전달받고 이를 사용하여 화면에 텍스트 위젯을 렌더링하는 간단한 구성 가능한 함수
@Composable
fun Greeting(name: String) {
    Text("Hello $name")
}
```

- 함수는 @Composable 주석으로 주석이 지정된다. 모든 구성 가능한 함수에는 이 주석이 있어야 하며, 데이터를 UI로 변환하기 위한 함수라는 것을 Compose 컴파일러에 알린다.
- 함수는 데이터를 받으며 매개변수를 통해 앱 로직이 UI를 형성할 수 있다.
- 함수는 UI에 텍스트를 표시하며, 텍스트 UI 요소인 `Text()` 를 호출한다.
- 구성 가능한 함수는 다른 구성 가능한 함수를 호출하여 UI 곛으 구조를 내보낸다.
- 이 함수는 빠르며 `멱등원(연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질)`이며 부작용이 없다.

## 선언형 패러다임 전환

- Compose의 선언형 접근 방식에서 위젯은 비교적 스테이리스(Stateless) 상태이며 setter 또는 getter 함수를 노출하지 않는다. (객체로 노출되지 않는다.)
- 동일한 구성 가능한 함수를 다른 인수로 호출하여 UI를 업데이트한다.
- `ViewModel`과 같은 아키텍처 패턴에 상태를 쉽게 제공할 수 있다.
- 컴포저블은 식별 가능한 데이터가 업데이트될 때마다 현재 애플리케이션 상태를 UI로 변환한다.
- 앱 로직은 최상위 구성 가능한 함수에 데이터를 제공한다 그러면 함수는 데이터를 사용하여 다른 컴포저블을 호출함으로써 UI를 형성하고 적절한 데이터를 해당 컴포저블 및 계층 구조 아래로 전달한다.
- 사용자가 UI와 상호작용할 때 UI는 `onClick`과 같은 이벤트를 발생시키며 이러한 이벤트를 앱 로직에 전달하여 앱의 상태를 변경해야 한다. 상태가 변경되면 구성 가능한 함수는 새 데이터와 함꼐 다시 호출되며 이렇게 하면 UI요소가 다시 그려지며 이 프로세스를 `재구성`이라고 한다.

## 동적 컨텐츠

구성 가능한 함수는 XML이 아닌 Kotlin으로 작성되기 때문에 동적일 수 있다.

```kotlin
@Composable
fun Greeting(names: List<String>) {
    for(name in names){
        Text("Hello $name")
    }
}
```

- `if`문을 사용하여 특정 UI 요소를 표시할지 여부를 결정할 수 있으며 루프를 사용할 수도 있다.


## 재구성

- 명령형 UI모델에서 위젯을 변경하려면 위젯에서 setter를 호출하여 내부 상태를 변경하지만 Compose에서는 새 데이터를 사용하여 구성 가능한 함수를 다시 호출한다. 이렇게 하면 함수가 `재구성`되며, 필요한 경우 함수에서 내보낸 위젯이 새 데이터로 다시 그려진다. (변경된 부분만 재구성됨)

```kotlin
@Composable
fun ClickCounter(clicks: Int, onClick: () -> Unit) {
    Button(onClick = onClick) {
        Text("I've been clicked $clicks times")
    }
}
```

- 버튼이 클릭될 때마다 clicks 값을 업데이트한다.
- Compose는 `Text` 함수를 사용해 람다를 다시 호출하여 새 값을 표시한다. 이 프로세를 `재구성`이라고 하며 값에 종속되지 않은 다른 함수는 재구성되지 않는다.
- 재구성은 입력이 변경될 때 구성 가능한 함수를 다시 호출하는 프로세스이다. 합수 입력이 변경될 때 발생하며, Compose는 새 입력을 기반으로 재구성할 때 변경되어썌을 수 있는 함수 또는 람다만 호출하고 나머지는 건너뛴다.
- 함수의 재구성을 건너 뛸 수 있으므로 구성 가능한 함수 실행의 부작용에 의존해서는 안된다.
  - 공유 객체의 속성에 쓰기
  - `ViewModel`에서 식별 가능한 요소 업데이트 (Updating an observable in ViewModel)
  - Shared preferences 업데이트 하기
- 구성 가능한 함수는 애니메이션이 렌더링될 때와 같이 모든 프레임에서와 같은 빈도로 재실행될 수 있다. 애니메이션 도중 버벅거림을 방지하려면 구성 가능한 함수가 빨라야 하며 Composable은 Shared preferences에서 읽거나 쓰지 않아야 한다. 대신 백그라운드 코루틴의 `ViewModel`로 읽기 및 쓰기를 이동한다.

```kotlin
@Composable
fun SharedPrefsToggle(
    text: String,
    value: Boolean,
    onValueChanged: (Boolean) -> Unit
) {
    Row {
        Text(text)
        Checkbox(checked = value, onCheckedChange = onValueChanged)
    }
}
```

- 구성 가능한 함수는 순서와 관계없이 실행할 수 있다.
- 구성 가능한 함수는 동시에 실행할 수 있다.
- 재구성은 최대한 많은 수의 구성 가능한 함수 및 람다를 건너뛴다.
- 재구성은 낙관적이며 취소될 수 있다.
- 구성 가능한 함수는 애니메이션의 모든 프레임에서와 같은 빈도로 매우 자주 실행될 수 있다.

## 구성 가능한 함수는 순서와 관계없이 실행할 수 있음

- 구성 가능한 함수에 다른 구성 가능한 함수 호출이 포함되어 있으며 함수는 순서와 관계없이 실행될 수 있다.
- Compose에서는 일부 UI요소가 다른 UI요소보다 우선순위가 높다는 것을 인식하고 그 요소를 먼저 그리는 옵셥이 있다.

```kotlin
@Composable
fun ButtonRow() {
    MyFancyNavigation {
        StartScreen()
        MiddleScreen()
        EndScreen()
    }
}
```

- `StartScreen`, `MiddleScreen`, `EndScreen` 호출은 순서와 관계없이 발생할 수 있다.
- 각 함수는 독립적이어야 한다.

## 구성 가능한 함수는 동시에 실행할 수 있음

- Compose는 구성 가능한 함수를 동시에 실행하여 재구성을 최적화할 수 있다. 이를 통해 Compose는 다중 코어를 활용하고 화면에 없는 구성 가능한 함수를 낮은 우선순위로 실행할 수 있다.
- 최적화는 구성 가능한 함수가 백그라운드 스레드 풀 내에서 실행될 수 있다.
- 애플리케이션이 올바르게 작동하려면 모든 구성 가능한 함수에 부작용이 없어야 한다. 대신 UI 스레드에서 항상 실행되는 `onClick`과 같은 콜백에서 부작용을 트리거한다.
- 구성 가능한 함수가 호출될 때 호출자와 다른 스레드에서 호출이 발생할 수 있다. 즉, 구성 가능한 람다의 변수를 수정하는 코드는 피해야 한다. 이러한 코드는 스레드로부터 안전하지 않으며 구성 가능한 람다의 허용되지 않는 부작용이기 때문이다.

```kotlin
// 부작용이 없는 코드
@Composable
fun ListComposable(myList: List<String>) {
    Row(horizontalArrangement = Arrangement.SpaceBetween) {
        Column {
            for (item in myList) {
                Text("Item: $item")
            }
        }
        Text("Count: ${myList.size}")
    }
}
```

```kotlin
// 로컬 변수에 쓰는 경우 스레드부터 안전하지 않거나 적절하지 않다.
@Composable
@Deprecated("Example with bug")
fun ListWithBug(myList: List<String>) {
    var items = 0

    Row(horizontalArrangement = Arrangement.SpaceBetween) {
        Column {
            for (item in myList) {
                Text("Item: $item")
                items++ // Avoid! Side-effect of the column recomposing.
            }
        }
        Text("Count: $items")
    }
}
```

- 위의 예제에서 `items`는 모든 재구성을 통해 수정된다.
- 수정은 애니메이션의 모든 프레임에서 또는 목록이 업데이트될때 실행될 수 있다.
- 어느 쪽이든 UI에 잘못된 개수가 표시가된다. `이 때문에 이와 같은 쓰기는 Compose에서 지원되지 않는다`.
- 이러한 쓰기를 금지함으로써 프레임워크가 구성 가능한 람다를 실행하도록 스레드를 변경할 수 있다.

## 재구성은 가능한 한 많이 건너뜀

- UI의 일부가 잘못된 경우 Compose는 업데이트해야 하는 부분만 재구성하기 위해 최선을 다한다.
- UI 트리에서 위 또는 아리에있는 컴포저블을 실행하지 않고 단일 버튼의 컴포저블을 다시 실행하는 것을 건너뛸 수 있다.

```kotlin
/**
 * 이름 리스트를 보여주고 헤더를 클릭할 수 있다.
 */
@Composable
fun NamePicker(
    header: String,
    names: List<String>,
    onNameClicked: (String) -> Unit
) {
    Column {
        // [names]가 변경할 때가 아닌 [header]가 변경할때 재구성 한다.
        Text(header, style = MaterialTheme.typography.h5)
        Divider()

        // LazyColumn은 RecyclerView의 Compose버전이다.
        // items()는 RecyclerView.ViewHolder의 역할과 비슷하다.
        LazyColumn {
            items(names) { name ->
                // 아이템의 [name] 이 업데이트 될때 어댑터의 아이템이 재구성된다.
                // [header] 변경될때는 재구성하지 않는다.
                NamePickerItem(name, onNameClicked)
            }
        }
    }
}

/**
 * 단일 이름을 보여주고 사용자는 클릭할 수 있다.
 */
@Composable
private fun NamePickerItem(name: String, onClicked: (String) -> Unit) {
    Text(name, Modifier.clickable(onClick = { onClicked(name) }))
}
```

## 재구성은 낙관적임

- Compose가 컴포저블의 매개변수가 변경되었을 수 있다고 생각할 때마다 재구성이 시작된다. 재구성은 `낙관적`이다. 즉, Compose는 매개변수가 다시 변경되기 전에 재구성을 완료할 것으로 예상한다.
- 재구성이 완료되기 전에 매개변수가 변경되면 Compose는 재구성에서 UI 트리를 삭제한다. 표시되는 UI에 종속되는 부작용이 있다면 구성이 취소된 경우에도 부작용이 적용된다. 이로 인해 일관되지 않은 앱 상태가 발생할 수 있다.
- 낙관적 재구성을 처리할 수 있도록 모든 구성 가능한 함수 및 람다가 멱등원이고 부작용이 없는지 확인해야 한다.

## 구성 가능한 함수는 매우 자주 실행될 수 있음

- 경우에 따라 구성 가능한 함수는 UI 애니메이션의 모든 프레임에서 실행될 수 있다.
- 함수가 기기 저장소에서 읽기와 같이 비용이 많이 드는 작업을 실행하면 이 함수로 인해 UI 버벅거림이 발생할 수 있다.
- 예를 들어 위젯이 기기 설정을 읽으려고 하면 잠재적으로 이 설정을 초당 수백 번 읽을 수 있으며 앱 성능에 치명적이다.
- 구성 가능한 함수에 데이터가 필요하다면 데이터의 매개변수를 정의해야 하며 비용이 많이 드는 작업을 구성 외부의 다른 스레드로 이동하고 mutableStateOf 또는 LiveData를 사용하여 Compose에 데이터를 전달할 수 있다.

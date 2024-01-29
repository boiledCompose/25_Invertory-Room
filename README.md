## RoomDB에서 데이터 읽기 및 업데이트

### Flow

`Flow`는 일반 데이터스트림을 나타내는 유형으로, 주로 데이터베이스의 항목을 가져올 때 DAO에서 반환 유형으로 많이 사용된다.   
`Flow`를 반환하면 지정된 수명 주기 동안 DAO에서 메서드를 한 번만 명시적으로 호출하면 된다.

흐름(`Flow`)에서 데이터를 가져오는 것을 흐름에서 수집이라고 한다. UI 레이어의 흐름에서 수집할 때 고려해야 할 사항이 존재한다: 
- 구성 변경과 같은 수명 주기 이벤트로 인해 액티비티가 다시 생성되고 `Flow`에서 재구성 및 수집이 다시 이루어진다.
- 이런 이벤트들로 인해 기존 데이터가 손실되지 않도록 값을 상태로 캐시해야 한다.
- `Flow`는 컴포저블의 수명 주기가 종료된 후와 같이 남은 관찰자가 없는 경우 취소해야 한다.

`ViewModel`에서 UI 레이어에 `Flow`를 노출할 때 권장되는 방법은 `StateFlow`를 사용하는 것이다. `StateFlow`를 사용하면 UI 수명 주기와 관계없이 데이터를 저장하고 관찰할 수 있다. `Flow`를 `StateFlow`로 변환하려면 `stateIn`연산자를 사용한다.

### stateIn

`stateIn` 연산자에는 세 가지 매개변수가 있다.

1. `scope`: `viewModelScope`가 `StateFlow`의 수명 주기를 정의한다. `viewModelScope`가 취소되면 `StateFlow`도 취소된다.
2. `started`: 파이프라인은 UI가 표시되는 경우만 활성화되야 함으로 `SharingStarted.WhileSubscribed()`를 사용한다. 마지막 사용자의 사라짐과 공유 코루틴 중지 사이의 지연을 구성하려면 해당 메서드에 `TIMEOUT_MILLIS`를 전달한다.
3. `initialValue`: 상태 흐름의 초깃값을 `HomeUiState()`로 설정한다.

```kotlin
//예제 코드
val homeUiState: StateFlow<HomeUiState> =
    itemsRepository.getAllItemsStream().map { HomeUiState(it) }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(TIMEOUT_MILLIS),
            initialValue = HomeUiState()
        )
```

### StateFlow

`StateFlow`를 사용하면 UI 수명 주기와 관계없이 데이터를 저장하고 관찰할 수 있다. 

이 `StateFlow`에서 값을 수집하고 최신 값을 `State`를 통해 나타내는 방법은 `collectAsState()`를 사용하는 것이다.

```kotlin
//예제 코드
val homeUiState by viewModel.homeUiState.collectAsState()
```

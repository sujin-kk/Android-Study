# 02-Coroutine Flow

[flow-공식문서](https://developer.android.com/kotlin/flow?hl=ko)


### 목차

- [Coroutine의 Flow](#coroutine의-flow)

- [데이터 스트림의 구성요소](#데이터-스트림의-구성요소)

- [Flow의 예외처리](#flow의-예외처리)

- [다른 Courtine Context](#다른-courtine-context)

- [LiveData와 Flow의 차이](#livedata와-flow의-차이)

- [Flow와 StateFlow 차이](#flow와-stateflow-차이)

- [StateFlow와 SharedFlow 차이](#stateflow와-sharedflow-차이)

- [collect와 colletLatest 차이](#collect와-colletlatest-차이)

- [Hot Stream과 Cold Stream](#hot-stream과-cold-stream)

---

### Coroutine의 Flow

- 데이터 스트림 (Data Stream)
- 코루틴 상에서 리액티브 프로그래밍을 지원하기 위한 구성요소

- **Reactive Programming**
    - 데이터가 변경될 때 이벤트를 발생시켜서 데이터를 계속해서 전달하도록 하는 프로그래밍 방식
    - ↔ 명령형 프로그래밍과 대응되는 개념
        
        [[RxJava] 반응형 프로그래밍 이해하기](https://kotlinworld.com/125)
        
    - `기존의 명령형 프로그래밍` : 데이터가 필요한 Consumer는 데이터를 요청한 후 받은 결과값을 일회성으로 수신 → 결론적으로 데이터가 필요할 때마다 결과값을 매번 요청해야 한다는 비효율성 존재
    - `리액티브 프로그래밍` : 데이터를 발행하는 Producer가 있고, 해당 Producer는 데이터가 필요한 Consumer가 Producer에게 구독 요청(Subscribe)을 함
    - Producer는 새로운 데이터가 들어오면 Consumer에게 지속적으로 데이터를 발행함
    - 즉, Producer가 Consumer에게 **지속적으로 데이터를 전달**하는데, 이것을 **Data Stream**이라고 함

- Flow를 이용한 리액티브 프로그래밍
    > 코루틴에서 데이터 스트림을 구현하기 위해서는 Flow를 사용해야 함
    

---

### 데이터 스트림의 구성요소
`Producer` (생산자, 발행자)
     
`Intermediary` (중간 연산자)
     
`Consumer` (소비자)

*→ flow의 세가지 핵심 구성요소*

<img width="734" alt="Untitled" src="https://user-images.githubusercontent.com/85485290/185615903-85aa1f83-7f5c-46c5-a371-b83a08774c95.png">


#### Producer

<img width="644" alt="Untitled 2" src="https://user-images.githubusercontent.com/85485290/185615834-7b5cb167-7210-4052-8bd6-96c510e7d56d.png">


- 프로듀서는 **데이터를 발행(생성)하는 역할**을 함
- flow에서 프로듀서는 `flow{ }` 블록 내부에서 `emit()`을 통해 데이터를 생성한다
- 안드로이드에서 프로듀서가 가져오는 데이터의 대표적인 DataSource는 두가지
    1. 서버의 데이터 = REST API (Remote DataSource)를 이용해 가져오는 데이터
    2. 기기의 DB (Local DataSource)에서 가져오는 데이터
    
- ex) 고정된 간격으로 최신 뉴스를 자동으로 가져오는 과정
- 정지 함수는 연속된 값을 여러 개 반환할 수 없으므로, 데이터 소스가 이러한 요구사항을 충족하는 흐름을 만들고 반환합니다. 이 경우 데이터 소스가 생산자의 역할을 합니다.
    1. `flow { }` 블록을 선언한다.
    2. 뉴스 데이터를 서버로부터 받아온다. (Remote DataSource, newsApi)
    3. Producer가 데이터를 생성한다 (`emit`)
    4. 2~3 과정을 5초마다 반복하여 데이터를 계속 생성한다 (5초마다 최신 뉴스로 업데이트)

```kotlin
class NewsRemoteDataSource(
    private val newsApi: NewsApi,
    private val refreshIntervalMs: Long = 5000
) {
    val latestNews: Flow<List<ArticleHeadline>> = flow {
        while(true) {
            val latestNews = newsApi.fetchLatestNews()
            emit(latestNews) // Emits the result of the request to the flow
            delay(refreshIntervalMs) // Suspends the coroutine for some time
        }
    }
}

// Interface that provides a way to make network requests with suspend functions
interface NewsApi {
    suspend fun fetchLatestNews(): List<ArticleHeadline>
}
```

- flow내의 흐름은 ***순차적***.
    - 프로듀서가 코루틴에 있으므로, 정지 함수를 호출하면 프로듀서는 정지 함수가 반환될 때까지 정지 상태로 유지 (정지 함수 = fetchLatestNews)
    - 프로듀서는 `fetchLatestNews` 네트워크 요청이 완료될 때까지 정지됨 → 이후에 결과를 스트림으로 내보냄
- `flow` 빌더에서는 프로듀서가 다른 `CoroutineContext`의 값을 `emit`할 수 없음.
    - 그러므로 새 코루틴을 만들거나 `withContext` 블록을 사용하여 다른 `CoroutineContext`에서 `emit`를 호출하면 안된다!
    - 이런 경우 [callbackFlow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/callback-flow.html) 같은 다른 흐름 빌더를 사용해야 함
   

#### Intermediary

<img width="636" alt="Untitled 3" src="https://user-images.githubusercontent.com/85485290/185616292-068d9abf-78d8-4696-a1e6-7f79fc8a63ec.png">


- 프로듀서가 데이터를 생성했으면, **중간연산자는 생성된 데이터를 수정함**
- 예를들어, 프로듀서가 A라는 객체로 이루어진 데이터를 발행했는데, B라는 객체로 이루어진 데이터가 필요한 경우 중간연사자를 통해 **A → B 객체로 수정**할 수 있음
- 대표적 중간 연산자
    - `map` (데이터 변형)
    - `filter` (데이터 필터링)
    - `onEach` (모든 데이터마다 연산 수행) 등,,
    
- ex) 모든 뉴스 데이터가 아닌 → 내가 좋아하는 뉴스 데이터만 가져오는 과정
- 기존 뉴스 데이터를 좋아하는 뉴스데이터로 변형하고 방출하기 위해 `map`을 사용한 후, 들어오는 userData를 `filtering` 하여  좋아하는 뉴스만 가져온다.
- 방출 된 좋아하는 뉴스데이터 하나마다 cache에 저장한다.

```kotlin
class NewsRepository(
    private val newsRemoteDataSource: NewsRemoteDataSource,
    private val userData: UserData
) {
    /**
     * Returns the favorite latest news applying transformations on the flow.
     * These operations are lazy and don't trigger the flow. They just transform
     * the current value emitted by the flow at that point in time.
     */
    val favoriteLatestNews: Flow<List<ArticleHeadline>> =
        newsRemoteDataSource.latestNews
            // Intermediate operation to filter the list of favorite topics
            .map { news -> news.filter { userData.isFavoriteTopic(it) } }
            // Intermediate operation to save the latest news in the cache
            .onEach { news -> saveInCache(news) }
}
```

#### Consumer

<img width="618" alt="Untitled 4" src="https://user-images.githubusercontent.com/85485290/185616352-e619a09b-2285-4d7d-bf71-821be660e611.png">


- 중간 연산자는 프로듀서가 생성한 데이터를 변환하여 → 컨슈머에게 데이터를 전달함
- `collect`를 이용해 전달된 데이터를 소비(수집)할 수 있음
- 안드로이드에서 보통 데이터의 컨슈머는 → **`UI 구성요소`**
- UI는 데이터를 소비하여 데이터에 맞게 UI를 그려냄

- ex) 중간연산자가 변형 한 ‘좋아하는 최신 뉴스' 데이터를 ViewModel에서 처리하여 View에서 사용하는 과정

```kotlin
class LatestNewsViewModel(
    private val newsRepository: NewsRepository
) : ViewModel() {

    init {
        viewModelScope.launch {
            // Trigger the flow and consume its elements using collect
            newsRepository.favoriteLatestNews.collect { favoriteNews ->
                // Update View with the latest favorite news
            }
        }
    }
}
```

- `flow`를 소비하면 고정된 간격으로 최신 뉴스를 새로고침하고 → 네트워크 요청 결과를 내보내는 프로듀서가 트리거 된다.
- 프로듀서는 `while(true)` 반복문으로 항상 활성 상태가 유지되므로, ViewModel이 삭제되어 `viewModelScope`가 취소되면 데이터 스트림이 종료된다.

---


### Flow의 예외처리

#### 1. catch

- 보통 `catch` 중간 연산자 사용해서 예외 처리
    - **catch는 flow의 upStream에만 동작**하여, downStream에 대해 예외를 잡지 못함
    - 그러므로 catch의 위치가 중요
    - try-catch 만으로도 예외처리는 가능하나, boiler plate 코드가 늘어나거나 관리하기 힘들 수 있음
    - 범위 내에서 자식 중 예외가 발생하면 다음 자식이 실행되지 않고 종료돼버림 → 각 자식마다 개별 예외처리를 또 해줘야 하는 단점 존재
    
- catch 예제
    - `upStream` : “에러 발생 시점 이후”에 catch를 사용하여 예외처리가 가능
    - `downStream` : “에러 발생 시점 이전"에 catch를 사용하여 예외처리를 하지 못하고 예러 발생

```kotlin
// upStream
flowOf(1, 2, "Three", 3)
    .onEach { check(it !is String) { "It is Not Number" } }
    .catch { println(it) }
    .collect {
        println(it)
    }

result :
1
2
java.lang.IllegalStateException: It is Not Number

// downStream

flowOf(1, 2, "Three", 3)
    .catch { println(it) }
    .onEach { check(it !is String) { "It is Not Number" } }
    .collect {
        println(it)
    }

result :
1
2
FATAL EXCEPTION: DefaultDispatcher-worker-1
    Process: com.onetwothree.flowsample, PID: 19986
    java.lang.IllegalStateException: It is Not Number
```

- ex) catch 후 collect가 호출되지 않는 경우 (downStream)
- 예외가 발생하면 새 항목이 수신되지 않았기 때문에 collect가 호출되지 않는다

```kotlin
class LatestNewsViewModel(
    private val newsRepository: NewsRepository
) : ViewModel() {

    init {
        viewModelScope.launch {
            newsRepository.favoriteLatestNews
                // Intermediate catch operator. If an exception is thrown,
                // catch and update the UI
                .catch { exception -> notifyError(exception) }
                .collect { favoriteNews ->
                    // Update View with the latest favorite news
                }
        }
    }
}
```

- ex) catch 후 emit하여 collect 호출되는 경우 → 캐시된 값을 emit하는 예제 (upStream)
- 예외가 발생하면 collect가 호출되어 예외로 인해 새 항목이 스트림에 내보내짐

```kotlin
class NewsRepository(...) {
    val favoriteLatestNews: Flow<List<ArticleHeadline>> =
        newsRemoteDataSource.latestNews
            .map { news -> news.filter { userData.isFavoriteTopic(it) } }
            .onEach { news -> saveInCache(news) }
            // If an error happens, emit the last cached values
            .catch { exception -> emit(lastCachedNews()) }
}
```


#### 2. runCatching

[Try-catch를 쓰기 힘들다면 runCatching을 써보자! - Qiita](https://rannte.tistory.com/entry/kotlinruncatching)

- `runCatching` 블록 안에 **성공/실패 여부**를 캡슐화 된 Result<T> 형태로 리턴
- Exception 발생 시에 catch 해서 Result.failure()를 실행 시킴
- **runCatching과 함께 사용할 수 있는 복잡한 케이스 처리 API**
    - `map` / `mapCathing`
    - `**recover**` / `recoverCatching`
    - map과 recover 블록에서 Exception이 발생하면 runCatching 밖으로 exception이 나가버리는 문제 → try-catch로 또 잡아줘야 하는데,,
    - mapCatching과 recoverCatching을 이용하면 exception이 발생해도 runCatching 안을 벗어나지 않고 onFailure() 에서 핸들링 가능
    
- **runCatching에 대한 return 값이 필요할 때**
    - `getOrNull()` : 실패할 경우 Null 리턴
    - `getOrDefault()` : 실패할 경우 인자로 전달한 값을 Default 값으로 리턴
    - `getOrElse { }` : 실패할 경우 else 블록 실행
    - `getOrThrow()` : 실패할 경우 그대로 exception throw
    

```kotlin
public inline fun <R> runCatching(block: () -> R): Result<R> {
    return try {
        Result.success(block())
    } catch (e: Throwable) {
        Result.failure(e)
    }
}
```

<img width="400" alt="Untitled 5" src="https://user-images.githubusercontent.com/85485290/185616630-ff8ba581-2c34-47fe-8980-eda4f230b25e.png"><img width="400" alt="Untitled 6" src="https://user-images.githubusercontent.com/85485290/185616641-651f7726-289c-42a3-bf8b-65cfd500f91f.png">


- 이 try-catch 구문을,,

```kotlin

result.let {
                try {
                    try {
                        _images.postValue(it)
                    } catch (e:Exception) {
                      // error handling  
                    }
                }catch (e:Exception) {
                    // error handling
                }
            }
```

- runCatching 으로 바꾸면

```kotlin
fun fetchImages(query: String?, page: Int, size: Int) =
        viewModelScope.launch {
            val result = daumApi.loadImages(query, page, size)

            runCatching {
               _images.postValue(result)
            }.onSuccess { 
                // 성공시만 실행
            }.onFailure { 
                // 실패시만 실행 (try - catch 블록의 catch와 유사)
            }
        }
```

#### 3. corutineExceptionHandler

- 코루틴은 CourtineScope 내부의 예외처리 Handler를 제공하고 있음
- CoroutineScope 내에서 발생한 Exception을 Catch 해주어 coroutineContext와 Throwable의 형태로 반환
- `coroutineExceptionHandler`는 Thread의 UncaughtExceptionHandler를 구현하는 것으로 작동함
- `UncaugthExceptionHandler` → 캐치되지 않은 런타임 예외를 처리하는 방법
    - 즉, Thread에서 캐치되지 않은 런타임 예외를 **한 곳에서 처리**할 수 있도록 도와준다!

> 1. coroutineExceptionHandler { } 블록 내에서 예외처리를 함
> 2. SingleLiveEvent를 이용해 예외 옵저빙

```kotlin
open class BaseViewModel : ViewModel() {

    private val _isError = SingleLiveEvent<Boolean>()
    val isError: LiveData<Boolean> get() = _isError

    protected val coroutineExceptionHandler = CoroutineExceptionHandler { _, throwable ->
        // error handling
        throwable.printStackTrace()
        _isError.call()
    }
}
```

- 참고 : Retrofit + Coroutine 에서 API 예외처리

[안드로이드 Retrofit + Coroutines의 API 응답 및 에러 핸들링 - Sandwich](https://velog.io/@skydoves/retrofit-api-handling-sandwich)

    
---
    
    
### 다른 Courtine Context

--- 
    
    
### LiveData와 Flow의 차이

[LiveData와 Flow, 그리고 StateFlow](https://yjyoon-dev.github.io/android/2022/02/12/android-02/)

#### LiveData

- 관찰 가능한 데이터 홀더 타입
- **ViewModel이나 DataBinding과 호환되어 MVVM 패턴을 구현**하기 위해 적극적으로 사용됨
- Retrofit이나 Room에서도 LiveData를 지원하기 때문에, Data Layer 에서도 활용할 수 있음
- **안드로이드의 수명 주기를 인식할 수 있음**
    - LiveData를 이용하여 UI를 갱신하면 메모리 누수가 없고, 중지된 컴포넌트로부터 앱이 튕기는 일을 방지할 수 있음
    

But,,

- **비동기 데이터 스트림을 지원하지 않음**
    - LiveData는 UI와 밀접하게 연관되어 있기 때문에 **오직 Main Thread에서만 관찰**됨
    - ViewModel을 통해 View를 업데이트할 때는 문제 X
    - `Data Layer`에서 사용 시 문제 O → 데이터를 받아오고 처리하는 과정은 Main Thread가 아닌, Work Thread에서 처리해야 성능적으로 유리하기 때문!
    - `Domain Layer`에서도 사용하기 부적합 → **Domain Layer는 플랫폼에 종속적이지 않은 순수한 Kotlin 및 Java 코드**로 이루어져야 하는데, **LiveData는 안드로이드 플랫폼에 종속적임,,**
    
- 순수 Kotlin에서 지원하는 클래스 중, **관찰 가능한 데이터 홀더 타입** + **비동기 데이터 스트림**을 지원하는 것 → `Flow`
- Flow는 Data Layer에서 유리하다!
- LiveData와 Flow의 차이점 중 하나가 Flow는 flatMapLatest와 같은 Stream 함수를 제공한다는 점!

> Flow의 한계점
> 
- Flow 스스로 안드로이드 생명주기를 알 수 없음
- Flow에는 상태 또는 값이 없음
- Flow는 cold stream 방식으로, 연속해서 들어오는 데이터를 처리할 수 없으며 collect 되었을 때만 새로운 Flow가 생성되고 값을 반환함 → DB 접근 또는 API 통신 시에 collect 할 때마다 중복해서 리소스 요청 수행하는 문제 존재

<img width="600" alt="Untitled 8" src="https://user-images.githubusercontent.com/85485290/185617036-890ada19-ee91-4ecf-a04c-97618eb8e311.png">


<aside>
💡 **ViewModel에서는 LiveData를 사용**하고 **Repository와 Data Source 영역에서는 비동기적으로 Flow**를 사용한다면?

</aside>

- `Retrofit`과 `Room`이 코루틴(suspend)를 지원한다.
- 이를 repository에서 `Flow`로 collect 할 수 있다.
- Flow를 `asLiveData()`를 통해 LiveData로 변환할 수 있다.


> 💡 Room이나 `Retrofit`에서 코루틴의 `suspend` 키워드로 비동기 인터페이스를 구현하고, Repository에서 Data Source로부터 **Flow를 받아온 뒤** ViewModel에서 이를 **LiveData로 변환**하여 View에서 관찰하게 하는 구조!


---
    
### Flow와 StateFlow 차이

[[Coroutine Flow] 2. Flow와 StateFlow의 차이는 무엇인가?](https://kotlinworld.com/232)

#### Flow에 대한 착각

> Flow는 데이터의 흐름(flow)를 발생시키기만 할 뿐, **데이터가 저장되진 않는다!**
> 

따라서 flow만을 이용해 안드로이드의 UIState를 업데이트 하기 위해서는 두가지 방법이 가능했는데,,
    
    

1. **화면이 재구성 될때마다 다시 서버 or DB로부터 데이터 가져오기**
    
    → **비효율적!** 예를들어, 화면이 회전되면 onDestory가 호출된 후 다시 onCreate가 호출되는데, 이때마다 새로운 데이터를 서버나 DB로부터 가져와야한다? 너무 비효율적,,
    
    
    
2. **Flow로 부터 collect한 데이터를 ViewModel에 저장해놓고 사용하기**
    
    → **효율적!** ViewModel은 onDestroy가 호출되더라도 살아있고, 뷰모델에서 해당 데이터를 저장하고 있으면 됨
    
    - 하지만 데이터를 저장하고 있으려면 별도의 데이터 홀더 변수를 만들어야 함
    - 또한 UI에서 해당 데이터 홀더 변수를 구독하기 위해서는 별도의 fetching 로직을 만들어야 함
    
    > 그래서 **ViewModel에서 데이터 홀더 변수와 flow를 같이 사용하는 방법** 존재
    - flow를 구독하고, **데이터 홀더 변수는 flow에서 마지막으로 발행한 데이터를 저장**하고 있으면 됨
    - UI에서는 flow에서 값을 발행하기 전에는 데이터 홀더 변수의 데이터를 사용하면 됨!
    - 데이터 홀더 변수가 마지막 데이터를 저장하고 있으므로 다시 서버로 데이터를 요청할 필요가 없음
    
    > But, UIState가 여러개이고, 모두를 구독하기 위해 비슷한 코드를 매번 작성해야한다 ? 이는 Boiler Plate,,,
    
    

#### **그래서 나온 StateFlow !**

- StateFlow의 특징 (Flow와의 차이)
    - **데이터 홀더(저장소) 역할을 하면서 Flow의 데이터 스트림 역할까지 함**
    - 항상 값을 갖고 있고, 오직 하나의 값을 가짐
    - **Hot Stream 방식**으로 collector가 없어도 생성 시 바로 활성화되며, 값이 업데이트 된 경우에만 반환
    - 여러개의 collector를 지원하기 때문에 중복 리소스 요청을 방지

→ **Flow가 LiveData를 완벽히 대체할 수 없었던 문제들을 모두 해결**

→ 안드로이드의 생명주기를 직접 알 수 없다는 문제 또한 `LifeCycleScope` 를 확장하여 해결 가능

→ UI단에서 StateFlow를 구독하여 UIState를 업데이트 하면, 화면이 재구성 될때마다 서버로 데이터 요청할 필요 X, UI는 단순히 StateFlow를 구독만 하고 있으면 됨!

-> `DataBinding` + `Coroutines`+ `StateFlow` 조합

#### Flow를 StaetFlow로 변환하기
    - `**asStateFlow()**`

    
---
    
### StateFlow와 SharedFlow 차이

[stateflow-sharedflow-공식문서](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow?hl=ko)


---

### collect와 colletLatest 차이

[[Coroutine Flow] collect와 collectLatest의 차이는 무엇인가?](https://kotlinworld.com/252?category=973477)
    
- Flow에서 발행하는 데이터는 collect의 action 파라미터에 의해 소비됨
- collect의 인자로 들어가는 action 블록은 flow에서 발행된 데이터를 순차적으로 받아 suspend fun을 수행함
- 하지만 이 collect를 잘못 사용하면 새로운 데이터가 발행되더라도 데이터 처리가 제대로 되지 않을 수 있음

<img width="598" alt="Untitled 7" src="https://user-images.githubusercontent.com/85485290/185616880-a37089b2-8697-4fb3-86d0-6c516f05a82d.png">
    
- collect는 새로운 데이터가 들어왔을 때, 이전 데이터의 처리가 끝나야만 새로운 데이터를 처리하지만, collectLatest는 새로운 데이터가 들어오면 이전 데이터 처리를 강제종료 시키고 새로운 데이터를 처리하기에 최신데이터로 UI를 구성하는데 특화되어있음.

    
---
    
### Hot Stream과 Cold Stream

- Hot Stream
    - 예외로, Stateflow는 Hot Stream 이다!
    - StateFlow는 마지막 값의 개념이 있으며, 생성하자마자 활성화 됨
    - StateFlow는 Hot이니 Observable
    

<aside>
    
💡 생성되자마자 발행을 시작 → 이후에 구독한 구독자가 생겼을때는 중간 어딘가에서부터 발행될 수 있음 → 데이터스트림을 생성하자마자 발행하는데, 여러곳에서 중간에 구독하면 앞에 데이터가 전달이 안될수도 있는 문제 O
    
</aside>
    

- Cold Stream
    - Flow는 Cold Steam이다 !
    - 일반 Flow는 마지막 값의 개념이 없고 collect 될 때만 활성화 됨
    - Flow는 Cold니 Subject

    
<aside>
    
💡 옵저버가 서브스크라이브를 모두 할 때까지 기다림 → 모든 시퀀스가 처음부터 발행되는것을 보장!

</aside>

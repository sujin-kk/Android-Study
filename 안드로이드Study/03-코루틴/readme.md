# 03-Coroutine

### 목차

+ [Coroutine](#coroutine)
  - [Thread의 구조](#thread의-구조)
  - [기존의 접근 방식과 한계점](#기존의-접근-방식과-한계점)
  - [코루틴이 기존 한계점을 극복하는 방법](#코루틴이-기존-한계점을-극복하는-방법)
  - [코루틴은 왜 스레드보다 가볍다고 할까?](#코루틴은-왜-스레드보다-가볍다고-할까?)
+ [Coroutine Context](#coroutine-context)
+ [Coroutine Scope](#coroutine-scope)
+ [Dispatcher](#dispatcher)
+ [supend fun](#supend-fun)
+ [withContext](#withcontext)
+ [안드로이드에서 코루틴 사용하기](#안드로이드에서-코루틴-사용하기)
+ [코루틴의 상태 관리](#코루틴의-상태-관리)
  - [Job의 상태](#job의-상태)
  - [Job의 상태 변수](#job의-상태-변수)
+ [코루틴 예외 다루기](#코루틴-예외-다루기)
+ [Supervisor Job 과 SupervisorScope](#supervisor-job-과-supervisorscope)
+ [Dispatchers.Main.Immediate](#dispatchersmainimmediate)

---

### Coroutine

```
💡 코루틴은 쓰레드 안에서 실행되는 일시 중단 가능한 작업의 단위이다. 하나의 쓰레드 안에 여러 코루틴이 존재할 수 있다.
```

- 스레드를 차단(block) 하지 않고, 실행을 중지(suspend) 할 수 있는 비동기 작업 실행 단위


[공식문서](https://developer.android.com/kotlin/coroutines?hl=ko)

[Reading Coroutine official guide thoroughly - Part 0](https://myungpyo.medium.com/reading-coroutine-official-guide-thoroughly-part-0-20176d431e9d)

[[Coroutine] 1. Coroutine 은 어떻게 스레드 작업을 최적화 하는가?](https://kotlinworld.tistory.com/139)

- **프로세스**
    - OS에 의해 메모리에 적재되고 실행되는 앱
    - 데이터와 메모리 등의 자원, 그리고 스레드로 구성되어 있음
    - 한 개 이상의 스레드를 가짐

- **스레드**
    - 프로세스 내에서 실행되는 실행 흐름의 최소 단위

- **UI 변경은 메인 스레드에서만 가능!**
    - 메인 스레드 : 프로세스가 실행될 때 기본적으로 실행되는 스레드
    - 서브 스레드에서 UI를 변경하기 위해 아래와 같은 방법 이용
        - Hnadler
        - RunOnUiThread
        - ~~AsyncTask~~

#### Thread의 구조

- 하나의 프로세스에는 여러 스레드가 있고, 각 스레드는 독립적으로 작업을 수행할 수 있음
- ex) JVM 프로세스 상의 스레드 구성

<img width="500" alt="Untitled" src="https://user-images.githubusercontent.com/85485290/187021434-efefb4e9-81e6-48df-846f-ed9ad7e42d5e.png">


- JVM 프로세스는 Main Thread가 종료되면 강제로 종료되며, JVM 프로세스에 속한 Thread들도 함께 강제로 종료됨
- 나머지 2개의 Thread에서는 Main Thread와 마찬가지로 작업을 수행할 수 있으며, 이 Thread들은 종료되더라도 다른 Thread에 영향을 미치지 않음
- 안드로이드에서 **Main Threads**는 UI를 그려주고 사용자가 화면을 눌렀을 때 이벤트를 전달받는 **가장 중요한 쓰레드** = 사용자와 인터랙션을 담당하는 쓰레드
    - 만약 이 메인 쓰레드가 높은 부하를 받는 작업에 의해 막히면, 우리의 안드로이드 앱은 멈추게 되고, 일정 시간 이상 블로킹될 때 앱이 강제로 종료돼 버린다.
    - **메인 쓰레드에서 많은 부하를 받는 작업은 지양해야 한다!**
    - **다른 쓰레드를 생성**해 해당 쓰레드에서 높은 부하를 갖는 작업을 수행하도록 해야 함
    

#### 기존의 접근 방식과 한계점

```
💡 그렇다면 기존에는 어떻게 높은 부하를 갖는 작업을 분산시켰을까?
```

1. Runnable 인터페이스를 구현하는 클래스를 만들고, 다음 Thread에 해당 클래스를 넣어 start 시키는 방식
- ex) 전체 5개의 쓰레드 중 0번과 1번이 사용 됨

```kotlin
fun main() {
	val exampleRunnable = ExampleRunnable()

	Thread(exampleRunnable).start()
	Thread(exampleRunnable).start()

	/*
	출력
	Runnable Running
	Runnable Running
	Thread[Thread-0,5,main]
	Thread[Thread-1,5,main]
	*/
}

class ExampleRunnable: Runnable {
	override fun run() {
		println("Runnalbe Running")
		println(Thread.currentThread())
	}
}
```

2. ExecuterService를 이용해 Thread Pool을 구성하여 작업을 던지는 방식

```kotlin
fun main() {
	val executerService = Executors.newFixedThreadPool(4)

	executerService.submit(ExampleRunnable())
	executerService.submit(ExampleRunnable())

	/*
	출력
	Runnable Running
	Runnable Running
	Thread[Thread-0,5,main]
	Thread[Thread-1,5,main]
	*/
}

class ExampleRunnable: Runnable {
	override fun run() {
		println("Runnalbe Running")
		println(Thread.currentThread())
	}
}
```

3. Rx 라이브러리를 이용
- Rx 라이브러리는 엄밀히 말하면 Reactive Programming을 돕기위한 것이지만, 데이터를 발행하는 Thread와 데이터를 구독하는 Thread를 손쉽게 제어할 수 있게 함으로써 쓰레드 작업을 쉽게할 수 있음

```kotlin
publisher.subscribeOn(Schedulers.io())
	.observeOn(AndroidSchedulers.mainThread())
```

> 위와 같은 3가지 접근 방식들의 한계점은 **작업의 단위가 Thread** 라는 점,,
> 

- Thread는 생성 비용 및 작업을 전환하는 비용이 비쌈
- Thread1이 작업 1 수행 도중, Thread2의 작업2 결과물이 작업 1을 수행하는데 필요한 상황
- 이 때, Thread1은 아무것도 하는 일 없이 Blocking되고, Thread2로 부터 결과를 받아 작업1을 재개하기 까지 많은 시간 소요되는 문제,,
- 다른 Thread 부터의 작업을 기다릴 때 Blocking 되면, 해당 Thread는 하는 작업 없이 다른 작업이 끝날때까지 기다려야 하기 때문에 자원이 낭비됨

→ 작업 단위가 Thread 일 때 생기는 고질적인 문제들,,

<img width="600" alt="Untitled 1" src="https://user-images.githubusercontent.com/85485290/187021449-82b77b3b-2134-4b44-8dac-387fe4f8b1c9.png">


#### 코루틴이 기존 한계점을 극복하는 방법

- 보통 메인 스레드에서 네트워크 요청을 보내면, 응답을 받을 때 까지 메인 스레드가 멈춤 → OS는 onDraw()를 호출할 수 없으므로 앱이 정지되어 버림
- **코루틴은 메인 스레드에서 비동기 작업을 수행하면서도, 메인 스레드를 대기 시키지 않는다!**
- 코루틴에서도 Thread라는 작업 단위를 사용하지만, **Thread 내부에서 작은 Thread 처럼 동작하는 코루틴이 존재**
- Thread 하나를 일시중단 가능한 다중 경량 Thread 처럼 활용하는 것이 Coroutine!

<img width="260" alt="Untitled 2" src="https://user-images.githubusercontent.com/85485290/187021459-bf2c0e14-8160-442b-8f4e-696d3b8c4424.png">


- 위에서 나온 Thread 문제 → 코루틴을 이용하여 쓰레드를 **Non Blocking** 하게 만들기!
- Thread1 에서 코루틴 2개 생성
    1. **Coroutine1 - 작업1, Thread2 - 작업2** → Coroutine1은 작업1 수행 도중, 작업2의 결과가 필요해 짐
    2. Coroutine1은 Thread1을 Blocking 하는 대신, 자신의 작업을 일시 중단하고 Coroutine2에 리소스 사용 권한을 넘겨줌 → Thread1의 **Coroutine2가 작업3을 위해 Thread1을 사용함**
    3. 이후 Thread2의 작업2가 종료되고, 작업3을 수행하던 Coroutine2가 자신의 작업을 마무리 하고 Coroutine2 **자신을 일시중단** 시킴 → 다시 Thread1의 제어 권한이 **Coroutine1로 돌아옴** → Thread1은 Coroutine1이 마저 작업을 수행할 수 있도록 Thread2로부터 결과를 전달받아 **작업1을 재개**한다!
    
- Blocking되는 상황이 줄어, Thread1이 **리소스를 최대한 활용**할 수 있음
- Thread는 만드는 비용이 큰데, 코루틴은 쓰레드를 만드는 대신, **하나의 쓰레드 상에서 자신을 일시 중지할 수 있도록 하여, 쓰레드 생성 비용을 줄임**

<img width="600" alt="Untitled 3" src="https://user-images.githubusercontent.com/85485290/187021468-0c55377a-5b39-47ff-8b54-0ce21ab17794.png">


#### 코루틴은 왜 스레드보다 가볍다고 할까?

> 코루틴 1개가 새로 생성되어 실행 ≠ 동시에 새로운 스레드 또한 생성?
> 
- 정확히 말하면 코루틴 생성 시 스케쥴러 생성에 따라 다르긴 함
- 사실, 코루틴은 스케쥴링 가능한 코드 블록 or 이러한 코드 블록들의 집합 이라고 볼 수 있다는데,,

<img width="664" alt="Untitled 4" src="https://user-images.githubusercontent.com/85485290/187021473-004c81e5-ee45-4c18-b99e-769991b70258.png">


1. 우리가 어떤 코루틴을 실행하기 위해서는 어떤 `CoroutineScope` 에 속해 있어야 함
    - 현재 코루틴 스코프가 갖는 `CoroutineContext`에서 Dispathcer는 `UI Dispatcher` 로 설정되어 있음 (= 현재 스코프에서 실행되는 중단 함수들은 **UI Thread** 에서 수행 됨을 의미)

2. 이제 이 스코프 안에서 코루틴을 하나 만들었음
    - 이 코루틴은 자신이 실행되는 스코프(부모)의 context를 그대로 상속하고, Dispatcher만 `ThreadPoolDispatcher`로 재정의 함 (재정의 하지 않으면, 기본적으로 부모 스코프로부터 모두 상속)
    - 이 코루틴에서 수행되는 함수는 ThreadPoolDispatcher를 이용하여 **워커(백그라운드) 스레드**에서 수행 됨
    - 이때, launch { } 와 같이 빌더를 실행했을 경우 마지막으로 넘긴 코드 블록 즉, 실제 수행하고자 하는 로직이 담긴 코드 블록은 Continuation 이라는 단위로 만들어짐.

> *어떤 일을 수행하기 위한 일련의 함수들의 연결을 각 함수의 반환값을 이용하지 않고 Continuation 이라는 추가 파라미터(Callback)를 두어 연결하는 방식으로 Continuation 단위로 dispatcher 를 변경한다거나 실행을 유예한다거나 하는 플로우 컨트롤이 용이해지는 이점이 있음.*
> 

3. 이렇게 Continuation 으로 변경 된 코드 블럭은 최초에 `suspend` 상태로 생성 되었다가 `resume()` 요청으로 인해 resumed 상태로 전환되어 실행됨
    - Continuation의 재개(resume)가 요청될 때마다 현재 컨텍스트의 dispatcher 에게 dispatch(스레드 전환) 가 필요한지 `isDispatchNeeded()` 함수를 이용해 확인 한 후 dispatch가 필요하면 `dispatch()`함수를 호출하여 적합한 스레드로 전달하여 수행됨

- 만약 코루틴 생성 시 Dispatcher를 재정의 하지 않고, UI Dispatcher를 그대로 상속 받아 사용했다면?
    
    → 일반적인 함수 호출과 동일하게 수행 되었을 것!
    
    **이것이 코루틴이 경량 스레드라고 불리는 이유**
    
    ```
    💡 코루틴은 Dispatcher에 의해 실행되는 환경(Thread)이 결정될 수는 있지만, 그 자체로는 환경을 새로 구성하거나 변경하진 않는다.
    ```
    

- 그래서 코루틴은 공식 가이드에 나와있는, 아래와 같은 코드가 **OOM(Out-Of-Memory)** 방식으로 동작할 수 있음

```kotlin
fun main(args: Array<String>) = runBlocking {
    repeat(100_000) {
        launch {
            delay(1000L)
            print(".")
        }
    }
}
```

- 위 코드는 코루틴을 10만개 수행하고 있는 코드.
- 위 예제에서 launch { } 코루틴 빌더는 Dispatcher 를 재정의 하지 않았기 때문에 현재 스코프(runBlocking)의 Dispatcher 를 그대로 사용.
- runBlocking 코루틴 빌더는 내부적으로 GlobalScope을 사용하며 Dispatcher 는 BlockingEventLoop 을 사용하는데, 이는 큐를 이용한 이벤트 루프 형태의 Dispatcher 구현임!
- → 그래서 위 코드는 실행 스레드에서 이벤트 루프 기반으로 10만번의 이벤트를 발생하여 점(.)을 출력하게 되며 **스레드 부하는 없으므로 OOM을 피할 수 있다**!!

---

### Coroutine Context

- 참고 문서

[코루틴 공식 가이드 읽고 분석하기- Part 1 - Dive1](https://myungpyo.medium.com/reading-coroutine-official-guide-thoroughly-part-1-7ebb70a51910)

[[Kotlin - Coroutine] Coroutine Context 와 Scope](https://stanleykou.tistory.com/entry/Kotlin-Coroutine-Coroutine-Context-%EC%99%80-Scope)

[[Coroutine] 11. Coroutine CoroutineContext를 다루는 방법 : Coroutine Dispatcher과 ExceptionHandler을 CoroutineContext를 이용해 관리하기](https://kotlinworld.com/152?category=973476)

```
💡 CoroutineContext는 다양한 요소(Element)의 집합이다. 주요 요소는 **Job**과 **Dispatcher.**
```

- Coroutine Context는 코루틴이 실행되는 환경!
    - `Dispatcher`와 `CoroutineExceptionHandler` 또한 코루틴이 실행되는 환경의 일부 → 모두 CoroutineContext를 확장하는 인터페이스의 구현체
- 어떤 스레드를 사용할 지 결정해줌
    - Dispatcher를 붙여서 코루틴을 분배해준다✨

- CoroutineContext는 4가지 메소드를 가짐

```kotlin
public interface CoroutineContext {
  /**
   * Returns the element with the given [key] from this context or `null`.
   * Keys are compared _by reference_, that is to get an element from the context the reference to its actual key
   * object must be presented to this function.
   */
  public operator fun <E : Element> get(key: Key<E>): E?
  /**
   * Accumulates entries of this context starting with [initial] value and applying [operation]
   * from left to right to current accumulator value and each element of this context.
   */
  public fun <R> fold(initial: R, operation: (R, Element) -> R): R
  /**
   * Returns a context containing elements from this context and elements from  other [context].
   * The elements from this context with the same key as in the other one are dropped.
   */
  public operator fun plus(context: CoroutineContext): CoroutineContext = ...impl...
  /**
   * Returns a context containing elements from this context, but without an element with
   * the specified [key]. Keys are compared _by reference_, that is to remove an element from the context
   * the reference to its actual key object must be presented to this function.
   */
  public fun minusKey(key: Key<*>): CoroutineContext
}

/**
 * Key for the elements of [CoroutineContext]. [E] is a type of element with this key.
 * Keys in the context are compared _by reference_.
 */
public interface Key<E : Element>

/**
 * An element of the [CoroutineContext]. An element of the coroutine context is a singleton context by itself.
 */
public interface Element : CoroutineContext {
  /**
   * A key of this coroutine context element.
   */
  public val key: Key<*>
  
  ...overrides...
}
```

- `get()` : 연산자(operator) 함수로써 주어진 key 에 해당하는 컨텍스트 요소를 반환.
- `fold()` : 초기값(initialValue)을 시작으로 제공된 병합 함수를 이용하여 대상 컨텍스트 요소들을 병합한 후 결과를 반환.
    - ex) 초기값을 0을 주고 특정 컨텍스트 요소들만 찾는 병합 함수(filter 역할)를 주면 찾은 개수를 반환할 수 있고, 초기값을 EmptyCoroutineContext 를 주고 특정 컨텍스트 요소들만 찾아 추가하는 함수를 주면 해당 요소들만드로 구성된 코루틴 컨텍스트를 만들 수 O
- `plus()` : 현재 컨텍스트와 파라미터로 주어진 다른 컨텍스트가 갖는 요소들을 모두 포함하는 컨텍스트를 반환. 현재 컨텍스트 요소 중 파라미터로 주어진 요소에 이미 존재하는 요소(중복)는 버려짐!
- `minusKey()` : 현재 컨텍스트에서 주어진 키를 갖는 요소들을 제외한 새로운 컨텍스트를 반환.

- `Key` 인터페이스 : Key는 `Element` 타입을 제네릭 타입으로 가져야 함
    - Element 는 CoroutineContext 를 상속하며 “key” 를 멤버 속성으로 갖는다.
    - CoroutineContext 를 구성하는 Element 들의 예를 들어보면 `CoroutineId`,  `CoroutineName`,  `CoroutineDispatcher`, `ContinuationInterceptor`,  `CoroutineExceptionHandler` 등이 있음.
    - 이런 요소(element)들은 각각의 key 를 기반으로 CoroutineContext 에 등록 됨!
    

> CoroutineContext 에는 코루틴 컨텍스트를 상속한 **요소(Element)들이 등록**될 수 있고, 각 요소들이 등록 될 때는 요소의 **고유한 키**를 기반으로 등록된다!
> 

**CoroutineContext** 는 인터페이스로써 이를 구현한 구현체로는 3가지 종류가 있음

1. `EmptyCoroutineContext`: 특별히 컨텍스트가 **명시되지 않을 경우** 이 singleton 객체를 사용.
2. `CombinedContext`: 두개 이상의 컨텍스트가 명시되면 **컨텍스트 간 연결**을 위한 컨테이너역할을 하는 컨텍스트.
3. `Element`: **컨텍트스의 각 요소**들도 CoroutineContext 를 구현.

<img width="600" alt="Untitled 5" src="https://user-images.githubusercontent.com/85485290/187021516-43f85355-b14e-463b-941d-269aeee03032.png">


- ex) Global Scope.launch { } 를 수행 → launch 함수의 첫번째 파라미터인 CoroutineContext에 어떤 값을 넘기는 지에 따라 변화되는 코루틴 컨텍스트의 상태
- 각각의 요소를 + 연산자를 이용해 연결(merge)
    - Element + Element + … 는 결국 하나로 병합 된 CoroutineContext (e.g. CombinedContext)를 만들어낸다!
    

> *주황색 테두리는 CombinedContext 로서 CoroutineContext 와 Element 를 묶어 하나의 CoroutineContext 가 되는 개념. → 그리고 내부 코드에서 ContinuationInterceptor 는 이 병합 작업이 일어날 때 항상 마지막에 위치하도록 고정되는데 이는 인터셉터로의 빠른 접근을 위해서라고 커멘트 되어 있다.*
> 

---

### Coroutine Scope

- 참고 문서

[공식문서 - 수명 주기를 인식하며 코루틴 사용하기](https://developer.android.com/topic/libraries/architecture/coroutines?hl=ko)

[[Android CoroutineScope] 1. Activity, ViewModel에서 올바른 CoroutineScope 사용법 : lifecycleScope과 viewModelScope의 활용](https://kotlinworld.com/198?category=973478)

```
💡 CoroutineScope는 **Coroutine job이 실행되는 Scope** 이다.
```

- **코루틴의 실행 범위를 지정해줌**
    - `CoroutineScope(Dispathcers.Main).launch { }`
        - 범위 내에서 동작. 일반적으로 많이 사용. 필요할때만 열고 완료되면 닫아주면 됨.
    - `GlobalScope.launch { }`
        - 앱이 종료할때까지 동작. 실행 시간이 비교적 긴 코루틴의 경우에 적합. 가급적 사용 X
    - `viewModelScope.launch { }`
        - ViewModel 인스턴스, ViewModel이 제거될 때 까지 동작
    
- 코루틴 스코프는 기본적으로 **코루틴 컨텍스트 하나**만 멤버 속성으로 정의하고 있는 인터페이스임

```kotlin
public interface CoroutineScope {
    /**
     * Context of this scope.
     */
    public val coroutineContext: CoroutineContext
}
```

- 코루틴 스코프는 **하나 이상의 코루틴을 관리**하고, 모든 코루틴은 **스코프 내에서 실행**되어야 함
- **Coroutine Scope가 해제**되면, CoroutineScope에 속한 **Coroutine Job들은 모두 해제**됨
- 만약 Coroutine Scope가 안드로이드 구성요소(Activity, ViewModel)의 Lifecycle에 따라 올바르게 할당/해제되지 않는다면, 해제되어야 하는 Job들이 계속해서 동작해서 **메모리 누수로 이어질 수 있음**,,

- 우리가 사용하는 모든 코루틴 빌더들
    - ex) 코루틴 빌더 - launch, async / 스코프 빌더- coroutineScope, withContext 등)은 CoroutineScope 의 확장 함수로 정의 됨
    - 다시말해, 이 빌더들은 CoroutineScope의 함수들인 것이고 이들이 코루틴을 생성할 때는 소속된 CoroutineScope 에 정의된 **CoroutineContext를 기반**으로 필요한 코루틴들을 생성함
    
- ex) 액티비티에서의 코루틴 스코프
- 아래 코드에서 앱을 실행 뒤, Activity에서 뒤로 가기를 누르면
- → Activity가 onDestroy 되더라도 여전히 stringFlow에 대한 collect가 수행됨 (코루틴 잡이 캔슬되지 않고, 계속 새로운 잡 수행)

<img width="335" alt="Untitled 6" src="https://user-images.githubusercontent.com/85485290/187021549-18af8975-230f-4814-a75f-0a36a0019087.png">


- **안드로이드를 위한 CoroutineScope**
    - `lifecycleScope`
    - `viewModelScope`
    
- **Activity에서 사용해야 하는 lifecycleScope**
- Activity에서 뒤로가기를 누르면 OnDestroy가 호출되므로 lifecycleScope가 취소되어 stringFlow를 collect하는 Job 또한 중지된다!

<img width="327" alt="Untitled 7" src="https://user-images.githubusercontent.com/85485290/187021551-38295c4b-0d81-4fdd-be03-2208e7073486.png">


- **ViewModel에서 사용해야 하는 viewModelScope**
- ViewModel은 Fragment 혹은 Activity의 Lifecycle에 binding 되므로 viewModelScope는 binding된 lifecycle에 맞춰 viewModelScope 내의 Job에 대한 취소를 하도록 함

<img width="241" alt="Untitled 8" src="https://user-images.githubusercontent.com/85485290/187021554-a5a1acb3-5789-4cd1-b750-d8b27688408f.png">

- lifecycleScope의 한계점
    - onDestory시에 collect가 중단됨
    - 이 말은 → **앱이 백그라운드로 내려가고 종료되지 않으면**, collect가 중단되지 않고 데이터가 계속 수집된다는 말
    - 위의 예시들에서 앱을 뒤로가기로 종료하지 않고, 홈버튼을 눌러 내린다면? → 계속해서 Job 실행, 유저가 UI 화면을 보지 않음에도 collect가 일어나는 문제,,
    

```
💡 **onStart**에서 Job을 생성 시작하고, **onStop**에서 Job을 cancel 하기
```

- Coroutine Job을 onStart에서 생성하고, onStop에서 cancel 하게 되면 ‘홈버튼'을 눌렀을 때도 flow에 관한 collect(데이터 수집)가 중지됨!

<img width="329" alt="Untitled 9" src="https://user-images.githubusercontent.com/85485290/187021561-8ad40da6-fb2b-4ea2-ba11-10f9288c958c.png">


- 하지만 위와 같이 매번 Coroutine Job을 생성하고 해제해주는 일은 보일러 플레이트 코드를 생성시키는 것일뿐,,
- 만약 하나라도 cancel 하는 것을 깜빡했을 땐? 백그라운드에서 작업이 계속 일어나 메모리 사용량이 높은 상태로 유지되어 시스템에 의해 App이 강제 종료될 수도 있음

- 안드로이드에서는 onStart에서 Job을 생성 시작하고, onStop에서 Job을 cancel 하는 API 제공!
- → `repeatOnLifecycle` 함수
- `flowOnLifecycle` 알아보기

> 코루틴 컨텍스트와 코루틴 스코프는 ‘어디에 쓸지 의도된 목적’이 다르다!
> 

<img width="382" alt="Untitled 10" src="https://user-images.githubusercontent.com/85485290/187021564-2a73e323-26dd-4986-968c-5c69603113f2.png">


---

### Dispatcher

- 코루틴에서는 스레드 풀을 만들고 → 디스패처를 통해서 코루틴에 대한 task 배분
- **스레드에 코루틴을 보낸다.**

```
💡 코루틴을 Dispatcher에 전송하면, Dispatcher는 자신이 관리하는 Thread Pool의 내부 상황에 맞춰 코루틴을 배분함.
```

1. 유저가 코루틴을 생성 후 디스패처에 전송

<img width="670" alt="Untitled 11" src="https://user-images.githubusercontent.com/85485290/187021574-3b316937-af9c-4691-90ce-efb27fa6fc10.png">


2. 디스패처는 자신이 잡고 있는 Thread Pool에서 자원이 남는 스레드가 어떤 스레드인지 확인한 후, 해당 스레드에 코루틴을 전송

<img width="663" alt="Untitled 12" src="https://user-images.githubusercontent.com/85485290/187021577-a69974e3-670d-44f2-b002-e1d09bf208ac.png">

3. 분배 받은 스레드는 해당 코루틴을 수행

<img width="333" alt="Untitled 13" src="https://user-images.githubusercontent.com/85485290/187021587-8579cde9-9528-4e48-8803-278ff60c2db5.png">


- 안드로이드의 Dispatcher (기본 3가지 정의)
    - `Dispatchers.Main` - 안드로이드 **메인 스레드**에서 코루틴을 실행하는 디스패처. **UI와 상호작용하는 작업**을 실행하기 위해서만 사용해야 함. 코루틴이 메인스레드에서 작동.
    - `Dispatchers.IO` - **로컬DB 또는 네트워크 I/O 작업, 이미지 다운로드, 파일 작업 등**을 실행하는데 최적화 되어 있는 디스패처. 코루틴이 백그라운드 스레드에서 작동.
    - `Dispatchers.Default` - **CPU를 많이 사용하는 작업**을 기본 스레드 외부에서 실행하도록 최적화 되어 있는 디스패처. 정렬 작업이나 JSON 파싱 작업 등에 최적화.
    - `Dispatchers.Unconfined` - **호출한 컨텍스트**(Context)를 **기본으로 사용**하는 디스패처. 다만 **작업이 중단된 후 다시 실행될 때 컨텍스트가 바뀌면 바뀐 컨텍스트를 따라감.**
    

---

### supend fun

- 참고 문서

[[Coroutine] 5. suspend fun의 이해](https://kotlinworld.tistory.com/144?category=973476)

```
💡 코루틴은 **일시중단** 가능하다.
```

- launch로 실행하든, async로 실행하든 내부에 해당 코루틴을 일시중단 해야하는 동작이 있으면 코루틴은 일시 중단됨.

- ex) 일시 중단 가능한 코루틴

<img width="660" alt="Untitled 14" src="https://user-images.githubusercontent.com/85485290/187021602-c0192951-0cc0-4248-8263-ed1706f11768.png">

- 위 그림을 코드로 표현

```kotlin
fun exampleSuspend() {

	val job3 = CoroutineScope(Dispathers.IO).async {
		// 2. IO Thread에서 작업3 수행
		(1..10000).sortedByDescending { it }
		// 5. 작업3 완료
	}

	val job1 = CoroutineScope(Dispathers.Main).launch {
		// 1. Main Thread에서 작업1 수행
		println(1)

		// 3. 작업1의 남은 작업을 위해 작업3의 결과값이 필요 
		// -> Main Thread는 작업1을 일시중단
		val job3Result = job3.await()
		// 6. 작업3 으로부터 결과를 전달 받음

		// 7. 작업1 재개
		job3Result.forEach {
			println(it)
		}
	}

	// 4. Main Thread에서 작업2가 수행되고 완료
	val job2 = CoroutineScope(Dispathers.Main).launch {
		println("Job2 수행 완료")
	}
}
```

1. Main Thread의 Coroutine1에서 job1이 수행되며 Main Thread의 자원을 점유
2. IO Thread의 Coroutine3에서 job3이 수행되며 IO Thread의 자원을 점유
3. Coroutine1에서 Coroutine3의 job3의 결과가 필요한 작업에 도달 → **이때 Coroutine1 일시 중단**
4. Coroutine2가 Main Thread를 점유 → job2를 수행하고 완료
5. Coroutine3의 job3 완료
6. Coroutine1이 job3의 결과를 전달 받음
7. **Coroutine1 재개**

- 결과 로그

```kotlin
I/System.out: 1
I/System.out: Job2 수행완료 // Coroutine2 작업
I/System.out: 10000 // Coroutine1 작업 재개
	9999
	9998
	9997
	9996
	9995
	..
```

- job3는 IO Thread 위에서 수행되는 async 작업이며 결과값을 반환받아야 하는 작업
- 1~10000까지를 내림차순으로 정렬해서 반환받는 작업이다보니, job1의 println(1) 보다는 많은 시간을 소모함
- 때문에 job1 코루틴은 job3의 결과를 기다려야 함 → 일시 중단이 필요
- 비동기 작업에서는 **다른 작업의 결과를 기다려야 할 때, 즉 일시 중단이 필요한 경우가 많음** → 코루틴에서는 해당 코루틴 작업을 잠시 일시 중단 가능하게 함
- **일시 중단 가능한 함수는 코루틴 내부에서 수행되어야 함**


```
💡 코루틴 일시 중단은 **코루틴 블록 내부**에서 수행되어야 한다.
```

- 만약 일시 중단이 코루틴 블록(launch, async) 내부에서 수행되지 않는다면?

<img width="498" alt="Untitled 15" src="https://user-images.githubusercontent.com/85485290/187021608-2d429ee1-e87c-4724-9b4a-6c3d1c4b0319.png">

- 바로 오류가 나는데, 이를 해결하는 방법은 **두가지!**
    1. 일시 중단 작업을 내부로 옮기기
    2. `fun` → `suspend fun` 으로 만들기

1. 일시 중단 해당 작업을 코루틴 내부로 옮기기
    - 일시 중단 가능한 작업(await)을 코루틴 내부로 옮기면, 해당 코루틴은 결과값이 올때까지 일시 중단 됨!

```kotlin
fun exampleSuspend() {
	
	val job3 = CoroutineScope(Dispathers.IO).async {
		(1..10000).sortedByDescending { it }
	}

	CoroutineScope(Dispathers.IO).launch {
		job3.await()
	}
}
```

2. 일시 중단 작업을 수행하는 fun을 suspend fun으로 만들기
    - suspend fun은 일시 중단 가능한 함수를 지칭하는 것
    - 즉, **suspend fun은 무조건 코루틴 내부에서 수행되어야 함**
    - suspend fun은 suspend fun 내부에서 수행 될 수 있음!

```kotlin
class MainActivity: AppCompatActivity() {
	...
	CoroutineScope(Dispatchers.Main).launch {
		exampleSuspend()
	}
}

suspend fun exampleSuspend() {
	
	val job3 = CoroutineScope(Dispathers.IO).async {
		(1..10000).sortedByDescending { it }
	}

	job3.await()
}
```

- 정리
    - 코루틴의 일시 중단 기능은 무조건 코루틴 내부에서만 수행할 수 있음
    - suspend fun은 일시 중단 가능한 함수로, 해당 함수 내에 일시 중단 가능한 작업이 있다는 것을 의미
    - suspend fun은 코루틴 내부 또는 suspend fun 내부에서만 사용 가능

---

### withContext

```
💡 `withContext` 는 결과값 수신이 필요한 코드의 순차화를 가능하게 한다!
```

- 기존에 다른 코루틴에 보내진 작업의 결과를 수신하려면,,,
- Deferred로 결과값을 감싸 받아, await 으로 수신될 때까지 기다려야 했음

```kotlin
suspend fun main() {

	val deferred: Deferred<String> = CoroutineScope(Dispatchers.IO).async {
		"Async Result"
	}

	val result = deferred.await()

	println(result)
}
```

> `withContext` 를 이용하면 비동기 작업을 순차 코드처럼 작성할 수 있음!
> 
- withContext 블록의 **마지막 줄의 값이 반환 값**이 됨
- withContext가 끝나기 전까지 해당 **코루틴은 일시정지** 됨

위의 Deferred와 await을 이용한 코드는 다음과 같이 수정 가능

1. withContext 블록은 IO 스레드의 작업이 끝나기 전까지 Main 스레드에서 수행되는 코루틴을 일시중단 되도록 만듦
2. IO 스레드의 작업이 끝나면 Async Result가 반환되어 result 변수에 셋팅
3. result 프린트

```kotlin
suspend fun main() {

	val result: String = withContext(Dispatchers.IO) {
		"Async Result" // 반환 값
	}

	println(result)
}
```

- 비동기 작업을 순차적으로 처리하도록 만들어 버리면 비동기 작업의 장점이 사라질 수 있으니 너무 남발하면 안됨

- withContext를 사용하면 실행 중간에 CoroutineContext를 바꿀 수 있음
- 보통 네트워크나 DB에서 데이터를 받아오는 작업을 하고 나서 UI를 갱신하는 코드에 많이 사용

```kotlin
CoroutineScope(Dispathcers.IO).launch {
	val result = async {
		delay(1000)
		"Hello"
	}

	withContext(Dispatchers.Main) {
		textView.text = result
	}
}
```

---

### 안드로이드에서 코루틴 사용하기

1. Dispatcher에 Coroutine 붙이기
    - 두가지 메소드 `launch { }` ,  `async { }` 를 통해 가능

|  | 결과 반환 | 반환 타입 |
| --- | --- | --- |
| launch | X | Job |
| async | O | Deferred<T> |
- **결과를 반환하지 않는 launch**
    - launch 수행 시 job 반환

```kotlin
with(CoroutineScope(Dispatchers.Main)) {
	val job: Job = launch { println(1) }
}
```

- **결과를 반환하는 async**
    - 결과 값이 Deferred로 감싸서 반환됨 → **Deferred는 미래에 올 수 있는 값을 담아 놓을 수 있는 객체**
    - 아래 예제는 async 블록의 마지막 줄에 있는 1이 반환 되어야 하므로, **Deferred<Int>** 반환
    - await() 이 수행되면 코루틴은 결과가 반환될 때까지 기다림 → **일시 중단**
    - await() 은 일시 중단 가능한 ‘코루틴 내부’ 에서만 사용 가능
    - 결과가 반환되면 코루틴은 다시 재개 → println이 수행되어 1이 출력

```kotlin
CoroutineScope(Dispatchers.Main).launch {
	val deferredInt: Deferred<Int> = async { 
		println(1)
		1 // 마지막 줄 반환
	}

	val value = deferredInt.await()
	println(value)
}
```

- **Coroutine Builder**
    - 코루틴 스코프의 확장 함수 → 새로운 코루틴을 실행하기 위해 사용
    - 4가지 존재

1. `launch { }`
    - 현재 스레드를 blocking 하지 않고, 새로운 코루틴을 실행
    - job 인스턴스를 반환 → 이 인스턴스는 코루틴에 대한 참조로 사용
    - **리턴 값이 없는 단순 작업 코루틴**을 위해 사용하는 builder
    
2. `async { }`
    - 현재 스레드를 blocking 하지 않고, 새로운 코루틴을 실행
    - Deferred<T>의 인스턴스를 반환
        - Deferred 인터페이스 : job 인터페이스를 확장한 것, Deferred 인스턴스를 코루틴을 취소하는 것 같이 job 처럼 사용 가능
    - 값을 얻기 위해 `await()` 함수 사용
    - **리턴 값을 반환받고 싶은 경우**에 사용하는 builder
    
3. `produce { }`
- **요소 스트림**을 만드는 코루틴을 위한 builder
- ReceiveChannel 인스턴스 반환

1. `runBlocking { }`
- 다른 builder 들과 다르게 **작업이 끝날 때까지 스레드를 blocking**
- T 타입의 결과를 반환

++ 그외 빌더

withContext / coroutineScope / supervisorScope / actor

1. Dispatcher 전환하며 Coroutine 붙이기
- ex) 파일 시스템으로부터 Array<Int>를 받아와서 → 정렬한 다음 → 텍스트뷰에 출력하는 과정
- 파일 입출력(Dispatchers.IO), Array 정렬(Dispatchers.Default), 텍스트 뷰 출력(Dispatchers.Main)

→ **Dispatcher Switching**을 하는 방법?

- 코루틴을 생성할 때 Dispatcher를 설정하는 것 만으로도 Dispatcher 전환 가능

```kotlin
CoroutineScope(Dispatchers.Main).launch {

	// 1. 데이터 입출력 : IO Dispatcher
	val deferredArray: Deferred<Array<Int>> = async(Dispatchers.IO) { 
		println(1)
		arrayOf(3, 1, 2, 4, 5) // 마지막 줄 반환
	}

	// 2. Sorting = 많은 CPU 작업 : Default Dispatcher
	val sortedDeferred = async(Dispatchers.Default) { 
		val value = deferredArray.await()
		value.sortedBy { it }
	}

	// 3. 설정하지 않으면 : 부모 Dispathcer = Main Dispatcher
	// TextView 셋팅 = UI 작업 : Main Dispatcher
	val textViewSettingJob = launch {
		val sortedArray = sortedDeferred.await()
		setTextView(sortedArray)
	}
}
```

- launch나 async로 job을 실행시켰으면, 멈추는 것 또한 가능
- `cancel()`

```kotlin
class MainActivity():AppCompatActivity(){

    override fun onCreate(savedInstanceState: Bundle?){
    
    	...
        
        val cancelTest = CoroutineScope(Dispatchers.Default).launch {
            val doJob = launch{
                for(i in 0..1000)
                    Log.e(TAG, "Coroutine Test doing Job1 $i")
            }
        }

        binding.button.setOnClickListener{
            cancelTest.cancel()
        }
    }

// cancelTest를 취소시킴 -> 
// doJob이 진행하고 있던 작업도 cancelTest 코루틴이 취소되면서 함께 취소 됨
```

- 하나의 코루틴에는 여러개의 launch 블록이 있을 수 O
- But, 이렇게 launch 된 작업들은 모두 새로운 코루틴으로 분기가 되어 실행되기 때문에 순서가 없음
- 정 순서를 정해야할 때 → `join()` 으로 순차적으로 실행 O

```kotlin
CoroutineScope(Dispatchers.Default).launch {
            launch {
                for (i in 0..5) {
                    delay(500)
                    Log.d("COROUTINE", "$i")
                }
            }.join()
            launch {
                for (i in 6..10) {
                    delay(500)
                    Log.d("COROUTINE", "$i")
                }
            }
        }
// 0 1 2 3 4 5 6 7 8 9 10
```

---

### 코루틴의 상태 관리

[[Coroutine] 7. Coroutine Job 상태 관리하기 : New, Active, Completed, Cancelling, Cancelled](https://kotlinworld.tistory.com/146?category=973476)

[[Coroutine] 8. Coroutine Job의 상태 변수 isActive, isCancelled, isCompleted 알아보기](https://kotlinworld.tistory.com/147?category=973476)

#### **Job의 상태**

```
💡 Job의 상태는 **생성 - 실행 중 - 실행 완료 - 취소 중 - 취소 완료** 총 5가지
```

- `생성(New)` : Job이 생성됨
- `실행 중(Active)` : Job이 실행 중
- `실행 완료(Completed)` : Job의 실행이 완료됨
- `취소 중(Cancelling)` : Job이 취소되는 중. Job이 취소되면 리소스 반환 등의 작업을 해야 하기 때문에 취소 중 상태에 있음
- `취소 완료(Cancelled)` : Job의 취소가 완료됨

- **Job을 생성하고 실행하는 방법**
    - `launch`를 통한 Job의 생성 및 실행
    - launch에 `CoroutineStart.LAZY` 옵션을 추가하여 Job이 바로 실행되지 않게 만들기
    - CoroutineStart.LAZY로 생성된 Job을 시작하는 방법 : `start()`, `join()`

> 하지만 Job이 실행 완료가 되지 않는다면?
> 
- Job이 항상 실행 성공하여 실행 완료가 되는 것은 아님 → **중간에 취소될 수 있어야 함**
- 또한 **취소에 대한 Exception 핸들링**이 필요!

- **cancel()을 이용한 Job의 취소**
    - `cancel()`에 cancel된 원인을 넣고 원인을 출력 가능(생략 가능)
    1. cancel()에 두가지 인자 `message: String` / `cause: Throwable` 을 넘겨서 취소 원인을 알릴 수 있음
    2. 또한 Job에 `getCancellationException()` 메소드를 사용해서 취소 원인을 알 수 있게 됨
    - cancel시 넘겨지는 Exception의 종류는 `CancellationException`으로 고정! cancel시 넘긴 Throwable은 반영되지 않음.

```kotlin
suspend fun main() {
	val job = CoroutineScope(Dispathers.IO).launch {
		delay(1000)
	}

	job.cancel(
		"Job Cancelled by User", 
		InterruptedException("Cancelled Forcibly")
	) // 원인 알림

	println(job.getCancellationException()) // 원인 출력

	delay(3000)
}

/*
출력
java.util.concurrent.CancellationException: Job Cancelled by User

Process finished with exit code 0
*/
```

- cancel 되었을 때 동작 Handling 하기
    - Job이 취소 완료 될 때 `invokeOnCompletion` 내의 메소드가 호출되는 데, 이를 활용하여 에러 핸들링 가능
    - throwable은 InterruptedException을 넘기더라도 `CancellationException`으로 잡히는 것을 볼 수 있음

```kotlin
suspend fun main() {
	val job = CoroutineScope(Dispathers.IO).launch {
		delay(1000)
	}

	// 취소된 원인 handling
	job.invokeOnCompletion { throwable -> 
		println(throwable)
	}

	job.cancel(
		"Job Cancelled by User", 
		InterruptedException("Cancelled Forcibly")
	) // 원인 알림

	delay(3000)
}

/*
출력
java.util.concurrent.CancellationException: Job Cancelled by User

Process finished with exit code 0
*/
```

- But → invokeOnCompletion은 Job이 **취소완료** 되었을 뿐만 아니라, **실행 완료** 되었을 때도 실행되는 문제
- 취소 없이 **실행 완료되면 throwable에 null**이 나옴 (Job의 상태 변수와 관계)
- 다음과 같이 Handling 가능

```kotlin
suspend fun main() {
	val job = CoroutineScope(Dispathers.IO).launch {
		delay(1000)
	}

	// 취소된 원인 handling
	job.invokeOnCompletion { throwable -> 
		when(throwable) {
			is CancellationException -> println("Cancelled")
			null -> println("Completed with no error")
		}
	}

	delay(3000)
}
```

#### Job의 상태 변수

```
💡 Job의 3가지 상태 변수 : isActive - isCancelled - isCompleted
```

- isActive : Job이 실행중인지 여부 표시
- isCancelled : Job이 cancel 요청 되었는지 여부 표시
- isCompleted : Job이 실행 완료 되었거나 cancel이 완료 되었는지 표시

- Job을 생성 → 실행 중 상태로 바로 넘기지 않기 위해서, **CoroutineStart.LAZY** 옵션으로 `생성`하여 상태변수를 확인해보기
    - Job이 CoroutineStart.LAZY로 생성되면, Job은 **생성됨(New) 상태에 머뭄.** 이때는 세가지 상태변수 모두 **false**이다.
    - start() 혹은 join()을 통해 Job이 **실행 중 상태**로 바뀌면 **isActive가 true**가 된다.

<img width="539" alt="Untitled 16" src="https://user-images.githubusercontent.com/85485290/187021666-be18b373-c001-4cbd-bf30-ee30f9823b38.png">

- Job의 `cancel`이 호출되었을 때 상태변수 확인해보기
    - Job은 취소 중 상태로 바뀜. `isCancelled는 true`로 바뀜 → 취소 중인 상태에서는 취소가 완료된 것은 아니므로 isCompleted는 false
    - 취소가 완료 되면 `isCompleted가 true`로 바뀜 → `invokeOnCompletion`은 isCompleted의 상태를 관찰하는 메소드로 **isCompleted가 false→ true로 바뀔 때 호출**됨 (그래서 앞에서 취소가 완료 되었을 때도 호출되었던 것, null 출력)

<img width="653" alt="Untitled 17" src="https://user-images.githubusercontent.com/85485290/187021668-014fe778-5a88-4a15-a710-7df475132522.png">

- Job이 `완료`되었을 때 상태 변수 확인해보기
    - Job이 완료되면 **isActive가 false**로 바뀌며, **isCompleted가 true**로 바뀜
    - isCompleted가 false → true로 바뀌었으므로, cancel과 마찬가지로 invokeOnCompletion을 통해 설정 된 람다식 호출O
	
<img width="640" alt="Untitled 18" src="https://user-images.githubusercontent.com/85485290/187021671-aae47ec1-024e-4d43-99cf-1bc0db643197.png">

- 모두 정리하면 다음과 같음

<img width="590" alt="Untitled 19" src="https://user-images.githubusercontent.com/85485290/187021673-6b7e4d07-2b1e-4fe7-906e-f363e32ea543.png">

---

### 코루틴 예외 다루기

[[Coroutine] 9. Coroutine Job에서 Exception이 발생했을 때 Exception Handling을 하는 방법](https://kotlinworld.tistory.com/148?category=973476)

- **Job의 Exception을 Handling 하는 방법**
    - `invokeOnCompletion`을 이용하는 방법
    - `CoroutineExceptionHandler`를 이용하는 방법
    
1. `invokeOnCompletion`을 이용한 Exception Handling
- But, 이 방법은 job이 완료가 되었을 때 호출되는 람다식에서 핸들링 하는 것
- 에러가 발생해서 job이 완료되어도, job이 모두 수행되어 람다식이 호출 되는 것이므로 에러를 핸들링 하는 더 general한 방법 필요!

```kotlin
suspend fun main() {
	val job = CoroutineScope(Dispathers.IO).launch {
		throw IllegalArgumentException()
	}

	// 취소된 원인 handling
	job.invokeOnCompletion { cause: throwable -> 
		println(cause)
	}

	job.start()

	delay(1000)
}

// java.lang.IllegalArgumentException 출력
```

1. `CoroutineExceptionHandler` 이용하기
- 코루틴 Job 내부에서 오류가 발생했을 때 에러를 처리할 수 있는 CoroutineContext → 에러가 발생했을 때 실행되는 람다식
- exceptionHandler는 CoroutineExceptionHandler를 구현하며 exception이 왔을 때, 받은 exception을 출력함

```kotlin
suspend fun main() {

	val exceptionHandler = CoroutineExceptionHandler { _, exception -> 
		println("CoroutineExceptionHandler : $exception")
	}

	val job = CoroutineScope(Dispathers.IO).launch(exceptionHandler) {
		throw IllegalArgumentException()
	}

	delay(1000)
}

/*
출력
CoroutineExceptionHandler : java.lang.IllegalArgumentException

Process finished with exit code 0
*/
```

- CoroutineExceptionHandler는 여러 Job에 붙일 수도 있음

```kotlin
suspend fun main() {

	val exceptionHandler = CoroutineExceptionHandler { _, exception -> 
		println("CoroutineExceptionHandler : $exception")
	}

	val job1 = CoroutineScope(Dispathers.IO).launch(exceptionHandler) {
		throw IllegalArgumentException()
	}

	val job2 = CoroutineScope(Dispathers.IO).launch(exceptionHandler) {
		throw InterruptedException()
	}

	delay(1000)
}

/*
출력
CoroutineExceptionHandler : java.lang.IllegalArgumentException
CoroutineExceptionHandler : java.lang.InterruptedException

Process finished with exit code 0
*/
```

- CoroutineException을 이용해 에러에 맞게 처리해보자
- when문을 이용하여 exception에 대한 타입 검사를 해서, 에러를 유형별로 처리할 수 있음!

```kotlin
suspend fun main() {

	val exceptionHandler = CoroutineExceptionHandler { _, exception -> 
		println("CoroutineExceptionHandler : $exception")
	}

	when(exception) {
		is IllegalArgumentException -> println("More Arguement Needed To Process Job")
		is InterruptedException -> println("Job Interrupted")
	}
	val job1 = CoroutineScope(Dispathers.IO).launch(exceptionHandler) {
		throw IllegalArgumentException()
	}

	val job2 = CoroutineScope(Dispathers.IO).launch(exceptionHandler) {
		throw InterruptedException()
	}

	delay(1000)
}

/*
출력
CoroutineExceptionHandler : java.lang.IllegalArgumentException
More Arguement Needed To Process Job
CoroutineExceptionHandler : java.lang.InterruptedException
Job Interrupted

Process finished with exit code 0
*/
```

---

### Supervisor Job 과 SupervisorScope

[[Coroutine] 12. SupervisorJob를 이용한 Coroutine Exception Handling](https://kotlinworld.tistory.com/153?category=973476)

- 코루틴 내부에서 코루틴이 수행될 수 있는데, 보통 자식 코루틴에 에러가 생기면 부모 코루틴 까지 취소되어 버린다  → 부모 코루틴이 취소되면 당연히 자식으로 있는 모든 코루틴이 취소된다,,,
- 이런 문제 방지를 위해 에러 전파 방향을 `부모에서 → 자식` 으로만 한정짓는게 `SupervisorJob`
- SupervisorJob은 `코루틴 컨텍스트`이고, `+ 연산자`로 다른 컨텍스트들과 혼합해서 쓸 수 있다!
- 매번 코루틴 컨텍스트에 SupervisorJob을 설정할 필요 없이, **특정 코루틴 블록 내부의 모든 코루틴에 SupervisorJob을 설정할 때 쓰는 것**이 `SupervisorScope`
- 자식 코루틴에서 Exception이 발생해도, 부모로부터 전파되지 않아, 부모 코루틴의 다른 자식 코루틴들은 영향을 받지 않고 계속 Job을 수행할 수 있다.
- 일반 Job 과의 차이점? → **한 코루틴이 실패해도 다른 코루틴이 취소되지 않는다**는 차이점이 있다.

---

### Dispatchers.Main.Immediate

- `Dispatchers.Main.immediate`와 `Dispatchers.Main`는 서로 다르다

```
💡 Immediate는 **job들의 실행 순서를 보장해준다!**
```

- ex) `Dispatchers.Main`
- 순서가 보장되지 않음

```kotlin
class BasicCoroutineFragment : Fragment(R.layout.fragment_coroutine_basic) {
    private val TAG = this::class.simpleName
    private val coroutineScopeMain = CoroutineScope(Dispatchers.Main)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        testDispatchersMain()
    }

    private fun testDispatchersMain() {
        println("$TAG Zero")
        coroutineScopeMain.launch {
            println("$TAG First")
        }
        println("$TAG Second")
    }
}

// Zero - Second - First
```

- ex) `Dispatchers.Main.Immediate`
- 순서대로 (Top→Down) 출력됨

```kotlin
class BasicCoroutineFragment : Fragment(R.layout.fragment_coroutine_basic) {
    private val TAG = this::class.simpleName
    private val coroutineScopeMainImmediate = CoroutineScope(Dispatchers.Main.immediate)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        testDispatchersMainImmediate()
    }

    private fun testDispatchersMainImmediate() {
        println("$TAG Zero")
        coroutineScopeMainImmediate.launch {
            println("$TAG First")
        }
        println("$TAG Second")
    }
}

// Zero - First - Second
```

- `lifecycleScope`나 `viewModelScope`는 `Dispatchers.Main.Immediate`에 바인딩 되어 있어서 **순서가 보장이 됨**
- Dispatchers.Main을 사용하면 순서가 보장되지 않아서, 코드 양이 길어지면 예외 상황이 발생할 수 있음 → 의도를 명확히 전달하려면 Immetdiate 사용!

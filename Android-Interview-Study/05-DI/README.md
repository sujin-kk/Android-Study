# 의존성 주입(DI)

[공식문서](https://developer.android.com/training/dependency-injection?hl=ko)

[의존성 주입이란 무엇이며 왜 필요한가?](https://kotlinworld.com/64)

### 의존성 주입 (Dependency Injection) 이란?

- 클래스에 대한 의존성을 인터페이스화 하여 코드 유연성이 증가한다!

```
클래스 간의 의존성을 클래스 '외부'에서 주입하는 것

쉽게 말해, 외부에서 의존 객체(인스턴스)를 생성하여 넘겨주는 것을 의미
```

- 그러면 의존성이 뭔데?
- 객체 지향 프로그래밍에서 클래스간의 ‘의존성이 있다' = ‘의존 관계가 있다’
- **한 클래스가 바뀔 때 다른 클래스가 영향을 받는것**
    - A 클래스에서 사용하던 B 클래스가 변경되었다.
    - A 클래스 내부에서 쓰던 B 클래스의 메소드가 사라짐 → 오류!
    - 해당 변경에 의존성을 갖는 모든 코드가 오류가 생길 것,, 안정성이 매우 떨어짐

[[Objects] 객체와 의존성](https://annajinee.tistory.com/39)

---

- 의존 관계를 없애는 것을 어떻게 해결하는데?
    - `인터페이스` 생성으로 해결할 수 있다!
    - 인터페이스의 메소드를 가져오는 것이므로, 실제 클래스에서 객체 내부의 메소드 변경이 있어도 그 객체를 쓰는 클래스에서는 오류가 날 일이 없다!

- 그러면 주입이 뭔데?
    - 클래스 **외부에서 객체를 생성**하여 해당 객체를 **클래스 내부에 주입**하는 것
    - 외부에서 인스턴스를 만들어 저장하는 공간을 **(IOC)** **Container** 라고 부름
    - 제어 권한이 더이상 computer가 아닌 container에 있고, 이것을 **IOC(Inversion of Control)** = **제어의 역전** 이라고 부름

- 그래서 뭐가 좋은데?
    - 공식문서에 따르면,,
    - **클래스 재사용 및 종속 항목 분리** - 종속 항목 구현을 쉽게 분리할 수 있다. 컨트롤 반전으로 인해 코드 재사용이 개선되었으며 클래스가 더 이상 종속 항목 생성 방법을 제어하지 않지만 대신 모든 구성에서 작동한다
    - **리팩토링 편의성** - 종속 항목은 API 노출 영역의 검증 가능한 요소가 되므로 구현 세부정보로 숨겨지지 않고, 객체 생성 타임 또는 컴파일 타임에 확인할 수 있다
    - **테스트 편의성** - 클래스는 종속 항목을 관리하지 않으므로 테스트 시 다양한 구현을 전달해 모든 사례를 테스트할 수 있다
    

---

[안드로이드에서 의존성 주입을 작성하는 방법 (종속 항목 수동 삽입)](https://developer.android.com/training/dependency-injection/manual?hl=ko)

1. **Constructor Injection (생성자 삽입)**
    - 생성자를 통해 의존 객체 전달
2. **Filed Injection (필드 삽입) = Setter Injection**
    - 안드로이드에서 Activity나 Fragment는 시스템이 인스턴스화 하기 때문에 생성자를 통한 주입이 불가능함
    - 그래서 참조가 필요한 클래스를 먼저 생성한 다음(lateinit) 의존성을 주입하는 방법이 있다!
    
    ```kotlin
    class Car {
        lateinit var engine: Engine
    
        fun start() {
            engine.start()
        }
    }
    
    fun main(args: Array) {
        val car = Car()
        car.engine = Engine()
        car.start()
    }
    ```
    

- 이렇게 객체(인스턴스)를 만들고 클래스 내부에 주입해주는 수동 DI 과정을 자동화 해주는 라이브러리 존재
    - **Dagger2**
        - 컴파일 시점에서 의존성 주입
        - 빌드가 완료된 파일은 어느 정도 의존성 주입에 대해 안정성 보장
        - 당연히 컴파일타임(빌드타임)이 오래걸림
        - 대신 런타임에 빠르게 동작하고 에러가 발생하지 않음
    - **Hilt**
        - 컴파일 시점에서 의존성 주입
        - 대거 기반으로 빌드되며, 대거의 장점을 가지되 안드로이드 jetpack 구성요소기 때문에 안드로이드 앱에 최적화
    - **Koin**
        - Kotlin DSL
        - 런타임 시점에서 의존성 주입
        - 빌드 타임은 짧지만, 런타임 에러가 발생할 수 있고 컴파일 시점의 오류 확인이 어려울 수 있음

---

### Hilt

- `@HiltAndroidApp`
    - Application 클래스에 붙여준다.
    - 애플리케이션의 기본 클래스를 비롯하여 Hilt 코드 생성을 트리거 해준다.
    - Application 객체의 수명 주기에 연결되며, 이 객체는 앱의 상위 구성요소이므로 다른 구성요소는 Application 에서 제공하는 종속 항목에 접근할 수 O
    
    ```kotlin
    @HiltAndroidApp
    class ExampleApplication : Application() { ... }
    ```
    

- `@AndroidEntryPoint`
    - 안드로이드 최상위 컴포넌트
    - 이 어노테이션을 붙여준 클래스에 Hilt가 종속 항목을 제공할 수 있다.
    - Activity나 Fragment에서 Hilt를 사용하고 싶으면(=인스턴스를 주입하고 싶으면) 붙여준다.
    - Android Lifecycle을 따르는 종속 컨테이너를 생성하고, 인스턴스를 주입하는 것!
    
    ```kotlin
    @AndroidEntryPoint
    class ExampleActivity : AppCompatActivity() { ... }
    ```
    
    - Hilt가 지원하는 Android class는 다음과 같다.
        - `Application`(`@HiltAndroidApp`을 사용하여)
        - `ViewModel`(`@HiltViewModel`을 사용하여)
        - `Activity`
        - `Fragment`
        - `View`
        - `Service`
        - `BroadcastReceiver`
        
- `@Inject`
    - 이 어노테이션으로 인스턴스를 주입할 수 있음
    - `@Inject` 어노테이션이 붙은 필드들에 인스턴스가 주입된다!
    - 주의 : **private한 필드에는 주입되지 않음**
    - Hilt는 Activity의 종속 컨테이너에서 빌드한 인스턴스를 사용해서 onAttach() 수명 주기 메소드 내에서 필드에 주입한다!
    
    ```kotlin
    @AndroidEntryPoint
    class ExampleActivity : AppCompatActivity() {
    
      @Inject lateinit var analytics: AnalyticsAdapter
      ...
    }
    ```
    
    - 필드를 주입하는 수명 주기 콜백 관련해서는,,
    
    [hilt-lifetimes](https://developer.android.com/training/dependency-injection/hilt-android?hl=ko#component-lifetimes)
    
- Hilt에 결합 정보 제공하기 (Binding 정의)
    - Hilt가 알고있는 서로 다른 타입들의 인스턴스를 제공하는 방법을 알려주는 것 → `바인딩(Binding)`
    - 필드 삽입을 실행하려면, 해당 필드(위에서는 AnalyticsAdapter)에서 필요한 종속 항목의 인스턴스를 제공하는 방법을 알아야하는데, **‘생성자 삽입'** 으로 해결할 수 있음
    - 클래스의 생성자에서 `@Inject` 를 사용하여 클래스에 인스턴스를 제공하는 방법을 Hilt에 알려준다!
    - 어노테이션이 붙은 클래스 생성자의 매개변수는 그 클래스의 종속 항목임
    - 즉, AnalyticsAdapter 클래스에는 AnalyticsService가 종속항목으로 있는 것!
    
    ```kotlin
    class AnalyticsAdapter @Inject constructor(
      private val service: AnalyticsService
    ) { ... }
    ```
    
- 생성자 삽입이 안되는 경우에는?
    - 예) 인터페이스에는 생성자를 삽입할 수 없음
    - 외부 라이브러리 클래스 등 우리가 건드릴 수 없는 유형도 생성자 삽입이 안됨
    - **‘Hilt 모듈’** 사용 → 결합정보 제공!
    - `@Module` 로 클래스에 지정할 수 있음 → 특정 유형의 인스턴스를 제공하는 법을 알려줌
    - `@InstallIn` 로 각 모듈을 사용하거나 설치할 Android 클래스를 Hilt에 알려야 함 = 어떤 안드로이드 컴포넌트와 바인딩 할 수 있는지
    - Hilt에서 주입할 수 있는 Android 클래스마다 연결할 수 있는 **Hilt 컴포넌트**가 있음
        - 예) Activity 컨테이너는 ActivityComponent와 연결되며 Fragment는 FragmentComponent와 연결됨

- `@Binds`

- `@Provides`
    - 인터페이스 뿐 아니라 외부 라이브러리에서 제공되는 클래스는 우리가 소유하고 있지 않음 (ex. Retrofit, OkHttpClient, Room Database 등)
    - 또 Builder 패턴으로 인스턴스를 생성해야 하는 경우도 생성자 삽입이 불가능
    
- `@EntryPoint`
    - Hilt가 지원하지 않는 클래스에 필드를 주입해야 할 수도 있음
    - 위 어노테이션으로 진입점 만들 수 O

---

### DI의 동작원리

- 스프링 DI 동작원리

[[Java 이야기] Spring DI의 동작원리(feat.Reflection)](https://better-dev.netlify.app/java/2020/08/15/thejava_7/)

- 스프링에서 DI 라이브러리를 사용할 때, 우린 인스턴스를 생성한 적이 없는데 Null이 되지 않음
    - WHY?? → Reflection에 대한 이해 필요
    
- `Reflection`
    - Reflection 이란?
    
    ```kotlin
    구체적인 클래스 타입을 알지 못해도 그 클래스의 메소드, 타입, 변수들에
    접근할 수 있도록 해주는 자바 API
    
    Reflection API 라고도 함
    ```
    
- JVM 에서 클래스 인스턴스의 생성주기는 다음과 같다.
    1. ClassLoader에 클래스 정보 로딩
    2. Heap 메모리에 클래스 인스턴스 할당

→ 클래스를 인스턴스화 하여 생성된 객체를 핸들링 하는 것!

- 이 클래스를 인스턴스화 하려 클래스에 접근할 때 리플렉션으로 해결할 수 있는 것!

- 그러나 안드로이드의 Dagger, Hilt, Koin 등은 리플렉션을 사용하지 않아 속도가 빠르다고 하는데,,
- 안드로이드 DI 동작원리

[DI는 왜 쓸까? 또한 어떻게 작동될까?](https://sungbin.land/di%EB%8A%94-%EC%99%9C-%EC%93%B8%EA%B9%8C-%EB%98%90%ED%95%9C-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%EB%90%A0%EA%B9%8C-482627090a1e)

- 안드로이드의 `super.onCreate`에서 주입이 어떻게 이루어질까?

### 안드로이드에서의 의존성 주입 모범 사례

[[DI][번역] 안드로이드에서 의존성 주입](https://velog.io/@sana/DI%EB%B2%88%EC%97%AD-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C%EC%97%90%EC%84%9C-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85)

- 주의점
- DI를 구현할 때, 단지 외부에서 파라미터로 객체를 넘겨줬다고 해서, 즉 주입해줬다고 해서 다 DI가 아니다!
- 주입받는 메서드 파라미터가 이미 특정 클래스 타입으로 고정되어 있다면 DI가 일어날 수 없음.
- DI에서 말하는 주입은 다이나믹하게 구현 클래스를 결정해서 제공받을 수 있도록 **인터페이스(또는 추상클래스) 타입**의 파라미터로 이뤄져야 함!

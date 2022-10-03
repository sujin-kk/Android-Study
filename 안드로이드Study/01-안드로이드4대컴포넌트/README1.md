# 안드로이드 4대 컴포넌트

<aside>
💡 안드로이드 앱을 구성하는 데 필요한 4가지 요소

</aside>

- 각 컴포넌트는 **독립적인 형태**로 존재하며 **고유한 기능을 수행**하고 **인텐트(Intent)**를 통해 상호작용한다.

![image](https://user-images.githubusercontent.com/85485290/193570328-468f4275-9ea1-4570-97b1-3a57d8efcd83.png)

---

## 0 - Intent(인텐트)

<aside>
💡 4대 컴포넌트끼리 유기적으로 정보전달을 가능하게 해주는 전달수단 → (작은 범위) 액티비티와 액티비티 사이를 묶어주는 객체

</aside>

- 컴포넌트끼리 통신하기 위한 메세지 시스템

- 명시적 인텐트 (Explicit Intent)
    - 의도가 명확하다. **어디 클래스로 보낼지 지정되어있음.** 액티비티 전환 등

- 암시적 인텐트 (Implicit Intent)
    - 의도가 불명확하다. 해당 액션에 대해서 정확히 실행할 액티비티에 대한 보장이 없다. **액션 정보를 가지고 있음(이런 액션을 취할거니깐 너가 정해줘)**

```kotlin
val intent = Intent(Intent.ACTION_DIAL) //ACTION_DIAL 액션이 있는 앱과 Uri가 tel:로 가능한 액티비티를 찾아서 실행을 해줘야함,
val TEST_DIAL_NUMBER = Uri.fromParts("tel", "5551212", null)
intent.setData(TEST_DIAL_NUMBER)
startActivity(intent)
```

가장 간단한 사용 예제 → startActivity(intent)를 통한 화면 전환

**Q. 화면 전환 이외에 인텐트를 활용해 본 경험을 말해주세요.**

A. putExtra, getExtra 등을 통해 다른 액티비티로 필요한 데이터를 전송할 때 인텐트를 활용하였습니다.

컴포넌트가 어떤 액션과 데이터를 처리할 수 있는지에 대한 정보를 intent filter에 기록하고 암시적 인텐트를 수신하여 작업을 수행하였습니다. 

→ 사용자 기기에 등록된 모든 앱 정보를 가지고와서 리스트로 띄우고 실행할 수 있는 앱을 실습하였습니다.

---

## 1 - Activity (액티비티)

<aside>
💡 사용자가 Application과 상호작용하며 실제로 포그라운드에서 사용자에게 보이는 화면 (액티비티 = 화면 하나에 대응)

</aside>

- Application에 화면이 하나도 없으면 사용자와 상호작용 할 수 없으므로 **적어도 하나의 액티비티는 반드시 필요하다.**
- 즉, Application은 한 개 이상의 액티비티들로 구성되어 있으며, 이 액티비티 들은 느슨하게 묶여있다.
- **다른 Application의 액티비티**를 인텐트를 통해 불러올 수 있다. (Ex. 안드로이드 기본 캘린더 인텐트 등)
- 액티비티는 입출력 기능이 없기에, **하나 이상의 View 또는 ViewGroup을 가지는 데**, View는 눈에 보이는 텍스트, 버튼, 이미지 등을 의미하고 ViewGroup은 레이아웃 등이 있다.

- **setContentView** 메소드
- 액티비티가 생성될 때 마다 호출되며, 액티비티 안에 뷰를 배치하는 명령

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
}
```

- Applicaiton에 포함 된 **모든 액티비티는 반드시 manifest에 등록**해야 한다. (manifest에 등록되지 않은 액티비티는 존재하지 않는 것으로 취급)

### Activity State

- 액티비티는 여러 상태중의 하나의 상태를 나타낼 수 있음.
    - **Starting** : 로딩 중에 있으며, 아직 전체가 로딩되지는 않았음.
    - **Running** : 로딩이 되었고, 스크린 상에 보여짐.
    - **Paused** : 부분적으로 가려지거나, 포커스를 잃은 상태로, shut down은 되지 않음 (화면에는 보임)
    - **Stopped** : 더 이상 활성화 상태는 아니지만, 메모리에는 있는 상태 (화면에 안보임)
    - **Destroyed** : Shut down 되었고, 더 이상 메모리에 없음.

- 액티비티들 간의 상태 전환은 callBack 함수 호출 이벤트에 의해 표현

![image](https://user-images.githubusercontent.com/85485290/193570388-823c9ff7-fcc2-485b-af1d-e33b8e16df7f.png)

---

### Activity Lifecycle

![image](https://user-images.githubusercontent.com/85485290/193570445-0411dcee-c5e4-4817-abb2-2dfc5c1400c9.png)


- `onCreate()`
    - **Activity가 처음 생성될 때 단 한번만 호출** (액티비티의 생성자처럼 생각), 다시 불려질 수 없음(비정상적인 종료 후 재시작의 경우 제외)
    - 화면을 만드는 작업 → setContentView() 메소드 호출하면 됨
    - setContentView()는 Running 상태 전에만 호출해주면 됨 (onStart()에서도 호출 가능)
    - 사용자 인터페이스 초기화 할 때 사용, Listener 초기화 등
    - 액티비티 객체를 생성하고 메뉴, 레이아웃, 이미지 같은 리소스들을 로딩
    - onCreate 이후에 액티비티가 존재할 수 있음
    - 실행 완료 시 onStart() / onResume()을 호출
    
- `onRestart()`
    - Activity가 stop 되었다가 다시 시작할 때 호출 됨
    - 일반적으로 사용되는 것은 아님 (onResume 을 더 많이 사용)
    
- `onStart()`
    - 사용자에게 Activity가 화면에 보여지기 직전에 호출
    - 활동이 ‘재개됨' 상태가 됨
    - 실행 완료 시 onResume()을 호출
    
- `onResume()`
    - 액티비티가 시작되고, 사용자와 상호작용을 할 수 있을 때 호출 → 이때부터 사용자가 앱을 사용할 수 있는 상태가 된다.
    - 전화가 오거나 기기의 화면이 꺼졌을 때 등 → 앱의 포커스가 끝날 때까지 이 상태에 머무르고, 다시 앱으로 돌아올 때 무조건 호출된다. → 액티비티가 다시 호출될 때 무조건 하면 되는 작업들
    - onResume()을 거쳐야 화면에 액티비티가 보임 (화면에 직접 출력), 이 함수를 호출해야만 Running 상태가 되는 것임
    - 액티비티가 Pause → Running 상태로 돌아올 때 onStart()가 아닌 **onResume()을 거치게 됨**

- `onPause()`
    - 다른 액티비티를 표시하기 직전에 호출됨
    - 화면의 일부분이 보여지지 않는 상태에 호출 = 액티비티가 여전히 부분적으로 보여질 때
    - onPause()를 거치면 화면에 안보임
        - Ex. A화면 → B화면 전환 될 때, B화면이 보이기 직전에 A화면은 onPause() 호출 → Pause상태 → Stopped 상태로 넘어갈 수 있음
        - Ex. 화면 실행 중, 다이얼로그가 뜰 때, 이 화면은 Pause 상태로 전환
    - 데이터 저장, 네트워크 호출 등의 resource를 많이 사용하는 작업 처리를 해서는 안된다. → 아주 잠깐 실행되기 때문에 완료되지 못할 수 있어서!
    - 상태를 저장하고 싶을 때 onPause()에서 값 저장 가능
    - Activity가 다시 시작되면 onResume() 호출, 완전히 중지 시 onStop() 호출
    
- `onStop()`
    - 액티비티 전부가 더이상 화면에 보여지지 않을 때 = 더이상 사용 되지 않을 때
        - Ex. 사용자가 다른 액티비티를 시작할 때
        - Ex. 사용자가 앱에서 전화를 받는 경우
    - 앱은 여전히 실행되고 있지만, 액티비티가 아닐 경우 호출
    - 주의할 점은 메모리 부족 시 호출 되지 않을 수 있음
    - onStop() 전에는 항상 onPuase()가 호출됨
    - 데이터베이스에 Commit하는 것과 같은 부하가 큰 강력한 종료 작업을 수행할 때 사용
    - Activity 실행 종료 시 onDestroy() 호출
    
- `onDestroy()`
    - 앱 전체가 종료되고 메모리에서 unload될 때 호출 = Activity가 완전히 끝났을 때 호출
    - finish() 호출 시에 또는 Activity 제거 시에 호출된다.
    - 언제 호출될 지 정확하게 알 수 없기에, 예상 가능한 시기에 호출 되는 onPause나 onStop을 더 많이 사용
    

### 생명주기를 왜 알아야 하는가?

```
onCreate에서 setContentsView()를 호출 되는 것을 알 수 있다.
뷰에 보이는 것은 처리했지만 내가 만든 앱을 쓰다가 전화가 오거나,
잠시 카톡을 보내고 오고 싶을 때가 있다.
하지만 우리의 앱은 API를 사용하여 외부 서버에서 실시간으로 데이터를 받아오는 리스트 뷰가 있다고 해보자. 
그 사이에 데이터가 새로운것들이 추가가 됐다.
하지만 우리는 데이터 받아오는 코드를 onCreate()에 구현을 해놓아버렸다. 
전화 받고 온다면 onCreate()는 호출되지 않을 것이다. 
따라서 우리는 onStart, onResume에서 변화를 체크해주는 작업을 해주면 좋을 것 같다.
```

```
다른 예로 종료시에 현재까지 하던 작업을 기억하고 싶다면, 
예를 들어 로그인 기록(사용자 데이터), 
사용하던 페이지나 작업을 기억하고 싶다(데이터 베이스나, sharedPreference 등). 
그것을 언제 기록할 것인가? onCreate에서? 어려울 것이다. 
종료되는 시점의 pause, stop, destroy를 잘 활용한다면 
체계적인 어플리케이션을 만들 수 있을 것이다, 

onSaveInstanceState를 잘 활용하면 아주 좋다! -> Bundle 이용
sharedPreference를 이용해서 onPause, onResume에 대해서 저장하고 불러오는 방식도 있다.
```

### Activity의 상태 저장 및 복원에 사용되는 콜백함수

- 안드로이드 시스템은 액티비티의 상태를 저장해야 하는 경우 **onSaveInstanceState()** 콜백 메소드를 호출
    - onResume()을 호출하고 Running → Pause 상태로 바로 가지 않음
    - 액티비티 상태 저장을 위해 onResume() → onSaveInstanceState() → onPause() 순서로 호출
    - Pause 상태 되기 전에 데이터 저장
    
- 복구해야 하는 경우에는 **onRestoreInstanceState()** 콜백 메소드를 호출
    - Pause → Stop이 되어 다시 onRestart() → onStart() 한 다음에 onResume()을 호출하여 Running 상태로 가기 전에 onRestoreInstanceState() 를 호출
    - 저장해 놓은 데이터를 불러온 뒤 Running 상태 전에 불러옴
    
- Finish() 메소드를 강제적으로 호출하는 경우에는 이 두개의 콜백 메소드 호출 X

- Bundle : Key와 Value로 구성된 Map 형태의 데이터 타입 → 여러가지 타입의 값을 저장할 수 있다.
    - 보통 saveInstanceState 등에 많이 쓰이며, intent에 데이터를 넘기는 용도로도 사용한다.

### Back Stack을 이용한 화면 관리

![image](https://user-images.githubusercontent.com/85485290/193570973-5967185d-19da-4576-a5d3-a62f7ad96318.png)

→ 화면전환을 여러번 했을 때 finishActivity를 안해준 경우, 뒤로가기를 하면 이전 액티비티가 나타나는 문제가 발생할 수 있다.

**intent로 화면전환 시에 flag 값을 실어서 스택 관리를 할 수 있다.**

- **FLAG_ACTIVITY_NEW_TASK**

새로운 Task를 생성하여 그 Task 안에 액티비티를 추가할 때 사용한다. 단, 기존에 존재하는 Task들 중에 생성하려는 액티비티와 동일한 affinity를 가지고 있는 Task가 있다면 그곳으로 액티비티가 들어가게 된다.

- **FLAG_ACTIVITY_SINGLE_TOP**

시작되고 있는 액티비티가 백 스택 맨 위에 있는 액티비티인 경우, 해당 액티비티의 새 인스턴스를 생성하는 대신 기존 인스턴스가 onNewIntent()에 대한 호출을 받는다. 

singleTop과 동일 동작

- **FLAG_ACTIVITY_CLEAR_TOP**

시작되고 있는 액티비티가 이미 현재 Task에서 실행 중인 경우, 해당 액티비티의 새 인스턴스를 시작하는 대신 그 위에 있는 모든 액티비티가 소멸되고 이 인텐트는 해당 액티비티의 재개된 인스턴스로, onNewIntent()를 통해 전달된다.

### Task Affinity

<aside>
💡 한 액티비티를 실행할 때 어떤 태스크에 속하게 할 지 지정하는 것

</aside>

- 액티비티마다 정해진 taskAffinity가 존재하는데, 기본적으로 앱의 패키지 명을 따른다.
- 예를들어, 내가 만든 앱의 activity의 taskAffinity를 유튜브로 설정하면 유튜브의 task로 내가 만든 앱의 액티비티가 실행된다.

- **Menifest에 액티비티마다 launch mode 속성을 추가하는 방법 → 특정 액티비티에 대해서 flag가 항상 같을때**
    - **standard** : 평소에 쓰던대로
    - **singleTop** : 스택 최상위에 있으면 인스턴스 안 만든다. onNewIntent() 후 onResume()호출
    - **singleTask** : 테스크를 새로 만들어서 액티비티를 추가한다. 그 액티비티 다시 호출하면 해당 Task에서만 onNewIntent()호출
    - **singleInstance** : singleTask와 같지만, 액티비티를 다시 호출하면 똑같은 액티비티라도 새로운 Task에서 호출


---

## 2-Service

<aside>
💡 화면을 가지지 않고 백그라운드(우리 눈에 보이지 않는 곳)에서 작업을 수행하는 프로세스

</aside>

- 액티비티와 반대로 사용자와 직접적인 상호작용을 하지 않는다.
- Application이 종료되어도 (화면이 보이지 않아도) **백그라운드에서 동작**할 수 있다.
- Ex. 음악 앱 같은 경우는 화면을 꺼놔도 백그라운드에서 계속 음악을 재생 시킨다.
- Ex. 타이머 앱 같은 경우는 화면을 꺼놔도 타이머는 계속 작동되므로 서비스 기능이다.
- 다른 component를 service에 binding하여 service와 상호작용 할 수 있다.
    - service는 network transcaction을 처리하고
    - 음악 재생
    - File I / O
    - Content Provider와 상호작용
    - 등을 모두 백그라운드에서 수행할 수 있다.

<aside>
💡 service는 자신의 hosting process의 main thread에서 실행된다. 절대 새로운 process 또는 thread가 생성되는것이 아니다.

</aside>

<aside>
💡 **따라서 blocking operation이 수행되어야 한다면 별도의 thread를 생성해야 한다.**

</aside>

![image](https://user-images.githubusercontent.com/85485290/193570714-fe882e25-229f-4183-8b33-824712395c75.png)

---

## 3 - BroadCast Receiver (브로드캐스트 리시버, 방송 수신자)

<aside>
💡 안드로이드 OS로부터 각종 이벤트와 정보를 받아오고, 이에 반응하여 처리하는 컴포넌트

</aside>

- 대부분 UI를 가지지 않으며, 수신기를 통해 디바이스의 상황을 감시하다가 이벤트가 발생하면 해당 이벤트에 맞게 정의해 둔 작업들을 수행하는 역할을 한다.
- Ex. 화면이 꺼지고 켜지거나, 네트워크 연결 해제, 배터리 부족, 전화 수신 등의 이벤트 정보를 수신한다.
- 브로드캐스트 리시버는 앱 하나에서 동작하는 것이 아니라 **안드로이드 시스템 전체에서 동작한다.**
- 안드로이드 시스템에서는 이벤트가 발생하면 해당 이벤트 정보를 기기 내의 모든 App에 Broadcast(방송)한다. App내에 이 메시지를 받아서 처리할 수 있는 BroadcastReceiver가 구현되어 있다면 해당 작업을 수행할 수 있다.
- 또한 하나의 App에서 브로드캐스트 리시버를 통해 이벤트를 수신받고, 브로드캐스트 리시버가 없는 다른 App에 동작을 수행하도록 할 수 있다.
- 브로드캐스트 리시버는 보통 action이라는 것을 통해서 각 이벤트를 구별하고, **이 action에 대응할 수 있는 Broadcast Reciever를 가진 App들만 작업을 수행할 수 있다.**


---

## 4 - Content Provider (콘텐트 프로바이더, 콘텐츠 제공자)

<aside>
💡 데이터를 관리하고, 다른 Application의 데이터를 제공해주는 컴포넌트

</aside>

- 데이터를 저장하고 불러와서 사용할 수 있는 시스템 (Ex. DB, 웹, 파일 시스템 등)
- 데이터들은 Content Provider가 허용하면 → 가져오는 것 뿐 아니라 수정도 가능하다.
- Content Provider를 통해 다른 App끼리 데이터를 공유하여 사용 가능 → 혹자는 서버와 비슷한 역할이라고도 함
- Ex. 카카오톡에서 우리가 기기에 저장한 연락처 데이터를 동기화하여 제공하는 것을 볼 수 있음, 지도 앱에서 현재 위치를 가져오는 것 등
- Content Provider를 사용하려면 권한을 획득해야 함 (Ex. 카카오톡에서 처음에 연락처에 접근해도 되냐는 권한 획득 요청을 함) → 보안 이슈 고려

### 핵심

- Content Provider는 앱의 데이터를 다른 앱에 전달하기 위한 component다.
- Contnet Resolver는 Content Provider에 접근하기 위한 class이다.

![image](https://user-images.githubusercontent.com/85485290/193570915-0227f5ac-36f7-46b9-9ec4-5176de1ac4a4.png)

- DB 정보를 얻고자 하는 Application에서 Content Resolver를 통해 Uri를 보낸다
- Content Provider는 Uri 를 parsing하여 필요한 CRUD 작업을 진행한다.
- Content Provider는 필요한 작업을 수행하고, 결과값(cursor 객체)를 Content Resolver에게 return 한다.

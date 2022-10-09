## Q. A액티비티 상태에서 B액티비티가 실행된다면 생명주기가 어떻게 되나요?

A. B 액티비티가 실행되어 전체 화면을 가리는 상태가 되면 A 액티비티는 정지상태가 되며 onPause → onStop을 차례로 호출한다. onStop이 호출 된 A 액티비티는 백스택으로 들어간다.

A : onPause()

B : onCreate() → onStart() → onResume()

A : onStop()

B 액티비티에서 뒤로가기 또는 finish()를 호출하여 A액티비티를 다시 실행할 경우 A액티비티는 onRestart → onStart → onResume을 통해 재실행된다.

A 액티비티의 launchMode → standard

A : onRestart() → onStart() → onResume()

B : onStop() → onDestroy()

A 액티비티의 launchMode → singleTop

A : onDestroy

A : onCreate() → onStart() → onResume()

B : onStop() → onDestroy()


## Q. 기본 다이얼로그가 show 되면 액티비티의 생명주기는 어떻게 되나요?

A. 아무런 메소드도 호출되지 않는다!


## Q. permission request dialog가 show 되면 액티비티의 생명주기는 어떻게 되나요? * stop까지 가는지 안가는지

A.  onPause 상태로 전환 → 다이얼로그 종료 시 다시 onResume 을 통해 재실행


## Q. 홈/화면 잠금 버튼을 눌렀을 때 액티비티의 생명주기는 어떻게 되나요?

A. onPause() → onStop()


## Q. 홈 버튼을 눌렀던 앱을 다시 누를경우의 생명주기는 어떻게 되나요?

A. onRestart() → onStart() → onResume()


## Q. 화면 회전을 했을 때 Activity의 동작에 대해서 설명해주세요.

A. 화면을 회전하게 되면 현재 액티비티가 소멸되고 새로운 orientation을 가진 액티비티가 생성되는데, 

onStop() → onDestroy() 후에 onCreate() → onStart() → onResume()을 호출한다.

화면이 회전되었을 때 데이터가 초기화 되는것을 막는 일반적인 방법은 onSavedInstanceState 메소드 또는 ViewModel을 사용하는 방식으로 유지할 수 있다.


## Q. global broadcast와 Local broadcast의 차이점이 무엇인지?

A. 우리가 사용하는 일반 broadcast는 global broadcast로, 핸드폰의 모든 receiver가 해당 broadcast를 수신하면 받을 수 있다. **하지만 Local broadcast는 sender와 receiver가 같은 app에 한정되어 구현된다.**


## Q. manifest로 receiver를 등록하는 방법과 context receiver의 차이점이 무엇인지?

A. 컨텐스트에 등록되면 컨텍스트가 유효한 동안에만 수신받을 수 있다. 앱에 등록되면 앱이 실행되는 동안 계속 수신받는다. - unregister를 안해주면 살아있음.
하지만 manifest에 등록되면 앱이 실행중이지 않는다면 앱을 실행하여 브로드캐스트를 받는다.


## Q. BroadcastReceiver에서 메세지 클릭시 앱에서 SMS를 받고, 문자보관함으로 이동하지 않게 하는 방법은 뭐가 있을까?

A. abortBroadcast()를 사용해야한다. 이것은 sendOrderedBroadcast사용시에만 사용할 수 있다. 우선순위도 그럼 최대로 높혀놔야겠지? 잘 되진 않는다.

- **안드로이드의 다양한 저장방법?**
    - shared preference  / datastore – 개인 기본 데이터를 키-값 쌍으로 저장
    - 내부 저장소 – 장치 메모리에 개인 데이터 저장
    - 외부 저장소 – 공유 외부 저장소에 공개 데이터 저장
    - SQLite 데이터베이스 – 구조화된 데이터를 개인 데이터베이스에 저장
    

- **Room을 어떤 상황에서 써야하는지 ? or 써본 경험?**
    - 스마트폰 내부에서 동작해야하는 소규모 데이터를 처리할 때 용이하다고 생각
    - (네트워크가 끊길 경우) 상태 보존 or 진행중인 사항에 대한 기록 필요
    - 외부 디비에서 데이터를 가져올 수 없는 상황, 네트워크에 연결되어 있지 않아도 일부 데이터를 보여줘야할때 
    
- **room 구성요소?**
    - 엔티티 / DAO / 룸데이터베이스

- **Room과 SharedPreferences 비교**
    - SharedPreferencse는 key와 value로 이루어져 있고 중복 key값이 들어올수 없다, 단순한 정보에 적합하다
    - Room은 중복 키값을 가지는 정보를 여러개의 행으로 저장할 수 있다, 방대한 양의 정보에 적합하다

- **Room과 SQLite 비교**
    - sqlite 는 쿼리문으로 직접써야하고 ?(물음표) 로 쿼리문 작성해주는게 불편하고 코드도 더러워진다. 그래서 ORM 방식에 다양한 기능을 제공해주는 Room 이 사용하기도 편리하고 좋다

- **디비에 데이터 삽입할 때 프라이머리 키가 겹치거나 해서 충돌이 날수도있음 어떻게 처리?**
    - `onConflict = OnConflictStrategy.REPLACE`  이렇게 하면 충돌시에 항목을 덮어쓰게 할 수 있음, IGNORE로 무시
    
![image](https://user-images.githubusercontent.com/85485290/193569342-904b7828-0585-41c4-b48f-cfacba0d1103.png)

    
- **DAO ↔ DTO?**
    - DAO(Data Access Obejct)는 db에 직접 접근하는 인터페이스로 데이터의 삽입,삭제,조회 등의 쿼리를 조작할 수 있음
    
    - DTO(Data Transfer Object)는 계층간의 데이터 교환을 위한 객체로, 주로 getter setter 메소드로 객체를 컨트롤함
    
- **room으로 데이터를 가져올 때 비동기적으로 처리하는 방법?**

[](https://developer.android.com/training/data-storage/room/async-queries?hl=ko)

```
Room에서 coroutine을 지원하면서 DAO에서 suspend function을 만들어 선언해 놓으면 해당 function들은 main thread에서 동작하지 않음!
-> 따로 백그라운드 쓰레드를 만들어서 접근할 필요 없는 것
```


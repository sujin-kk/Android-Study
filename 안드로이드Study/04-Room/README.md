# 05-Room

### Room이란?

[room 공식문서](https://developer.android.com/training/data-storage/room?hl=ko)

[안드로이드 Room 사용법](https://medium.com/%EC%8A%AC%EA%B8%B0%EB%A1%9C%EC%9A%B4-%EA%B0%9C%EB%B0%9C%EC%83%9D%ED%99%9C/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-room-%EC%82%AC%EC%9A%A9%EB%B2%95-1b7bd07b4cee)

```
안드로이드 기기 내장 DB에 저장하기 위해 사용하는 Jetpack 라이브러리
```

- **ORM(Object Relational Mapping)** 라이브러리 → DB 데이터를 Java/Kotlin 객체로 맵핑
- SQLite를 내부적으로 사용하고 있지만, DB를 구조적으로 분리하여 데이터 접근의 편의성을 높여주고 유지보수에 편리
- 다양한 **Annotation**을 통해 컴파일시 코드들을 자동으로 만들어주며 LiveData, RxJava와 같은 Observation 형태를 지원하고 MVP, MVVM과 같은 아키텍쳐 패턴에 쉽게 활용할 수 있음!

- SQLite
    - 어플리케이션의 효과적인 데이터 관리를 위해 파일 형식으로 데이터를 저장함
    - 소규모 데이터를 관리하는데 적합한 관계형 DB
    - 임베디드 기기, IoT, 응용 프로그램 파일 형식, 웹사이트(트래픽이 적거나 중간 정도인), 엔터프라이즈 데이터용 캐시 처리에 적합
   

#### SQLite를 쓰지 않고 Room을 쓰길 권장하는 이유✨

[Incrementally migrate from SQLite to Room](https://medium.com/androiddevelopers/incrementally-migrate-from-sqlite-to-room-66c2f655b377)

![Untitled](https://user-images.githubusercontent.com/85485290/189627013-8c52372e-8cb4-4677-bc73-977105d8ad80.png)


- SQLite는 컴파일시 쿼리에 대한 에러를 확인하지 못하지만, Room은 쿼리 유효성 검사를 할수 있음
- SQLite는 schema 가 변경이 될때 SQL 쿼리를 수동으로 업데이트 해야하지만 Room은 쉽게 업데이트가 가능
- SQLite는 ORM을 지원하지 않아서, 데이터를 객체로 변환하는 작업을 거쳐야함
- Room은 LiveData와 같은 옵저버 패턴의 데이터를 다룰 때 Observation 으로 생성하여 동작할 수 있지만 SQLite는 할 수 없음

---

- 매우 간단한 정보를 저장할 때
    - ex) 자동 로그인 여부 true/false를 저장하려 할 때?
    - **SharedPreferences / DataStore**
    
    [[Android Datastore] 1. Datastore을 사용해야 하는 이유 : SharedPreferences는 왜 대체되어야 하는가?](https://kotlinworld.com/342)
    
- 대량의 데이터를 처리
    - **Realm**
    - 속도가 빠르고 안정적, 비동기 지원이 되는 장점 but 앱 용량이 커진다는 단점

---

### Room의 주요 구성요소 3가지
    - Room Database
    - Entities
    - Data Access Objects(DAO)

<img width="500" alt="Untitled 1" src="https://user-images.githubusercontent.com/85485290/189627120-71711aa3-2387-4fdd-8b75-7b2a9fc264da.png">

#### 1. Entity
    - 개체
    - 관련이 있는 속성들이 모여 **하나의 정보 단위**를 이룬 것
    - 데이터베이스의 테이블같은 존재
    
    
    > Entity(개체)와 Object(객체)는 비슷해 보이지만 다른 의미를 가지고 있음.
    > 
    
    > *객체는 개체를 포함한 더 큰 개념이다! (객체 > 개체)*
    > 
    
    > *대상에 대한 **정보뿐만 아니라 동작, 기능, 절차 등을 포함하는 것이 객체**이다.*


    
- Entity 생성
    - 테이블의 이름은 따로 정하지 않으면 클래스 이름을 사용
    - 만약 테이블 이름을 정해주고 싶다면 `@Entity(tableName="userProfile")` 붙여줌

> ‘@’ : 어노테이션이란, 데이터를 설명하는 데이터
> 

```kotlin
@Entity
data class User (
    var name: String,
    var age: String,
    var phone: String
){
    @PrimaryKey(autoGenerate = true) var id: Int = 0
}
```

- `@Entity(tableName = “테이블 이름”)`
    - 테이블 이름을 지정.
    - 기본값은 Class 명과 동일.
    - Serializable을 implements하여 다른 클래스들 처럼 Intent에 담아서 이동O
    - 주의 : 대소문자를 구분하지 않음!
- `@PrimaryKey`
    - 기본 키 값을 지정.
    - 기본 키는 Unique하므로 중복 X
    - `autoGenerate=true`로 설정해주면 자동으로 Key값을 생성.
- `@ColumnInfo`
    - 테이블 컬럼 명을 지정.
    - 생략한다면 기본 값 = 멤버 변수의 변수명
- `@Ignore`
    - Database에는 저장하기 싫지만 이 클래스에는 있어야 하는 멤버변수가 있는 경우
    - @Ignore annotation을 사용하면 Database에 저장되지 않음!
    

#### 2. DAO

[Data Access Objects - DAO in Room](https://blog.mindorks.com/data-access-objects-dao-in-room)

- 데이터에 접근할 수 있는 메소드를 정의해 놓은 **인터페이스**
    - `@Insert` : 테이블에 데이터 삽입
    - `@Update` : 테이블의 데이터 수정
    - `@Delete` : 테이블의 데이터 삭제

```kotlin
@Dao
interface UserDao {
    @Insert
    fun insert(user: User)
 
    @Update
    fun update(user: User)
 
    @Delete
    fun delete(user: User)
}
```

- 삽입/수정/삭제 외의 다른 기능을 하는 메소드?
    - `@Query` : 어떤 동작을 할 건지 sql 문법으로 작성

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM User") // 테이블의 모든 값을 가져와라
    fun getAll(): List<User>
 
    @Query("DELETE FROM User WHERE name = :name") // 'name'에 해당하는 유저를 삭제해라
    fun deleteUserByName(name: String)
}
```

- `@Ignore`
    - 파라미터로 넘겨받은 데이터를 테이블에 저장.
    - `onConflict = OnConflictStrategy.REPLACE` 이렇게 하면 중복시에 항목을 덮어쓰게 할 수 있음.
    - 반면에 REPLACE 대신 `IGNORE`하면 기존데이터를 유지!
    

#### 3. Room Database
- 데이터베이스를 생성하고 관리하는 데이터베이스 객체 → `abstract class` 생성
    - version?
    - 앱을 업데이트하다가 entity의 구조를 변경해야 하는 일이 생겼을 때 **이전 구조와 현재 구조를 구분해주는 역할**
    - 만약 구조가 바뀌었는데 버전이 같다면 에러가 뜨며 디버깅 되지 않음
    - 처음 데이터베이스를 생성하는 상황이라면 그냥 1을 넣어주면 됨
- 하나의 데이터베이스가 여러 entity를 가진다면 arrayOf() 안에 콤마로 구분해서 entity 넣음!

```kotlin
@Database(entities = arrayOf(User::class, Student::class), version = 1)
abstract class UserDatabase: RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

- 데이터베이스 객체를 instance 할 때 **싱글톤으로 구현**하기를 권장
    - 여러 instance에서 액세스를 꼭 할 일이 거의 없고, 객체 생성에 비용이 많이 들어감

![Untitled 2](https://user-images.githubusercontent.com/85485290/189627496-d8b36a3c-f179-409d-a081-c76f2c9c1ec7.png)

- `companion object`로 객체를 선언해서 사용할 수 있음

```kotlin
@Database(entities = [User::class], version = 1)
abstract class UserDatabase: RoomDatabase() {
    abstract fun userDao(): UserDao
 
    companion object {
        private var instance: UserDatabase? = null
 
        @Synchronized
        fun getInstance(context: Context): UserDatabase? {
            if (instance == null) {
                synchronized(UserDatabase::class){
                    instance = Room.databaseBuilder(
                        context.applicationContext,
                        UserDatabase::class.java,
                        "user-database"
                    ).build()
                }
            }
            return instance
        }
    }
}
```

- 데이터 베이스 사용

```kotlin
var newUser = User("김똥깨", "20", "010-1111-5555")
 
        // 싱글톤 패턴을 사용하지 않은 경우
        val db = Room.databaseBuilder(
                applicationContext,
                AppDatabase::class.java,
                "user-database"
        ).build()
        db.UserDao().insert(newUser)
 
        // 싱글톤 패턴을 사용한 경우
        val db = UserDatabase.getInstance(applicationContext)
        db!!.userDao().insert(newUser)
```

- 오래걸리는 작업이라 바쁘니까 딴 얘 시켜! → 에러 발생?

> *Cannot access database on the main thread since it may potentially lock the UI for a long period of time*
> 

1. `allowMainThreadQueries()` 로 강제로 실행시키기
    
    → 나중에 문제 생길 수 있음!
    
2. **비동기 실행**
    
    → `Coroutine` 사용 가능!
    

```kotlin
// 싱글톤 패턴을 사용한 경우
        val db = UserDatabase.getInstance(applicationContext)
        CoroutineScope(Dispatchers.IO).launch { // 비동기 실행
            db!!.userDao().insert(newUser)
        }
```

- ROOM 사용 시 유용한 정보

[[번역] 안드로이드 Room 7가지 유용한 팁](https://medium.com/harrythegreat/%EB%B2%88%EC%97%AD-%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-room-7%EA%B0%80%EC%A7%80-%EC%9C%A0%EC%9A%A9%ED%95%9C-%ED%8C%81-18252a941e27)

- Room으로 Offline Cahce 구현

[Room을 이용해서 Offline Cache 구현하는 방법 정리 #Android](https://developer88.tistory.com/323)


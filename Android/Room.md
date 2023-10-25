## Room

안드로이드 Jetpack 구성 요소 중 하나로, 2018년 1월 22일에 <a href = "https://developer.android.com/jetpack/androidx/releases/room#1.1.0-alpha1">1.1.0- alpha 1</a>  버전으로 처음 공개됨

서버와의 통신이 불가능한 경우, 데이터를 캐싱하여 기기 내부에 저장하고 있다가 로컬 환경에서도 데이터를 제공 할 수 있게 함

#### 장점

- SQL 쿼리의 컴파일 시점 검증
- 어노테이션으로 보일러 플레이트 코드 최소화 및 반복성 감소
- 데이터베이스 마이그레이션 경로 간소화

> 위와 같은 사항들로 인해 안드로이드 공식 문서에서는 SQLite 대신 Room을 사용 할 것을 권장하고 있음

### Room 사용하기 - gradle

2023.10.24 기준 Room 최신 버전 2.6.0

``` kotlin
implementation("androidx.room:room-runtime:2.6.0")
annotationProcessor("androidx.room:room-compiler:2.6.0")
```

## Room 핵심 컴포넌트 파헤쳐보기

- **Database Instance**
- **Database Entity**
- **DAO (Data Access Object)**

해당 Database와 관련된 DAO 객체를 제공하고, 정의된 DAO 객체를 통해 데이터를 삽입하거나 삭제할 수 있음

<img width = "40%" src = "https://developer.android.com/static/images/training/data-storage/room_architecture.png"/>

### 간단하게 구현해보기 - Room Entity

``` kotlin
import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity
internal data class Recruitment(
    @PrimaryKey val recruitmentId: Long,
    @ColumnInfo("company_name") val companyName: String,
    @ColumnInfo("train_pay") val trainPay: Long,
    val military: Boolean,
)
```

-> Room database의 column 정의

- @Entity : 해당 클래스가 Room database의 entity임을 명시

  - Entity 어노테이션이 명시된 클래스는 최소 한 개의 PrimaryKey를 가지고 있어야 함

    ``` kotlin
    @Entity(primaryKeys = ["companyName", "trainPay"])
    data class User(
        val companyName: String?,
        val trainPay: String?,
    )
    ```

    - 어노테이션 내부 primaryKeys argument를 이용하여 PrimaryKey들을 정의 할 수도 있음

- @PrimaryKey : 해당 필드가 PrimaryKey임을 명시

  - Room Entity안에 최소 한 개 이상 포함되어야 함

- @ColumnInfo : 따로 해당 Column의 속성 명을 정의하고자 할 때 필드에 명시

  - Column의 이름은 기본적으로 변수명을 따라가지만 @ColumnInfo()로 정의할경우 Column name을 custom 할 수 있음

### Room DAO 정의하기

``` kotlin
import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.Query
import com.untitled.roompractice.entity.Recruitment

@Dao
internal interface RecruitmentDao {
    @Query("select * from recruitment")
    fun getRecruitments(): List<Recruitment>

    @Query("select * from recruitment where recruitmentId = (:recruitmentId)")
    fun getRecruitmentDetails(recruitmentId: Long): Recruitment

    @Insert
    fun addRecruitment(vararg recruitment: Recruitment)

    @Delete
    fun deleteRecruitment(recruitment: Recruitment)
}
```

- @Dao : 해당 인터페이스가 DAO (Data Access Object) 임을 명시
  - Dao는 interface 또는 abstract class로 정의될 수 있음
  - abstract class로 정의된 Dao의 경우 Database를 인자로 받는 생성자를 가질 수 있음

- @Query : 해당 method가 SQL query 문임을 명시
  - :{name} 형식으로 SQL 문 내부에서 함수 인자를 참조할 수 있음
- @Insert : 해당 method가 data insert 문임을 명시
- @Delete: 해당 method가 data delete 문임을 명시

### Room Database 정의하기

``` kotlin
import androidx.room.Database
import androidx.room.RoomDatabase
import com.untitled.roompractice.dao.RecruitmentDao
import com.untitled.roompractice.entity.Recruitment

@Database(
    entities = [Recruitment::class],
    version = 1,
)
internal abstract class DemoDatabase : RoomDatabase() {
    abstract fun recruitmentDao(): RecruitmentDao
}
```

Room Database 클래스는 아래와 같은 조건들을 충족해야함

- 클래스는 반드시 @Database 어노테이션으로 명시되어야 함
  - entities 인자에는 @Entity로 정의된 Database Entity들이 array 형태로 전달되어야 함
  - 반드시 abstract class여야 하며 RoomDatabase를 상속받아야 함
  - 각 Dao 클래스를 반환하는 인자가 없는 함수를 정의해야함

### Room Database 사용하기

``` kotlin
private val database by lazy {
    Room.databaseBuilder(
        context = this,
        klass = DemoDatabase::class.java,
        name = "demo-database"
    ).build()
}
```

**주의**

``` kotlin
CoroutineScope(Dispatchers.IO).launch {
      recruitmentAdapter = RecruitmentAdapter(database.recruitmentDao().getRecruitments())
      binding.recyclerViewRecruitment.adapter = recruitmentAdapter
}
```

- database에 query하는 작업은 항상 **worker thread**에서 이루어져야 함

  

### 발생한 이슈

``` markdown
Cannot find implementation for com.untitled.roompractice.database.DemoDatabase. DemoDatabase_Impl does not exist
```

- abstract class인 RoomDatabase의 implementation 클래스를 찾을 수 없음

``` kotlin
kapt("androidx.room:room-compiler:2.6.0")
```

- Java - annotationProcessor 추가
- Kotlin - kapt 추가

### 참고자료

https://developer.android.com/training/data-storage/room#kotlin

https://developer.android.com/training/data-storage/room/accessing-data

https://stackoverflow.com/questions/47274677/room-cannot-find-implementation
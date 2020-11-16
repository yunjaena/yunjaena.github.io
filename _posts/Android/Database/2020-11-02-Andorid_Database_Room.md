---
title: Android Database - Room
author: Yun-Jae Na
date: 2020-11-02 15:00:00 +0900
categories: [Android, Database]
tags: [Android, Android Database]
image : https://drive.google.com/uc?export=view&id=184o_vVVVd_epsuvEVPsuuORTPNCSn1gv
---

## Android Database Room

Room 은 SQLite에 대한 추상화 레이어를 제공하여 원할한 데이터베이스 엑세스를 지원하는 동시에 SQLite를 완벽히 활용한다. 다양한 Annotation을 통해 컴파일시 코드들을 자동으로 만들어주며 LiveData, RxJava와 같은 Observation 형태를 지원 또한 MVP, MVVM 등과 같은 아키텍쳐 패턴에서 활용하기 쉽다.

공식 문서는 해당 링크를 참조하면 된다.
- [https://developer.android.com/training/data-storage/room](https://developer.android.com/training/data-storage/room)

## Before Start

앱 또는 모듈의 **build.gradle** 파일에 라이브러리를 추가해준다.

```groovy
dependencies {
  def room_version = "2.2.5"

  implementation "androidx.room:room-runtime:$room_version"
  annotationProcessor "androidx.room:room-compiler:$room_version"

  // optional - RxJava support for Room
  implementation "androidx.room:room-rxjava2:$room_version"

  // optional - Guava support for Room, including Optional and ListenableFuture
  implementation "androidx.room:room-guava:$room_version"

  // optional - Test helpers
  testImplementation "androidx.room:room-testing:$room_version"
}
```

## Entity 정의

Entity 정의는 간단하다 Entity를 선언할 클래스에 **@Entity** 어노테이션을 붙여주고 primary key 에는 **@PrimaryKey** 어노테이션을 column 에는 **@ColumInfo** 어노테이션을 사용해서 지정해주면 된다.

```kotlin
@Entity
data class User(
    @PrimaryKey val uid: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

## DAO(= Data Access Object) 정의

Database의 data를 접근하는 객체를 정의하기 위해서 **@Query** 어노테이션을 사용하여 data의 리스트를 받아올 수 있고 **@Insert**, **@Delete** 어노테이션을 활용해서 삽입 및 삭제를 할 수 있다.

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    fun getAll(): List<User>

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    fun loadAllByIds(userIds: IntArray): List<User>

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
           "last_name LIKE :last LIMIT 1")
    fun findByName(first: String, last: String): User

    @Insert
    fun insertAll(vararg users: User)

    @Delete
    fun delete(user: User)
}
```

## 데이터베이스 생성

RoomDatabase를 상속받은 abstract 클래스를 생성해서 해당 데이터베이스에서 사용하는 entity 및 version을 어노테이션을 사용하여 정의해준다.

```kotlin
@Database(entities = arrayOf(User::class), version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

위에 클래스를 활용해서 데이터베이스 인스턴스를 가져온다.

```kotlin
val db = Room.databaseBuilder(
            applicationContext,
            AppDatabase::class.java, "database-name"
        ).build()

```

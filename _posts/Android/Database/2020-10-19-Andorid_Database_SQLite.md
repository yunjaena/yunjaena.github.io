---
title: Android Database - SQLite
author: Yun-Jae Na
date: 2020-10-19 16:38:00 +0900
categories: [Android, Database]
tags: [Android, Android Database]
---

## Android SQLite

Android 에서 database를 사용할 때 필요한 API는 **android.database.sqlite** 패키지로 제공한다. 하지만 2017년 Google I/O 에서 Android Architecture Components(AAC)를 발표하면서 Android Developer 사이트에는 SQLite를 직접 사용하는것은 low level API 이며 구현시 오류발생의 가능성이 높아 안전하게 사용하기 위해 AAC 의 **ROOM**을 사용하는 것을 추천하고 있다. **Room은 SQLite를 추상화 하여 쉽게 database에 대한 인터페이스를 제공하는 라이브러리이다.** Room은 추후에 다루어 보도록한다.

![Android Developer SQLite Warning](https://drive.google.com/uc?export=view&id=14vKUbd6eSCN0lPI1uD8w8vleLwz9yJBC)
_[Android Developer SQLite Warning](https://developer.android.com/training/data-storage/sqlite)_

## Schema 및 Contract 정의

SQL database의 기본 원칙 중 하나는 **schema** 이다. **schema**는 database 구성 체계에 관한 공식적인 선언이며 개발자가 database를 생성할 때 사용하는 SQL 문에 반영된다. 체계적인 자체 문서화 방식으로 schema의 레이아웃을 명시적으로 지정하는 **contract class**라고 하는 **compainon class**를 생성하면 도움이 될 수 있다.

**Contract class**는 URI, 테이블 및 열의 이름을 정의하는 상수를 유지하는 컨테이너이다. contract class를 통해 동일한 패키지의 다른 모든 클래스에 동일한 상수를 사용할 수 있다. 이렇게 하면 어느 한 곳에서 열 이름을 변경하고 이 변경사항을 코드 전체에 전파할 수 있다.

**Contract class**를 구성하는 좋은 방법은 클래스의 루트 수주에 있는 database 전체에 전역적인 정의를 추가하는 것이며 다음 각 테이블의 내부 클래스를 생성한다. 각 내부 클래스는 상응하는 테이블의 열을 열거한다.

>[BaseColums](https://developer.android.com/reference/android/provider/BaseColumns) 인터페이스를 구현함으로써 **_ID**라고 하는 기본 키 필드를 상속할 수 있으며 CusorAdapter와 같은 일부 Android 클래스는 내부 클래스가 이러한 기본 키 필드를 가지고 있을 것이라 예상한다. 기본 키 필드가 반드시 필요한 것은 아니지만 database가 Android 프레임워크와 조화롭게 작동하는데 도움이 된다.

아래의 코드는 Android Developer 페이지에 있는 코드이다. 다음 Contract 는 테이블 이름과 RSS 피드를 나타내는 단일 테이블의 열 이름을 정의한다.

```kotlin
object FeedReaderContract {
    // Table contents are grouped together in an anonymous object.
    object FeedEntry : BaseColumns {
        const val TABLE_NAME = "entry"
        const val COLUMN_NAME_TITLE = "title"
        const val COLUMN_NAME_SUBTITLE = "subtitle"
    }
}
```

## SQL Helper를 사용하여 database 생성

Database의 모양을 정의한 후에는 Database 및 테이블을 생성 및 유지하는 메서드를 구현해야한다. 다음은 테이블을 생성하고 삭제하는 일반적인 구문이다.

```kotlin
private const val SQL_CREATE_ENTRIES =
        "CREATE TABLE ${FeedEntry.TABLE_NAME} (" +
                "${BaseColumns._ID} INTEGER PRIMARY KEY," +
                "${FeedEntry.COLUMN_NAME_TITLE} TEXT," +
                "${FeedEntry.COLUMN_NAME_SUBTITLE} TEXT)"

private const val SQL_DELETE_ENTRIES = "DROP TABLE IF EXISTS ${FeedEntry.TABLE_NAME}"
```

기기의 내부 저장소에 저장한 파일과 마찬가지로 **Android는 database를 앱의 비공개 폴더에 저장한다.** 기본적으로 이 공간은 다른 앱이나 사용자가 액세스할 수 없기 때문에 저장된 데이터는 안전하게 유지된다.

이 클래스를 사용하여 database의 참조를 가져오면 시스템은 앱이 시작되고 있는 동안이 아닌 필요한 때에만 database 생성 및 업데이트와 같이 장시간 실행될 수 있는 작업을 실행한다. 개발자는 **getWritableDatabase()** 또는 **getReadableDatabase()** 를 호출하기만 하면 된다.

>**getWritableDatabase()** 또는 **getReadableDatabase()** 는 장시간 실행될 수 있기 떄문에 백그라운드 스레드에서 호출해야 한다.

**SQLiteOpenHelper** 를 사용하면 **onCreate()** 및 **onUpgrade()** 콜백 메서드를 재정의하는 서브클래스를 생성해야 한다. 또한 onDowngrade() 또는 onOpen() 메서드를 구현할 수 있지만 이러한 메서드는 필수는 아니다.

```kotlin
// SQLiteHelper 구현 예시
class FeedReaderDbHelper(context: Context) : SQLiteOpenHelper(context, DATABASE_NAME, null, DATABASE_VERSION) {
    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL(SQL_CREATE_ENTRIES)
    }
    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        // This database is only a cache for online data, so its upgrade policy is
        // to simply to discard the data and start over
        db.execSQL(SQL_DELETE_ENTRIES)
        onCreate(db)
    }
    override fun onDowngrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        onUpgrade(db, oldVersion, newVersion)
    }
    companion object {
        // If you change the database schema, you must increment the database version.
        const val DATABASE_VERSION = 1
        const val DATABASE_NAME = "FeedReader.db"
    }
}
```
Database에 액세스하려면 다음과 같이 SQLiteOpenHelper의 서브클래스를 인스턴스화한다.

```kotlin
val dbHelper = FeedReaderDbHelper(context)
```

## Database에 정보 삽입

다음과 같이 **[ContentValues](https://developer.android.com/reference/android/content/ContentValues)** 객체를 insert() 메서드에 전달하여 데이터를 database에 삽입할 수 있다.

```kotlin
// Gets the data repository in write mode
val db = dbHelper.writableDatabase

// Create a new map of values, where column names are the keys
val values = ContentValues().apply {
    put(FeedEntry.COLUMN_NAME_TITLE, title)
    put(FeedEntry.COLUMN_NAME_SUBTITLE, subtitle)
}

// Insert the new row, returning the primary key value of the new row
val newRowId = db?.insert(FeedEntry.TABLE_NAME, null, values)
```

여기서 **insert()** 첫 번째 인수는 Database Table 이름이고 두 번째는 **ContentValues()** 가 비어 있을 때 즉, 어떤 값도 삽입하지 않았을때 실행할 작업을 프레임워크에 알려준다. 열 이름을 지정하면 프레임워크는 행을 삽입하고 **열의 값을 null**로 설정한다. 위 코드와 같이 **null**을 지정하면 프레임워크는 값이 없을 때 행을 삽입하지 않는다.

**insert()** 메서드는 새로 생성된 행의 ID를 반환하거나 데이터 삽입 시 오류가 발생하면 -1를 반환한다. 오류는 database의 기존 데이터와 충돌하는 경우에도 발생할 수 있다.

## Database에서 정보 읽어오기

database에서 정보를 읽어오려면 **query()** 메서드를 사용하고 이 메서드에 선택 기준 및 원하는 열을 전달한다. 쿼리 결과는 **Cursor** 객체로 반환된다.

```kotlin
val db = dbHelper.readableDatabase

// Define a projection that specifies which columns from the database
// you will actually use after this query.
val projection = arrayOf(BaseColumns._ID, FeedEntry.COLUMN_NAME_TITLE, FeedEntry.COLUMN_NAME_SUBTITLE)

// Filter results WHERE "title" = 'My Title'
val selection = "${FeedEntry.COLUMN_NAME_TITLE} = ?"
val selectionArgs = arrayOf("My Title")

// How you want the results sorted in the resulting Cursor
val sortOrder = "${FeedEntry.COLUMN_NAME_SUBTITLE} DESC"

val cursor = db.query(
        FeedEntry.TABLE_NAME,   // The table to query
        projection,             // The array of columns to return (pass null to get all)
        selection,              // The columns for the WHERE clause
        selectionArgs,          // The values for the WHERE clause
        null,                   // don't group the rows
        null,                   // don't filter by row groups
        sortOrder               // The sort order
)
```

세 번쨰 및 네 번째 인수 (**selection** 및 **selectionArgs**)는 결합되어 WHERE 절을 생성한다.

커서의 행을 알아볼꺼면 **moveToNext()** 메서드를 사용한다. 커서는 -1 위치에서 시작하므로 해당 메소드를 호출하면 결과의 첫 번째 항목에 **읽기 위치**가 배치되고 커서가 결과 세트의 마지막 항목을 이미 지나갔는지 여부가 반환된다.

**getString()** 또는 **getLong()** 과 같은 Cursor get 메서드 중 하나를 호출해서 열의 값을 읽어 올 수 있다. 각 get 메서드에서 원하는 열의 index 위치를 전달해야 하며 이 위치는 **getColumnIndex()** 또는 **getColumnIndexOrThrow()** 를 호출하여 가져올 수 있다. 결과에 대해 반복이 완료가 되면 커서의 **close()** 메소드를 호출하여 리소스를 해제한다.

```kotlin
// 커서에 저장된 모든 항목 ID를 가져와서 목록에 추가
val itemIds = mutableListOf<Long>()
with(cursor) {
    while (moveToNext()) {
        val itemId = getLong(getColumnIndexOrThrow(BaseColumns._ID))
        itemIds.add(itemId)
    }
}
```

## Database에서 정보 삭제

테이블에서 행을 삭제하려면 행을 식별하는 선택 기준을 **delete()** 메서드에 제공해야 한다.

```kotlin
// Define 'where' part of query.
val selection = "${FeedEntry.COLUMN_NAME_TITLE} LIKE ?"
// Specify arguments in placeholder order.
val selectionArgs = arrayOf("MyTitle")
// Issue SQL statement.
val deletedRows = db.delete(FeedEntry.TABLE_NAME, selection, selectionArgs)
```

delete() 메서드의 반환 값은 database에서 삭제된 행 수를 나타낸다.

## Database 업데이트

Database 값의 일부를 수정해야 하면 **update()** 메서드를 사용한다.
테이블을 업데이트하려면 ContentValues 를 사용한다.

```kotlin
val db = dbHelper.writableDatabase

// New value for one column
val title = "MyNewTitle"
val values = ContentValues().apply {
    put(FeedEntry.COLUMN_NAME_TITLE, title)
}

// Which row to update, based on the title
val selection = "${FeedEntry.COLUMN_NAME_TITLE} LIKE ?"
val selectionArgs = arrayOf("MyOldTitle")
val count = db.update(
        FeedEntry.TABLE_NAME,
        values,
        selection,
        selectionArgs)
```

**update()** 메서드의 반환 값은 database에서 업데이트에 영향받는 행의 수이다.

## Database 연결 유지

Database가 닫혀 있을 때 **getWritableDatabase()** 및 **getReadableDatabase()** 호출에는 리소스가 많이 사용되므로 database에 액세스해야 하는 동안에는 최대한 database 연결을 열린 상태로 두어야 한다. 일반적으로 호출 활동의 **onDestroy()** 에서 database를 닫는 것이 가장 좋다.

```kotlin
override fun onDestroy() {
    dbHelper.close()
    super.onDestroy()
}
```

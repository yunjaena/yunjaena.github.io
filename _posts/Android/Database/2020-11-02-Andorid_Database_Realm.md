---
title: Android Database - Realm
author: Yun-Jae Na
date: 2020-11-02 15:00:00 +0900
categories: [Android, Database]
tags: [Android, Android Database]
image : https://drive.google.com/uc?export=view&id=1c5-x5XBmygaix9BUjBD0A3c05hnGHQlh
---

## Android Realm Database

Realm(렘)은 오픈소스 데이터베이스 관리시스템(DBMS) 이며 모바일 환경을 주요 타깃으로 삼은 데이터베이스이다.  
Realm은 매우 작은 리소스를 사용하고 사용하기 쉽고 더 빠르게 데이터와 상호 작용 가능하다.  
NoSQL 데이터베이스를 지향하며 **rawSQL을 사용할 수 없어 Realm API를 통해서 실행된다.**


한국어 공식 문서도 제공하지만 kotlin 인 경우 현제 영어 문서만 제공하고 있다.  
Java : [https://realm.io/docs/java/latest/](https://realm.io/docs/java/latest/)  
Kotlin : [https://realm.io/docs/kotlin/latest/](https://realm.io/docs/kotlin/latest/)

![SQL Queries](https://drive.google.com/uc?export=view&id=1__tRX-MRdzIqHRCcj0LvDRCrGvCUPRhn){: width="500"}
_Realm Queries_

## 시작하기

프로젝트 레벨의 gradle 파일에 아래와 같이 class path를 추가해준다.
```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "io.realm:realm-gradle-plugin:7.0.0-beta"
    }
}
```

애플리케이션 레벨의 gradle 파일에 플러그인을 추가해준다.

```groovy
apply plugin: 'com.android.application'
apply plugin: 'kotlin-kapt'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'realm-android'
```

## Kotlin 으로 사용시 주의 사항

1. 모델 클래스는 **open** 으로 선언되어야 한다.
```kotlin
enum class MyEnum {
    Value1, Value2
}
open class EnumTest: RealmObject() {
    // Custom private backing field representing the enum
    private var strField: String = MyEnum.Value1.name

    // Public field exposing setting/getting the enum
    var enumField: MyEnum
        get() = MyEnum.values().first { it.name == strField }
        set(value) {
            strField = value.name
        }
}
// Queries
val results = realm.where<EnumTest>().equalTo("strField", MyEnum.Value1.name).findAll()
```

2. primitive type wrapper class 구분
```kotlin
schema
  .addField("field", Long::class.java) // Non-nullable (long)
  .addField("field", Long::class.javaObjectType) // Nullable (Long)
  .addField("field", Long::class.javaPrimitiveType) // Non-nullable (long)
```

## Realm 초기화

Application 클래스에서 Realm 을 초기화 해준다.

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        Realm.init(this)
    }
}
```

AndroidManifest.xml에 Application class를 등록해준다.
```xml
<application
  android:name=".MyApplication"
  ...
/>
```

## Realm 설정하기

Realm 을 설정하기 위해 **RealmCongiguration** 객체를 사용한다. default로 설정하기 위해서는 아래와 같이 사용된다.

```kotlin
val config = RealmConfiguration.Builder().build()
```

이와 같이 사용하는 경우 **Context.getFilesDir**에 위치에 default.realm 파일을 저장한다.

전형적으로 사용하는 방법은 아래와 같다.

```kotlin
// The RealmConfiguration is created using the builder pattern.
// The Realm file will be located in Context.getFilesDir()
// with name "myrealm.realm"
val config = RealmConfiguration.Builder()
    .name("myrealm.realm")
    .encryptionKey(getMyKey())
    .schemaVersion(42)
    .modules(MySchemaModule())
    .migration(MyMigration())
    .build()
// Use the config
val realm = Realm.getInstance(config)
```

### RealmConfiguration을 Default로 설정하기

Custom Application class에서 **RealmConfiguration** 클래스에 있는 **setDefaultConfiguration** 매서드를 사용해서 default를 설정할 수도 있다.

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        // The default Realm file is "default.realm" in Context.getFilesDir();
        // we'll change it to "myrealm.realm"
        Realm.init(this)
        val config = RealmConfiguration.Builder().name("myrealm.realm").build()
        Realm.setDefaultConfiguration(config)
    }
}

class MyActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val realm = Realm.getDefaultInstance() // opens "myrealm.realm"
        try {
            // ... Do something ...
        } finally {
            realm.close()
        }
    }
}
```

## Realm 모델 생성하기

**RealmObject** class 를 상속받아 구현한다. (**open** 으로 정의하기!!)

```kotlin
open class User(
    var name: String = "",
    var age: Int = 0,

    @Ignore
    var sessionId: Int = 0
): RealmObject()
```

### Required fields

- 필드에서 null을 허용하지 않을꺼면 **@Require**를 필드에 추가해준다.

### 속성 인덱싱

- **@Index** annotation을 추가해주면 그 필드를 기준으로 인덱싱된다. 인덱싱을 하면, 데이터 추가가느려지며 파일이 커지는 단점이 있다. 대신 query가 빨라진다.
- **String, Byte, Shore, Int, Long, Boolean, Date** 필드에서만 사용 가능하다.

### Primary Key

- **@PrimaryKey** annotation 을 사용해서 기본키를 설정한다.  
- **@PrimaryKey** 는 **@Index** annotation을 포함하고 있다.
- Primary key를 사용하면 **copyToRealmOrUpdate** 메서드를 사용할 수 있는데 업데이트 시, 키를 이용해서 객체를 찾으면 그 객체를 업데이트를 하고 못찾으면 새로운 객체를 생성한다. (@PrimaryKey 가 선언되어 있지 않으면 exception 발생)
- **Realm.createObject** 는 모든 필드에 기본값이 채워져서 객체를 생성하는데 기본키가 중복이 될 수도 있다.
  - 이를 해결하기 위해서 **copyToRealm** 메서드를 사용하는데 똑같은 키를 가지면 exception이 발생한다.

```kotlin
val obj = MyObject()
obj.id = 42
obj.name = "Fish"
realm.executeTransaction { realm ->
    // This will create a new object in Realm or throw an exception if the
    // object already exists (same primary key)
    // realm.copyToRealm(obj);

    // This will update an existing object with the same primary key
    // or create a new object if an object with no primary key = 42
    realm.copyToRealmOrUpdate(obj)
}
```

### @Ignore

- Realm 필드를 저장하고 싶지 않을 때 사용한다.  
- static, trasient은 항상 @Ingnore로 설정된다.

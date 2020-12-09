---
title: RxJava - 프로젝트 환경 구축 및 Hello RxJava
author: Yun-Jae Na
date: 2020-12-09 20:00:00 +0900
categories: [Android, RxJava]
tags: [Android, RxJava]
image : https://drive.google.com/uc?export=view&id=1EKGE5Tbptu_0S3-y3Pazr-1vSTtfNy1K
---

## RxJava 프로젝트 환경 구축 순서

- JDK 설치  (1.8 이상의 JDK가 설치 되어 있다고 가정함)
- IDE 다운로드 및 설치
- IDE에 프로젝트 생성
- RxJava를 위한 의존 라이브러리 설치
- 정상적으로 동작하는지 Hello RxJava 코드 작성 및 실행

### IDE 다운로드 및 설치

- JetBrain의 IntelliJ IDE 다운로드 및 설치 : [https://www.jetbrains.com/ko-kr/idea/](https://www.jetbrains.com/ko-kr/idea/)

### IDE 실행

- Create New Project
- Gradle 선택
- 프로젝트 이름 설정

### RxJava를 위한 의존 라이브러리 설치

- build.gradle 파일로 이동
- dependecies에 아래의 코드 추가 (만약 아래 코드 추가후에 자동 import가 되지 않는 경우 preferences -  Gradle - Automatically import this project on changes in build script files 활성화를 한다. )
```groovy
compile group: 'io.reactivex.rxjava2', name: 'rxjava', version: '2.2.6'
```

### Hello Rxjava 코드 작성

- 클래스를 생성한다. ( ex) HelloJava.class )
- windows : ctrl + shift + enter , mac : cmd + shift + return 을 누르면 클래스 안으로 커서 이동
- 메인 메소드를 작성(psvm 라이브 템플릿 사용)
- 아래의 코드를 입력 후 실행

```java
import io.reactivex.Observable;

  public class HelloRxJava {

    public static void main(String[] args){
        Observable<String> observable = Observable.just("hello", "RxJava"); // 데이터를 생성하고 통제하는 생성자쪽 코드.
        observable.subscribe(data -> System.out.println(data)); // 데이터를 구독하는 소비자쪽 코드.
    }

}
/* 출력
hello
RxJava
*/
```

> 이 글은 inflearn에 있는 Kevin의 알기 쉬운 RxJava 1부를 공부하고 작성한 글입니다.   
> [강의영상 링크](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-1#description)

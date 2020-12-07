---
title: RxJava - 리엑티브(Reative) 프로그래밍이란
author: Yun-Jae Na
date: 2020-12-07 15:00:00 +0900
categories: [Android, RxJava]
tags: [Android, RxJava]
image : https://drive.google.com/uc?export=view&id=1EKGE5Tbptu_0S3-y3Pazr-1vSTtfNy1K
---

## 리액티브 프로그래밍(Reactive Programming) 이란?

**변화의 전파**와 **데이터 흐름**과 관련된 **선언적 프로그래밍** 패러다임이다.

- 변화의 전파와 데이터 흐름 : 데이터가 변경 될 때 마다 이벤트를 발생시켜서 데이터를 계속적으로 전달한다.
- 선언적 프로그래밍 : 실행할 동작을 구체적으로 명시하는 명령형 프로그램이과 달리 선언형 프로그래밍은 단순히 목표를 선언한다.

## 리액티브의 개념이 적용된 예

- Push 방식 : 데이터의 변화가 발생했을 떄 변경이 발생한 곳에서 데이터를 보내주는 방식
  - RTC(Real Time Communication)
  - 소켓 프로그래밍
  - DB Trigger
  - Spring의 ApplicationEvent
  - Angular의 데이터 바인딩
  - 스마트폰의 Push 메시지


- Pull 방식 : 변경된 데이터가 있는지 요청을 보내 질의하고 변경된 데이터를 가져오는 방식
  - 클라이언트 요청 & 서버 응답 방식의 애플리케이션
  - Java와 같은 절차형 프로그래밍 언어

## 리액티브 프로그래밍을 위해 알아야 될 것들

- Observable : 데이터 소스 (변경되는 데이터 관찰)
- 리액티브 연산자(Operators) : 데이터 소스를 처리하는 함수
- 스케쥴러(Scheduler) : 스레드 관리자
- Subscriber : Observable이 발행하는 데이터를 구독하는 구독자
- 함수형 프로그래밍 : RxJava에서 제공하는 연산자(Operator) 함수를 사용
- doOnNext : 데이터가 발생한 후 onNext 함수가 호출된 후 호출되는 메소드
- subscribeOn : 데이터가 발행, 데이터의 흐름을 결정짓는 스레드를 결정
- observeOn : 발행된 데이터를 가공하고 구독해서 처리하는 스레드를 결정
- 리액티브 기본 동작 흐름 : 데이터 발행 -> 데이터 가공 -> 데이터 구독 -> 결과 처리

> 이 글은 inflearn에 있는 Kevin의 알기 쉬운 RxJava 1부를 공부하고 작성한 글입니다.   
> [강의영상 링크](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-1#description)

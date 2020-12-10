---
title: RxJava - Reative Streams란?
author: Yun-Jae Na
date: 2020-12-10 18:00:00 +0900
categories: [Android, RxJava]
tags: [Android, RxJava]
image : https://drive.google.com/uc?export=view&id=1EKGE5Tbptu_0S3-y3Pazr-1vSTtfNy1K
---

## Reactive Streams란?

- 리액티브 프로그래밍 라이브러리의 표준 사양이다. => [참고링크](https://github.com/reactive-streams/reactive-streams-jvm/)
- 리액티브 프로그래밍에 대한 인터페이스만 제공한다.
- RxJava는 이 Reactive Streams의 인터페이스들을 구현한 구현체이다.
- Reactive Streams는 Publisher, Subscriber, Subscription, Processor 라는 4개의 인터페이스를 제공한다.
  - Publisher(생산자) : 데이터를 생성하고 통지한다.
  - Subscriber(소비자) : 통지된 데이터를 전달받아서 처리한다.
  - Subscription(구독) : 전달 받을 데이터의 개수를 요청하고 구독을 해지한다.
  - Processor : Publisher와 Subscriber의 기능이 모두 있음.

## Publisher와 Subscriber간의 프로세스 흐름

![Publisher 와 Subscriber](https://drive.google.com/uc?export=view&id=1P-Lu-1uY-YN38Yo_t-cER6in9dtX9TG_)

## Cold Publisher & Hot Publisher

### Cold Publisher(차가운 생산자)

- 생산자는 소비자가 구독 할때마다 데이터를 처음부터 새로 통지한다.
- 데이터를 통지하는 새로운 타임 라인이 생성된다.
- 소비자는 구독 시점과 상관없이 통지된 데이터를 처음부터 전달 받을 수 있다.

![Cold publisher](https://drive.google.com/uc?export=view&id=1gcWrn7kVkZqiwIf_si1MA3CjRUElwnyi)


```java
import io.reactivex.Flowable;

public class ColdPublisherExample {
    public static void main(String[] args) {
        Flowable<Integer> flowable = Flowable.just(1,3,5,7);

        flowable.subscribe(data -> System.out.println("구독자1: " + data));
        flowable.subscribe(data -> System.out.println("구독자2: " + data));
    }
}
/*
구독자1: 1
구독자1: 3
구독자1: 5
구독자1: 7
구독자2: 1
구독자2: 3
구독자2: 5
구독자2: 7
*/
```

### Hot Publisher(뜨거운 생산자)

- 생산자는 소비자 수와 상관없이 데이터를 한번만 통지한다.
- 즉, 데이터를 통지하는 타임 라인은 하나이다.
- 소비자는 발행된 데이터를 처음부터 전달 받는게 아니라 구독한 시점에 통지된 데이터들만 전달 받을 수 있다.

![Hot Publisher](https://drive.google.com/uc?export=view&id=1ZxusWKCyiomzyBn7nLcWDFP3L4bjPt21)

```java
import io.reactivex.processors.PublishProcessor;

public class HotPublisherExample {
    public static void main(String[] args) {
        PublishProcessor<Integer> processor = PublishProcessor.create();
        processor.subscribe(data -> System.out.println("구독자1: " + data));
        processor.onNext(1);
        processor.onNext(3);

        processor.subscribe(data -> System.out.println("구독자2: " + data));
        processor.onNext(5);
        processor.onNext(7);

        processor.onComplete();
    }
}
/*
구독자1: 1
구독자1: 3
구독자1: 5
구독자2: 5
구독자1: 7
구독자2: 7
*/
```


> 이 글은 inflearn에 있는 Kevin의 알기 쉬운 RxJava 1부를 공부하고 작성한 글입니다.   
> [강의영상 링크](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-1#description)

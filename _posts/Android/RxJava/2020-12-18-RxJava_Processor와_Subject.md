---
title: RxJava - Processor와 Subject
author: Yun-Jae Na
date: 2020-12-18 12:50:00 +0930
categories: [Android, RxJava]
tags: [Android, RxJava]
image : https://drive.google.com/uc?export=view&id=1EKGE5Tbptu_0S3-y3Pazr-1vSTtfNy1K
---

## Projector와 Subject란?

- Processor는 Reactive Streams에서 정의한 Publisher 인터페이스와 Subscriber 인터페이스를 둘 다 상속한 확장 인터페이스이다.

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

- 즉, Processor는 Publisher(생산자)의 기능과 Subscriber(소비자)의 기능을 모두 가지고 있다.
- Processor는 Hot Publisher(뜨거운 생산자)이다.
- Subject는 Reactive Streams Processor와 동일한 기능을 하나 배압 기능이 없는 추상 클래스이다.
- Processor와 Subject의 구현 클래스에는 다음과 같은 클래스가 있다.
  - PublishProcessor / PublishSubject
  - AsyncProcessor / AsyncSubject
  - BehaviorProcessor / BehaviorSubject
  - ReplayProcessor / ReplaySubject


## Cold Publisher / Hot Publisher 리뷰

### Cold Publisher(차가운 생성자)

![Cold publisher](https://drive.google.com/uc?export=view&id=1gcWrn7kVkZqiwIf_si1MA3CjRUElwnyi)

### Hot Publisher(뜨거운 생산자)

![Hot Publisher](https://drive.google.com/uc?export=view&id=1ZxusWKCyiomzyBn7nLcWDFP3L4bjPt21)

## Subject

### PublishSubject

- 구독 전에 통지된 데이터는 받을 수 없고, 구독한 이후에 통지 된 데이터만 받을 수 있다.
- 데이터 통지가 완료 된 이후에 소비작가 구독하면 완료 또는 에러 통지를 받는다.

![PublishSubject](https://drive.google.com/uc?export=view&id=1C9raqlJIjdts02z664A1ylGbpTkPfbYn)

```java
/**
 * 소비자가 구독한 시점 이 후에 통지 된 데이터만 소비자에게 전달되는 PublishSubject 예제
 */
public class PublishSubjectExample {
    public static void main(String[] args){
        PublishSubject<Integer> subject = PublishSubject.create();

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 1 : " + price));
        subject.onNext(3500);
        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 2 : " + price));
        subject.onNext(3300);
        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 3 : " + price));
        subject.onNext(3400);

        subject.subscribe(
                price -> Logger.log(LogType.ON_NEXT, "# 소비자 4 : " + price),
                error -> Logger.log(LogType.ON_ERROR, error),
                () -> Logger.log(LogType.ON_COMPLETE)
        );

        subject.onComplete();

    }
}
/*
onNext() | main | 01:17:51.318 | # 소비자 1 : 3500
onNext() | main | 01:17:51.329 | # 소비자 1 : 3300
onNext() | main | 01:17:51.329 | # 소비자 2 : 3300
onNext() | main | 01:17:51.330 | # 소비자 1 : 3400
onNext() | main | 01:17:51.330 | # 소비자 2 : 3400
onNext() | main | 01:17:51.330 | # 소비자 3 : 3400
onComplete() | main | 01:17:51.333
*/
```

### AsyncSubject

- 완료 전까지 아무것도 통지하지 않고있다가 완료했을 때 마지막으로 통지한 데이터와 완료만 통지한다.
- 모든 소비자는 구독 시점에 상관없이 마지막으로 통지된 데이터와 완료 통지만 받는다.
- 완료 후에 구독한 소비자라도 마지막 통지된 데이터와 완료 통지를 받는다.

![AsyncSubject](https://drive.google.com/uc?export=view&id=1zG03UQASNvBYJ1CrPWx4ltnjQU1c-5jK)

```java
/**
 * 구독 시점에 상관없이 모든 소비자들이 마지막으로 통지된 데이터만 전달 받는 AsyncSubject 예제
 */
public class AsyncSubjectExample {
    public static void main(String[] args) {
        AsyncSubject<Integer> subject = AsyncSubject.create();
        subject.onNext(1000);

        subject.doOnNext(price -> Logger.log(LogType.DO_ON_NEXT, "# 소비자 1 : " + price))
                .subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 1 : " + price));
        subject.onNext(2000);

        subject.doOnNext(price -> Logger.log(LogType.DO_ON_NEXT, "# 소비자 2 : " + price))
                .subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 2 : " + price));
        subject.onNext(3000);

        subject.doOnNext(price -> Logger.log(LogType.DO_ON_NEXT, "# 소비자 3 : " + price))
                .subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 3 : " + price));
        subject.onNext(4000);

        subject.onComplete();

        subject.doOnNext(price -> Logger.log(LogType.DO_ON_NEXT, "# 소비자 4 : " + price))
                .subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 4 : " + price));
    }
}
/*
doOnNext() | main | 02:26:45.367 | # 소비자 1 : 4000
onNext() | main | 02:26:45.369 | # 소비자 1 : 4000
doOnNext() | main | 02:26:45.370 | # 소비자 2 : 4000
onNext() | main | 02:26:45.370 | # 소비자 2 : 4000
doOnNext() | main | 02:26:45.370 | # 소비자 3 : 4000
onNext() | main | 02:26:45.370 | # 소비자 3 : 4000
doOnNext() | main | 02:26:45.371 | # 소비자 4 : 4000
onNext() | main | 02:26:45.371 | # 소비자 4 : 4000
*/
```

### BehaviorSubject

- 구독 시점에 이미 통지된 데이터가 있다면 이미 통지된 데이터의 마지막 데이터를 전달 받은 후, 구독 이 후에 통지된 데이터를 전달 받는다.
- 처리가 완료된 이후에 구독하면 완료나 에러 통지만 전달 받는다.

![BehaviorSubject](https://drive.google.com/uc?export=view&id=1nf_v2Y781BrL7Q7uFE3Wna89RUUWXUfo)

```java
/**
 * 구독 시점에 이미 통지된 데이터가 있다면 이미 통지된 데이터의 마지막 데이터를 전달 받은 후,
 * 구독 이후부터 통지 된 데이터를 전달 받는 예제
 */
public class BehaviorSubjectExample {
    public static void main(String[] args){
        BehaviorSubject<Integer> subject = BehaviorSubject.createDefault(3000);

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 1 : " + price));
        subject.onNext(3500);

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 2 : " + price));
        subject.onNext(3300);

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 3 : " + price));
        subject.onNext(3400);
    }
}
/*
onNext() | main | 03:03:42.635 | # 소비자 1 : 3000
onNext() | main | 03:03:42.639 | # 소비자 1 : 3500
onNext() | main | 03:03:42.640 | # 소비자 2 : 3500
onNext() | main | 03:03:42.640 | # 소비자 1 : 3300
onNext() | main | 03:03:42.640 | # 소비자 2 : 3300
onNext() | main | 03:03:42.641 | # 소비자 3 : 3300
onNext() | main | 03:03:42.641 | # 소비자 1 : 3400
onNext() | main | 03:03:42.641 | # 소비자 2 : 3400
onNext() | main | 03:03:42.641 | # 소비자 3 : 3400
*/
````

### ReplaySubject

- 구독 시점에 이미 통지된 데이터가 있다면 이미 통지된 데이터 중에서 최근 통지된 데이터를 지정한 개수만큼 전달 받은 후, 구독 이 후에 통지된 데이터를 전달 받는다.
- 이미 처리가 완료된 이후에 구독하더라도 지정한 개수 만큼의 최근 통지된 데이터를 전달 받는다.

![ReplaySubject](https://drive.google.com/uc?export=view&id=1-Ki5Dfh3hZZDHLKoCF2kf_fLB8myk6MH)

```java
public class ReplaySubjectExample01 {
    public static void main(String[] args){
        ReplaySubject<Integer> subject = ReplaySubject.create();
        subject.onNext(3000);
        subject.onNext(2500);

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 1 : " + price));
        subject.onNext(3500);

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 2 : " + price));
        subject.onNext(3300);

        subject.onComplete();

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 3 : " + price));

    }
}
/*
onNext() | main | 04:03:24.885 | # 소비자 1 : 3000
onNext() | main | 04:03:24.888 | # 소비자 1 : 2500
onNext() | main | 04:03:24.888 | # 소비자 1 : 3500
onNext() | main | 04:03:24.888 | # 소비자 2 : 3000
onNext() | main | 04:03:24.888 | # 소비자 2 : 2500
onNext() | main | 04:03:24.888 | # 소비자 2 : 3500
onNext() | main | 04:03:24.889 | # 소비자 1 : 3300
onNext() | main | 04:03:24.889 | # 소비자 2 : 3300
onNext() | main | 04:03:24.891 | # 소비자 3 : 3000
onNext() | main | 04:03:24.892 | # 소비자 3 : 2500
onNext() | main | 04:03:24.892 | # 소비자 3 : 3500
onNext() | main | 04:03:24.892 | # 소비자 3 : 3300
*/
```

```java
public class ReplaySubjectExample02 {
    public static void main(String[] args){
        ReplaySubject<Integer> subject = ReplaySubject.createWithSize(2);
        subject.onNext(3000);
        subject.onNext(2500);

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 1 : " + price));
        subject.onNext(3500);

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 2 : " + price));
        subject.onNext(3300);

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 3 : " + price));
        subject.onNext(3400);

        subject.onComplete();

        subject.subscribe(price -> Logger.log(LogType.ON_NEXT, "# 소비자 4 : " + price));
    }
}
/*
onNext() | main | 04:04:56.095 | # 소비자 1 : 3000
onNext() | main | 04:04:56.098 | # 소비자 1 : 2500
onNext() | main | 04:04:56.098 | # 소비자 1 : 3500
onNext() | main | 04:04:56.099 | # 소비자 2 : 2500
onNext() | main | 04:04:56.099 | # 소비자 2 : 3500
onNext() | main | 04:04:56.099 | # 소비자 1 : 3300
onNext() | main | 04:04:56.099 | # 소비자 2 : 3300
onNext() | main | 04:04:56.100 | # 소비자 3 : 3500
onNext() | main | 04:04:56.100 | # 소비자 3 : 3300
onNext() | main | 04:04:56.100 | # 소비자 1 : 3400
onNext() | main | 04:04:56.100 | # 소비자 2 : 3400
onNext() | main | 04:04:56.100 | # 소비자 3 : 3400
onNext() | main | 04:04:56.102 | # 소비자 4 : 3300
onNext() | main | 04:04:56.102 | # 소비자 4 : 3400
*/
```

> 이 글은 inflearn에 있는 Kevin의 알기 쉬운 RxJava 2부를 공부하고 작성한 글입니다.   
> [강의영상 링크](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-2)

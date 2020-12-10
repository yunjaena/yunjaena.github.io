---
title: RxJava - Flowable과 Observable
author: Yun-Jae Na
date: 2020-12-10 19:00:00 +0900
categories: [Android, RxJava]
tags: [Android, RxJava]
image : https://drive.google.com/uc?export=view&id=1EKGE5Tbptu_0S3-y3Pazr-1vSTtfNy1K
---

## Flowable과 Observable의 비교

| Flowable |  Observable   |
|----------|:-------------:|
| Reactive Streams 인터페이스를 구현함 |  Reactive Streams 인터페이스를 구현하지 않음 |
| Subscriber에서 데이터를 처리한다. |Observer에서 데이터를 처리한다.   |
| 데이터 개수를 제어하는 **배압 기능**이 있음 | 데이터 개수를 제어하는 **배압 기능**이 없음 |
| Subscription으로 전달 받는 데이터 개수를 제어할 수 있다. | 배압 기능이 없기때문에 데이터 개수를 제어할 수 없다. |
| Subscription으로 구독을 해지한다. | Disposable로 구독을 해지한다. |

## 배압(Back Pressure)이란?

- Flowable에서 데이터를 통지하는 속도가 Subscriber에서 통지된 데이터를 전달받아 처리하는 속도 보다 빠를 때 밸런스를 맞추기 위해 데이터 통지량을 제어하는 기능을 말한다.

![Back Pressure](https://drive.google.com/uc?export=view&id=1cw-vg7S-buDOflGd9s4IP-QUEmcpQxTa)

### 배압(Back Pressure) 예시 코드

```java
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import com.itvillage.utils.TimeUtil;
import io.reactivex.Flowable;
import io.reactivex.schedulers.Schedulers;

import java.util.concurrent.TimeUnit;

public class MissingBackpressureExceptionExample {
    public static void main(String[] agrs) throws InterruptedException {
        Flowable.interval(1L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.DO_ON_NEXT, data))
                .observeOn(Schedulers.computation())
                .subscribe(
                        data -> {
                            Logger.log(LogType.PRINT, "# 소비자 처리 대기 중..");
                            TimeUtil.sleep(1000L);
                            Logger.log(LogType.ON_NEXT, data);
                        },
                        error -> Logger.log(LogType.ON_ERROR, error),
                        () -> Logger.log(LogType.ON_COMPLETE)
                );

        Thread.sleep(2000L);
    }
}
/*
doOnNext() | RxComputationThreadPool-2 | 21:12:57.559 | 0
doOnNext() | RxComputationThreadPool-2 | 21:12:57.562 | 1
print() | RxComputationThreadPool-1 | 21:12:57.562 | # 소비자 처리 대기 중..
doOnNext() | RxComputationThreadPool-2 | 21:12:57.562 | 2
doOnNext() | RxComputationThreadPool-2 | 21:12:57.562 | 3
doOnNext() | RxComputationThreadPool-2 | 21:12:57.562 | 4
doOnNext() | RxComputationThreadPool-2 | 21:12:57.562 | 5
doOnNext() | RxComputationThreadPool-2 | 21:12:57.562 | 6
.
.
.
doOnNext() | RxComputationThreadPool-2 | 21:12:57.647 | 126
doOnNext() | RxComputationThreadPool-2 | 21:12:57.648 | 127
onNext() | RxComputationThreadPool-1 | 21:12:58.563 | 0
onERROR() | RxComputationThreadPool-1 | 21:12:58.564 | io.reactivex.exceptions.MissingBackpressureException: Can't deliver value 128 due to lack of requests
*/
```

## 배압 전략(BackpresuureStrategy)

- RxJava에서는 BackpressureStrategy를 통해 Flowable이 통지 대기 중인 데이터를 어떻게 다룰지에 대한 배압 전략을 제공한다.

### Missing 전략

- 배압을 적용하지 않는다.
- 나중에 onBackpressureXXX()로 배압 적용을 할 수 있다.

### ERROR 전략

- 통지된 데이터가 버퍼의 크기를 초과하면 MissingBackpressureException 에러를 통지한다.
- 즉, 소비자가 생산자의 통지 속도를 따라 잡지 못할 떄 발생한다.
- 해당 코드와 같은 경우 : [링크](#배압back-pressure-예시-코드)

## BUFFER 전략 : DROP_LATEST

- 버퍼가 가득 찬 시점에 버퍼내에서 가장 최근에 버퍼로 들어온 데이터를 DROP 한다.
- DROP된 빈 자리에 버퍼 밖에서 대기하던 데이터를 채운다.

```java
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import com.itvillage.utils.TimeUtil;
import io.reactivex.BackpressureOverflowStrategy;
import io.reactivex.Flowable;
import io.reactivex.schedulers.Schedulers;

import java.util.concurrent.TimeUnit;

/**
 * - DROP_LATEST 전략  : 생산자쪽에서 데이터 통지 시점에 버퍼가 가득 차있으면 버퍼내에 있는 데이터 중에서
 * 가장 최근에 버퍼안에 들어온 데이터를 삭제하고 버퍼 밖에서 대기하는 데이터를 그 자리에 채운다.
 */
public class BackpressureBufferExample01 {
    public static void main(String[] args) {
        System.out.println("# start : " + TimeUtil.getCurrentTimeFormatted());
        Flowable.interval(300L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log("#inverval doOnNext()", data))
                .onBackpressureBuffer(
                        2,
                        () -> Logger.log("overflow!"),
                        BackpressureOverflowStrategy.DROP_LATEST)
                .doOnNext(data -> Logger.log("#onBackpressureBuffer doOnNext()", data))
                .observeOn(Schedulers.computation(), false, 1)
                .subscribe(
                        data -> {
                            TimeUtil.sleep(1000L);
                            Logger.log(LogType.ON_NEXT, data);
                        },
                        error -> Logger.log(LogType.ON_ERROR, error)
                );

        TimeUtil.sleep(2800L);
    }
}
/*
# start : 23:16:11.557
#inverval doOnNext() | RxComputationThreadPool-2 | 23:16:11.992 | 0
#onBackpressureBuffer doOnNext() | RxComputationThreadPool-2 | 23:16:11.992 | 0
#inverval doOnNext() | RxComputationThreadPool-2 | 23:16:12.292 | 1
#inverval doOnNext() | RxComputationThreadPool-2 | 23:16:12.591 | 2
#inverval doOnNext() | RxComputationThreadPool-2 | 23:16:12.891 | 3
overflow! | RxComputationThreadPool-2 | 23:16:12.893
onNext() | RxComputationThreadPool-1 | 23:16:12.998 | 0
#onBackpressureBuffer doOnNext() | RxComputationThreadPool-1 | 23:16:12.998 | 1
#inverval doOnNext() | RxComputationThreadPool-2 | 23:16:13.191 | 4
#inverval doOnNext() | RxComputationThreadPool-2 | 23:16:13.492 | 5
overflow! | RxComputationThreadPool-2 | 23:16:13.492
#inverval doOnNext() | RxComputationThreadPool-2 | 23:16:13.792 | 6
overflow! | RxComputationThreadPool-2 | 23:16:13.792
onNext() | RxComputationThreadPool-1 | 23:16:14.004 | 1
#onBackpressureBuffer doOnNext() | RxComputationThreadPool-1 | 23:16:14.004 | 3
#inverval doOnNext() | RxComputationThreadPool-2 | 23:16:14.091 | 7
#inverval doOnNext() | RxComputationThreadPool-2 | 23:16:14.392 | 8
overflow! | RxComputationThreadPool-2 | 23:16:14.392
*/
```

### BUFFER 전략 : DROP_OLDEST

- 버퍼가 가득 찬 시점에 버퍼내에서 가장 오래전에(먼저) 들어온 데이터를 DROP한다.
- DROP 된 빈 자리에는 버퍼 밖에서 대기하던 데이터를 채운다.

```java
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import com.itvillage.utils.TimeUtil;
import io.reactivex.BackpressureOverflowStrategy;
import io.reactivex.Flowable;
import io.reactivex.schedulers.Schedulers;

import java.util.concurrent.TimeUnit;

/**
 * - DROP_OLDEST 전략 : 생산자쪽에서 데이터 통지 시점에 버퍼가 가득 차있으면 버퍼내에 있는 데이터 중에서 가장 먼저(OLDEST) 버퍼
 * 안에 들어온 데이터를 삭제하고 버퍼 밖에서 대기하는 데이터를 채운다.
 */
public class BackpressureBufferExample02 {
    public static void main(String[] args){
        System.out.println("# start : " + TimeUtil.getCurrentTimeFormatted());
        Flowable.interval(300L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log("#inverval doOnNext()", data))
                .onBackpressureBuffer(
                        2,
                        () -> Logger.log("overflow!"),
                        BackpressureOverflowStrategy.DROP_OLDEST)
                .doOnNext(data -> Logger.log("#onBackpressureBuffer doOnNext()", data))
                .observeOn(Schedulers.computation(), false, 1)
                .subscribe(
                        data -> {
                            TimeUtil.sleep(1000L);
                            Logger.log(LogType.ON_NEXT, data);
                        },
                        error -> Logger.log(LogType.ON_ERROR, error)
                );

        TimeUtil.sleep(2500L);
    }
}
/*
# start : 00:12:33.570
#inverval doOnNext() | RxComputationThreadPool-2 | 00:12:34.078 | 0
#onBackpressureBuffer doOnNext() | RxComputationThreadPool-2 | 00:12:34.079 | 0
#inverval doOnNext() | RxComputationThreadPool-2 | 00:12:34.377 | 1
#inverval doOnNext() | RxComputationThreadPool-2 | 00:12:34.677 | 2
#inverval doOnNext() | RxComputationThreadPool-2 | 00:12:34.978 | 3
overflow! | RxComputationThreadPool-2 | 00:12:34.980
onNext() | RxComputationThreadPool-1 | 00:12:35.084 | 0
#onBackpressureBuffer doOnNext() | RxComputationThreadPool-1 | 00:12:35.084 | 2
#inverval doOnNext() | RxComputationThreadPool-2 | 00:12:35.277 | 4
#inverval doOnNext() | RxComputationThreadPool-2 | 00:12:35.577 | 5
overflow! | RxComputationThreadPool-2 | 00:12:35.577
#inverval doOnNext() | RxComputationThreadPool-2 | 00:12:35.877 | 6
overflow! | RxComputationThreadPool-2 | 00:12:35.877
onNext() | RxComputationThreadPool-1 | 00:12:36.089 | 2
#onBackpressureBuffer doOnNext() | RxComputationThreadPool-1 | 00:12:36.089 | 5
#inverval doOnNext() | RxComputationThreadPool-2 | 00:12:36.177 | 7
*/
```

## DROP 전략

- 버퍼에 데이터가 모두 채워진 상태가 되면 이후에 생성되는 데이터를 버리고(DROP), 버퍼가 비워지는 시점에 DROP되지 않은 데이터부터 다시 버퍼에 담는다.

```java
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import com.itvillage.utils.TimeUtil;
import io.reactivex.Flowable;
import io.reactivex.schedulers.Schedulers;

import java.util.concurrent.TimeUnit;

public class BackpressureDropExample {
    public static void main(String[] args){
        Flowable.interval(300L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log("#inverval doOnNext()", data))
                .onBackpressureDrop(dropData -> Logger.log(LogType.PRINT, dropData + " Drop!"))
                .observeOn(Schedulers.computation(), false, 1)
                .subscribe(
                        data -> {
                            TimeUtil.sleep(1000L);
                            Logger.log(LogType.ON_NEXT, data);
                        },
                        error -> Logger.log(LogType.ON_ERROR, error)
                );

        TimeUtil.sleep(5500L);
    }
}
/*
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:27.949 | 0
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:28.204 | 1
print() | RxComputationThreadPool-2 | 00:27:28.205 | 1 Drop!
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:28.504 | 2
print() | RxComputationThreadPool-2 | 00:27:28.504 | 2 Drop!
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:28.805 | 3
print() | RxComputationThreadPool-2 | 00:27:28.805 | 3 Drop!
onNext() | RxComputationThreadPool-1 | 00:27:28.957 | 0
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:29.103 | 4
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:29.403 | 5
print() | RxComputationThreadPool-2 | 00:27:29.403 | 5 Drop!
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:29.705 | 6
print() | RxComputationThreadPool-2 | 00:27:29.705 | 6 Drop!
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:30.005 | 7
print() | RxComputationThreadPool-2 | 00:27:30.005 | 7 Drop!
onNext() | RxComputationThreadPool-1 | 00:27:30.108 | 4
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:30.303 | 8
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:30.604 | 9
print() | RxComputationThreadPool-2 | 00:27:30.604 | 9 Drop!
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:30.904 | 10
print() | RxComputationThreadPool-2 | 00:27:30.904 | 10 Drop!
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:31.205 | 11
print() | RxComputationThreadPool-2 | 00:27:31.205 | 11 Drop!
onNext() | RxComputationThreadPool-1 | 00:27:31.308 | 8
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:31.504 | 12
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:31.804 | 13
print() | RxComputationThreadPool-2 | 00:27:31.804 | 13 Drop!
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:32.104 | 14
print() | RxComputationThreadPool-2 | 00:27:32.104 | 14 Drop!
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:32.405 | 15
print() | RxComputationThreadPool-2 | 00:27:32.405 | 15 Drop!
onNext() | RxComputationThreadPool-1 | 00:27:32.509 | 12
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:32.703 | 16
#inverval doOnNext() | RxComputationThreadPool-2 | 00:27:33.004 | 17
print() | RxComputationThreadPool-2 | 00:27:33.004 | 17 Drop!
*/
```

### LATEST 전략

- 버퍼에 데이터가 모두 채워진 상태가 되면 버퍼가 비워질 때까지 통지된 데이터는 버퍼 밖에서 대기하며 버퍼가 비워지는 시점에 가장 나중에(최근에) 통지된 데이터부터 버퍼에 담는다.

```java
/**
 * 통지된 데이터로 채워진 버퍼의 데이터를 소비자가 모두 소비하면 버퍼 밖에서 대기중인 통지된 데이터 중에서
 * 가장 나중에(최근에) 통지된 데이터부터 다시 버퍼에 채운다.
 */
public class BackpressureLatestExample {
    public static void main(String[] args){
        Flowable.interval(300L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log("#inverval doOnNext()", data))
                .onBackpressureLatest()
                .observeOn(Schedulers.computation(), false, 1)
                .subscribe(
                        data -> {
                            TimeUtil.sleep(1000L);
                            Logger.log(LogType.ON_NEXT, data);
                        },
                        error -> Logger.log(LogType.ON_ERROR, error)
                );

        TimeUtil.sleep(5500L);
    }
}
/*
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:17.666 | 0
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:17.921 | 1
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:18.221 | 2
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:18.520 | 3
onNext() | RxComputationThreadPool-1 | 01:32:18.673 | 0
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:18.820 | 4
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:19.121 | 5
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:19.421 | 6
onNext() | RxComputationThreadPool-1 | 01:32:19.676 | 3
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:19.721 | 7
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:20.021 | 8
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:20.320 | 9
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:20.621 | 10
onNext() | RxComputationThreadPool-1 | 01:32:20.681 | 6
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:20.921 | 11
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:21.222 | 12
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:21.522 | 13
onNext() | RxComputationThreadPool-1 | 01:32:21.685 | 10
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:21.820 | 14
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:22.122 | 15
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:22.421 | 16
onNext() | RxComputationThreadPool-1 | 01:32:22.690 | 13
#inverval doOnNext() | RxComputationThreadPool-2 | 01:32:22.722 | 17
*/
```

## Flowable 과 Observable 내부 동작 원리

### Flowable

```java
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import io.reactivex.BackpressureStrategy;
import io.reactivex.Flowable;
import io.reactivex.FlowableEmitter;
import io.reactivex.FlowableOnSubscribe;
import io.reactivex.schedulers.Schedulers;
import org.reactivestreams.Subscriber;
import org.reactivestreams.Subscription;

public class HelloRxJavaFlowableCreateExample {
    public static void main(String[] args) throws InterruptedException {
        Flowable<String> flowable =
                Flowable.create(new FlowableOnSubscribe<String>() {
                    @Override
                    public void subscribe(FlowableEmitter<String> emitter) throws Exception {
                        String[] datas = {"Hello", "RxJava!"};
                        for(String data : datas) {
                            // 구독이 해지되면 처리 중단
                            if (emitter.isCancelled())
                                return;

                            // 데이터 통지
                            emitter.onNext(data);
                        }

                        // 데이터 통지 완료를 알린다
                        emitter.onComplete();
                    }
                }, BackpressureStrategy.BUFFER); // 구독자의 처리가 늦을 경우 데이터를 버퍼에 담아두는 설정.

        flowable.observeOn(Schedulers.computation())
                .subscribe(new Subscriber<String>() {
                    // 데이터 개수 요청 및 구독을 취소하기 위한 Subscription 객체
                    private Subscription subscription;

                    @Override
                    public void onSubscribe(Subscription subscription) {
                        this.subscription = subscription;
                        this.subscription.request(Long.MAX_VALUE);
                    }

                    @Override
                    public void onNext(String data) {
                        Logger.log(LogType.ON_NEXT, data);
                    }

                    @Override
                    public void onError(Throwable error) {
                        Logger.log(LogType.ON_ERROR, error);
                    }

                    @Override
                    public void onComplete() {
                        Logger.log(LogType.ON_COMPLETE);
                    }
                });

        Thread.sleep(500L);
    }
}
```

```java
// Lamda 사용
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import io.reactivex.BackpressureStrategy;
import io.reactivex.Flowable;
import io.reactivex.schedulers.Schedulers;

public class HelloRxJavaFlowableCreateLamdaExample {
    public static void main(String[] args) throws InterruptedException {
        Flowable<String> flowable =
                Flowable.create(emitter -> {
                    String[] datas = {"Hello", "RxJava!"};
                    for(String data : datas) {
                        // 구독이 해지되면 처리 중단
                        if (emitter.isCancelled())
                            return;

                        // 데이터 발행
                        emitter.onNext(data);
                    }

                    // 데이터 발행 완료를 알린다
                    emitter.onComplete();
                }, BackpressureStrategy.BUFFER);

        flowable.observeOn(Schedulers.computation())
                .subscribe(data -> Logger.log(LogType.ON_NEXT, data),
                        error -> Logger.log(LogType.ON_ERROR, error),
                        () -> Logger.log(LogType.ON_COMPLETE),
                        subscription -> subscription.request(Long.MAX_VALUE));

        Thread.sleep(500L);
    }
}
```

### Observable

```java
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import io.reactivex.Observable;
import io.reactivex.ObservableEmitter;
import io.reactivex.ObservableOnSubscribe;
import io.reactivex.Observer;
import io.reactivex.disposables.Disposable;
import io.reactivex.schedulers.Schedulers;

public class HelloRxJavaObservableCreateExample {
    public static void main(String[] args) throws InterruptedException {
        Observable<String> observable =
                Observable.create(new ObservableOnSubscribe<String>() {
                    @Override
                    public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                        String[] datas = {"Hello", "RxJava!"};
                        for(String data : datas){
                            if(emitter.isDisposed())
                                return;

                            emitter.onNext(data);
                        }
                        emitter.onComplete();
                    }
                });

        observable.observeOn(Schedulers.computation())
                .subscribe(new Observer<String>() {
                    @Override
                    public void onSubscribe(Disposable disposable) {
                        // 아무 처리도 하지 않음.
                    }

                    @Override
                    public void onNext(String data) {
                        Logger.log(LogType.ON_NEXT, data);
                    }

                    @Override
                    public void onError(Throwable error) {
                        Logger.log(LogType.ON_ERROR, error);
                    }

                    @Override
                    public void onComplete() {
                        Logger.log(LogType.ON_COMPLETE);
                    }
                });

        Thread.sleep(500L);
    }
}
```

```java
// Lamda 사용
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import io.reactivex.Observable;
import io.reactivex.schedulers.Schedulers;

public class HelloRxJavaObservableCreateLamdaExample {
    public static void main(String[] args) throws InterruptedException {
        Observable<String> observable =
                Observable.create(emitter -> {
                    String[] datas = {"Hello", "RxJava!"};
                    for(String data : datas){
                        if(emitter.isDisposed())
                            return;

                        emitter.onNext(data);
                    }
                    emitter.onComplete();
                });

        observable.observeOn(Schedulers.computation())
                .subscribe(
                        data -> Logger.log(LogType.ON_NEXT, data),
                        error -> Logger.log(LogType.ON_ERROR, error),
                        () -> Logger.log(LogType.ON_COMPLETE),
                        disposable -> {/**아무것도 하지 않는다.*/}
                );

        Thread.sleep(500L);
    }
}
```
> 이 글은 inflearn에 있는 Kevin의 알기 쉬운 RxJava 1부를 공부하고 작성한 글입니다.   
> [강의영상 링크](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-1#description)

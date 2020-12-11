---
title: RxJava - Single vs Maybe vs Completable
author: Yun-Jae Na
date: 2020-12-11 10:00:00 +0900
categories: [Android, RxJava]
tags: [Android, RxJava]
image : https://drive.google.com/uc?export=view&id=1EKGE5Tbptu_0S3-y3Pazr-1vSTtfNy1K
---

## Single

- 데이터를 1건만 통지하거나 에러를 통지한다.
- 데이터 통지 자체가 완료를 의미하기 때문에 완료 통지는 하지 않는다.
- 데이터를 1건만 통지하므로 데이터 개수를 요청할 필요가 없다.
- onNext(), onComplete()가 없으며 이 둘을 합한 onSuccess()를 제공한다.
- Single의 대표적인 소비자는 SingleObserver이다.
- 클라이언트의 요청에 대응하는 서버의 응답이 Single을 사용하기 좋은 대표적인 예다.

```java
/**
 * Single 클래스를 사용하여 현재 날짜와 시각을 통지하는 예제
 */
public class SingleCreateExample {
    public static void main(String[] args) {
        Single<String> single = Single.create(new SingleOnSubscribe<String>() {
            @Override
            public void subscribe(SingleEmitter<String> emitter) throws Exception {
                emitter.onSuccess(DateUtil.getNowDate());
            }
        });

        single.subscribe(new SingleObserver<String>() {
            @Override
            public void onSubscribe(Disposable disposable) {
                // 아무것도 하지 않음.
            }

            @Override
            public void onSuccess(String data) {
                Logger.log(LogType.ON_SUCCESS, "# 날짜시각: " + data);
            }

            @Override
            public void onError(Throwable error) {
                Logger.log(LogType.ON_ERROR, error);
            }
        });
    }
}
/*
onSuccess() | main | 11:02:22.407 | # 날짜시각: 2020-12-11 11:02:22
*/
```

```java
public class SingleLamdaExample {
    public static void main(String[] args) {
        Single<String> single = Single.create(emitter -> emitter.onSuccess(DateUtil.getNowDate()));

        single.subscribe(
                data -> Logger.log(LogType.ON_SUCCESS, "# 날짜시각: " + data),
                error -> Logger.log(LogType.ON_ERROR, error)
        );
    }
}
/*
onSuccess() | main | 11:02:22.407 | # 날짜시각: 2020-12-11 11:02:22
*/
```

## Maybe

- 데이터를 1건만 통지하거나 1건도 통지하지 않고 완료 또는 에러를 통지한다.
- 데이터 통지 자체가 완료를 의미하기 때문에 오나료 통지는 하지 않는다.
- 단,데이터를 1건도 통지하지 않고 처리가 종료될 경우에는 완료 통지를 한다.
- Maybe의 대표적인 소비자는 MaybeObserver이다.

```java
import com.itvillage.utils.DateUtil;
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import io.reactivex.Maybe;
import io.reactivex.MaybeEmitter;
import io.reactivex.MaybeObserver;
import io.reactivex.MaybeOnSubscribe;
import io.reactivex.disposables.Disposable;

/**
 * Maybe 클래스를 이용하여 데이터를 통지하는 예제
 * 데이터 통지가 없을 경우 onComplete()가 통지가 된다.
 */
public class MaybeCreateExample {
    public static void main(String[] args){
        Maybe<String> maybe = Maybe.create(new MaybeOnSubscribe<String>() {
            @Override
            public void subscribe(MaybeEmitter<String> emitter) throws Exception {
                emitter.onSuccess(DateUtil.getNowDate());

//                emitter.onComplete();
            }
        });

        maybe.subscribe(new MaybeObserver<String>() {
            @Override
            public void onSubscribe(Disposable disposable) {
                // 아무것도 하지 않음.
            }

            @Override
            public void onSuccess(String data) {
                Logger.log(LogType.ON_SUCCESS, "# 현재 날짜시각: " + data);
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
    }
}
/*
onSuccess() | main | 11:11:21.523 | # 현재 날짜시각: 2020-12-11 11:11:21
*/
```

```java
import com.itvillage.utils.DateUtil;
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import io.reactivex.Maybe;

public class MaybeLamdaExample {
    public static void main(String[] args) {
        Maybe<String> maybe = Maybe.create(emitter -> {
            emitter.onSuccess(DateUtil.getNowDate());
//            emitter.onComplete();
        });

        maybe.subscribe(
                data -> Logger.log(LogType.ON_SUCCESS, "# 현재 날짜시각: " + data),
                error -> Logger.log(LogType.ON_ERROR, error),
                () -> Logger.log(LogType.ON_COMPLETE)
        );
    }
}
/*
onSuccess() | main | 11:11:21.523 | # 현재 날짜시각: 2020-12-11 11:11:21
*/
```

```java
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import io.reactivex.Maybe;

public class MaybeJustExample {
    public static void main(String[] args) {
//        Maybe.just(DateUtil.getNowDate())
//                .subscribe(
//                        data -> Logger.log(LogType.ON_SUCCESS, "# 현재 날짜시각: " + data),
//                        error -> Logger.log(LogType.ON_ERROR, error),
//                        () -> Logger.log(LogType.ON_COMPLETE)
//                );

        Maybe.empty()
                .subscribe(
                        data -> Logger.log(LogType.ON_SUCCESS, data),
                        error -> Logger.log(LogType.ON_ERROR, error),
                        () -> Logger.log(LogType.ON_COMPLETE)
                );
    }
}
/*
Maybe.just() 코드 결과 -> onSuccess() | main | 11:18:36.774 | # 현재 날짜시각: 2020-12-11 11:18:36
Maybe.empty() 코드 결과 -> onComplete() | main | 11:17:07.133
*/
```

```java
// Single 객체를 파라미터로 전달받아 Maybe 객체를 만들 수 있다.
import com.itvillage.utils.DateUtil;
import com.itvillage.utils.LogType;
import com.itvillage.utils.Logger;
import io.reactivex.Maybe;
import io.reactivex.Single;

public class MaybeFromSingle {
    public static void main(String[] args){
        Single<String> single = Single.just(DateUtil.getNowDate());
        Maybe.fromSingle(single)
                .subscribe(
                        data -> Logger.log(LogType.ON_SUCCESS, "# 현재 날짜시각: " + data),
                        error -> Logger.log(LogType.ON_ERROR, error),
                        () -> Logger.log(LogType.ON_COMPLETE)
                );
    }

}
/*
onSuccess() | main | 11:20:18.107 | # 현재 날짜시각: 2020-12-11 11:20:17
*/
```

## Completable

- 데이터 생산자이지만 데이터를 1건도 통지하지 않고 완료 또는 에러를 통지한다.
- 데이터 통지의 역할 대신에 Completable 내에서 특정 작업을 수행한 후, 해당 처리가 끝났음을 통지하는 역할을 한다.
- Completable의 대표적인 소비자는 CompletableObserver이다.

```java
/**
 * Completable 을 사용하여 어떤 작업을 수행한 후, 완료를 통지하는 예제
 */
public class CompletableCreateExample {
    public static void main(String[] args) throws InterruptedException {
        Completable completable = Completable.create(new CompletableOnSubscribe() {
            @Override
            public void subscribe(CompletableEmitter emitter) throws Exception {
                // 데이터를 통지하는것이 아니라 특정 작업을 수행한 후, 완료를 통지한다.
                int sum = 0;
                for (int i = 0; i < 100; i++) {
                    sum += i;
                }
                Logger.log(LogType.PRINT, "# 합계: " + sum);

                emitter.onComplete();
            }
        });

        completable.subscribeOn(Schedulers.computation())
                .subscribe(new CompletableObserver() {
                    @Override
                    public void onSubscribe(Disposable disposable) {
                        // 아무것도 하지 않음
                    }

                    @Override
                    public void onComplete() {
                        Logger.log(LogType.ON_COMPLETE);
                    }

                    @Override
                    public void onError(Throwable error) {
                        Logger.log(LogType.ON_ERROR, error);
                    }
                });

        TimeUtil.sleep(100L);
    }
}
/*
print() | RxComputationThreadPool-1 | 11:24:29.090 | # 합계: 4950
onComplete() | RxComputationThreadPool-1 | 11:24:29.097
*/
```

```java
public class CompletableLamdaExample {
    public static void main(String[] args) {
        Completable completable = Completable.create(emitter -> {
            // 데이터를 발행하는것이 아니라 특정 작업을 수행한 후, 완료를 통지한다.
            int sum = 0;
            for (int i = 0; i < 100; i++) {
                sum += i;
            }
            Logger.log(LogType.PRINT, "# 합계: " + sum);

            emitter.onComplete();
        });

        completable.subscribeOn(Schedulers.computation())
                .subscribe(
                        () -> Logger.log(LogType.ON_COMPLETE),
                        error -> Logger.log(LogType.ON_ERROR, error)
                );

        TimeUtil.sleep(100L);
    }
}
/*
print() | RxComputationThreadPool-1 | 11:24:29.090 | # 합계: 4950
onComplete() | RxComputationThreadPool-1 | 11:24:29.097
*/
```


> 이 글은 inflearn에 있는 Kevin의 알기 쉬운 RxJava 1부를 공부하고 작성한 글입니다.   
> [강의영상 링크](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-1#description)

---
title: RxJava - 테스트(2)
author: Yun-Jae Na
date: 2021-01-12 23:28:00 +0930
categories: [Android, RxJava]
tags: [Android, RxJava]
image : https://drive.google.com/uc?export=view&id=1EKGE5Tbptu_0S3-y3Pazr-1vSTtfNy1K
---

## 테스트를 위한 TestSubscriber / TestObserver

- 테스트 용도로 사용되는 소비자 클래스이다.

- assertXXX 함수를 이용해 통지된 데이터를 검증할 수 있다.

- awaitxxxxx 함수를 이용해서 지정된 시간 동안 대기하거나 완료 또는 에러 이벤트가 발생할 때까지 대기할 수 있다.

- 완료, 에러, 구독 해지 등의 이벤트 발생 결과 값을 이용해서 데이터를 검증할 수 있다.

### assertEmpty

- 테스트 시점까지 통지받은 데이터가 없다면 테스트에 성공한다.

- Observable.empty()로 생성 시, 완료를 통지를 하기때문에 테스트가 실패한다.

- 즉, 통지 이벤트 자체가 없는지를 테스트할 수 있다.

```java
/**
 * assertEmpty를 사용하여 해당 시점까지 통지된 데이터가 있는지 검증하는 예제
 */
public class AssertEmptyTest {
    // 테스트 실패 예제
    @Test
    public void getCarStreamEmptyFailTest() {
        // when
        Observable<Car> observable = SampleObservable.getCarStream();
        TestObserver<Car> observer = observable.test();

        // then
        observer.awaitDone(100L, TimeUnit.MILLISECONDS).assertEmpty();

    }

    // 테스트 성공 예제
    @Test
    public void getCarStreamEmptySuccessTest() {
        // when
        Observable<Car> observable = SampleObservable.getCarStream();
        TestObserver<Car> observer = observable.delay(1000L, TimeUnit.MILLISECONDS).test();

        // then
        observer.awaitDone(100L, TimeUnit.MILLISECONDS).assertEmpty();

    }
}
```

### assertValue

- 통지된 데이터가 한개인 경우에 사용한다.

- 즉, 통지된 데이터가 한개뿐이므로 파라미터로 입력된 값과 같다면 테스트에 성공한다.

```java
/**
 * assertValue를 이용한 데이터 검증 예제
 */
public class AssertValueTest {
    @Test
    public void assertValueTest() {
        Observable.just("a")
                .test()
                .assertValue("a");
    }

    @Test
    public void getCarMarkerAssertValueTest(){
        SampleObservable.getCarMakerStream()
                .filter(carMaker -> carMaker.equals(CarMaker.SAMSUNG))
                .test()
                .awaitDone(1L, TimeUnit.MILLISECONDS)
                .assertValue(CarMaker.SAMSUNG);
    }
}
```

### assertValues

- 통지된 데이터가 한개 이상인 경우에 사용한다.

- 즉, 통지된 데이터의 값과 순서가 파라미터로 입력된 데이터의 값과 순서와 일치하면 테스트에 성공한다.

```java
/**
 * assertValues를 이용하여 조건에 맞는 1개 이상의 데이터의 값과 순서과 일치하는지 검증하는 예제
 */
public class AssertValuesTest {
    @Test
    public void getCarMakerAssertValueTest() {
      SampleObservable.getDuplicatedCarMakerStream()
              .filter(carMaker -> carMaker.equals(CarMaker.CHEVROLET))
              .test()
              .awaitDone(1L, TimeUnit.MILLISECONDS)
              .assertValues(CarMaker.CHEVROLET, CarMaker.CHEVROLET);
    }
}
```

### assertNoValues

- 해당 시점까지 통지된 데이터가 없으면 테스트에 성공한다.

- 완료 통지와 에러 통지는 테스트 대상에서 제외된다.

```java
/**
 * assertNoValues를 이용해 통지 시점까지 통지된 데이터가 없는지 검증하는 예제
 */
public class AssertNotValueTest {
    @Test
    public void assertNoValuesTest() {
        Observable.interval(200L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.ON_NEXT, data))
                .filter(data -> data > 5)
                .test()
                .awaitDone(1000L, TimeUnit.MILLISECONDS)
                .assertNoValues();
    }
}
/*
onNext() | RxComputationThreadPool-1 | 21:23:52.615 | 0
onNext() | RxComputationThreadPool-1 | 21:23:52.790 | 1
onNext() | RxComputationThreadPool-1 | 21:23:52.989 | 2
onNext() | RxComputationThreadPool-1 | 21:23:53.190 | 3
onNext() | RxComputationThreadPool-1 | 21:23:53.388 | 4
*/
```

### assertResult

- 해당 시점까지 통지를 완료했고, 데이터와 파라미터로 입력된 데이터의 값과 순서가 같으면 테스트에 성공한다.

- assertValues와의 차이점은 해당 시점까지 완료 통지를 받았으냐 받지 않았으냐이다.

```java
/**
 * assertResult를 사용하여 통지 완료 후, 통지된 데이터와 파라미터로 입력된 데이터의 값과 순서가 일치하는지 검증하는 예제
 */
public class AssertResultTest {
    // 테스트 실패 예제 (롼료 통지가 없음)
    @Test
    public void assertResultFailTest(){
        Observable.interval(200L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.ON_NEXT, data))
                .filter(data -> data > 3)
                .test()
                .awaitDone(1100L, TimeUnit.MILLISECONDS)
                .assertResult(4L);
    }

    // 테스트 성공 예제
    @Test
    public void assertResultSuccessTest(){
        Observable.interval(200L,  TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.ON_NEXT, data))
                .take(5)
                .filter(data -> data > 3)
                .test()
                .awaitDone(1100L, TimeUnit.MILLISECONDS)
                .assertResult(4L);
    }
}
```

### assertError

- 해당 시점까지 에러 통지가 있으면 테스트에 성공한다.

- 단순히 에러 통지가 있었는지의 여부와 구체적으로 발생한 에러가 맞는지를 테스트할 수 있다.

```java
/**
 * assertError를 이용하여 데이터 통지 중에 에러가 발생했는지를 검증하는 예제
 */
public class AssertErrorTest extends RxJavaTest {
    // 단순히 예외가 발생했는지를 테스트하는 예제
    @Test
    public void assertErrorTest01(){
        Observable.interval(100L, TimeUnit.MILLISECONDS)
                .map(data -> {
                    long value;
                    if(data == 4)
                        value = data / 0;
                    else
                        value = data / 2;
                    return value;
                })
                .test()
                .awaitDone(1000L, TimeUnit.MILLISECONDS)
                .assertError(Throwable.class);
    }

    // 구체적인 예외 클래스를 비교 테스트하는 예제
    @Test
    public void assertErrorTest02(){
        Observable.interval(100L, TimeUnit.MILLISECONDS)
                .map(data -> {
                    long value;
                    if(data == 4)
                        value = data / 0;
                    else
                        value = data / 2;
                    return value;
                })
                .test()
                .awaitDone(1000L, TimeUnit.MILLISECONDS)
                .assertError(error -> error.getClass() == ArithmeticException.class);
    }

}
```

### assertComplete

- 해당 시점까지 완료 통지가 있으면 테스트에 성공한다.

```java
/**
 * assertComplete을 이용하여 A 지점과 B 지점의 매출 합계 처리가 지정된 시간안에 끝나는지 검증하는 예제
 */
public class AssertCompleteTest {
    @Test
    public void assertCompleteTest() {
        SampleObservable.getSalesOfBranchA()
                .zipWith(
                        SampleObservable.getSalesOfBranchB(),
                        (a, b) -> {
                            TimeUtil.sleep(100L);
                            return a + b;
                        }
                )
                .test()
                .awaitDone(3000L, TimeUnit.MILLISECONDS)
                .assertComplete();
    }
}
```

### assertNotComplete

- 해당 시점까지 완료 통지가 없으면 테스트에 성공한다.

```java
/**
 * assertNotComplete를 이용하여 A 지점과 B 지점의 매출 합계 처리가 지정된 시간안에 끝나지않았는지 검증하는 예제
 */
public class AssertNotCompleteTest {
    @Test
    public void assertNotCompleteTest() {
        SampleObservable.getSalesOfBranchB()
                .zipWith(
                        SampleObservable.getSalesOfBranchB(),
                        (a, b) -> {
                            TimeUtil.sleep(1000L);
                            return a + b;
                        }
                )
                .test()
                .awaitDone(3000L, TimeUnit.MILLISECONDS)
                .assertNotComplete();
    }
}
```

### awaitDone

- 파라미터로 지정된 시간 동안 대기시키거나 지정된 시간 전에 완료 통지나 에러 통지가 있을떄까지만 대기시킨다.

```java
/**
 * awaitDone 을 이용해서 지정된 시간 또는 완료 통지가 있을떄까지 대기하는 예제
 */
public class AssertDoneTest {
    // 지정된 시간까지 완료 통지가 없이, 해당 시점까지 전달 받은 데이터의 개수가 맞는지 검증하는 예제
    @Test
    public void awaitDoneTest01() {
        Observable.interval(200L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.DO_ON_NEXT, data))
                .take(5)
                .doOnComplete(() -> Logger.log(LogType.DO_ON_COMPLETE))
                .doOnError(error -> Logger.log(LogType.DO_ON_ERROR, error.getMessage()))
                .test()
                .awaitDone(500L, TimeUnit.MILLISECONDS)
                .assertNotComplete()
                .assertValueCount(2);
    }

    // 지정된 시간 전에 완료 통지가 있어, 완료 통지 시점까지만 대기하고 전달 받은 데이터의 개수가 맞는지 검증하는 예제
    @Test
    public void awaitDoneTest02() {
        Observable.interval(200L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.DO_ON_NEXT, data))
                .take(5)
                .doOnComplete(() -> Logger.log(LogType.DO_ON_COMPLETE))
                .doOnError(error -> Logger.log(LogType.DO_ON_ERROR, error.getMessage()))
                .test()
                .awaitDone(1500L, TimeUnit.MILLISECONDS)
                .assertComplete()
                .assertValueCount(5);
    }
}
```

### await

- 생산자쪽에서 완료 통지나 에러 통지가 있을떄까지 쓰레드를 대기시킨다.

- 파라미터로 지정된 시간동안 대기하며, 대기 시간내에 완료 통지가 있었는지 여부를 검증한다.

```java
/**
 * await 을 이용해서 쓰레드에서 통지가 끝날때까지 또는 지정한 시간만큼 대기하는 예제
 */
public class AwaitTest {
    // 생산자쪽에서 완료 통지를 보낼때까지 대기한 후, 완료 및 통지된 데이터 개수를 검증하는 예제
    @Test
    public void awaitTest() throws InterruptedException {
        Observable.interval(100L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.DO_ON_NEXT, data))
                .take(5)
                .doOnComplete(() -> Logger.log(LogType.ON_COMPLETE))
                .doOnError(error -> Logger.log(LogType.DO_ON_ERROR, error.getMessage()))
                .test()
                .await()
                .assertComplete()
                .assertValueCount(5);
    }

    // 지정한 시간동안 대기하면서 대기 시간내에 완료 통지를 받았는지 여부를 검증하는 예제
    @Test
    public void awaitTest02() throws InterruptedException {
        boolean result = Observable.interval(1000L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.DO_ON_NEXT, data))
                .take(5)
                .doOnComplete(() -> Logger.log(LogType.DO_ON_NEXT))
                .doOnError(error -> Logger.log(LogType.DO_ON_ERROR, error.getMessage()))
                .test()
                .await(2000L, TimeUnit.MILLISECONDS);

        assertThat(result, is(false));
    }
}
```

### awaitCount

- 파라미터로 지정된 개수만큼 통지될 때 까지 쓰레드를 대기시킨다.

```java
/**
 * awaitCount 를 이용해 지정된 개수만큼 통지가 될 떄까지 대기
 */
public class AwaitCountTest {
    // 지정된 개수만큼 대기하고 완료 통지 유무, 통지 유무, 통지된 데이터 개수 및 데이터 개수 및 데이터의 값과 순서를 검증하는 예제
    @Test
    public void awaitCountTest() {
        Observable.interval(200L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.DO_ON_NEXT, data))
                .take(5)
                .doOnComplete(() -> Logger.log(LogType.DO_ON_COMPLETE))
                .doOnError(error -> Logger.log(LogType.DO_ON_ERROR, error.getMessage()))
                .test()
                .awaitCount(3)
                .assertNotComplete()
                .assertValueCount(3)
                .assertValues(0L, 1L, 2L);
    }
}
```
### isTerminated

```java
public class TestObserverEventTest {
    // 완료 통지 이벤트가 발생해서 종료 되었는지를 검증하는 예제
    @Test
    public void isTerminalEventTest(){
        boolean result = Observable.interval(200L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.DO_ON_NEXT, data))
                .take(5)
                .doOnComplete(() -> Logger.log(LogType.DO_ON_COMPLETE))
                .doOnError(error -> Logger.log(LogType.DO_ON_ERROR, error.getMessage()))
                .test()
                .awaitCount(5)
                .isTerminated();

        assertThat(result, is(true));
    }

    // 에러 통지 이벤트가 발생해서 종료 되었는지를 검증하는 예제
    @Test
    public void isErrorEventTest(){
        boolean result = Observable.interval(200L, TimeUnit.MILLISECONDS)
                .doOnNext(data -> Logger.log(LogType.DO_ON_NEXT, data))
                .map(data -> {
                    if(data == 2)
                        throw new RuntimeException("Error happened");
                    return data;
                })
                .doOnComplete(() -> Logger.log(LogType.DO_ON_COMPLETE))
                .doOnError(error -> Logger.log(LogType.DO_ON_ERROR, error.getMessage()))
                .test()
                .awaitCount(5)
                .isTerminated();

        assertThat(result, is(true));
    }
}
```
> 이 글은 inflearn에 있는 Kevin의 알기 쉬운 RxJava 2부를 공부하고 작성한 글입니다.   
> [강의영상 링크](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-2)

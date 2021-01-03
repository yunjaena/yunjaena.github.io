---
title: RxJava - 테스트(1)
author: Yun-Jae Na
date: 2021-01-01 16:40:00 +0930
categories: [Android, RxJava]
tags: [Android, RxJava]
image : https://drive.google.com/uc?export=view&id=1EKGE5Tbptu_0S3-y3Pazr-1vSTtfNy1K
---

## 테스트를 위한 blockingXXX 함수

- 비동기 처리 결과를 테스트하려면 현재 쓰레드에서 호출 대상 쓰레드의 실행 결과를 반환 받을때까지 대기할 수 있어야 한다.

- RxJava에서는 현재 쓰레드에서 호출 대상 쓰레드의 처리 결과를 받을 수 있는 blockingXXX 함수를 제고안다.

- Observable에서 통지되고 가공 처리된 결과 데이터를 현재 쓰레드에 반환하므로, 반환된 결과 값과 예상되는 기대값을 비교해서 단위 테스트를 수행할 수 있다.

## RxJava의 API를 사용하지 않은 Unit Test

- 생산자에서 처리하는 쓰레드를 별도로 지정을 하였을 경우 두 개의 쓰레드가 동시에 돌아가면서 예상하는 값이 제대로 나오지 않게 된다.

```java
/**
 * RxJava의 API를 사용하지 않은 Unit Test 예제
 */
public class UnitTestNotByRxJava {
    @Test
    public void getCarMakerSyncTest() {
        List<CarMaker> carMakerList = new ArrayList<>();
        SampleObservable.getCarMakerStream()
                .subscribe(data -> carMakerList.add(data));

        assertThat(carMakerList.size(), is(5));
    }
}
/*
Expected: is <5>
     but: was <0>
java.lang.AssertionError:
*/
```

### blockingFirst

- 생산자가 통지한 첫번쨰 데이터를 반환한다.

- 통지된 데이터가 없을 경우 NoSuchElementException을 발생 시킨다.

![blockingFirst](https://drive.google.com/uc?export=view&id=12Slz_BnuMZoSzZSW8WA15M_u_mbv7sZF)

```java
/**
 * blockingFirst를 사용한 통지된 첫번째 데이터를 테스트하는 예제
 */
public class BlockingFirst {
    // Car 리스트 중에서 첫번째 Car를 테스트
    @Test
    public void getCarStreamFirstTest(){
        // when
        Car car = SampleObservable.getCarStream().blockingFirst();
        String actual = car.getCarName();

        // then
        assertThat(actual, is("말리부"));
    }

    @Test
    public void getSalesOfBranchAFirstTest(){
        // when
        int actual = SampleObservable.getSalesOfBranchA()
                .take(6)
                .blockingFirst();

        // then
        assertThat(actual, is(15_000_000));
    }
}
```

### blockingLast

- 샘산지가 통지한 마지막 데이터를 반환한다.

- 통지된 데이터가 없을 경우 NoSuchElementException을 발생 시킨다.

- 결과를 반환하는 시점이 완료를 통지하는 시점이므로 완료 통지가 없는 데이터 통지일 경우 사용할 수 없다.

![blockingLast](https://drive.google.com/uc?export=view&id=1Dwp0OKGpjWZw5pRzQsyfdu1b8Jf6MqwR)

```java
/**
 * blocingLast를 사용한 통지된 마지막 데이터를 테스트하는 예제
 */
public class BlockingLastTest {
    // Car 리스트 중 마지막 Car 테스트
    @Test
    public void getCarStreamLastTest() {
        // when
        Car car = SampleObservable.getCarStream().blockingLast();
        String actual = car.getCarName();

        // then
        assertThat(actual, is("SM5"));
    }

    // A 지점의 월간 매출액 중 6월 달 매출액 테스트
    @Test
    public void getSalesOfBranchAtLastTest() {
        // when
        int actual = SampleObservable.getSalesOfBranchA()
                .take(6)
                .blockingLast();

        // then
        assertThat(actual, is(40_000_000));
    }
}
```

### blockingSingle

- 생산자가 한 개의 데이터를 통지하고 완료되면 해당 데이터를 반환한다.

- 2개 이상의 데이터를 통지할 경우에는 IllegalArgumentException를 발생시킨다.

![blockingSingle](https://drive.google.com/uc?export=view&id=1klogGk8qui4X_E-z7cd8mOF2TKi2dpUH)

```java
**
 * blockingSingle을 사용한 통지된 첫번째 데이터를 테스트하는 예제
 */
public class BlockingSingleTest {

    // A 지점의 월간 매출 중에 30,000,000원 이상인 매출액의 첫번쨰 데이터를 테스트
    @Test
    public void totalSalesOfBranchTest(){
        int actual = SampleObservable.getSalesOfBranchA()
                .filter(sale -> sale > 30_000_000)
                .take(1)
                .blockingSingle();

        assertThat(actual, is(35_000_000));
    }

    // A 지점의 월간 매출 중에 30,000,000원 이상인 매출액의 첫번쨰 데이터를 테스트
    @Test(expected = IllegalArgumentException.class)
    public void totalSalesOfBranchTest2(){
        int actual = SampleObservable.getSalesOfBranchA()
                .filter(sale -> sale > 30_000_000)
                .take(2)
                .blockingSingle();
    }
}
```

### blockingGet

- 생산자가 0개 또는 1개의 데이터를 통지하고 완료되면 해당 데이터를 반환한다.

- 즉, 생산자가 Maybe일 경우 사용할 수 있다.

![blockingGet](https://drive.google.com/uc?export=view&id=1O2vt4VKAXLOjdBWaxOHzhfMq7V7l8IOH)

```java
/**
 * blockingGet 을 사용한 통지 데이터 테스트 예제
 */
public class BlockingGetTest {
    @Test
    public void blockingGetEmptyTest() {
        // then
        assertThat(Observable.empty().firstElement().blockingGet(), is(nullValue()));
    }

    // A 지점의 월간 매출 합계 테스트
    @Test
    public void totalSalesOfBranchATest() {
        // when
        int totalSales = SampleObservable.getSalesOfBranchA()
                .reduce((a, b) -> a + b)
                .blockingGet();
        // then
        assertThat(totalSales, is(326_000_000));
    }

    // A, B, C 지점의 연간 매출 합계 테스트
    @Test
    public void salesAllBranchTest() {
        // when
        int totalSales = Observable.zip(
                SampleObservable.getSalesOfBranchA(),
                SampleObservable.getSalesOfBranchB(),
                SampleObservable.getSalesOfBranchC(),
                (a, b, c) -> a + b + c
        )
                .reduce((a, b) -> a + b)
                .blockingGet();

        // then
        assertThat(totalSales, is(992_000_000));

    }
}
```

### blockingIterable

- 생산자가 통지한 모든 데이터를 받을 수 있는 Iterable을 통해 Iterator를 얻게 한다.

- 구독 후, Iterator의 next() 메서드를 호출하는 시점부터 처리한다.

![blockingIterable](https://drive.google.com/uc?export=view&id=102PXBD5b-cqY5r-zRdCXvie02MSVSxw1)

```java
/**
 * blockingIterable을 사용한 통지 데이터 테스트 예제
 */
public class BlockingIterableTest {
    // 전체 CarMaker의 요소가 맞는지 모두 테스트한다.
    @Test
    public void getCarMakerIterableTest() {
        // when
        Iterable<CarMaker> iterable = SampleObservable.getCarMakerStream()
                .blockingIterable();

        Iterator<CarMaker> iterator = iterable.iterator();

        // then
        assertThat(iterator.hasNext(), is(true));

        assertThat(iterator.next(), is(CarMaker.CHEVROLET));
        assertThat(iterator.next(), is(CarMaker.HYUNDAE));
        assertThat(iterator.next(), is(CarMaker.SAMSUNG));
        assertThat(iterator.next(), is(CarMaker.SSANGYOUNG));
        assertThat(iterator.next(), is(CarMaker.KIA));

    }
}
```

### blockingForEach

- 생산자가 통지한 데이터를 순차적으로 모두 통지한다.

- 통지된 각각의 데이터가 모두 조건에 맞아야 true를 반환한다.

![blockingForEach](https://drive.google.com/uc?export=view&id=1mYVGW3ZXummWe36YCXTbtVwtoP_Ads8e)

```java
/**
 * blockingForEach를 사용해 통지된 데이터 저부를 테스트한다.
 */
public class BlockingForEachTest {
    // A 구간의 속도 중에서 110 이상인 속도만 필터링 되었는지 테스트
    @Test
    public void getSpeedOfSectionAForEachTest(){
        SampleObservable.getSpeedOfSectionA()
                .filter(speed -> speed > 110)
                .blockingForEach(speed -> assertThat(speed, greaterThan(110)));
    }
}
```

### blockingSubscribe

- 통지된 원본 데이터를 호출한 원본 쓰레드에서 부수적인 처리를 할 수 있도록 해준다.

- 소비자가 전달 받은 데이터로 어떤 부수적인 처리 할 떄 이 처리 결과를 테스트 할 수 있다.

![blockingSubscribe](https://drive.google.com/uc?export=view&id=1P3mFDfmR0jmF6LjfyZQRoegYuSlyFO-o)

```java
/**
 * blockingSubscribe를 사용해 구독 후 소비자의 처리로 인해 부수 작용이 발생한 결과를 테스트하는 예제
 */
public class BlockingSubscribeTest {

    // A 지점의 월간 매출 합계를 부수 작용으로 테스트
    @Test
    public void avgTempOfSeoulTest(){
        Calculator calculator = new Calculator();

        SampleObservable.getSalesOfBranchA()
                .blockingSubscribe(data -> calculator.setSum(data));

        assertThat(calculator.getSum(), is(326_000_000));
    }
}
```



> 이 글은 inflearn에 있는 Kevin의 알기 쉬운 RxJava 2부를 공부하고 작성한 글입니다.   
> [강의영상 링크](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-2)

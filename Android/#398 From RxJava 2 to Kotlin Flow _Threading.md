# [From RxJava 2 to Kotlin Flow: Threading](https://androidweekly.us2.list-manage.com/track/click?u=887caf4f48db76fd91e20a06d&id=786841c330&e=7b5d1e4925)(Android Weekly #398)

## [기본 알고가기]

##### subscribeOn : 생산자 처리 작업
- upStream 작업 설정
- 이친구는 한번만 설정

##### observeOn : 소비자 처리 작업

-  downStream 작업 설정

![img](https://k.kakaocdn.net/dn/dbR4uV/btqySsa31sU/aP9kou0qnTjDcdYYVycXJk/img.png)

<hr>

## [Main Issue]

Rxjava2의 Observable과 코틀린의 Flow는 모두 Cold Stream이다.

즉, subscription이 있기 전까지 내부 코드가 실행되지 않는다.



> In RxJava there are also other types such as `Flowable`, `Single`, etc. In the article we’ll talk about `Observable` only because everything else applies to other types as well.

Rxjava에는 Flowable과 Single과 같은 다른 유형도 있지만 우리는 Observable에 대해 이야기 할것이다.

왜냐하면 다른 모든것들 또한 적용되기 때문이다.



### subscribeOn

subscribeOn은 연산자 체인에서 ``어떤 스케줄러에 의해 시작될것인가``를 의미한다.

**하지만, 연산자 체인안에서 subscribeOn 의 위치는 상관이 없다.**

또한 다양한 스케줄러를 위하여 여러개를 쓸 필요없다. 어차피 위에가 이긴다.

![img](https://miro.medium.com/max/510/1*LBKRVDIpr_rxjylKadaSgg.png)

### observeOn

observeOn은 호출이 되면 아래 스케줄러를 바꾼다.

![img](https://miro.medium.com/max/533/1*uCiZwTFtiPoh8KZdMUDLpg.png)

빨간색 : ioThread

초록색 : mainThread



![img](https://miro.medium.com/max/515/1*ES5NJCpKCmVVhdD36UurzQ.png)



### just + defer

![img](https://miro.medium.com/max/533/1*4EVWYZCJ4u9mPrJazHQaeg.png)

just 에서 무거운 작업을 할 시 subscribeOn을 타지않고 바로 계산된다. (즉, mainThread에서 계산된다.)



이러한 문제를 해결하는 방법은 ``Observable.just``를 ``Observable.defet``로 감싸면 된다.

그러면 subscribeOn에 지정된 스레드에서 계산이 될 것이다.

![img](https://miro.medium.com/max/608/1*x8ASbBk5N1KnVIDaT8KRHA.png)



### flatMap concurrency and parallelism

Flatmap에서 동시성과 병렬성또한 까다롭다.

네트워크에서 id목록을 로드하는 경우를 예로 들어보자

![img](https://miro.medium.com/max/597/1*KmPkrVtEedgkXu08JxSMew.png)

> 위 결과는 1,2,3으로 항상 동일하다.

우리는 병렬적으로 처리될것이라 생각하지만 그렇지 않다.

- SubscribeOn은 fromIterable의 thread를 지정한다.
- 따라서 flatMap은 하나의 스레드에서 작동한다.

이를 해결하기 위해서는 flatMap도 하나의 Observable이기 때문에 해당 스트림에 스레드를 등록해주어야 한다.

![img](https://miro.medium.com/max/592/1*RfeS9DYoGIMoj05cI-zkpA.png)

> 위 결과는 1,2,3 혹은 2,1,3 등 병렬적으로(비동기적으로) 처리되기 때문에 결과가 일관적이지 않다.

## 이후 내용은 코루틴을 공부하지 않았기 때문에 이후에 다시 작성하도록 함 (2020-01-28)
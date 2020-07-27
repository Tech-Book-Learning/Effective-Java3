### Item78 - 공유 중인 가변데이터는 동기화해 사용해라

한마디로, `volatile` 과 `synchronized` 키워드를 꼭 사용하라는 말이다!  

* 여러 쓰레드가 가변 데이터를 공유한다면, 해당하는 데이터를 읽고 쓰는 동작은 반드시 동기화하자. 

* 혹은 `atomic` 패키지를 활용해서 잘 사용하도록 하자. 

* 혹은 **가변 데이터**는 단일 쓰레드에서만 쓰자 꼭

### Item79 - 과도한 동기화는 피하라

동기화에 대한 영역은 최대한 작게 하자!  

### Item80 - 스레드보다는 실행자, 태스트, 스크림을 애용하라

* 실행자: `Excutors`에 있는 ThreadPool 을 애용하자
* 태스크: **Runnable** 과 **Callable** 을 적극적으로 활용하자!
* 스트림: 스트림에는 **parellalStream**이 존재하며, ForkJoinPool로 이루어져 있으니 잘 활용하자!

### Item81 - wait와 notify보다는 동시성 유틸리티를 애용하라

wait와 notify는 언제나 시스템이 하나의 상태를 가진다는 말과 동일하다. 

억지로 스레드에게 상태를 정의하지 말고, 최대한 사용하지 않는 방안을 고려해보자.!

가끔은 대기중인 스레드가 notify 없이도 깨어나버린다!!

시간 측정 관련해서!

> 시간 간격을 잴 때는 System.currentTimeMillis 보다는 System.nanoTime 을 사용하자!  

### Item82 - 스레드 안전성 수준을 문서화하라

제목이 곧 내용이다

### Item83 - 지연 초기화는 신중히 사용하라

대부분의 상황에서는 지연 초기화보다는 일반 초기화가 낫다. 

만약, 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면, **이중검사 관용구를 사용하라.**

```java
private violatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result != null) { // 첫번째 검사, 락 사용 X
    return result;
  }
  
  synchronized(this) {
    if (field == null) { // 두번째 검사, 락 사용
      field = computeFieldValue();
    }
    return field;
  }
}
```

### Item84 - 프로그램의 동작을 스레드 스케줄러에 기대지 말라

우선, 스레드는 **당장 처리해야 할 작업이 없다면 실행되서는 안된다.**  

스레드 스케줄러에게 의존하게 된다면, 원인을 찾기 힘든 `에러`들이 난무할 것이다.
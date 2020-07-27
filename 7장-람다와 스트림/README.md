### Item42 - 익명 클래스보다는 람다를 사용하라

`Java8`에서는 함수형 인터페이스라 부르는 인터페이스들의 인스턴스를 **람다식**을 사용해 만들 수 있다! <br>
이전에 예시로 보았던 계산기를 기억하는가? <br>
람다를 쓰게 되면 깔끔하게 가독성 있는 코드로 나타낼 수 있다! <br>
```java
public enum Operation {
	PLUS ("+", (x, y) -> x + y),
	MINUS ("-", (x, y) -> x - y),
	TIMES ("*", (x, y) -> x * y),
	DIVIDE ("/", (x, y) -> x / y);

	private final String symbol;
	private final DoubleBinaryOperator op;

	Operation(String symbol, DoubleBinaryOperator op) {
		this.symbol = symbol;
		this.op = op;
	}

	public double apply(double x, double y) {
		return op.applyAsDouble(x, y);
	}
}
```
이처럼 깔끔해진다! 다만...
```text
람다는 이름이 없고 문서화도 못한다. 따라서 코드 자체로 동작이 명확히 설명되지 않으면 쓰지 말아야 한다!
```

### Item43 - 람다보다는 메서드 참조를 사용하라

메서드 참조의 예
```java
// 정적
Integer::parseInt == str -> Integer.parseInt(str)
// 한정적(인스턴스)
Instant.now()::isAfter == Instant then = Instant.now(); t -> then.isAfter(t)
// 비한정적(인스턴스)
String::toLowerCase == str -> str.toLowerCase()
// 생성자
TreeMap<K,V>::new == () -> new TreeMap<K,V>()
```
메서드 참조는 람다식을 더 가독성이 있을 때가 있고, 아닐 때가 있다 <br>
**메서드 참조 쪽이 짧고 명확하면, 메서드 참조를 쓰고 그렇지 않으면 람다를 사용하라**

### Item44 - 표준 함수형 인터페이스를 사용하라

우리가 자주쓰는 람다식의 경우 대게 **표준 함수형 인터페이스화** 되어 있을 확률이 높다  

인터페이스 | 함수 시그니처 | 예
----- | ----- | -----
UnaryOperator<T> | T apply(T t) | String::toLowerCase
BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add
Predicate<T> | boolean test(T t) | Collection::isEmpty
Function<T,R> | R apply(T t) | Arrays::asList
Supplier<T> | T get() | Instant::now
Consumer<T> | void accept(T t) | System.out::println

### Item45 - 스트림은 주의해서 사용하라

스트림과 같은 **함수형 프로그래밍**의 장점은 바로 가독성이다 <br>
위의 장점을 무시하고 과하게 적용할려고 하면 바보다 <br>
**스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다** <br>
원래 기존코드보다 스트림을 이용해 리팩토링한 결과가 나아 보일때만 반영하자 <br>
다음은 스트림을 쓸때 안성맞춤인 조건들이다
1. 원소들의 시퀀스를 일관되게 변환한다.
2. 원소들의 시퀀스를 필터링한다.
3. 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기, 연결하기)
4. 원소들의 시퀀스를 컬렉션에 모은다
5. 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.


### Item46 - 스트림에서는 부작용 없는 함수를 사용하라

우리가 사용하는 `Stream`은 안전성이 보장된 API이다 <br>
하지만 스트림 함수를 커스터마이징 할 일이 생긴다면 반드시 `side effect`가 없다는 보장이 되어야 한다<br>
또한 **forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산할 때는 적합하지 않다** <br>
스트림을 올바로 사용하려면 `수집기(Collectors)`를 잘 알아둬야 한다! <br>
가장 중요한 수집기 팩터리는 `toList`, `toSet`, `toMap`, `groupingBy`, `joining`이다!

### Item47 - 반환 타입으로는 스트림보다 컬렉션이 낫다

왜 그럴가? <br>
기본적으로 `Collection` 인터페이스는 `stream`메서드와 `iterable`를 지원하기 때문이다 <br>
따라서 반환타입은 **Collection**이 올바르다! <br>
다만 **컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다** <br>

### Item48 - 스트림 병렬화는 주의해서 적용하라

대체로 데이터 소스가 `Stream.iterate`거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다 <br>
스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다 <br>
또한 스트림을 잘못 병렬화하면 **성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다** <br>

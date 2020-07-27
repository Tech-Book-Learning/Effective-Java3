### Item49 - 매개변수가 유효한지 검사하라

메서드 내부나 생성자에 잘못된 매개변수가 들어갈 수가 있다  
이러한 것을 방지하기 위하여 적극적으로 `Exception`을 활용하도록 하자  
또한 `@throws` 자바독 태그는 **센스**다  
1. IllegalArgumentException
2. IllegalStateException
3. NullPointerException
null에 관련한 검사는 `java7`에 추가된
```java
Obejcts.requireNonNull(strategy, "전략");
```
와 같은 메서드를 적극적으로 활용하도록 하자!


### Item50 - 적시에 방어적 복사복을 만들라

클라이언트가 여러분의 불변식을 깨드리려 혈안이 되어 있다고 가정하고 **방어적**으로 프로그래밍해야 한다  
다음 예시에 문제점을 찾아보자
```java
public final class Period {
	private final Date start;
	private final Date end;

	public Period(Date start, Date end) {
		if (start.compareTo(end) > 0) {
			throw new IllegalAccessException(
				start + "가 " + end + "보다 늦다.");
		}
		this.start = start;
		this.end = end;
	}

	public Date start() {
		return start;
	}

	public Date end() {
		return end;
	}
	...
}
```
`Period`클래스는 얼핏보면 불변처럼 보인다  
하지만 Date가 가변이기에 쉽게 깨뜨릴 수 있다
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 수정했다
```
다행히도 `java8`부터는 쓸모없는 Date클래스 대신 **LocalDateTime**클래스를 사용하면 된다  
이 Period의 경우 **생성자에서 받은 가변 매개변수 각각을 방어적으로 복사**해야 한다  
```java
public Period(Date start, Date end) {
	this.start = new Date(start.getTime());
	this.end = new Date(end.getTime());

	if (this.start.compareTo(this.end) > 0) {
		throw new IllegalAccessException(
				this.start + "가 " + this.end + "보다 늦다.");
	}
}
```
`가변인수`는 항상 `방어적 복사본`을 만드는 것이 중요하다!  
*하지만 이게 끝이 아니다*  
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // p의 내부를 변경했다
```
그래서 getter 또한 마찬가지로 방어적 복사본을 반환해야 한다!  
```java
public Date start() {
	return new Date(start.getTime());
}

public Date end() {
	return new Date(end.getTime());
}
```
**래퍼 클래스**의 특성상 클라이언트는 래퍼에 넘긴 객체를 직접 접근할 수 있다  
이를 못하게 꼭 `방어적 복사본`을 만들자!


### Item51 - 메서드 시그니처를 신중히 설계하라

메서드를 만드는 규칙에는 다음과 같은 것들이 있다
1. **메서드 이름**을 신중히 짓자
2. 편의 메서드를 너무 많이 만들지 말자
3. **매개변수 목록**은 4개 이하로 짧게 유지하자 = 같은 타입의 매개변수 여러개는 쫌 그렇다
또한 매개변수를 지정할 때에는  
* 매개변수의 타입으로는 클래스보다는 **인터페이스**가 더 낫다
* boolean보다는 원소 2개짜리 **열거 타입**이 낫다
왜냐하면 `확장`에 열려 있기 때문이다!  


### Item52 - 다중정의는 신중히 사용하라

다중정의 = `오버로딩`은 항상 신중하게 생각하자  
왜냐하면 **재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택되기 때문이다**  
한마디로 `컴파일 타임`에 모든 것이 정해진다  
그래서 **다중정의는 혼동을 일으키는 상황을 초래한다**  
예시를 보면서 주의하는 경각심을 가지자  
```java
public Class CollectionClassifier {
	public static String classify(Set<?> s) {
		return "집합";
	}

	public static String classify(List<?> lst) {
		return "리스트";
	}

	public static String classify(Collection<?> c) {
		return "그 외";
	}

	public static void main(String[] args) {
		Collection<?>[] collections = {
			new HashSet<String>(),
			new ArrayList<BigInteger>(),
			new HashMap<String, String>().values()
		};

		for (Collection<?> c : collections)
			System.out.println(classify(c));
	}
}
```
결과는?
```text
"그 외"
"그 외"
"그 외"
```


### Item53 - 가변인수는 신중히 사용하라

`...` 이라는 훌륭한 가변인수가 있다  
하지만 이것을 남발하는 건 바보 같은 짓이다  
예시를 볼까?  
```java
static int min(int... args) {
	if (args.length == 0)
		throw new IllegalArgumentException("인수가 1개이상 필요합니다.");
	int min = args[0];
	for (int i = 1; i < args.length; i++)
		if (args[i] < min)
			min = args[i];
	return min;
}
```
정말 지저분한 코드고, 인수가 0개 들어가면 런타임에 실패하는 코드다  
차라리 첫 번째 매개변수를 받고, 가변인수를 받으면 된다!
```java
static int min(int firstArg, int... remainingArgs) {
	int min = firstArg;
	for (int arg : remainingArgs)
		if (arg < min)
			min = arg;
	return min;
}
```


### Item54 - null이 아닌, 빈 컬렉션이나 배열을 반환하라

`null`을 반환하는 로직을 작성하면, 반드시 다른 곳에서 null check를 하는 번거로움이 발생한다  
```java
List<Chesse> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
	return "치즈치즈";
```
이런이런 벌써부터 너무나 귀찮아진다  
shop에 있는 getCheeses 메서드를 개선해보자!!
```java
public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? Collections.emptyList()
		: Collections.unmodifiableList(cheesesInStock);
}
```
반드시 **null이 아닌, 빈 배열이나 컬렉션을 반환하라**


### Item55 - 옵셔널 반환은 신중히 하라

값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야 하는 메서드라면 `Optional`을 반환해야 하는 상황이다  
하지만 너무나 맹신하지는 말자  
**컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다**  
차라리 빈 리스트나 빈 배열을 반환하자!  
또한, Optional을 반환하는 메서드에서는 **절대 null을 반환하지 말자**  


### Item56 - 공개된 API 요소에는 항상 문서화 주석을 작성하라

제목이 곧 내용이다
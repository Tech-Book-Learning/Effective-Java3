### Item1 - 생성자 대신 정적 팩터리 메서드를 고려하라

[정적 팩터리 메서드 패턴](https://github.com/huisam/JinLearnedList/blob/master/DesignPattern/abstractfactory)

*왜 기본적인 생성자를 쓰지 말고 정적 팩터리 메서드를 쓰라고 할까요?*

기본적으로 다음과 같은 **장점**이 있습니다
1. 생성자가 이름을 가질 수 있다 `ex) Shape.createRectangle()`
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다. `ex) Boolean.valueOf()`
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다 `ex) Enum 활용`
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다 `ex) JDBC Driver`

그렇다면 **단점**은 무엇일까요?
1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다
	>API가 아니기 때문이다

흔히 정적 팩터리 메서드의 **Naming Convention**은 다음과 같다
* from : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환 `Date d = Date.from(instant)`
* of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 메서드 `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`
* instance 혹은 getInstance : 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다
* create 혹은 newInstance : 위와 같지만, 매번 새로운 인스턴스를 생성함을 보장한다 `Object newArray = Array.newInstance(classobject, arrayLen);`
* getType : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다 `FileStore fs = Files.getFileStore(path)`
* newType : newInstance와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서들르 정의할 때 쓴다 `BufferedReader br = Files.newBufferedReader(path)`
* type : getType와 newType의 간결한 버전

### Item2 - 생성자에 매개변수가 많다면 빌더를 고려하라

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
이런 Code와
```java
NutritionFacts cocaCola = new NutritionFacts.Builder()
					.servingSize(240)
					.servings(8)
					.calories(100)
					.sodium(35)
					.carbohydrate(27)
					.build();
```
이러한 빌더 패턴의 코드차이는 벌써 **가독성**에서 부터 다르다<br>
하지만 매개변수가 3개 이하라면 일반적인 **생성자**를 써도 무관하다<br>

### Item3 - private 생성자나 열거 타입으로 싱글턴임을 보증하라

위 설명은 [블로그 링크](https://huisam.tistory.com/entry/Singleton)로 대체합니다!

### Item6 - 불필요한 객체 생성을 피하라

기본적으로 정규표현식을 사용해서 원하는 문자열을 얻고 싶을 때<br>
다음 처럼 작성할 수 있다
```java
static boolean isRomanNumeral(final String s) {
	return s.matches("\d\d\d");
}
```
이 방식의 문제점은 무엇일까요?<br>
함수가 호출될 때마다 **String 객체**에서 정규표현식용 `Pattern`인스턴스를 매번 만드는 성능상의 이슈가 있습니다.

*그럼 어떻게 바꿔야 할까요?*

재사용성을 높이는 방식입니다.
```java
public class RomanNumerals{
	private static final Pattern ROMAN = Pattern.compile("\d\d\d");

	static boolean isRomanNumeral(final String s) {
		return ROMAN.matcher(s).matches();
	}
}
```
이렇게 개선하면 `ROMAN`이라는 하나의 인스턴스로 성능을 개선하고,<br>
재사용성을 높일 수 있게 됩니다!<br>

**박싱된 기본 타입보다는 기본 타입을 사용하자**
```java
private static long sum() {
	Long sum = 0L;
	for (long i = 0; i <= Interger.MAX_VALUE; i++)
		sum += i;

	return sum;
}
```
이 코드의 문제점이 무엇일까요?<br>
**바로 매번 객체를 만드는 끔찍한 일을 하고 있습니다**<br>
절대로 `불필요한 객체 생성`은 최대한 하지 맙시다!<br>


### Item7 - 다 쓴 객체 참조를 해제하라

Java에서는 `가비지 콜렉터`라는 엄청난 친구가 있다<br>
이 `가비지 콜렉터`는 SW상에 더이상 참조되지 않는 메모리 영역이 있다면<br>
알아서 메모리를 해제 하는 역할을 담당하고 있다<br>

하지만 프로그래머가 쓸데없이 계속 참조하는 코드를 작성하면,<br>
**영영 메모리 해제**가 불가능하다<br>

잘못된 예부터 보자
```java
public Object pop() {
	if (isEmpty())
		throw new EmptyStackException();
	return elements[--size];
}
```
얼핏 보면 괜찮아 보이지만,<br>
계속해서 add되고 pop되는게 반복되면 이전 메모리가 절대 해제되지 않는다!<br>

```java
public Object pop() {
	if (isEmpty())
		throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null;
	return result;
}
```
이러한 메모리 누수에 대한 사례는 겉으로 잘 드러나지 않고, 수년간 시스템에 잠복되는 경우가 많으니 각별히 주의하도록 하자!<br>
> 리뷰를 요청하거나, 리뷰를 요청하거나, 리뷰를 요청하거나...

### Item8 - finallizer와 cleaner 사용을 피하라

**제곧내**(제목이 곧 내용)

### Item9 - try-finally 보다는 try-with-resources 사용하라

우선 try-finally는 정말 코드상으로 역겨워지는 방식중의 하나다.<br>
다음 같은 예시를 볼까?
```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(dst);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```
와후 정말 끔찍하다

`Java7`에서 이러한 문제들을 이미 해결했다!!<br>
`AutoCloseable` 인터페이스를 상속받은 객체는 모두 try-with-resource 문법을 이용할 수 있다!!!<br>
**한마디로 열어 놓으면 지가 알아서 닫는다는 말이다**<br>
```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
			out.write(buf, 0, n);
	}
}
```
정말 짧고 간결해지지 않았는가?

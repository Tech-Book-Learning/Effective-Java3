### Item34 - int 상수 대신 열거 타입을 사용하라

`C++`에서는 `enum`이 강제로 상수가 할당되지만 <br>
java에서는 **완전한 형태의 클래스**라서 정말 강력하다
```java
public static final int APPLE_FUJI = 1;
public static final int APPLE_PIPPIN = 2;
public static final int APPLE_GRANNY_SMITH = 3;
```
이렇게 사용하지는 말자ㅠㅠ <br>
개선하면~
```java
public enum Apple {
	FUJI, PIPPIN, GRANNY_SMITH
}
```
이러한 열거 타입은 단순 매칭되는 시스템에 강력하게 적용할 수 있다 <br>
**열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다** <br>
이런식으로 말이다~!
```java
public enum Operation {
	PLUS("+") {
		public double apply(double x, double y) { return x + y; }
	},
	MINUS("-") {
		public double apply(double x, double y) { return x - y; }
	},
	TIMES("*") {
		public double apply(double x, double y) { return x * y; }
	},
	DIVIDE("/") {
		public double apply(double x, double y) { return x / y; }
	};

	private String final symbol;

	Operation(String symbol) { this.symbol = symbol; }

	public abstract double apply(double x, double y);

}
```
그럼 이렇게 좋은 열거 타입을 언제 써야하나? <br>
필요한 원소를 **컴파일타임에 다 알 수 있는 상수 집합**이라면 열거 타입을 사용하자 <br>
열거 타입에 정의된 상수 개수가 영원히 `고정불변`일 필요는 없다! <br>

### Item35 - ordinal 메서드 대신 인스턴스 필드를 사용하라

`ordinal` 메서드 : 열거 타입에서 `몇 번째 위치`인지 반환하는 메서드 <br>
잘못된 예를 먼저 보여주자면
```java
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET;

	public int numberOfMussicians() { return ordinal() + 1; }

}
```
진짜 바보 같은 코드다 <br>
만약, *상수 선언 순서가 바뀐다면?*  <br>
바로 오동작하게 되는 코드다! <br>
따라서 **열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지말고, 인스턴스 필드에 저장하자**
```java
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4);

	private final int numberOfMussicians;

	Ensemble(int size) { this.numberOfMussicians = size; }

	public int numberOfMussicians() { return numberOfMussicians; }

}
```

### Item36 - 비트 필드 대신 EnumSet을 사용하라

비트 필드도 상수로 사용하는 것은 정말 바보같은 짓이다
```java
public class Text {
	public static final int STYLE_BOLD = 1 << 0;
	public static final int STYLE_ITALIC = 1 << 1;
	public static final int STYLE_UNDERLINE = 1 << 2;

	public void applyStyles(int styles) { ... }

}
```
점점 상수가 많아질 수록 우리가 표현할 수 있는 비트는 한정되어 있다 (long = 64bit) <br>
그러니 EnumSet을 사용해서 수정하자!
```java
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE; }

	public void applyStyles(Set<Style> styles) { ... }

}
```
요렇게 선언 하였으면
```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
요렇게 사용하면 된다!! <br>
열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다!

### Item37 - ordinal 인덱싱 대신 EnumMap을 사용하라

`EnumMap` 자료구조는 **Key**값을 해당 Enum 객체의 ordinal로 판별하기 때문에 <br>
**성능도 우수**하고, **불변성**을 보장한다! <br>
```java
public V put(K key, V value) {
    typeCheck(key);

    int index = key.ordinal();
    Object oldValue = vals[index];
    vals[index] = maskNull(value);
    if (oldValue == null)
        size++;
    return unmaskNull(oldValue);
}
```
실제로 EnumMap의 get메서드다!

```text
배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라
```


### Item38 - 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

위에 작성된 예시중에 Operation이라는 객체를 인터페이스화 시키라는 말이다!
```java
public interface Operation {
	double apply(double x, double y);
}
```
이 인터페이스를 상속 받아서 Enum타입 객체를 만든다!
```java
public enum BasicOperation implements Operation {
	PLUS("+") {
		public double apply(double x, double y) { return x + y; }
	},
	MINUS("-") {
		public double apply(double x, double y) { return x - y; }
	},
	TIMES("*") {
		public double apply(double x, double y) { return x * y; }
	},
	DIVIDE("/") {
		public double apply(double x, double y) { return x / y; }
	};

	private String final symbol;

	Operation(String symbol) { this.symbol = symbol; }

}
```
그러면 다른 연산자(Exp(^), Remainder(%))들이 추가되도 유연하게 확장이 가능해진다! <br>

```text
열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다!
```

### Item39 - 명명 패턴보다 애너테이션을 사용하라

우리가 흔히 아는 애너테이션이 있다 `@Override` <br>
이 애너테이션은 오버라이딩된 메서드인지 체크하고 아닐경우 컴파일 에러를 띄우는 역할이다
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
애너테이션에는 2가지 정책이 중요하다 <br>
1. 해당 애너테이션의 `Target`이 누구인지
2. 어떤 단계 `Retention` 까지 **보존정책**을 유지할 것인지

크게 `Target`은 여러 종류들이 있다
```text
TYPE : class, interface, enum 에 붙이는 애너테이션
FIELD : 인스턴스 필드에 붙이는 애너테이션
METHOD : 메소드에 붙이는 애너테이션
PARAMETER : 메소드 인자에 붙이는 애너테이션
CONSTRUCTOR : 생성자에 붙이는 애너테이션
LOCAL_VARIABLE : 지역변수에 붙이는 애너테이션
PACKAGE : 패키지에 붙이는 애너테이션
```
`Retention`은 딱 3가지다
1. SOURCE : compiler까지만 유지한다
2. CLASS : class file까지만 유지한다 = Not Runtime
3. RUNTIME : runtime까지 유지한다

그러니까 **애너테이션**으로 할 수 있는 일을 명명 패턴으로 처리할 이유가 없다! <br>
적극적으로 활용하자~!

### Item40 - @Override 애너테이션을 일관되게 사용하라

`@Override` 애너테이션을 사용하면 실수를 줄일 수 있다
```java
class Bigram {
	private final char first;
	private final char second;

	public Bigram(char first, char second) {
		this.first = first;
		this.second = second;
	}

	public boolean equals(Bigram b) {
		return b.first == first && b.second == second;
	}

	public int hashCode() {
		return 31 * first + second;
	}
}
```
얼핏 보면 잘 작성한 코드 같다 <br>
하지만 프로그래머가 잘못 작성한 코드다 <br>
머릿속에서는 `equals`메서드가 오버라이딩 된 것 같지만 <br>
아쉽게도 `equals`메서드의 인자는
```java
public boolean equals(Obejct b);
```
이렇게 된다 <br>
그러니까 `@Override` 애너테이션을 사용하는 것이다!! <br>
**우리의 실수를 막아줄 것이다**

### Item41 - 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

* 마커 인터페이스 : 빈 interface

마커 인터페이스가 마커 애너테이션보다 2가지면에서 낫다
1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 그렇지 않다
2. 적용 대상을 더 정밀하게 지정할 수 있다

그렇다면 어느때에 **마커 인터페이스** 써야 할까? <br>
```text
이 마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있을까?
```
라는 대답에 `그렇다`라면 사용하면 된다!<br>
**마커 애너테이션**의 적용대상이 `ElementType.TYPE`이면 잠시 여유를 갖고 생각하는게 좋다 <br>
**애너테이션**이 옳은지, **마커 인터페이스**가 낫진 않은지 말이다

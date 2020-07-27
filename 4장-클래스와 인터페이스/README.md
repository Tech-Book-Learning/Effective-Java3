### Item15 - 클래스와 멤버의 접근 권한을 최소화하라

한마디로 `정보 은닉` 혹은 `캡슐화(Encapsulation)`을 하라는 말과 일맥상통한다 <br>
책에서는 다음과 같은 장점이 있다고 한다
* 시스템 **개발 속도**를 높일 수 있다 
* 시스템 **관리 비용**을 낮출 수 있다
* 정보은닉 자체가 성능을 높여주지는 않지만, **성능 최적화**에 도움을 준다
	> 독립적인 컴포넌트 운용이 가능하기 때문이다
* 소프트웨어 **재사용성**을 높인다
* 거대한 시스템을 제작하는 난이도를 낮춰준다

정보 은닉을 위한 기본 원칙은 **모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다**

### Item16 - public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

한마디로 이렇게 쓰지 말자 제발
```java
class Point {
	public double x;
	public double y;
}
```
객체의 정보에 대한 접근이 필요하면 `getter`를 이용하자

### Item17 - 변경 가능성을 최소화하라

불변 클래스 : 인스턴스의 내부 값을 수정할 수 없는 클래스 <br>
정말 확정된 도메인 객체의 경우에는 `불변 클래스`로 만드는 것이 중요하다 <br>
불변 클래스의 규칙은 다음과 같다
1. 객체의 상태를 변경하는 메서드(setter)를 제공하지 않는다
2. class를 확장할 수 없도록 한다 ex) final class
3. 모든 필드를 final로 선언한다
4. 모든 필드를 private으로 선언한다
5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다

불변 객체를 쓰는 중요한 이유는 `함수형 프로그래밍`에 기초적인 형태이기 때문이다 <br>
함수형 프로그래밍의 대표적인 예로는 `java8`에서 나온 `stream API`이다 <br>
예시를 통해 이해하자
```java
public final class Complex {
	private final double re;
	private final double im;

	public Complex(double re, double im) {
		this.re = re;
		this.im = im;
	}

	public Complex plus(Complex c) {
		return new Complex(re + c.re, im + c.im);
	}

	public Complex minus(Complex c) {
		return new Complex(re - c.re, im - c.im);
	}
}
```

메서드를 통한 호출의 결과값이 `새로운 객체`를 만드는 것이다! <br>
이렇게 설계를 하게 되면 예상치 못한 `버그`를 방지할 수 있고 <br>
객체가 정말 **객체답게 행동**할 수 있는 기반을 만들어준다 <br>
그렇기 때문에 불변 객체는 `스레드에 안전`하다 <br>
<br>
*그렇다면 이렇게 좋은 불변 객체의 단점이 뭘까?* <br>
바로 `값이 다르면 반드시 독립된 객체로 만들어야 된다` = 메모리, 성능을 많이 잡는다 <br>

### Item18 - 상속보다는 컴포지션을 사용하라

상속은 객체지향에 있어서 정말 좋지만 **메서드 호출과 달리 상속은 캡슐화를 깨뜨린다** <br>
상속은 반드시 하위 클래스가 상위 클래스의 `진짜` 하위 타입인 상황에서만 써야 한다! <br>
class B가 class A와 `is-a` 관계일 때만 상속하자! <br>
다른말로 class A를 상속하는 class B를 작성하려고 한다면 `B가 정말 A인가?` 에 대한 확신이 있어야 한다 <br>


### Item19 - 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

상속용 클래스는 **재정의**할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다 <br>
대표적인 예로 `List 인터페이스`에는 다음과 같은 메서드들이 있다
```java
public boolean remove(Object o); // Object o 를 찾아서 삭제
public boolean remove(int index); // index번째의 Object를 삭제
```
메서드 오버로딩이 되어 있는 remove지만 둘의 역할은 완전히 다르다 <br>
따라서 우리는 **상속용으로 설계한 클래스**는 반드시 배포전에 **하위 클래스**를 만들어 검증해야 한다 <br>
또한 생성자에서 절대로 **상속 가능한 메서드**를 호출하지 않도록 하자 = `의존성 제거` <br>
```text
private, final, static 메서드는 재정의가 불가능하니 활용하자~!
```
상속을 하지 못하게 막을꺼면 `final`키워드 혹은 `private 생성자`를 적극적으로 활용하자!

### Item20 - 추상 클래스보다는 인터페이스를 우선하라

이유는 너무나 당연하다 <br>
추상 클래스(Abstract Class)는 단일상속이고, 인터페이스(interface)는 다중상속이다 <br>
객체를 좀 더 유연하게 적용할려면 단일보다는 다중이 매력적일 때가 훨신 많다 <br>
그래서 인터페이스는 **믹스인(mixin)** 정의에 안성 맞춤이다!

### Item21 - 인터페이스는 구현하는 쪽을 생각해 설계하라

*왜 그럴까?* <br>
생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법이다

### Item22 - 인터페이스는 타입을 정의하는 용도로만 사용하라

제발 **상수 인터페이스**를 사용하는 바보짓은 하지 말자ㅠ
```java
public interface PhysicalConstants {
	static final double AVOGADROS_NUMBER = 6.02;

	static final double BOLTZMANN_CONSTANT = 1.38;

	static final double ELECTRON_MASS;
}
```
차라리 이럴거면 `열거 타입(Enum)`을 쓰자 <br>
혹은 **생성자를 private으로 한 class**로 설계하는 것이 바람직하다

### Item23 - 태그 달린 클래스보다는 클래스 계층 구조를 활용하자

우선 코드를 작성해보겠다
```java
class Figure {
	enum Shape { RECTANGLE, CIRCLE, };

	// 태그 필드 - 현재 모양을 나타낸다.
	final Shape shape;

	// 다음 필드는 사각형(RECTANGLE)일 때만 쓰인다
	double length;
	double width;

	// 다음 필드는 원(CIRCLE)일 때만 쓰인다
	double radius;

	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.width = width;
		this.length = length;
	}

	double area() {
		switch(shape) {
			case RECTANGLE:
				return length * width;
			case CIRCLE:
				return Math.PI * (radius * radius);
			default:
				throw new AssertionError(shape);
		}
	}
}
```
*무엇이 문제인지 보이십니까?* <br>
Figure Class에서는 너무 많은 필드와 너무 많은 책임을 가지고 있다 <br>
도대체 왜 도형에 대한 판단을 Figure가 직접하는가? Rectangle, Circle이 직접하는게 올바르다 <br>
나같으면 공통부분을 추상화해서 상속받는 형식으로 **리팩토링**할 것이다 <br>
여러분들도 해보셨으면 좋겠다

### Item24 - 멤버 클래스는 되도록 static으로 만들라

멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 `static`을 붙여서 <br>
정적 멤버 클래스로 만들자! <br>
이는 가비지 컬렉션의 인스턴스를 수거하지 못하는 **메모리 누수**를 막을 수 있다

### Item25 - 톱레벨 클래스는 한 파일에 하나만 담으라

예를 들어 `Desert.java`라는 파일이 있다고 가정하자 
```java
class Desert {
	static final String NAME = "cake";
}

class Utensil {
	static final String NAME = "pen";
}
```
한 파일안에 2개 이상의 클래스를 정의하는 것은 올바르지 않다 <br>
혹시나 `Utensil.java`라는 파일을 만들어서 Utensil class를 정의할 수 있는 상황이 올 수 있다 <br>
이는 컴파일 단계에서 중복 정의 오류를 내뱉을 것이다 <br>

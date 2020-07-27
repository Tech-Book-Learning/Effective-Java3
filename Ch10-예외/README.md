### Item69 - 예외는 진짜 예외 상황에만 사용해라

제목 그대로 예외상황에만 쓰고 **절대로 일상적인 제어 흐름용**으로 쓰지 마라  


### Item70 - 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

호출하는 쪽에서 복구하리라 여겨지는 상황이면 `검사 예외`를 사용하라  
반면에 프로그래밍 오류일 때는 `런타임 예외`를 사용하자  
다시 말해서 **비검사 throwable**은 모두 `RuntimeException`의 하위 클래스 여야 한다!  


### Item71 - 필요 없는 검사 예외 사용은 피하라

### Item72 - 표준 예외를 사용하라

자주 쓰는 예외클래스를 적극적으로 활용하자  
다만 **Exception, RuntimeException, Throwable, Error**는 직접 재사용하지 말자  

|예외|주요 쓰임|
|---|---|
|IllegalArgumentException|**허용하지 않는 값**이 인수로 건네졌을 때|
|IllegalStateException|객체가 메서드를 **수행하기에 적절하지 않은 상태**일 때|
|NullPointerException|null을 허용하지 않는 메서드에 **null**을 건냈을 떄|
|IndexOutOfBoundsException|**인덱스**가 범위를 넘어섰을 떄|
|ConcurrentModificationException|허용하지 않는 **동시 수정**이 발견됬을 떄|
|UnsupportedOperationException|호출한 **메서드를 지원**하지 않을 때|

간단히 설명하면, 인수 값이 무엇이었든 어차피 실패하면 `IllegalStateException`  
그렇지 않으면 `IllegalArgumentException`을 던지자  

### Item73 - 추상화 수준에 맞는 예외를 던지라

상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다  
또한 이것을 무턱대고 `남용`해서는 곤란하다!  
```java
try {
	...
} catch (LowerLevelException cause) {
	throw new HigherLevelException(cause);
}
```

### Item74 - 메서드가 던지는 모든 예외를 문서화하라

한마디로 `@throws`태그로 문서화하라는 말이다  
단 **비검사 예외**는 제외하자  

### Item75 - 예외의 상세 메세지에 실패 관련 정보를 담으라

실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드 값을 실패 메세지에 담아야 한다 
단, `암호 키` 혹은 `비밀번호`는 절대로 담으면 안된다   
```java
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
	super(String.format(
		"최솟값: %d, 최댓값 : %d, 인덱스: %d",
		lowerBound, upperBound, index));
	this.lowerBound = lowerBound;
	this.upperBound = upperBound;
	this.index = index;
}
```


### Item76 - 가능한 한 실패 원자적으로 만들라

호출된 메서드가 실패하더라도 해당 객체는 메서드 호출 전 상태를 유지해야 한다  


### ITem77 - 예외를 무시하지 말라

`catch` 블록을 비워두는 바보 같은 짓을 하지는 말자  
어쨋든 예외를 무시하기로 했다면 **이유를 주석으로 남기고 예외 변수도 ignored**로 바꾸자!!  
```java
try {
	...
} catch (Exception ignored) {
	// 여기서는 처리하지 않는다
}
```

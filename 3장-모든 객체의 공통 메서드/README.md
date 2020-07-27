### Item10 - equals는 일반 규약을 지켜 재정의하라

### Item11 - equals를 재정의하려거든 hashCode도 재정의하라

사실 이 두 아이템의 경우에는 `IDE`가 직접 해결해주는 경우가 많다 <br>
우리가 알아야 할 것은 **직접 만든 비즈니스 로직의 객체**는 반드시 equals와 hashCode를 재정의해야 합니다 <br>
그래야만 `Collection Framework`로 응용할 수 있는 **확장성** 이 생기기 때문이죠~! <br>

### Item12 - toString을 항상 재정의하라

실전에서 `toString`메서드는 **객체가 가진 주요 정보를 모두 반환하는게 좋다** <br>
*왜 그럴까?* <br>
주로 Log 수집에 대한 정보를 담당하기 때문이다 <br>
필자는 예전에 다음과 같은 Code를 작성한 적이 있었다 <br>
```java
@Override
public String toString() {
    StringBuilder history = new StringBuilder();
    for (String carName : trialHistory.keySet()) {
        history.append(carName).append(" : ");
        for (int i = 0; i < trialHistory.get(carName); i++) {
            history.append('-');
        }
        history.append('\n');
    }
    return history.toString();
}
```
이러한 View를 나타내기 위한 로직은 절대로 `toString` 메서드에 나타내는 것이 아니다 <br>
정말 쉽게 `toString`을 사용하려면 `IDE`를 잘 응용해서 쓰자~!

### Item13 - clone 재정의는 주의해서 진행하라

사실 `clone`메서드를 재정의하는 것보다는 **복사 생성자**와 **복사 팩터리** 방식을 구현하는 것이 좋다
```java
public Yum(Yum yum) { ... }

public static Yum newInstance(Yum yum) { ... } 
```
이 방식이 훨씬 더 가독성이 좋고, 원본의 구현 타입에 얽매이지 않고 객체 생성이 가능하다!

### Item14 - Comparable을 구현할지 고려하라

우리는 아주 범용성이 높은 인터페이스를 알고 있다 <br>
```java
class T implements Comparable<T>{

}
```
우선순위를 고려할 필요성이 있는 `TreeMap`, `PriorityQueue`, `TreeSet` 와 같은 컬렉션을 구현할 때 <br>
`compareTo` 메서드를 재정의하여 사용하면 된다! <br>
그렇게 되면 우리는 자신만의 **일급 컬렉션**을 쓸 수 있게 되는 것이다!! <br>


# final keyword 
지금까지 final 키워드에 대하여 '안다'고만 생각했었지 제대로 어디에 사용이 가능하고 어디에 불가능한지 확인을 못해본 것 같다. 그리하야 오늘은 `final` 키워드의 사용범위를 확인해보려 한다.
### 1. class 에서의 final
보통의 class 는 상속이 가능하다.
일반적인 public class 를 extents 하면 문제없이 상속관계가 이루어진다.
하지만 final 로 정의된 class 를 상속하려 하면 상속할 수 없다고 나온다.
```java
public final class FinalParentsClass {
    ...
}
```
```java
public class ChildrenClass extends FinalParentsClass { 
    // 에러 발생. Cannot inherit from final 'FinalParentsClass'
    ...
}
```
가끔 자바 라이브러리 등을 사용할 때 라이브러리에서 원하는만큼의 기능을 제공하지 않거나 custom 기능을 제공하는 라이브러리라면 특정 class 를 상속받아 확장시키고는 한다. 하지만 해당 클래스가 final 로 정의되어 있다면 우리는 이러한 확장을 할 수 없게 될 것이다. 대표적으로는 java.lang 패키지의 기본자료형들을 감싸고 있는 wrapper 클래스와 String 클래스가 final 로 선언되어 있다. (`Integer`, `Short`, `String` 등)
### 2. Method 에서의 final
먼저 클래스가 final 이라면 내부 메소드 들을 굳이 final 로 선언하지 않아도 상속 자체가 불가능하기 때문에 final 클래스의 멤버변수나 멤버클래스는 final 로 선언하지 않아도 된다. 그렇다면 보통의 class의 내부메소드에 final 키워드를 사용할 수 있다는 말이 된다.
```java
public class ParentsClass {
    public final void finalParentsMethod() {}
}
```
```java
public class ChildClass extends ParentsClass {}
```
상속은 문제없이 된다. 그도 그럴것이 `ParentsClass`는 현재 final 클래스가 아닌 일반 클래스이다.
하지만 메소드 오버라이드를 하려고 한다면 어떨까?
```java
public class ChildClass extends ParentsClass {
    @Override
    // 에러 발생. Method does not override method from its superclass
    public void finalParnetsMethod() {}
}
```
final 키워드가 붙은 메소드는 오버라이드를 할 수 없다. final 키워드의 목적이 '수정불가능' 인 부분을 고려해보면 당연한 동작이다.
그렇다면 사용할 때는 어떤 제한이 있을까?
```java
public class ChildClass extends ParentsClass {
    public ChildClass() {
        finalParentsMethod();
    }
}
```
```java
public static void main(String[] args) {
    new ParentsClass().finalParentsMethod();
}
```
자식클래스의 생성자에서 호출하거나 새로 생성한 인스턴스에서 해당 메소드를 호출하는 것에는 전혀 제한사항이 없다.
그렇다, `final` 키워드는 사용항의 제한을 두기 위한 키워드가 아닌 것이다. 단지 '수정불가', '작성자가 선언한 형태로만 사용가능'의 기능을 가진다.

마지막으로 final 이 붙은 변수에는 어떠한 제한이 있는지 확인해보자.

###3. final 변수
먼저 일반 클래스 내부에 final 변수를 선언해보자.
```java
public class ParentsClass {
    final String mother; // lint time 에서 바로 에러가 발생한다.
}
```
일단 변수만 선언하면 바로 lint time 에서 에러가 발생한다. 그도 그럴것이 '변경할 수 없다' 는 건 변수가 생성되기 전에 값이 들어가 있어야 한다는 의미이기도 하다. 그렇다면 2가지 방법으로 초기화가 가능할 것이다.

첫번째로, 변수 선언시 바로 초기화하는 방법이다.
```java
public class ParentsClass {
    final String mother = "Mrs.Lee";
}
```
두번째로, 생성자메서드에서 초기화하는 방법이다.
```java
public class ParentsClass {
    final String mother;
    
    public ParentsClass() {
        this.mother = "Mrs.Lee";
    }
}
```
에러는 사라졌다. 클래스가 '인스턴스화 될 때' 값을 초기화 해준다면 문제없이 final 변수를 사용할 수 있다는 것을 알았다. 그렇다면 final 변수의 getter, setter 를 만들 수 있을까? (다른 메서드에서 값을 변경한다거나 뭐 그런)
```java
public class ParentsClass {
    private final String mother = "Mrs.Lee";
    
    public String getMother() {
        return mother;
    }
    
    public void setMother(String mother) {
        this.mother = mother;
        // lint time 에 이 부분에서 에러가 발생한다.
        // final 변수는 초기값이 정해지면 이 값을 변경할 수 없다.
    }
}
```
(예상했던 대로) final 변수는 값을 재설정할 수 없다. 이렇게 setter 메소드를 만든다거나 다른 메소드에서 값을 변경한다거나 상속받은 곳에서 값을 변경한다거나 하는 등등의 모든 '변경작업' 이 불가능하다.

그래서 **클래스에서 공동으로 사용하는(static) 변경할 수 없는(final)** 요소를 선언할 때에는 `static final` 예약어들을 앞에 붙여 선언한다. final 의 결정적인 기능인 '런타임 내에 변수가 변경될 가능성이 없는' 점이 상수임을 가능케 하는 것이다.

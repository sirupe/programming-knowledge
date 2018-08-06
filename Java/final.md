# final keyword 
지금까지 final 키워드에 대하여 '안다'고만 생각했었지 제대로 어디에 사용이 가능하고 어디에 불가능한지 확인을 못해본 것 같다. 

그리하야 오늘은 `final` 키워드의 사용범위를 확인해보려 한다.
### 1. class 에서의 final
보통의 class 는 상속이 가능하다. 일반적인 public class 를 extents 하면 문제없이 상속관계가 이루어진다.

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
가끔 자바 라이브러리 등을 사용할 때 라이브러리에서 원하는만큼의 기능을 제공하지 않거나 custom 기능을 제공하는 라이브러리라면 특정 class 를 상속받아 확장시키고는 한다. 하지만 해당 클래스가 final 로 정의되어 있다면 우리는 이러한 확장을 할 수 없게 될 것이다. 

대표적으로는 java.lang 패키지의 기본자료형들을 감싸고 있는 wrapper 클래스와 String 클래스가 final 로 선언되어 있다. (`Integer`, `Short`, `String` 등)
### 2. Method 에서의 final
먼저 클래스가 final 이라면 내부 메소드 들을 굳이 final 로 선언하지 않아도 상속 자체가 불가능하기 때문에 final 클래스의 멤버변수나 멤버클래스는 final 로 선언하지 않아도 된다. 

그렇다면 보통의 class의 내부메소드에 final 키워드를 사용할 수 있다는 말이 된다.
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

final 키워드가 붙은 메소드는 오버라이드를 할 수 없다. final 키워드의 목적이 '수정불가능' 인 부분을 고려해보면 당연한 동작이다. 그렇다면 자식 클래스에서 부모 클래스에 final 로 선언된 메소드를 사용할 수는 있을까?
```java
public class ChildClass extends ParentsClass {
    public ChildClass() {
        // 별다른 제약 없이 사용이 가능하다
        finalParentsMethod();
    }
}
```
final 키워드는 '변경불가' 의 기능을 하므로 당연히 '접근제한'과는 관련이 없다. '접근제한'을 시키려면 엄한 final 을 붙이지 말고 '접근제어자' 키워드를 사용하도록 하자. 

더불어 final 키워드를 붙임으로써 오버라이드가 안되는 것은  접근제어자를 사용해서 오버라이드를 할 수 없는 것과는 엄연히 의미가 다른 것이다. final 키워드를 이용한 것은 '어디서든 변경을 할 수 없게 하겠다'는 취지인 것이고 '접근제어자' 를 이용한 것은 '특정 밤위 밖에서는 사용할 수 없게 하겠다'는 취지인 것이다. 

일례로 protected 나 default 메소드인 경우 범위만 가능하다면 메소드를 오버라이드 하여 사용할 수 있는 것이다.
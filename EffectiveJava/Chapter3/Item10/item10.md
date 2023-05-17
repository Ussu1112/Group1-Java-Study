## item10. equals는 일반 규약을 지켜 재정의하라

equals를 재정의 하지 않는 경우 ? -> 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다.

### 재정의 할 필요가 없는 경우

- 각 인스턴스가 본질적으로 고유하다
- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다
- 상위 클래스에서 재정의한 eqauls가 하위 클래스에도 딱 들어 맞는다.
- 클래스가 private 이거나 package-private이고 equals 메소드를 호출할 일이 없다.

```java
// 실수로라도 호출되는 것을 막고싶다면..
@Override public boolean eqauls(Object o){
    throw new AssertionError();
}
```

### 재정의를 해야하는 경우
객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인 해야할 때

Integer, String 등 값을 표현해주는 값 클래스인 경우 
(Enum의 경우에도 값 클래스이지만 논리적으로 같은 객체가 2가지 이상 만들어지지 않으니 재정의 하지 않아도 된다.)

### equals 메소드 재정의 시 지켜야 할 일반 규약
[수학에서 동치관계](https://ko.wikipedia.org/wiki/%EB%8F%99%EC%B9%98%EA%B4%80%EA%B3%84)

equals 메서드는 동치관계(equivalence relation)를 구현하며, 아래 다섯 가지를 만족한다.
- 반사성(reflexivity) : null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true다.
- 대칭성(symmetry) : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true면 y.eqauls(x)도 true다.
- 추이성(transitivity) : null이 아닌 모든 참조 값 x,y,z 에 대해, x.equals(y)가 true이고, y.equals(z)도 true면 x.equals(z)도 true다.
- 일관성(consistency): null이 아닌 모든 참조 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.


### equals 구현 방법
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

##### 구현 시 주의사항
- float와 double을 제외한 기본 타입 필드는 == 연산자로 비교하고, 참조 타입 필드는 각각의 equals 메서드로,
float와 double 필드는 각각 정적 메서드인 Float.compare(float, float)와 Double.compare(double, double)로 비교한다.
-> 부동소수 값은 다루기 때문
- 어떤 필드를 먼저 비교하느냐에 따라 equals의 성능을 좌우한다.
최상의 성능을 바란다면 다를 가능성이 크거나 비교하는 비용이 싼(혹은 둘 다 해당해는) 필드를 먼저 비교한다.

> equals를 다 구현 하였다면 세가지를 자문해본다. 대칭적인가? 추이성이 있는가? 일관적인가?


```java
// 전형적인 eqauls 메서드
@Override
public boolean equals(Object o){    
    if(o == this)
        return true;
    if(!(o instanceof INSTANCE))
        return false;
    INSTANCE is = (INSTANCE)o;
    return is.intA = intA && is.intB == intB && is.intC == intC; 
}
``` 

- equals를 재정의할 땐 hashCode도 반드시 재정의하자.
- 너무 복잡하게 해결하려 들지말자. -> 필드들의 동치성만 검사해도 충분하다.
- Object 외의 타입을 매개변수로 받는 equals 메소드는 선언하지 말자.


```java
// 잘못된 예 
public boolean equals(MyClass o){
    // 입력 타입은 반드시 Object여야 한다.
    }
    
@Override
public public boolean equals(MyClass o){
    // 일관되게 @Override를 사용하면 컴파일할 때 오류를 뱉어 실수를 알려준다!
    } 
```


> (책에서는 구글의 AutoValue 프레임워크가 작성을 돕는다고 하지만 엄청 오래 된듯 하다..?)
> IDE에서 제공하는 자동생성을 사용하여 작성 시 IDE 는 나중에 클래스가 수정된 걸 자동으로 알아채지 못하니 테스트코드를 작성해야한다.
> 사람이 직접 작성 시 오류가 발생할 수 있으니 그래도 자동생성을 사용하는게 낫다.


## 정리
꼭 필요한 경우가 아니면 eqauls를 재정의하지 말자. 많은 경우에 Object의 equals가 원하는 비교를 정확히 해준다.
재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.
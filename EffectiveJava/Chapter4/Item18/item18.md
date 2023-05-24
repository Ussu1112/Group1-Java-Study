## Item18. 상속보다는 컴포지션을 사용하라.

**상속은 코드를 재사용하는 아주 강력한 수단이지만, 항상 최선은 아니다.**


<br>

### 상속을 왜 쓰지말라고 하는 것일까?

프로그램 설계 시 상위 클래스와 하위 클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서
**확장할 목적으로 설계**되었고 **문서화도 잘 된** 클래스라면 얼마든지 안전하게 사용할 수 있지만
대부분 그렇지 않다.

코드를 수정하는 누군가에 의해 상위 클래스가 수정될 때 하위 클래스는 그 여파로 코드 한 줄 건드리지
않았지만 오작동 할 수가 있다.

> 그렇기 때문에 메소드 호출과 달리 상속은 캡슐화를 깨뜨릴 수 있다.

<br>

### 상속의 잘못된 예1

###### HashSet을 사용하여 Set에 몇개가 들어갔는지 세는 프로그램
```java
public class InstrumentedHashSet<E> extends HashSet<E> {

    private int addCount = 0;

    public InstrumentedHashSet() {
        super();
    }

    public InstrumentedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
public class Main {
    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));

        System.out.println(s.getAddCount()); // 6        
    }
}
```


재정의한 addAll을 호출하여 인스턴스 s에 값을 3개 넣었지만 카운트는 6개가 되었다.
이는 addAll함수의 내부 구현 방식 중 각 원소에 대해 add 메소드를 사용하여 넣게 되어있었다.

이처럼 자신의 다른 부분을 사용하는 **자가사용(self-use)** 여부는 내부 구현 방식에 해당되며,
사용하는 개발자 입장에서는 이러한 점을 모두 알 수 없다.

<br>

### 상속의 잘못된 예2

상위 클래스에 새로운 메소드를 추가하는 경우?

보안 때문에 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야만 하는 프로그램인 경우

> 컬렉션 프레임워크 이전부터 존재했던 Hashtable과 Vector를 컬렉션 프레임워크에 포함시킬 때에도
> 보안상 구멍이 존재하여 수정이 필요했다.



<br>


### 그래서 해결방법은?

**상속 대신! 컴포지션을 사용하자**

> 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션(Composition; 구성)이라 한다.

컴포지션은 기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.

새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환해준다.
여기서 반환하는 방식은 전달(forwarding)이라 하며, 새 클래스의 메서드들은 전달 메서드(forwarding method)라 부른다.


> 컴포지션 사용 결과 : 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며,
> 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.

<br>

###### 재사용 가능한 전달클래스
```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    @Override
    public boolean add(E e) {
        return s.add(e);
    }
    @Override
    public int size() {
        return s.size();
    }
    @Override
    public boolean isEmpty() {
        return s.isEmpty();
    }
    @Override
    public boolean contains(Object o) {
        return s.contains(o);
    }
    @Override
    public Iterator<E> iterator() {
        return s.iterator();
    }
    @Override
    public Object[] toArray() {
        return s.toArray();
    }
    @Override
    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }
    @Override
    public boolean remove(Object o) {
        return s.remove(o);
    }
    @Override
    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }
    @Override
    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }
    @Override
    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }
    @Override
    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }
    @Override
    public void clear() {
        s.clear();
    }
}
```

###### 전달클래스를 컴포지션하여 생성한 집합 클래스(래퍼 클래스)
```java
public class InstrumentedSet<E> extends ForwardingSet<E>{
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

**InstrumentedSet**클래스는 HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 아주 유연하다.
컴포지션 방식은 한 번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과 함께 사용할 수 있다.

<br>

### 래퍼클래스 (Wrapper Class)

다른 Set 인스턴스틑 감싸고 있다는 뜻에서**InstrumentedSet**같은 클래스를 래퍼 클래스라고 한다.
래퍼 클래스와 비슷한 개념의 패턴으로 **데코레이터 패턴**이 있다.

[//]: # (래퍼 클래스는 단점이 거의 없으며, 콜백함수와 어울리지 않는다. 콜백함수 실행 시 SELF문제를 발생한다.)


<br> 

> 스택은 벡터가 아니고, 속성 목록(Properties)도 해시테이블이 아니므로 때문에 컴포지션을 사용했다면 더 좋았을 것이다.   
> _컴포지션을 써야 할 상황에 상속을 사용했을 경우에 대한 예는 책속에!_

<br>

### 결론

상속은 반드시 하위 클래스가 상위 클래스의 '진짜'하위 타입인 상황에서만 쓰여야 한다.

이 장에서 대표적으로 설명하는 상속의 단점은 바로 SOLID원칙의 리스코프 치환 원칙(LSP)이다.
하위 클래스의 인스턴스는 상위형 객체 참조 변수에 대입해 상위 클래스의 역할을 하는 데 문제가 없어야 한다.

상속은 강력하지만 캡슐화를 해치기 때문에 순수한 is-a 관계에서 사용해야한다.
is-a 관계에서도 상위 클래스가 확장에 대해 고려되지 않았다면 여전히 문제가 된다.
상속의 취약점을 피하기 위해 래퍼 클래스를 구현할 인터페이스가 있다면 컴포지션을 사용하자.
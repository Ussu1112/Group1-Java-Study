# Item20 - 추상 클래스보다는 인터페이스를 우선하라

인터페이스와 추상클래스는 자바가 지원하는 다중 구현 메커니즘이다.

인터페이스와 추상클래스의 가장 큰 차이는 구현(implements)와 확장(extends)이다.

### 왜 인터페이스를 사용해야 하는가

추상 클래스를 구현하는 클래스는 반드시 **추상 클래스의 하위 클래스**가 되어야 한다.

자바는 단일 상속만 지원하기 때문에, 추상 클래스 방식은 **새로운 타입을 정의하는 데 제약**을 안게 된다.

반면 인터페이스는 선언한 메서드를 구현하는 클래스라면 어떤 클래스를 상속했든 같은 타입으로 취급할 수 있다. 이는 프로그래밍 시 유연성을 제공한다.

예를 들어 가수와 작곡가 인터페이스가 있다고 가정해보자.

```java
public interface Singer{
		AudioClip sing(Song s);
}

public interface Songwriter{
		Song compose(int chartPosition);
}
```

작곡을 하는 가수는 어떻게 표현할 수 있을까?

```java
public interface SingerSongwriter extends Singer, Songwriter{
		AudioClip sing(Song s);
		Song compose(int chartPosition);
}
```

너무 간단하다.

하지만 가수와 작곡가가 추상클래스로 선언되어 있었다면??? 가수가 먼저인가 작곡가가 먼저인가?

음,, 간단한 작업이 되진 않을 것 같다. 계층구조에 대해 심히 고민하다가 인터페이스로 구현할 듯

위의 예제는 계층구조를 만든 인터페이스의 예시이다.

인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

```java
public interface Animal {
		public String getName();
}

public class Dog {
		private String name;
		
		@Override
		public String getName(){
				return name;
		}
}

public class Cat {
		private String name;

		@Override String getName(){
				return name;
		}
}
```

`Dog`과 `Cat`은 계층구조를 가지지 않고 `Animal`이라는 타입으로 선언될 수 있다.

인터페이스는 추상클래스와 다르게 다중상속이 가능하고, 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.

새로운 자바 버전에 `Comparable`, `Iterable` 인터페이스가 추가되었을 때, 기존의 수많은 클래스가 이를 구현한 채 릴리즈될 수 있었다.

하지만 만약 새로운 인터페이스가 아닌 추상클래스였다면? 아주 복잡하고 어려운 작업을 수행해야 했을 것이고, 계층구조에 혼란을 일으킬 수 있을 것이다.

### 디폴트 메서드

자바8 이후 인터페이스에 디폴트 메서드를 선언할 수 있게 되었다.

디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 `@implSpec` 자바독 태크를 붙여 문서화 해야한다.

> equals 나 hashCode 같은 메서드는 디폴트로 제공해서는 안된다.
>

왜지 구체 클래스마다 동등성 검사 기준이 다를 수 있고, hash코드 생성 방식이 다를 수 있어서 인가

### 템플릿 메서드 패턴

아직은 디자인 패턴을 공부할 단계는 아닌 것 같다. 읽어보고 패스하자

[[Design Pattern] 템플릿 메소드 패턴(Template Method Pattern)에 대하여](https://coding-factory.tistory.com/712)

### 그래서 왜 추상클래스보다 인터페이스가 좋은데?

- 상속에 있어 추상클래스보다 인터페이스가 더 자유롭다.
- 계층구조 없이 타입 프레임워크를 만들 수 있음. (한 타입으로 선언 가능한 여러 구체를 생성할 수 있음)
- 자바8 부터 디폴트 메서드를 지원하면서 추상클래스 상위 호환이 되었음.
- 인터페이스는 추상클래스보다 더 추상적인 개념을 표현할 수 있다.
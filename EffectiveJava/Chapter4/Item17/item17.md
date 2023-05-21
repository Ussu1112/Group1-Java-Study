# 아이템 17 : 변경 가능성을 최소화하라
이번 아이템의 핵심 내용은 제목 그대로 클래스의 변경 가능성을 최소화 하는 것이 좋다는 원칙에 대한 것이다. 클래스를 불변으로 만드는 것이 해당 클래스의 디자인과 구현을 더 쉽게 하고, 사용도 더 편리하게 할 수 있다는 것이다. 이번 아이템을 읽어보며 몇 가지 의문점들이 생겼는데, 차례대로 탐구해 보겠다.

먼저, 불변 객체의 예시 코드는 다음과 같다.
```java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                           re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                           (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        return Double.compare(c.re, re) == 0
               && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }
    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
위의 예시코드는 이펙티브 자바에서 나오는 , 복소수를 나타내는 Complex 클래스이다. 이 클래스는 객체의 불변성 특징을 보여준다.
- 모든 필드는 private final로 선언돼서 하위 클래스 생성이 불가능하다.
- plus(), minus(), times() 등 메서드들을 보면, return 타입이 Complex로서 새로운 객체를 반환한다. 이를 통해 객체의 불변성이 보장된다.

## 왜 클래스를 불변으로 만드는 게 유리한가?
책에 따르면, 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉽고, 오류가 생길 여지도 적으며 훨씬 안전하다고 한다. 클래스를 이렇게 불변으로 만드는 규칙은 다섯 가지가 있다.

### 불변 클래스 설계 규칙
- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않음
- 클래스를 확장할 수 없도록 함
- 모든 필드를 final로 선언
- 모든 필드를 private으로 선언
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 함

### 1.2. 불변 클래스가 유리한 이유
- 쓰레드 세이프 : 여러 쓰레드에서 동시에 사용하더라도 동기화를 고려할 필요가 없음. 멀티스레드 환경에서의 동시성 문제를 제거할 수 있음.
- 객체의 자유로운 공유성 : 불변 객체, 즉 static 객체는 불변성 덕분에 안전하게 공유 가능하다. 이를 통해 불필요한 메모리 사용을 줄일 수 있다.
- 높은 신뢰성 : 불변 객체는 변하지 않는 상태가 보장되기 때문에 높은 신뢰성을 제공한다. 이를 통해 보다 안정적인 시스템 설계 및 구축이 가능하다.
- 컬렉션에서의 범용성 : 불변 클래스의 인스턴스는 컬렉션에서 안전하게 사용 가능하다. Map의 Key를 예로 들면, 불변 객체는 변경되지 않기 때문에 자료 구조 변경 도중 상태가 변경돼서 예상치 못한 결과를 초래하는 것을 막는다.

이렇게 얘기하면 프로그래밍 과정 중 모든 클래스를 불변으로 만들어야 할 것 같은 느낌이 든다. 하지만 당연히 불변 클래스의 단점도 존재한다.

### 1.3. 불변 클래스의 단점
- 성능 최적화 : final 필드 사용이 성능에 부정적인 영향을 줄 수 있다. 컬렉션을 불변으로 활용할 경우 복사할 때 많은 리소스를 잡아먹고, 이는 비효율적이다.
- 하위 클래스에서의 재사용성 : 상위 클래스의 필드를 final로 선언하면 하위 클래스에서 그 필드를 오버라이딩 하거나 변경할 수 없다. 이는 자바의 지향성인 상속과 다형성을 위배한다.
- 데이터 변경 제한 : 객체의 필드값이 변경돼야 하는 DB 같은 동적인 시스템에서 불변 객체는 제한적이다.
- 프레임워크 & 라이브러리와의 호환성 : 자바 스프링 프레임워크에서 빈을 초기화 하는 경우 불변 객체로서 선언할 수 없기 때문에 사용이 제한적이다. 예를 들어, DI에서 final 필드에 대한 의존성 주입은 생성자를 통해서만 가능하다. 하지만 DI는 보통 setter를 통해서도 많이 이루어지기 때문에 사용이 제한적이라고 하는 것이다.

## 2. 불변 클래스에서 생성자와 정적 팩토리를 비교하면 어떨까?
위에서 불변 클래스의 장단점 및 기본적인 내용을 언급했다. 이때, 불변 객체를 만드는 방법 중 생성자를 통한 방법이 일반적이지만, 더 유연한 방법으로 정적 팩토리 방식이 있다고 한다.

### 2.1. 생성자를 통한 초기화
- 직관적이고 명확함
- 파라미터 숫자가 많아질수록 코드 가독성이 떨어짐
- 항상 새로운 객체를 반환 -> 불필요한 객체 생성이 불가피함

### 2.2. 정적 팩토리 메서드를 통한 초기화
- 메서드 네이밍을 통해 생성 객체의 의미를 명확하게 표현할 수 있음
- 기존 객체 재활용 또는 캐싱이 가능함
- 정적 팩토리 메서드만 제공하면 하위 클래스 생성이 불가능함

### 2.3. 예시 코드로 비교
```java
public final class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

}
```
위 코드는 생성자로만 불변 클래스를 초기화하는 코드이다.

```java
public final class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

}
```
위 코드는 정적 팩토리 메서드를 통해 불변 클래스를 초기화하는 코드이다. 차이점은 valueOf() 메서드가 추가되었는데, new Complex 객체를 return한다.

이때 메서드 네이밍을 "valueOf"라고 한 이유는 메서드의 파라미터의 값과 같은 값을 가진 객체를 return한다는 의미를 가지고 있기 때문이다. 다만, 이건 절대적인 건 아니고 결국 설계자 마음이라고 한다.

## 3. 이런 방식을 실무적인 관점에서 보면 어떠한가?
결론적으로 불변 클래스를 활용하면 복잡성과 버그 가능성을 소거하고, 안정성을 높일 수 있는 장점이 있지만 가변 클래스가 더 유리한 경우도 존재한다. 예를 들면, 필드가 자주 변경되는 클래스, 시간에 따라 필드가 변하는 클래스 등에서는 가변 클래스가 더 유리하다. 따라서 개발자는 상황에 따라 적절한 선택을 해야 한다.
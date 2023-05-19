이번 Item의 핵심 내용은 제목 그대로 "equals()와 hashCode()를 동시에 오버라이딩 해야 한다"는 것이다. 예시에 나온 HashMap()을 직접 구현해 봄으로써 확인해 보았다.

# 왜 equals()와 hashCode()를 동시에 오버라이딩 해줘야 하는걸까?
예시에 나온 HashMap()을 직접 구현해 봄으로써 왜 그런지 확인해 보자.

```java
class PhoneNumber {

    int number;

    public PhoneNumber(int number) {
        this.number = number;
    }
}
public class Main03 {

    public static void main(String[] args) {
        Map<PhoneNumber, String> map = new HashMap<>();

        map.put(new PhoneNumber(001), "Song");
        System.out.println(map.get(new PhoneNumber(001)).hashCode());

    }
}
```
PhoneNumber 클래스를 다음과 같이 정의하고 메인함수에서 map.get()을 통해 PhoneNumber 객체의 hashCode를 출력하려 했다.

```java
Exception in thread "main" java.lang.NullPointerException
	at test.Main03.main(Main03.java:13)
```
결과는 위와 같이 NullPointerException이 발생했다. 왜 이 예외가 터진걸까? 다음 HashMap의 Key 검색 과정을 통해 예외가 터진 이유를 알아보자.

## HashCode를 계산하는 단계
- HashMap은 먼저 Key의 hashCode() 메서드를 호출해 해시코드를 얻는다.
- 이 해시코드는 Map의 내부 버킷 배열에서 특정 버킷을 결정하는 데 사용되기 때문에 만약 두 Key가 같다면 둘의 해시코드도 반드시 같아야 한다.
- PhoneNumber 클래스에서는 hashCode()를 오버라이딩 하지 않았기 때문에 두 PhoneNumber 객체는 서로 다른 hashCode를 가진다.

## Bucket 내에서 동일한 키를 찾는 단계
- 해시코드를 통해 올바른 버킷을 결정하면, HashMap은 그 버킷 내에서 equals() 메서드를 통해 동일한 키를 찾는다.
- 만약 equals() 메서드가 오버라이딩 되지 않았다면, 동일한 키를 찾지 못한다.

따라서 PhoneNumber 클래스에서 equals()와 hashCode() 메서드를 오버라이딩 하지 않았기 때문에 HashMap에서는 두 키를 서로 다른 키로 인식하게 된다. 그래서 HashMap은 Key를 찾기 못했기 때문에 null을 반환하게 되고, 이것이 NullPointerException 예외가 발생한 이유다.



이런 이유 때문에 hashCode()와 equals()를 적절히 오버라이딩 하지 않으면 컬렉션 사용 시 문제가 될 수 있다.

# 해시코드를 캐싱하는 경우
클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기 보다는 캐싱하는 방식을 고려할 수 있다고 한다. 이는 해시 기반 컬렉션에서 객체를 검색하거나 삽입할 때 비용의 문제가 될 수 있다.



계산 비용이 크다는 것은 어떤 객체의 hashCode() 메서드를 통해 해시코드를 계산할 때, 이 메서드가 복잡하거나 시간이 오래 걸릴 경우 그 객체를 해시 기반 컬렉션에 삽입 또는 검색 시 큰 비용이 들 수 있다는 것이다.



이로 인해, 해시코드 캐싱을 고려할 수 있다. 다음은 불변 객체에 대하여 해시코드를 캐싱하는 예시 코드다.
```java
public final class ImmutableClass {
    private final int value;
    private int hashCode; // 캐싱된 해시코드

    public ImmutableClass(int value) {
        this.value = value;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        ImmutableClass that = (ImmutableClass) o;
        return value == that.value;
    }

    @Override
    public int hashCode() {
        // 처음 호출될 때 해시코드 계산
        if (hashCode == 0) {
            hashCode = Objects.hash(value);
        }
        // 계산된 해시코드 반환
        return hashCode;
    }
}
```

# 해시코드 초기화 시 지연 초기화 전략을 사용하는 경우
지연 초기화(Lazy Initialization)란 객체의 초기화를 그것이 실제로 필요한 시점까지 미루는 방법이다. 이를 통해 불필요한 초기화 비용을 줄이고, 리소스 사용을 최적화 할 수 있다. 다음은 이에 대한 예시 코드다.
```java
public final class ImmutableClass {
    private final int value;
    private Integer hashCode; // Integer로 선언하여 null 가능

    public ImmutableClass(int value) {
        this.value = value;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        ImmutableClass that = (ImmutableClass) o;
        return value == that.value;
    }

    @Override
    public int hashCode() {
        // 처음 호출될 때 해시코드 계산(지연 초기화)
        if (hashCode == null) {
            hashCode = Objects.hash(value);
        }
        // 계산된 해시코드 반환
        return hashCode;
    }
}
```
코드를 보면 해시코드를 캐싱하는 코드와 거의 비슷해 보인다. 차이점은 int 아니면 Integer 라는 것이다. 지연 초기화 전략을 사용할 경우 hashCode 변수가 Integer로 선언되는데, Integer는 null값을 가지는 것이 가능하므로 호출 전까지 null 상태를 유지할 수 있다.
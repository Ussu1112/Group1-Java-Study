# Item4 - 인스턴스화를 막으려거든 private 생성자를 사용하라

단순히 정적 필드와 정적 메서드만 가지는 클래스를 만들고 싶을 수 있을 것이다.
객체지향과 어울리지 않는 클래스이긴 하나 쓰임새가 있다.

아래는 그 예이다.


- `Math`, `Arrays`: 기본 타입 값이나 배열 관련 정적 메서드를 모아 놓음
    
    ```java
    public class Arrays {
        private static final int MIN_ARRAY_SORT_GRAN = 1 << 13;
    
        // Suppresses default constructor, ensuring non-instantiability.
        private Arrays() {}
    
        static final class NaturalOrder implements Comparator<Object> {
            @SuppressWarnings("unchecked")
            public int compare(Object first, Object second) {
                return ((Comparable<Object>)first).compareTo(second);
            }
            static final NaturalOrder INSTANCE = new NaturalOrder();
        }
    		...
    }
    ```
    
   
- `Collections`: 특정 인터페이스(`Collection`)를 구현하는 객체를 생성해주는 메서드를 모아놓음
    
    ```java
    public class Collections {
        // Suppresses default constructor, ensuring non-instantiability.
        private Collections() {}
    	
    		public static final Set EMPTY_SET = new EmptySet<>();
    		// 빈 set 인스턴스를 반환
    		public static final <T> Set<T> emptySet() {
            return (Set<T>) EMPTY_SET;
        }
    
    		...
    }
    ```
    
- `final` 클래스
    
    ```java
    public final class FinalClass {
    		int a = 10; // 자동으로 상수
    		
    		void test() { // 자동으로 final
    		}
    }
    ```
    


이러한 유틸성 클래스는 인스턴스로 만들어 사용하려고 설계하지 않았다.

따라서 기본 생성자를 private로 선언하여 인스턴스화를 막는다.
생성자를 선언하지 않으면 컴파일러에 의해 public 한 기본 생성자가 선언되기 때문이다.

해당 방식은 상속을 불가능하게 하는 효과도 있다. 자식 클래스에서 부모 클래스의 생성자를 호출할 수 없기 때문이다.

생성자가 존재하는데 호출할 수는 없다.

이는 그다지 직관적이지 않기 때문에 적절한 주석을 달아두는 게 좋다.

`Collections`, `Arrays` 클래스에는 아래와 같은 주석이 달려있다.

```java
// Suppresses default constructor, ensuring non-instantiability.
private Collections() {}
```

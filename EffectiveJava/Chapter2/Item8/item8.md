# finalizer와 cleaner란 무엇인가
finalizer와 cleaner는자원 정리를 위한 방법을 제공한다. 이들은 다루기 어렵고, 알맞게 사용하지 않으면 예상치 못한 결과를 초래할 수 있으므로 주의해서 사용해야 한다.

## 1.1. finalizer
객체가 가비지 컬렉션에 의해 제거될 때 JVM에 의해 호출된다. 이 메서드를 오버라이드하여 객체의 정리 로직을 구현할 수 있습니다. 그러나 이 방법은 몇 가지 문제가 있다.

finalize() 메서드의 호출 시점은 JVM에 따라 다르므로, 객체의 메모리 해제 시점을 예측하기 어렵다.

finalize() 메서드가 실패하거나 중단되면, JVM은 그것을 무시하고 계속 진행한다.

finalize() 메서드가 오래 실행되면, 가비지 컬렉션의 효율성이 떨어질 수 있다.

다음은 finalizer를 활용한 코드의 예시이다.
```java
public class FinalizerExample {

    @Override
    protected void finalize() throws Throwable {
        try {
            System.out.println("Finalize of Sub Class");
        } catch (Throwable throwable) {
            throw throwable;
        } finally {
            System.out.println("Calling finalize of Super Class");
            super.finalize();
        }
    }
}
```
이 코드는 finalizer 메서드를 오버라이딩 했다. 먼저 "Finalize of Sub Class"를 출력하고 그 아래에 자원 해제나 클린업 작업 코드를 넣는다.

마지막으로 finally에서 super.finalize()를 통해 상위 클래스의 finalize 메서드를 실행한다.

## 1.2. cleaner
Cleaner는 finalize()와 유사하게 동작하지만, 이를 사용하면 메모리 누수와 같은 문제를 피할 수 있다. Cleaner는 등록된 객체가 가비지 컬렉션에 의해 수집될 때 특정 작업을 수행하도록 등록할 수 있다.

다음은 clear를 활용한 코드의 예시이다.
```java
import java.lang.ref.Cleaner;

public class CleanerExample {

    private static final Cleaner cleaner = Cleaner.create();

    static class CleanableResource implements Runnable {

        String name;

        CleanableResource(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println("Cleaning " + name);
        }
    }

    public static void main(String[] args) {
        CleanableResource resource = new CleanableResource("Resource 1");
        Cleaner.Cleanable cleanable = cleaner.register(resource, resource);
        resource = null;
        System.gc();
    }
}
```
leanerExample 클래스에서 먼저 Cleaner 객체를 생성한다. cleaner Cleaner.create()를 통해 생성된다.

main 메서드에서 인스턴스를 생성하고 register 메서드를 통해 클린업 작업을 등록한다. 이때 register의 파라미터는 클린업 대상 객체, 클린업 작업을 수행할 Runnable 객체다.

resource = null을 통해 참조를 제거한다. 이를 통해 가비지 컬렉터에 의해 메모리 수거가 가능한 상태가 만들어진다.

System.gc()를 호출해서 가비지 컬렉션을 요청한다. 이를 통해 JVM에 의해 실제로 가비지 컬렉션이 수행된다.

# 2. finalizer와 cleaner의 문제점
## 2.1. 즉시 수행된다는 보장이 없다.
finalizer와 cleaner는 객체가 가비지 컬렉션 되었을 때 실행된다. 가비지 컬렉터는 더 이상 참조되지 않는 객체를 메모리에서 제거하는 역할을 하는데, 가비지 컬렉터가 언제 실행될지는 JVM에 따라 다르다. 따라서 그 시점을 정확히 예측할 수 없다. 이로 인해 finalizer와 cleaner로는 제때 실행되어야 하는 작업을 할 수 없다. 왜냐하면 메모리 사용을 정확히 예측할 수 없기 때문이다.
## 2.2. 수행 시점 뿐만 아니라 수행 여부조차 보장하지 않는다.
개발자는 가비지 컬렉터가 언제 실행될지, 어떤 객체가 가비지 컬렉션의 대상이 될지 정확히 예측하거나 제어할 수 없다. 따라서 finalizer나 cleaner가 호출되는 시점을 보장할 수도 없는 것이다. 그래서 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다.
## 2.3. 성능 문제가 존재한다.
finalizer를 사용하면 가비지 컬렉션 과정이 복잡해진다. 일반적으로 객체는 가비지 컬렉터에 의해 한 번의 패스로 메모리에서 제거된다. 그러나 finalizer가 있는 객체는 두 번의 패스를 필요로 한다. 첫 번째 패스에서는 finalizer가 호출되고, 두 번째 패스에서는 실제로 객체가 회수된다. 이 두 단계의 절차는 가비지 컬렉션의 성능을 저하시키며, 메모리 회수가 지연되는 결과를 초래한다.
## 2.4. 보안 문제를 일으킬 수 있다.
생성자에서 예외가 발생하면 객체의 생성이 완료되지 않는다. 하지만, 이 객체에는 finalizer가 있을 수 있으며, 이 finalizer는 예외 발생에도 불구하고 여전히 호출될 수 있다. 이로 인해, finalizer는 완전히 초기화되지 않은 객체의 상태에 접근할 수 있게 된다. 이는 보안 문제를 일으킬 수 있으며, 특히 민감한 정보가 포함된 객체의 경우 정보 유출의 위험을 초래할 수 있다.
직렬화는 객체의 상태를 바이트 스트림으로 변환하는 과정이다. 이 과정에서 예외가 발생하면, 변환되는 중인 객체의 일부만 직렬화되게 된다. 이렇게 되면, 완전히 직렬화되지 않은 객체의 상태를 다른 부분에서 읽을 수 있게 되며, 이는 보안 문제를 일으킬 수 있다. 특히, 민감한 정보가 포함된 객체의 경우 정보 유출의 위험이 있다.
# 3. 결론
finalizer 또는 cleaner는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하는 게 좋다. 이때도 불확실성과 성능 저하에 주의해야 한다.
즉, 자원의 소유자가 closer 메서드를 호출하지 않는 것에 대비한 안전망 역할로서 활용할 수 있다는 것이다.
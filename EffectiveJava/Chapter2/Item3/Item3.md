## Item3. private 생성자나 열거 타입으로 싱글턴임을 보증하라.

싱글턴(Singleton)이란?
인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
함수와 같은 무상태(stateless) 객체나 설계상 유일해야하는 컴포넌트에 사용한다.

> 클래스를 싱글턴으로 생성시 이를 사용하는 클라이언트를 테스트하기가 어려워
> 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면
> 가짜(mock) 구현으로 대체할 수 없기 때문  
> Customer -> Customers(Singleton)

### 싱글턴 생성방식1 - public static final 필드 방식
```java
public class staticFactorySingleton {
    public static final publicSingleton INSTANCE = new publicSingleton();
    
    private publicSingleton() {}
}
```
해당 방식은 생성자의 접근제어자가 private으로 클라이언트가 생성할 방법이 없다.

public static final 필드 방식의 장점
- 해당 API가 싱글턴임이 확실히 드러나고, public static 필드가 final이기 때문에 절대로 다른 객체를 참조 불가능하다.
- 간결함

### 싱글턴 생성방식2 - 정적 팩터리 방식
```java
public class staticFactorySingleton {
    private static final staticFactorySingleton INSTANCE = new staticFactorySingleton();
    
    private staticFactorySingleton() {}
    
    public static staticFactorySingleton getInstance() {
        return INSTANCE;
    }
    
    public void doSomething() { }
}
```

정적 팩터리 방식의 장점
- API를 변경하지 않고 싱글턴이 아니게 바꿀 수 있다. 인스턴스를 반환하던 팩터리 메서드에서
스레드 별로 다른 인스턴스를 반환하게끔 변경한다.
- 원한다면 제네릭 싱글턴로 변경가능하다.


```java 
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("Unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
```


- 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다

### 두 생성방식의 예외 - 리플렉션 API

리플렉션API를 통해서라면 클래스의 생성자가 호출이 가능하여, 오직 하나만 생성가능하다라는 법칙을 깰 수 있다.

```java
public class Main {
    public static void main(String[] args) throws Exception {
        staticFactorySingleton singleton1 = staticFactorySingleton.getInstance();
        staticFactorySingleton singleton2 = staticFactorySingleton.getInstance();

        System.out.println(singleton1); //item3.staticFactorySingleton@7c30a502
        System.out.println(singleton2); //item3.staticFactorySingleton@7c30a502

        Class<?> clazz = Class.forName("item3.staticFactorySingleton");
        Constructor<?> constructor = clazz.getDeclaredConstructor();

        constructor.setAccessible(true);
        staticFactorySingleton singleton3 = (staticFactorySingleton) constructor.newInstance();
        System.out.println(singleton3); //item3.staticFactorySingleton@49e4cb85

        publicSingleton singleton4 = publicSingleton.INSTANCE;
        publicSingleton singleton5 = publicSingleton.INSTANCE;

        System.out.println(singleton4); //item3.publicSingleton@43a25848
        System.out.println(singleton5); //item3.publicSingleton@43a25848

        Class<?> clazz2 = Class.forName("item3.publicSingleton");
        Constructor<?> constructor2 = clazz2.getDeclaredConstructor();

        constructor2.setAccessible(true);
        publicSingleton singleton6 = (publicSingleton) constructor2.newInstance();
        System.out.println(singleton6); //item3.publicSingleton@3ac3fd8b
    }
}
```
리플렉션 API를 통해 생성하는 것을 방어하기 위해 두 번째 객체 생성시에 예외를 던진다.

### 두 생성방식의 예외 - 직렬화, 역직렬화

```java
public class Main {
    public static void main(String[] args) throws Exception {
        staticFactorySingleton singleton1 = staticFactorySingleton.getInstance();
        System.out.println(singleton1); // item3.staticFactorySingleton@7c30a502

        // serialization
        File file = new File("./Singleton.file");
        try (FileOutputStream fos = new FileOutputStream(file);
             ObjectOutputStream oos = new ObjectOutputStream(fos)) {
            oos.writeObject(singleton1);
            oos.flush();
        }

        // deserialization
        staticFactorySingleton result = null;
        try (FileInputStream fis = new FileInputStream(file);
             ObjectInputStream ois = new ObjectInputStream(fis)) {
            result = (staticFactorySingleton) ois.readObject();
        }

        System.out.println(result); // item3.staticFactorySingleton@5f375618
    }
}
```

싱글턴 클래스를 직렬화 후 역직렬화를 진행하면 새로운 인스턴스를 생성하게 된다.
INSTANCE의 진위여부를 판단 하기 위해서 클래스에 readResolve() 을 선언하여 싱글턴임을 보장한다.

```java
public class staticFactorySingleton implements Serializable {
    private static final staticFactorySingleton INSTANCE = new staticFactorySingleton();
    private staticFactorySingleton() {    }
    public static staticFactorySingleton getInstance() {
        return INSTANCE;
    }

    private Object readResolve() {
        // 가짜 객체는 가비지컬렉터로 보내고 진짜 객체를 반환
        return INSTANCE;
    }
}
```



### 원소가 하나인 열거타입 방식을 활용하자.
```java
public enum Singleton {
    INSTANCE;
    public void leaveTheBuilding() {}
}
```

복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 객체를 생성하지 못하게 막아준다.
**대부분 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.**

만들려는 싱글턴이 Enum외의 클래스를 상속해야한다면 Enum 클래스는 사용할 수 없다.









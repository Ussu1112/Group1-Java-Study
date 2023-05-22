# 아이템 2 : 생성자에 매개변수가 많다면 빌더를 고려하라
이 아이템의 핵심 내용은 "생성자나 정적 팩토리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다"는 것이다. 인스턴스 하나를 초기화 하는 데 무슨 방법이 이렇게 많은걸까? 이 의문에서 출발하여 내용을 전개해 보겠다.

# 1. 인스턴스를 초기화하는 방법이 여러 가지로 분기하게 된 원인은 무엇인가?
각각의 상황과 필요성에 따라 더 효율적인 초기화 방법이 다르기 때문이다. 간단한 객체 생성에는 생성자 만으로도 충분하지만, 생성자를 이해하기 쉬워야 하는 상황에서는 정적 팩토리 메서드를 쓰고, 필드가 많아짐에 따라 많은 파라미터가 필요한 상황에서는 빌더 패턴이 유리하다.

# 2. 각 방법은 어떤 특징을 가지고, 어떤 상황에서 유리한가?

## 2.1. 생성자
너무 당연한 개념이기 때문에 내용은 생략하고 예시 코드만 첨부한다.

```java
public class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

### 2.1.1. JavaBeans 패턴
파라미터가 많아질 경우 생성자를 다루기에 비효율적인 상태가 된다. 이때 이것을 해결하기 위해 등장한 방법이 JavaBeans 패턴이다. 이 방법은 파라미터가 없는 생성자로 객체를 만들고, setter()를 호출해서 원하는 파라미터를 초기화한다.

```java
public class NutritionFacts {
    private int servingSize = -1;     
    private int servings = -1;        
    private int calories = 0;         
    private int fat = 0;              
    private int sodium = 0;           
    private int carbohydrate = 0;     

    public NutritionFacts() { }

    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calories = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}

NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

## 2.2. 정적 팩토리 메서드
인스턴스를 반환하는 정적 메서드이다. 이 메서드는 생성자를 통해 인스턴스를 직접 생성하는 대신 호출하고, 메서드 네이밍을 통해 객체 생성의 의도를 더 명확하게 나타낼 수 있다.

```java
public class Person {
    private String name;
    private int age;
    
    private Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public static Person createAdult(String name) {
        return new Person(name, 20);
    }
    
    public static Person createChild(String name) {
        return new Person(name, 10);
    }
}
```
위 코드에서 createAdult()와 createChild()를 보면 메서드 네이밍을 통해 인스턴스 생성의 목적을 분명히 알 수 있다.

## 2.3. 빌더 패턴
객체를 생성하는 복잡한 과정을 추상화하고, 객체 생성을 단계별로 수행하도록 하는 패턴이다. 이 방법은 생성자의 파라미터가 많을 경우 유리하다. 생성자 패턴과 JavaBeans 패턴의 한계를 극복하기 위해 등장했다.

```java
public class Person {
    private String name;
    private int age;

    private Person() {}

    public static class Builder {
        private Person person;

        public Builder() {
            person = new Person();
        }

        public Builder withName(String name) {
            person.name = name;
            return this;
        }

        public Builder withAge(int age) {
            person.age = age;
            return this;
        }

        public Person build() {
            return person;
        }
    }
}

Person person = new Person.Builder()
                    .withName("John")
                    .withAge(25)
                    .build();
```
위 코드는 빌더 패턴을 이용하 예시 코드이다. 인스턴스 생성을 단계별로 구성해서 파라미터 처리에 있어 유연성이 높다. 또한 직관적으로 이해하기 쉬워 가독성도 높아진다.

### 2.3.1. 빌더 패턴과 getter / setter는 근본적으로 무슨 차이점이 있는가?
두 방법은 메서드 측면에서 비슷해 보일 수 있지만, 근본적인 차이점이 있다.
- 불변성 보장의 측면 : 빌더를 사용해서 인스턴스를 만들면 그 인스턴스는 불변 객체가 된다. 하지만, getter & setter를 사용하면 객체의 필드가 언제든지 변경될 수 있다.
- 추상화 및 생성주기 관리의 측면 : 빌더를 통해 객체 생성 과정을 추상화 하며, 그 과정이 단계별로 관리될 수 있도록 한다. 여기서 추상화라고 하는 것의 의미에 대해서 설명하자면, 객체 생성을 담당하는 코드는 객체가 어떻게 구성되는지에  대한 세부사항을 알 필요가 없다는 것이다. 단지 빌더가 주는 메서드만 호출하면 된다.

```java
Person person = new Person.Builder()
                .withName("Song")
                .withAge(20)
                .withAddress("Address")
                .withPhoneNumber("010-1234-5678")
                .build();
```
위 코드를 보면 객체 생성 과정이 단계별로 나눠져 있고, 객체 생성 주기가 추상화되어 메서드만 호출되고 있음을 알 수 있다.
# Item16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.

> 아이템 16은 정보 은닉을 강조하고, 클래스의 내부 상태를 캡슐화하여 외부에서 직접적인 접근을 제한하는 것이 좋다는 원칙을 제시합니다. 

## 인스턴스 필드만을 모아놓은 퇴보한 클래스

```
Class User {
    public String id;
    public String password;
}
```
위에 코드와 같이 퇴보한 클래스란 인스턴스 필드만을 모아놓은 클래스를 말하며,
`캡슐화의 이점`을 제공하지 못하는 클래스들입니다.

왜 인스턴스 필드만 모아놓은 코드가 퇴보한 클래스인지 한번 살펴보도록 하겠습니다.

쇼핑몰에 가입을 한다고 가정해봅시다. <br>
위에 코드처럼 User 클래스가 필드를 public으로 선언하여 사용자에게 공개하면 어떻게 될까요? <br>
```
    public static void main(String[] args) {
        User user = new User("java", "1234");
        System.out.println(user.id); // java
        System.out.println(user.password); // 1234
        
        user.id = "C";
        System.out.println(user.id); // C
    }
}
```
위에 코드처럼 객체의 필드에 직접 접근할 수 있어 사용자가 값을 변경할 수 있게 됩니다. <br>
이처럼 필드에 대한 접근이 제한되지 않고, public으로 공개되어 있으면 외부에서는 필드에 어떠한 값이든 할당 할 수 있으며 읽을 수 있게 되는 것입니다.<br>
즉, 이러한 접근 제한 없이 필드를 변경하게 되면, 객체의 내부 상태를 원치 않는 방향으로 임의로 수정할 수 있기 때문에 <br>
API를 수정하지 않고는 `내부 표현`을 바꿀 수 없고, 외부에서 필드에 접근할 때 `부수작업`을 수행할 수도 없으며 `객체의 일관성`을 훼손하거나 `불변 객체의 불변성`을 해치는 등의 문제를 일으킬 수 있습니다. <br>

[참고로 java.awt.package 패키지에 Point 클래스의 필드가 직접 노출되어있으나 접근은 불가하다.]

## 해결방법 1. 캡슐화한 클래스
- 필드를 모두 `private`로 바꾼다.
- public 접근자(`getter`), 변경자(`setter`) 메서드를 추가한다.
```
class User {

    private String id;
    private String password;
    
    User(String id, String password) {
        this.id = id;
        this.password = password;
    }

    public String getId() { return id; }
    public String getPassword() { return password;}
    
    public void setId(String id) { this.id = id; }
    public void setPassword(String password) { this.password = password; }
}
```

## 해결방법 2. private 중첩클래스
```
public class TopPoint {
	private static class Point {
		public double x;
		public double y;
	}

	public Point getPoint() {
		Point point = new Point();
		point.x = 3.5;
		point.y = 4.5;
		return point;
	}
}
```


## 캡슐화의 이점
그렇다면 캡슐화의 이점인 내부표현, 외부에서 필드에 접근할 때 부수작업, 객체의 일관성, 불변 객체의 불변성에 대해서도 알아봅시다.

### 내부 표현
내부 표현(Internal representation)이란 객체가 자신의 상태나 데이터를 어떻게 구성하고 저장하는지를 나타내는 것을 말합니다. <br>
즉, 객체의 내부에서 데이터를 표현하는 방식이나 구조를 의미합니다. <br>

### 부수작업
필드에 직접 접근하면 해당 필드에 값을 할당하거나 값을 읽을 수 있으며, <br>
필드에 접근하는 코드 자체에 추가적인 로직을 직접 삽입하기 어렵기 때문에 부수 작업을 수행하기 어렵습니다.
그러나 필드에 접근할 때 부수 작업을 수행하고 싶은 경우에는 일반적으로 접근자 메서드(getter와 setter)를 사용합니다. <br>
접근자 메서드를 통해 필드에 접근하면 필드의 가시성을 조절하고, 필드에 대한 유효성 검사, 부작용 처리 등의 로직을 추가할 수 있습니다. <br>
이를 통해 필드에 접근할 때 부수 작업을 수행할 수 있습니다.
```
class User {

    private String id;
    private String password;
    
    User(String id, String password) {
        this.id = id;
        this.password = password;
    }

    public String getId() { return id; }
    public String getPassword() { return password;}
    
    public void setId (String id) {
        if (id != null) {
            this.id = id;
        } else {
            throw new IllegalArgumentException("id cannot be null");
        }
    public void setPassword(String password) { this.password = password; }
}
```

### 객체의 일관성
객체의 일관성(consistency)은 객체의 상태와 데이터가 어떤 규칙 또는 제약 조건을 따라 변화해야 함을 의미합니다. <br> 
객체의 일관성은 객체의 내부 데이터가 항상 유효하고 정확한 상태를 유지하도록 보장하는 것을 의미합니다.

### 불변 객체의 불변성
객체의 불변성(Immutability)은 객체가 생성된 후에 그 상태를 변경할 수 없는 성질을 가지는 것을 의미합니다.


## 결론
public 클래스의 경우 가변 필드를 직접 노출해서는 안 되며, 불변 필드라고 해도 덜 위험하지만 안심할 수 없다. <br>
하지만 package-private 클래스나 private 중첩 클래스에서는 필드를 노출하는 편이 나을 때도 있어 상황에 맞게 사용하자
## Item12. toString을 항상 재정의하라.

> toString은 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.

toString을 잘 구현한 클래스는 다른 개발자가 사용하기에 훨씬 편하고,
클래스를 사용한 시스템은 디버깅하기 쉽다.

실전에서 toString은 그 **객체가 가진 주요 정보** 모두를 반환하는 게 좋다.

### toString 작성 시 해야할 주석방법

toString 입력 시에는 반환 값에 대한 포맷을 명시하거나 의도를 밝혀야 한다.

```java
public class PhoneNumber {
    int areaCode;
    int prefix;
    int lineNum;
    
    /*
     * ex) 포맷에 들어가는 내용과 반환 값에 대한 설명
     * 예상 입력 값과 출력 값            
     */
    @Override
    public String toString() {
        return String.format("%03d-%03d-%03d", areaCode, prefix, lineNum);
    }
    /* OR
     * 포맷이 정해져있지 않은 일반적인 toString이라면
     * 클래스에 관한 대략적인 설명 일반적인 출력 내용
     * 변경 가능성 등을 주석에 입력
     * */
}
```

### toString을 제대로 사용하지 않는다면 ?

toStiring을 그대로 파싱하여 사용하는 경우에 포맷이 바뀌거나 하는 경우에 유연하게 대처할 수 없다.

toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하여 값을 확인하고 활용한다.

### 정리

IDE에서 자동 생성해주는 toString() 함수의 경우 필드의 내용을 모두 나타내 주긴 하지만
클래스의 **의미**까지 파악할 수 없기 때문에 주석을 활용하여 객체의 값에 대한 설명을 덧붙히는 편이 좋다.

이번 아이템의 주제에 맞게 "toString을 항상 재정의하라" 라는 뜻은 결국에 프로젝트 
혹은 그보다 더 작은 기능 안에 클래스.. 객체.. 의 설명을 내포하고 있다고 생각한다.

다른 개발자의 이해를 돕기위해 기능이 어떻게 돌아가는지만 주석처리하면 되지 않나? 라는 생각만 있었는데,
이번 아이템을 읽으면서 디버깅을 쉽게 하거나 클래스를 활용하기 위해 필수적으로 작성을 해야한다고 생각한다.


[Java Doc 작성방법](https://johngrib.github.io/wiki/java/javadoc/)


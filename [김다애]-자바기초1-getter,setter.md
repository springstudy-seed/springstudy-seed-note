# 08.14~08.20 : 자바 기초 다지기1

Created: 2022년 8월 20일 오후 7:55
Last Edited Time: 2022년 8월 21일 오후 3:58
Stakeholders: 익명
Status: 스터디전
Type: Technical Spec

# Summary(Quiz)

?클래스와 메소드 차이

?비동기 메소드와 동기 메소드

?자바 생성자와 메소드 비교

?getter와 setter의 사용이유

# 자바 클래스와 메소드

자바의 모든 것은 속성(attribute) 및 메서드(method)와 함께 클래스 및 개체와 연결된다. 예를 들어 자동차에는 무게와 색상과 같은 속성와 엑셀, 브레이크와 같은 메소드가 있다.

- class는 한가지 개체의 속성과 기능등을 묶는 역할을 하고
- method는 특정 작업을 수행하는데 사용되는 기능이다.
- method는 호출될때만 실행되는 코드 블록이며, 매개변수(parameter)를 메소드에 전달한다.

(매개변수가 메소드에 전달되면, 이 데이터는 “인수”라고 한다)

# Static vs  Non-static method

자바를 하다보면 static 또는 public 속성과 메소드를 볼 수 있는데, 이 둘의 차이는 무엇일까? 이 둘의 차이는 클래스에 액세스 하는 방식에 있다. 

- static메소드의 경우 클래스속성에 접근할때, 클래스의 객체를 만든 후 접근할 수 있으며,
- public의 경우 클래스 객체 생성 후 접근하는 모습을 볼 수 있다.

```java
//static과 public 메소드의 차이를 보여주는 예시
public class Main {
  // Static method
  static void myStaticMethod() {
    System.out.println("Static methods can be called without creating objects");
  }

  // Public method
  public void myPublicMethod() {
    System.out.println("Public methods must be called by creating objects");
  }

  // Main method
  public static void main(String[] args) {
    myStaticMethod(); // Call the static method
    // myPublicMethod(); This would compile an error

    Main myObj = new Main(); // Create an object of Main
    myObj.myPublicMethod(); // Call the public method on the object
  }
}
```

### 클래스 속성 Access

To static, public, protected, private, final과 같이 클래스의 접근을 제어하는 키워드들을 Java Modifier라고 한다. Java Modifier는 Access Modifiers와 Non-Access Modifiers로 나뉜다.

- Access Modifiers : 접근권한 제어 - `public` `private` `default` `protected`
- Non-Access Modifiers : 액세스 레벨은 아니지만 다른 기능을 제공 - `final` `abstract` `static` `transient` `synchronized` `volite`

# Java Encapsulation

캡슐화의 의미는 “민감한”데이터를 사용자에게 숨겨져있는지 확인하는 것이다. 이를 수행하기위해

- 클래스 변수/속성을 `private`로 선언하는데

우리는 접근이 불가한 클래스의 변수/속성값에 어떻게 액세스하며 업데이트 할까?

### Getter 와 Setter 메소드 사용 이유

 `get`과 `set` 메소드를 사용하면 이를 해결할 수 있다.  

```java
public class Person {
  private String name; // private = restricted access

  // Getter
  public String getName() {
    return name;
  }

  // Setter
  public void setName(String newName) {
    this.name = newName;
  }
}

public class Main {
  public static void main(String[] args) {
    Person myObj = new Person();
    myObj.setName("John"); // Set the value of the name variable to "John"
    System.out.println(myObj.getName());
  }
}

// Outputs "John"
```

### 캡슐화를 사용하는 이유

이처럼 클래스 속성 및 메서드를 적절히 제어하며, get과 set 메소드를 사용하여 `read-only`, `write-only`으로 설정할 수 있다. 

- Attribute 및 method 제어
- 유연성 - 다른 부분에 영향을 주지 않고 코드의 한 부분을 변경 가능
- 데이터 보안의 강화

# 각종 클래스(Abstract, Inner, interface)

이외에도 1) 자바 상속과 관련된 `extend` . 2) 상속에 의해 서로 연관된 클래스가 많을때 발생하는 다형성과 관련된 `Inner`, `outer`클래스. 3) 다중상속과 추상화와 관련된 `interface` 클래스등이 존재한다.
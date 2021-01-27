# Content

- [Java Static 이란?](#java-static-이란)

<br>

# Java Static 이란?

변수나 메소드 앞에 `static` 키워드를 붙여 사용한다.

`static` 은 같은 클래스에 있는 경우 같은 메모리 주소를 바라본다는 점을 이용해서 메모리 효율 최적화, 데이터 공유 등을 위해 사용한다.

한글로는 "정적인" 이라는 뜻인데 한번에 이해하기가 쉽지 않다.

동적인 다른 자원들과 달리 프로그램 상의 변하지 않는 자원이라고 생각하면 된다.

<br>

## 생성 시기와 소멸 시기

`static` 키워드를 사용하면 객체가 생성되는 시점이 아닌 Class Loader 가 클래스를 Load 할 때 Data 영역에 메모리가 할당되게 된다.

이 영역은 같은 유형의 클래스마다 공유되며 Process 가 종료되는 시점에서 해제되므로 `static` 키워드의 생명주기 역시 Class Load 시 생성되고 Process 종료 시 해제되게 된다.

<br>

## static 변수

클래스 내에서 `static` 으로 선언된 변수는 모든 클래스 객체에서 같은 메모리 주소를 바라본다.

예를 들어 `Student` 라는 클래스가 있다. 이 클래스에는 학생들이 살고 있는 도시인 `teacher` 변수를 갖는다.

```java
public class Student {
  String name;
  String teacher;
}
```

학생들은 모두 같은 반이라서 선생님이 같다고 가정하자.

만약 반의 선생님이 바뀌어도 모든 학생들이 동시에 바뀌어야 한다.

보통 한 반에 학생이 30 명이라고 가정하면, 선생님이 바뀌었을 때 30 명의 학생들을 전부 바꿔줘야 한다.

하지만 이럴 때 `teacher` 변수를 `static` 으로 선언한다면 `Student` 객체의 모든 `teacher` 가 같은 메모리를 공유해서 데이터를 바꾸면 동시에 바뀐다.

<br>

## static 메소드

`static` 메소드는 객체 생성 없이 사용할 수 있으며 보통 유틸리티 클래스에서 주로 사용한다.

`static` 메소드 내에서는 `static` 변수만 사용 가능하고 `this` 호출이 불가능하다.

"이메일 형식 검사", "현재 시간 구하기" 등은 새로운 객체를 할당할 필요가 없기 때문에 `static` 으로 작성하는 게 좋다.

<br>

## static 을 응용항 싱글톤 패턴 (Singleton pattern)

디자인 패턴 중 하나인 싱글톤 패턴은 `static` 의 개념을 이용한 패턴이다.

싱글톤은 단 하나의 객체만 생성하는 패턴이다.

싱글톤 클래스를 만드는 규칙은 다음과 같다.

1. 싱글톤 클래스의 생성자를 `private` 로 하여 외부에서 생성하지 못하게 한다.
2. 클래스 내부에 `static` 변수를 선언한다.
3. `getInstance()` 메소드를 `static` 으로 만들고 내부에서 싱글턴 객체를 선언하여 리턴한다.

```java
class Singleton {
  private static Singleton instance;

  // 생성자를 private 로 하여 외부에서 생성 불가능하게 만듬
  private Singleton() { }

  public static Singleton getInstacne() {
    if (instance == null) {
      instance = new Singleton();
    }

    return instance;
  }
}
```

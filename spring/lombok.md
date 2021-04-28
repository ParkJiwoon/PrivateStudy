# Lombok 이란?

Lombok은 애너테이션을 기반으로 `constructor`, `getter`, `setter` 등 반복적으로 작성해야 하는 메서드를 자동으로 생성하는 라이브러리입니다.

`Model` 이나 `Entity` 같은 도메인 클래스에서 반복되는 코드를 `@` 어노테이션을 이용하여 간단하게 적용할 수 있습니다.

개발자는 어노테이션만 사용하면 컴파일 되는 과정에서 자동으로 그에 맞는 코드들을 생성해줍니다.

<br>

## Lombok 을 사용하지 않는 클래스

```java
class Person {
  private String name;
  private Integer age;

  public Person() { }
  public Person(String name, Integer age) {
    this.name = name;
    this.age = age;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public Integer getAge() {
    return age;
  }

  public void setAge(Integer age) {
    this.age = age;
  }

  @Override
  public String toString() {
    return "Person{" +
           "name='" + name + '\'' +
           "age='" + age + '\'' +
           '}';
  }
}
```

클래스에 필요한 메서드들을 직접 입력해야 합니다.

(요즘은 IDE 의 도움을 받아 자동으로 Generate 할 수 있습니다)

<br>

## Lombok 을 사용하는 클래스

```java
@ToString
@Getter
@Setter
class Person {
  private String name;
  private Integer age;
}
```

롬복을 사용하면 위와 같이 코드를 간단하게 줄일 수 있으며 어떤 롬복이 구현되어있는지 어노테이션만 보고도 확인할 수 있습니다.

Lombok 의 더 많은 종류와 기능을 확인하시려면 Reference 의 활용법 링크를 참고해주세요.

<br>

## 장점

1. 코드의 길이가 짧아진다.
2. 사용하려는 어노테이션이 명시적이다.

<br>

## 단점

1. Lombok 을 모르는 사람들에게는 한눈에 와닿지 않고 어노테이션의 무분별한 사용은 오히려 가독성을 해칠 수 있다.
2. Lombok 을 제대로 이해하지 않고 코드를 작성하거나 수정하면 의도치 않은 결과를 얻을 수 있다. (아래 Reference 의 주의점 링크 참고)

<br>

# Reference

- [Lombok 사용상 주의점(Pitfall)](https://kwonnam.pe.kr/wiki/java/lombok/pitfall)
- [[Java] Lombok이란? 및 Lombok 활용법](https://mangkyu.tistory.com/78?category=872426)

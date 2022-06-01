# Factory 패턴 (1/3) - Simple Factory

# 1. Overview

Factory 패턴은 객체 생성 역할을 별도의 클래스 (Factory) 에게 위임하는 것이 가장 궁극적인 목표입니다.

디자인 패턴 중 Facotry 와 관련된 패턴은 크게 두 가지가 있습니다.

팩토리 메서드 패턴과 추상 팩토리 패턴인데요.

이 두 가지 패턴의 베이스가 되는 가장 단순한 형태의 Factory 패턴이 존재합니다.

엄밀히 따지면 디자인 패턴이 아니라 객체 지향 프로그래밍에서의 자주 쓰이는 관용구 느낌이라 별도의 이름은 없지만 Simple Factory 라는 이름으로 많이 불립니다.

이 글에서는 Simple Factory 에 대해 알아보고 이후 나머지 두 패턴에 대해 알아볼 예정입니다.

<br>

# 2. Simple Factory

Simple Factory 는 굉장히 단순합니다.

객체는 여러 곳에서 생성될 수 있는데, 호출하는 쪽이 객체의 생성자에 직접 의존하고 있으면 나중에 변경되었을 때 수정되어야 하는 코드가 많이 발생합니다.

그래서 생성자 호출 (`new`) 을 별도의 클래스 (`Factory`) 에서 담당하고 클라이언트 코드에서는 팩토리를 통해 객체를 생성합니다.

<br>

## 3. Example

```java
public interface Pet {
}

public class Cat implements Pet {
}

public class Dog implements Pet {
}
```

애완 동물을 한번 예시로 들어봅니다.

공통 인터페이스인 `Pet` 을 정의하고 이를 구현하는 `Cat`, `Dog` 클래스를 만들었습니다.

이제 클라이언트 코드에서 `Cat`, `Dog` 을 사용하기 위해 생성할 수 있습니다.

<br>

## 3.1. Before

```java
Pet cat = new Cat();
Pet dog = new Dog();
```

일반적인 사용법은 `new` 를 사용해 구현 클래스를 생성한 후 호출하는 겁니다.

하지만 이렇게 하면 `Client` 와 클래스들 사이에 다음과 같은 의존관계가 생깁니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/design-pattern/images/screen_2022_05_29_05_02_35.png?raw=true">

이렇게 구현 클래스를 직접 의존하고 있으면 해당 클래스의 생성자나 전처리 코드가 변경되었을 때 사용하는 모든 `Client` 코드를 변경해야 합니다.

그래서 **객체의 생성만을 담당하는 별도의 Factory 클래스**를 만들어 생성 역할을 넘겨봅니다.

<br>

## 3.2. After

```java
public interface Pet {
    enum Type {
        CAT, DOG
    }
}
```

우선 `Pet` 인터페이스에 `enum` 으로 타입을 선언합니다.

<br>

```java
public class PetFactory {

    public Pet createPet(Pet.Type petType) {
        switch (petType) {
            case CAT:
                return new Cat();
            case DOG:
                return new Dog();
            default:
                throw new IllegalArgumentException("Pet 타입이 아닙니다");
        }
    }
}
```

`PetFactory` 를 만든 후 `Pet.Type` 에 따라 다른 객체를 생성해서 반환합니다.

<br>

```java
PetFactory petFactory = new PetFactory();
Pet cat = petFactory.createPet(Pet.Type.CAT);
Pet dog = petFactory.createPet(Pet.Type.DOG);
```

`PetFactory` 를 선언한 후 생성 메서드만 호출하면 실제 구현 클래스인 `Cat`, `Dog` 에 의존하지 않은 코드를 작성할 수 있습니다.


<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/design-pattern/images/screen_2022_05_29_05_29_30.png?raw=true">

의존 관계를 그림으로 표현하면 위와 같이 변경됩니다.

`Client` 에서 구현 클래스를 직접 의존하지 않기 때문에 나중에 클래스 이름이 변경되거나, 생성자가 변경되는 경우에도 `PetFactory` 내부만 수정하면 됩니다.

<br>

## 4. Simple Factory 의 한계

Simple Factory 는 앞서 말했듯이 디자인 패턴으로 분류되지는 않습니다.

이 패턴은 객체의 생성 역할을 담당하며 확장이 용이하다는 장점이 있지만 변경에 닫혀 있어야 한다는 OCP 원칙에 위배됩니다.

만약 새로운 애완 동물 구현 클래스로 `Bird` 가 추가 되었다고 가정합니다.

그럼 `PetFactory` 내부에 존재하는 `switch` 문에 해당 클래스를 추가해줘야 합니다.

객체지향 원칙은 확장을 할 때 기존 코드에 영향을 주지 않는 것을 지향합니다.

팩토리 메서드나 추상 팩토리 패턴을 활용한다면 기존 클래스에 영향을 주지 않고 확장이 가능합니다.
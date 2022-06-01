# Factory 패턴 (3/3) - Abstract Factory 패턴

# 1. Overview

Factory 패턴 시리즈의 마지막인 추상 팩토리 패턴입니다.

추상 팩토리는 얼핏 보면 [팩토리 메서드 패턴](https://bcp0109.tistory.com/367)과 비슷하다고 느낄 수도 있습니다.

가장 큰 차이점은 팩토리 메서드 패턴은 어떤 객체를 생성 할지에 집중하고 추상 팩토리 패턴은 연관된 객체들을 모아둔다는 것에 집중합니다.

<br>

# 2. Abstract Method

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/design-pattern/images/screen_2022_05_31_04_12_38.png?raw=true">

추상 팩토리 패턴은 연관된 객체들의 생성을 하나의 팩토리에서 담당합니다.

<br>

# 3. Example

스포츠 팀을 만든다고 가정합니다.

스포츠 팀에는 플레이어와 매니저가 필수로 존재하기 때문에 **팀의 구성요소**라고 볼 수 있습니다.

<br>

## 3.1. ProductA (Manager)

```java
public interface Manager {
}

public class SoccerManager implements Manager {
}

public class TennisManager implements Manager {
}
```

- `Manager` 인터페이스와 클래스를 정의합니다.
- 축구팀과 테니스팀이 존재하기 때문에 두 개의 `Manager` 구현 클래스를 정의합니다.

<br>

## 3.2. ProductB (Player)

```java
public interface Player {
}

public class SoccerPlayer implements Player {
}

public class TennisPlayer implements Player {
}
```

- `Player` 인터페이스와 클래스를 정의합니다.
- 매니저와 마찬가지로 축구 선수와 테니스 선수를 정의합니다.

<br>

## 3.3. Factory (StaffFactory)

```java
public interface StaffFactory {
    Manager createManager();
    Player createPlayer();
}

public class SoccerStaffFactory implements StaffFactory {

    @Override
    public Manager createManager() {
        return new SoccerManager();
    }

    @Override
    public Player createPlayer() {
        return new SoccerPlayer();
    }
}

public class TennisStaffFactory implements StaffFactory {

    @Override
    public Manager createManager() {
        return new TennisManager();
    }

    @Override
    public Player createPlayer() {
        return new TennisPlayer();
    }
}
```

- Product 를 생성하는 Factory 클래스를 정의합니다.
- `Manager`, `Player` 는 축구라는 하나의 공통점으로 묶을 수 있습니다.
- 그래서 `SoccerManager`, `SoccerPlayer` 를 생산하는 `SoccerStaffFactory` 와 반대로 테니스 객체들을 생성하는 `TennisStaffFactory` 를 정의합니다.
- 단순하게 생각하면 팩토리 메서드 패턴과 동일하지만 **공통된 집합을 모아둔다**는 점이 특징입니다.

<br>

## 3.4. Client

```java
public class AbstractFactoryApp {
    public static void main(String[] args) {
        use(new SoccerStaffFactory());
        use(new TennisStaffFactory());
    }

    private static void use(StaffFactory factory) {
        Manager manager = factory.createManager();
        Player player = factory.createPlayer();
    }
}
```

- 구체 클래스가 아닌 인터페이스에 의존하게 작성할 수 있습니다.
- 어떤 Factory 를 넘겨받는지에 관계 없이 클라이언트는 `Manager`, `Player` 를 생성해서 사용할 수 있습니다.

<br>

## 3.5. 의존 관계 다이어그램

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/design-pattern/images/screen_2022_06_02_01_56_33.png?raw=true">

의존 관계 그림을 보면 맨 처음에 봤던 다이어그램과 동일한 것을 볼 수 있습니다.

<br>

# 4. 장단점

- 장점
  - 팩토리 메서드 패턴과 마찬가지로 수정에는 닫혀 있고 확장에는 열려 있습니다.
  - 여러 개의 비슷한 집합 객체 생성을 하나의 팩토리에 모아둘 수 있습니다. (위 예시 뿐만 아니라 자동차의 부품 등)
- 단점
  - 팩토리 메서드 패턴과 마찬가지로 클래스 갯수가 늘어납니다.

<br>

# Reference

- [Wikipedia - Abstract Factory Pattern](https://en.wikipedia.org/wiki/Abstract_factory_pattern)
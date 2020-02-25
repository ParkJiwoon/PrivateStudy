# when
## when 조건이 겹치는 경우 우선순위가 어떻게 될까

1.
    ```ruby
    a = 1
    b = 2
    c = 3

    case
    when a == 1 && b == 2
        1
    when a == 1 && b == 2 && c == 3
        2
    end
    # 출력: 2
    ```
2.
    ```ruby
    a = 1
    b = 2
    c = 3

    case
    when a == 1 && b == 2 && c == 3
        1
    when a == 1 && b == 2
        2
    end
    # 출력: 1
    ```
## 결론
```&&``` 조건이 겹치면 위에 있는 조건이 우선시된다
# 정렬

기본적으로 `sort` 또는 `sort_by` 가 있는데 배열 자체가 바뀌진 않는다.

만약 배열 자체를 바꾸고 싶다면 새로 할당해줘야 한다.

<br>

## 기본 정렬

```ruby
# print: [2, 3, 4]

[3, 2, 4].sort
```

<br>

## 내림차순 정렬

`sort_by` 메소드는 *__Schwartzian algorithm__* 을 사용하기 때문에 `objects` 나 `collections` 같은 복잡한 자료구조를 다룰 때 더 효율적이라고 한다.

```ruby
# print: [4, 3, 2]

[3, 2, 4].sort { |a, b| b <=> a } 
[3, 2, 4].sort.reverse


# print: [{:name=>"Bill"}, {:name=>"Jane"}, {:name=>"John"}]

names = [{name: "John"}, {name: "Jane"}, {name: "Bill"}]
names.sort_by { |h| h[:name] }
```

<br>

## 커스텀 정렬

직접 두 요소를 비교해서 우선순위 주고싶은 조건일 때 -1 을 리턴하면 된다.

냅두고 싶으면 0

뒤로 보내고 싶으면 1 을 리턴한다.

```ruby
# print: [2, 4, 3]

[3, 2, 4].sort do |a, b|
    case
    when a == 3 then 1      # 원소 3은 무조건 가장 뒤로 보낸다
    when a < b then -1      # 앞의 수 a 가 뒤의 수 b 보다 작으면 앞으로 보낸다
    else 0
    end
end
```

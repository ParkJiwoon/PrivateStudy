# Ruby Regular Expressions

루비 정규 표현식은 문자열 내의 특정 패턴을 찾는 데 도움을 줍니다.

루비에서 정규식을 사용할 때는 양 끝에 슬래쉬(`/`) 를 사용합니다.

## Example

### 1. =~
```ruby
# 'world' 단어 찾기
"Hello world!" =~ /world/   # 6
"Hello world!" =~ /worla/   # nil
```

`=~` 식으로 비교하면 정규 표현식이 시작되는 인덱스를 리턴합니다.

해당되는 인덱스가 존재하지 않으면 `nil` 을 리턴합니다.

<br>

### 2. match
```ruby
if "Hello world!".match(/world/)    # <MatchData "world">
    puts "world is found"
end
```

`match(/regex/)` 메소드를 통해 정규식 포함 여부를 알 수 있습니다.

만약 정규 표현식에 해당되는 값이 없다면 `nil` 을 리턴합니다.

간단히 `true / false` 여부만 알고싶다면 `include?` 메소드를 사용할 수 있습니다.

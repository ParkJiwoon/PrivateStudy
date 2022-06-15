# Ruby 의 as_json 과 to_json 의 차이

# Overview

Ruby 에는 Class 를 Json 으로 표현하는 두 가지 방법이 있습니다.

`as_json` 과 `to_json` 인데 이 두가지는 약간의 차이가 존재합니다.

**`to_json` 은 String 을 반환하고 `as_json` 은 Hash 를 반환**합니다.

<br>

# Example

```ruby
class User
  attr_accessor :name, :age
end

user = User.new("Alice", 22)

# to_json
puts user.to_json       # {"name":"Alice", "age":22}
puts user.to_json.class # String

# as_json
puts user.as_json       # {"name"=>"Alice", "age"=>22}
puts user.as_json.class # Hash
```

<br>

# Reference

- [Stackoverflow - Difference between as_json and to_json method in Ruby](https://stackoverflow.com/questions/38301957/difference-between-as-json-and-to-json-method-in-ruby)
# Ruby

# 1. Variables

```rb
number = 1
puts number # => 1

large_number = 2
```

변수 이름이 길어질 때는 snake_case 를 사용합니다.

<br>

# 2. Data Types

Ruby 에는 다음과 같은 타입들이 있습니다.

기본적으로 최상위 타입은 전부 Object 입니다.

- Numbers
- Strings (texts)
- True, False, and Nil
- Symbols
- Arrays
- Hashes

<br>

## 2.1. Numbers

```rb
a = 1   # Integer
b = 1.2 # Float

c = 1_000_000 # == 1,000,000
```

숫자 타입입니다.

Integer, Float, BigDecimal 등등이 존재합니다.

<br>

## 2.2. Strings

```rb
"hi" + "hi"         # => "hihi"
"hi" * 3            # => "hihihi"

"hello".upcase      # => "HELLO"
"hello".capitalize  # => "Hello"
"hello".length      # => 5
"hello".reverse     # => "olleh"
```

문자열입니다.

큰 따옴표가 아니라 작은 따옴표로 묶어서 사용할 수도 있습니다.

<br>

## 2.3. Symbols

Symbol 은 문자열과 비슷하지만 조금 다릅ㄴ디ㅏ.

앞에 콜론이 붙어있으면 Symbol 입니다. (`:something`)

Symbol 은 보통 텍스트를 `데이터` 가 아닌 `코드` 로 사용할 때 많이 사용합니다.

예를 들면 Hash Key 같은 경우 Key 로 사용한다는 것에 의미가 있죠.

그리고 String 과 비교했을 때 가장 두드러지는 부분은 **같은 Symbol 은 동일한 객체를 참조한다**는 점입니다.

String 은 매번 다른 객체를 생성하지만 Symbol 은 동일한 객체를 재사용합니다.

```rb
"string".object_id # => 2409957680
"string".object_id # => 2682952200
"string".object_id # => 2409974840

:symbol.object_id # => 881948
:symbol.object_id # => 881948
:symbol.object_id # => 881948
```

<br>

## 2.4. Array

```rb
a = [1, 2, 3]

a[0]        # 1
a << 4      # a == [1, 2, 3, 4]
a[6] = 7    # a == [1, 2, 3, 4, nil, nil, 7]
a.first     # 1
a.last      # 7
a.length    # 7
a.compact   # [1, 2, 3, 4, 7]
```

`a.compact` 처럼 새로운 배열을 뽑을 때 현재 배열 자체를 바꾸고 싶다면 마지막에 ! 를 붙여주면 됩니다. (`a.compact!`)

<br>

## 2.5. Hash

```rb
dictionary = { "one" => "eins", "two" => "zwei", "three" => "drei" }
dictionary["one"]               # "eins"
dictionary["zero"] = "null"     # insert
```

Key, Value 로 지정할 수 있는 타입에는 제한이 없습니다.

위 예시에서는 둘다 String 이었지만 Key 로 Integer 를 사용해도 되고 Value 에 배열이 들어가도 됩니다.

가장 많이 사용되는 Key 타입은 Symbol 입니다.

Symbol Key Hash 는 좀더 간편하게 표현할 수 있게 지원해줍니다.

아래 두 문법은 동일한 기능을 갖습니다.

```rb
a = { one: "eins", two: "zwei", three: "drei" }
a = { :one => "eins", :two => "zwei", :three => "drei" }

a[:one] # => "eins"
```

<br>

# 2. String interpolation

```rb
str = "Ruby"
puts "Hello, #{str}"
puts "1 + 2 = #{1 + 2}"
```

`#{}` 로 감싸면 다른 변수를 넣을 수 있습니다.

<br>

# 3. Method

```rb
def add(a, b)
  a + b
end

add(1, 2) # => 3
add 1, 2  # => 3
```

Ruby 의 메서드는 return 이 없어도 맨 마지막 값을 자동으로 리턴합니다.

메서드를 호출할 때는 괄호로 감싸지 않아도 호출 가능합니다.

<br>

## 3.1. 괄호 생략

```rb
def print
  puts "Hello, World!"
end

print() # => Hello, World!
print # => Hello, World!
```

메서드를 정의할 때 파라미터가 필요 없다면 생략할 수 있습니다.

호출 역시 마찬가지로 생략 가능합니다.

<br>

## 3.2. Default parameter

```rb
def print(greeting = "Hello", target = "World")
  puts "#{greeting} #{target}"
end

print               # Hello World
print("Hi")         # Hi World
print("Hi", "Ruby") # Hi Ruby
```

파라미터 값이 없을 때의 기본값을 넣어줄 수 있습니다.

<br>

# 4. Class

```rb
class Person
end
```

클래스는 위와 같이 정의할 수 있습니다.

`person = Person.new` 처럼 객체를 생성할 수 있습니다.

<br>

## 4.1. 인스턴스 변수

```rb
class Person
  def print_name
    puts "My Name is #{@name}"
  end
end

p1 = Person.new
p1.print_name   # My Name is
p1.name = "woody"
p1.print_name   # My Name is woody
```

인스턴스 변수는 클래스 내에서 `@` 가 붙어있는 변수를 의미합니다.

일회성으로 사용되고 끝나는 변수와 달리 생성된 인스턴스 내에서 계속 사용할 수 있습니다.

<br>

## 4.2. 클래스 변수

```rb
class Person
  @@name = "default"

  def name
    @@name
  end

  def name=(name)
    @@name = name
  end
end

p1 = Person.new
p2 = Person.new
p1.name     # default
p2.name     # default

p1.name = "woody"
p1.name     # woody
p2.name     # woody
```

클래스 변수는 모든 객체가 공유하는 변수입니다.

위 코드에서 볼수있듯이 `p1` 의 변수를 바꾸었지만 `p2` 의 변수도 똑같이 바뀐 것을 볼 수 있습니다.

이것이 인스턴스 변수와의 차이점입니다.

<br>

## 4.3. 상수

```rb
class Person
  AGE = 24
end

Person::AGE # 24
```

상수는 변하지 않는 변수입니다.

객체를 직접 생성하지 않아도 바로 사용할 수 있습니다.

<br>

## 4.4. 인스턴스 메서드


# Reference

- [Ruby For Beginners](https://ruby-for-beginners.rubymonstas.org/)
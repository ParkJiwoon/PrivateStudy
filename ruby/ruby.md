# Ruby

# Overview

Ruby 언어에 관한 정보를 정리합니다.

나중에 더 추가될 수도 있습니다.

<br>

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

Symbol 은 문자열과 비슷하지만 조금 다릅니다.

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

## 4.1. Ruby Class 의 인스턴스, 클래스란?

Ruby 에서는 인스턴스 변수, 인스턴스 메서드, 클래스 변수, 클래스 메서드 등 인스턴스와 클래스라는 단어가 많이 나옵니다.

Ruby 에서 말하는 인스턴스, 클래스는 다음과 같습니다.

- 인스턴스
  - 객체를 생성 (`new`) 해야 사용할 수 있는 변수, 메서드
  - 인스턴스 변수는 객체끼리 독립된 값을 가짐
- 클래스
  - 객체 생성 없이 클래스 자체로 생성할 수 있는 변수, 메서드
  - 클래스 변수는 같은 클래스로 생성된 여러 객체가 공유함

<br>

## 4.2. 인스턴스 변수

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

## 4.3. 클래스 변수

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

## 4.4. 상수

```rb
class Person
  AGE = 24
end

Person::AGE # 24
```

상수는 변하지 않는 변수입니다.

객체를 직접 생성하지 않아도 바로 사용할 수 있습니다.

<br>

## 4.5. 인스턴스 메서드, 클래스 메서드

```rb
class Sample
  def print
    puts "Instance Method"
  end

  def self.print
    puts "Class Method"
  end
end

Sample.new.print # Instance Method
Sample.print     # Class Method
```

인스턴스 메서드는 객체를 생성한 뒤에 사용하는 메서드, 클래스 메서드는 그대로 사용하는 메서드입니다.

인스턴스 메서드는 `def` 처럼 평범하게 정의하면 되지만 클래스 메서드는 `def self` 를 사용해서 정의해야 합니다.

<br>

## 4.6. 생성자

```rb
class Person
  def initialize(name)
    @name = name
  end
end

Person.new("woody") # => #<Person:0x000000012cebfeb0 @name="woody">
Person.new          # ArgumentError (wrong number of arguments (given 0, expected 1))
```

Ruby 에서는 `initialize` 메서드로 생성자를 선언할 수 있습니다.

생성자를 선언하지 않으면 파라미터가 없는 기본 생성자를 사용할 수 있습니다.

인스턴스 변수를 초기화할 때는 일반적으로 생성자를 사용합니다.

<br>

## 4.7. Accessor (Getter, Setter)

```rb
class Person
  attr_accessor :name
  attr_reader :age
  attr_writer :address
end

p = Person.new

# accessor (getter, setter)
p.name # => nil
p.name = "adsf"
p.name # => "asdf"

# reader (getter)
p.age # => nil
p.age # NoMethodError (undefined method `age=' for #<Person:0x000000012cb9ce90 @name="asdf">)

# writer (setter)
p.address # NoMethodError (undefined method `address' for #<Person:0x000000012cb9ce90 @name="asdf">)
p.address = "Seoul"
```

`attr_accessor`, `attr_reader`, `attr_writer` 를 사용해서 Getter, Setter 를 생성해줄 수 있습니다.

반대로 private 을 사용해서 Getter, Setter 를 숨길 수도 있습니다.

Accessor 로 지정한 변수는 인스턴스 변수로 사용 가능합니다.

<br>

## 4.8. 상속

```rb
class Fruit
  attr_accessor :name

  def print
    puts "This is Fruit"
  end

  def super_method
    puts "This is super class method"
  end
end

class Apple < Fruit
  def print
    puts "This is Apple"
  end
end

```

`<` 키워드를 사용해서 상속을 표현할 수 있습니다.

상속받은 클래스에서는 상위 클래스의 메서드나 변수를 사용할 수 있으며 오버라이드도 가능합니다.

<br>

## 4.9. 클래스 확장

클래스 확장이란 기존에 정의된 클래스에 새로운 메서드나 변수를 추가하는 것을 의미합니다.

새로운 기능을 추가하는 방법은 총 세가지지만 기존에 정의된 메서드를 중복 정의하는 경우 새롭게 덮어 씌운다는 공통점이 있습니다.

우선 `Game` 이라는 기본적인 클래스를 정의합니다.

```rb
class Game
  def initialize
    @name = "lol"
  end
end
```

<br>

**1. `<<` 키워드 사용**

```rb
# 인스턴스
game = Game.new

class << game
  attr_accessor :age
  
  def print_one
    puts "print_one: #{@name}"
  end
end

# 클래스
class << Game
  def print_class
    puts "print_class"
  end
end
```

`<<` 키워드를 사용하면 동적으로 새로운 변수, 메서드를 추가할 수 있습니다.

<br>

**2. 확장 함수로 정의**

```rb
# 인스턴스
def game.print_two
  puts "print_two: #{@name}"
end

# 클래스
def Game.print_class
  puts "print_class"
end
```

Kotlin 의 확장 함수와 비슷한 문법입니다.

<br>

**3. 클래스 분할 정의**

```rb
class Game
  def print_three
    puts "print_three: #{@name}"
  end

  def self.print_class
    puts "print_class"
  end
end
```

Ruby 는 클래스를 여러 번 정의할 수 있습니다.

분할 정의된 클래스의 내용은 기존 클래스에 그대로 추가됩니다.

<br>

# 5. Module

모듈 (Module) 이란 여러 가지 기능을 모은 것을 의미합니다.

모듈은 크게 두 가지 목적으로 사용합니다.

- Namespace 구분
  - 중복된 이름의 클래스를 사용할 때 충돌이 발생하지 않도록 합니다
  - V1, V2 처럼 버전 구분을 할 때 유용
- 중복된 기능 모으기
  - 여러 곳에서 사용하는 중복된 기능을 분리해서 각각의 모듈에 담을 수 있습니다

<br>

## 5.1. Definition

```rb
module Sample
end
```

모듈은 위와 같이 정의할 수 있습니다.

<br>

## 5.2. Namespace

```rb
module V1
  class API
    def self.call
      puts "Call API v1"
    end
  end
end

module V2
  class API
    def self.call
      puts "Call API v2"
    end
  end
end

V1::API.call # Call API v1
V2::API.call # Call API v2
```

원래 동일한 이름의 클래스와 메서드를 선언하면 분할 정의로 취급하여 기존 메서드는 사라졌습니다.

하지만 모듈을 사용해 분리했기 때문에 두 클래스는 같은 이름과 같은 메서드를 같지만 다른 클래스, 메서드처럼 동작합니다.

<br>

## 5.3. Module Function

```rb
module Person
  ADDRESS = "Seoul"

  def age
    @age
  end

  def age=(age)
    @age = age
  end

  def company
    "company"
  end

  module_function :age, :age=
end

Person::ADDRESS # => "Seoul"
Person.age      # => nil
Person.age = 24
Person.age      # => 24
Person.company  # NoMethodError (undefined method `company' for Person:Module)
```

모듈에서 변수나 메서드를 정의할 수 있습니다.

모듈은 객체처럼 생성이 불가능하기 때문에 사실상 모든 변수와 메서드는 클래스 변수, 메서드라고 볼 수 있습니다.

정의된 메서드를 모듈에서 직접 사용하기 위해선 `module_function` 키워드를 사용해서 지정해줘야 합니다.

<br>

## 5.4. Mixin

Module 을 Class 에서 참조하면 마치 클래스의 메서드인것처럼 사용할 수 있습니다.

이걸 mix-in 이라고 부릅니다.

모듈을 mixin 하는 방법에는 `include`, `prepend`, `extend` 총 세가지가 있습니다.

<br>

### 5.4.1. Include

```rb
module IncludeModule
  def hello
    puts "Hello, Include"
  end
end

class Person
  include IncludeModule

  def hello
    puts "Hello, Person"
  end
end

person = Person.new
person.hello    # Hello, Person
```

`include` 키워드를 사용해서 모듈을 참조하면 모듈의 메서드를 **인스턴스 메서드**로 사용할 수 있습니다.

만약 동일한 이름의 메서드가 모듈과 클래스 양쪽에 정의되어 있다면 클래스의 메서드를 우선시합니다.

<br>

### 5.4.2. Prepend

```rb
module PrependModule
  def hello
    puts "Hello, Prepend"
  end
end

class Person
  prepend PrependModule

  def hello
    puts "Hello, Person"
  end
end

person = Person.new
person.hello    # Hello, Prepend
```

`prepend` 는 `include` 와 마찬가지로 모듈의 메서드를 **인스턴스 메서드**로 사용할 수 있습니다.

하지만 `include` 와 다르게 동일한 이름의 메서드가 정의되어 있을 경우 모듈의 메서드를 우선시합니다.

<br>

### 5.4.3. Extend

```rb
module ExtendModule
  def hello
    puts "Hello, Extend"
  end
end

class Person
  extend ExtendModule
end

Person.hello    # Hello, Extend
```

`extend` 는 다른 mixin 과 다르게 **클래스 메서드**로 사용 가능합니다.

중복 정의된 메서드는 `include` 와 마찬가지로 클래스에 있는 걸 우선시합니다.

<br>

### 5.4.4. Ancestors

만약 여러 개의 모듈을 한번에 `include` 하면 어떻게 될까?

`include`, `prepend` 가 여러개 섞여 있으면 어떻게 될까?

Ruby 에서는 어떤 오브젝트가 젤 우선시 되는지 `ancestors` 메서드로 확인할 수 있습니다.

<br>

```rb
module Include1
end

module Include2
end

module Prepend1
end

module Prepend2
end

class Human
end

class Person < Human
  include Include1
  include Include2
  prepend Prepend1
  prepend Prepend2
end

```

`Person` 클래스에서는 여러 모듈을 동시에 mixin 하고 있습니다.

`prepend` -> `Person` -> `include` 순서까지는 쉽게 짐작할 수 있지만 같은 키워드를 사용한 모듈들은 어떤게 우선시 되는지 알 수 없습니다.

게다가 상위 클래스에도 중복된 메서드가 정의되어 있다면 더욱 더 헷갈립니다.

이를 확인할 수 있는게 `ancestors` 메서드입니다.

<br>

```rb
Person.ancestors
=> [Prepend2, Prepend1, Person, Include2, Include1, Human, ActiveSupport::ToJsonWithActiveSupportEncoder, Object, PP::ObjectMixin, ActiveSupport::Dependencies::Loadable, ActiveSupport::Tryable, JSON::Ext::Generator::GeneratorMethods::Object, Kernel, BasicObject]
```

`Person.ancestors` 를 사용하면 해당 클래스가 참조하고 있는 모든 오브젝트가 나옵니다.

그리고 중복 정의된 메서드가 있으면 앞 순서에 있는 오브젝트의 메서드가 우선시됩니다.

그래서 `Person` 클래스에서 중복된 메서드가 있으면 `Prepend2` 모듈의 메서드가 사용 됩니다.

모든 클래스는 오브젝트라서 `BasicObject` 가 최상단에 있는 걸 알 수 있습니다.

<br>

# Reference

- [Ruby For Beginners](https://ruby-for-beginners.rubymonstas.org/)
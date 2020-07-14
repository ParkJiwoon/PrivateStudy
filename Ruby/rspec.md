# RSpec - Ruby Test Framework

RSpec 프레임워크는 BDD 프로세스에서 사용하는 툴입니다.

따라서 RSpec 테스트 코드에는 자세한 설명이 첨부되어 있습니다.

> **BDD (Behavior-Driven Development) 란 ?**
>
> TDD (Test-Driven Development) 의 유닛 테스트에서 더 나아가 테스트 케이스 자체가 요구사양이 되도록 개발하는 방식입니다.
> 단위 테스트 보다는 행위에 집중하여 테스트 메소드의 이름을 *"이 클래스가 어떤 행위를 해야한다. (should do something)"* 라는 식의 문장으로 작성하며 행위를 위한 테스트에 집중을 합니다.
> 
> - Feature : 테스트에 대상의 기능/책임을 명시한다.
> - Scenario : 테스트 목적에 대한 상황을 설명한다.
> - Given : 시나리오 진행에 필요한 값을 설정한다.
> - When : 시나리오를 진행하는데 필요한 조건을 명시한다.
> - Then : 시나리오를 완료했을 때 보장해야 하는 결과를 명시한다.

<br>

## RSpec 기본 구조

테스트의 기대되는 값과 실제 값을 비교하여 성공 여부를 리턴해줍니다.

```ruby
expect(테스트 객체).to 비교 matcher(예상되는 값)
```

<br>

## describe

`describe` 키워드를 사용해서 어떤 함수를 설명하려는 지 명확하게 합니다.

예를 들어 클래스 함수를 참조할 때는 `.` 또는 `::` 을 접두어로 하고 인스턴스 함수를 참조할 때는 `#` 을 접두어로 사용합니다.

```ruby
describe '.authenticate' do
describe '#admin?' do
```

<br>

## 설명을 짧게 유지하기

명세의 설명이 40문자를 넘어가면 `context` 를 사용해서 쪼개야 합니다.

```ruby
# Bad
it 'has 422 status code if an unexcepted params will be added' do

# Good
context 'when not valid' do
  it { is_expected.to respond_with 422 }
end
```

<br>

## 각 테스트는 한 가지만을 확인

테스트가 한 가지만을 확인해야 에러를 찾기도 쉽고 실패하는 테스트를 바로 찾을 수 있습니다.

같은 예제 안에서 여러 가지 예상이 나온다면 여러 행동으로 나누어야 합니다.

```ruby
# 분리형
it { is_expected.to rsespond_with_content_type(:json) }
it { is_expected.to assign_to(:resource) }

# 통합형
it 'creates a resource' do
  expect(response).to respond_with_content_type(:json)
  expect(response).to assign_to(:resource)
end
```

<br>

## 가능한 모든 케이스를 테스트

유효한 경우, 유효하지 않은 경우를 모두 테스트 해야 합니다.

예를 들어 **User 삭제** 기능을 테스트 하려는데 `before_action` 으로 **User 찾기** 가 걸려있다면

```ruby
before_action :find_user

def destory
  render 'show'
  @user.destory
end
```

보통은 삭제가 잘 되었는지만 확인하지만 유저를 찾지 못한 경우에 대해서도 테스트를 해주어야 합니다.

```ruby
# Bad
it 'show user'

# Good
describe '#destory' do
  context 'when user is found' do
    it 'responds with 200'
    it 'shows the user'
  end

  context 'when user is not found' do
    it 'responds with 404'
  end
end
```

<br>

## Expect vs Should

새 프로젝트에서는 항상 `expect` 문법만 사용합니다.

```ruby
# Bad
it 'creates a resource' do
  response.should respond_with_content_type(:json)
end

context 'when not valid' do
  it { should respond_with 422 }
end

# Good
it 'creates a resource' do
  expect(response).to respond_with_content_type(:json)
end

context 'when not valid' do
  it { is_expected.to respond_with 422 }
end
```

<br>

## 필요한 데이터만 만들기

필요 이상으로 많은 데이터를 만들 필요는 없습니다.

최대한 적은 양의 데이터로 모든 케이스를 테스트 할 수 있어야 합니다.

<br>

# Reference

- [RSpec 가이드라인](http://www.betterspecs.org/ko)

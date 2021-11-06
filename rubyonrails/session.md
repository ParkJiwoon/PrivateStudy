# Rails 세션

# 1. Overview

Rails Session 은 우리가 알고있는 세션과 크게 다르지 않습니다.

사용자에 대한 일부 데이터를 저장하기 위해 사용합니다.

쿠키를 사용할 수도 있지만 외부에 노출되는 쿠키와 달리 서버에만 저장해야 하는 중요한 데이터들은 세션에 저장하면 편리합니다.

<br>

# 2. 사용법

Rails Session 은 `Hash` 처럼 사용하면 됩니다.

다른 Controller 에서 저장한 데이터를 꺼내 쓸 수 있습니다.

<br>

```ruby
# app/controllers/index_controllers.rb
def create
    # ...
    session[:current_user_id] = @user.id
    # ...
end
```

예를 들어 `IndexController` 에서 저장한 세션값을..

<br>

```ruby
# app/controllers/users_controllers.rb
def index
    # ...
    current_user = User.find_by_id(session[:current_user_id])
    # ...
end
```

`UserController` 에서 꺼내서 사용할 수 있습니다.

<br>

# 3. Session Stores

세션 저장소의 종류는 쿠키, 데이터베이스, Memcached, Redis 등등 다양합니다.

쿠키 세션 저장소를 제외한 모든 세션 저장소는 동일한 프로세스로 동작합니다.

<img src="https://user-images.githubusercontent.com/28972341/140613146-23e29d31-0ccb-4519-89f7-aebb838011c7.png" width="80%">

<br><br>

## 3.1. Session 값 저장

1. `session[:current_user_id] = 1` 을 호출했는데 기존에 사용하던 세션(Session ID) 이 없는 경우
2. Rails 는 `09497d46978bf6f32265fefb5cc52264` 와 같은 임의의 Session ID 를 사용하여 `sessions` 테이블에 새로운 record 를 저장
3. 해당 record 의 `data` 속성에 `{current_user_id: 1}` (Base64-encoded) 값도 함께 저장
4. 생성한 Session ID (`09497d46978bf6f32265fefb5cc52264`) 는 `Set-Cookie` 를 사용하여 브라우저에게 전달


<br>

## 3.2. Session 값 가져오기

1. 브라우저가 서버에 요청할 때 `Cookie:` 헤더를 사용해서 동일한 쿠키값을 전달
   - (1번 예시) `Cookie: _my_app_session=09497d46978bf6f32265fefb5cc52264; path=/; HttpOnly`
2. 코드에서 `session[:current_user_id]` 을 호출
3. 쿠키에 있는 Session ID 값으로 `sessions` 테이블에 있는 record 를 가져옴
4. record 에 있는 `data` 속성에서 `current_user_id` 값을 가져옴


<br>

# Reference

- [How Rails Sessions Work](https://www.justinweiss.com/articles/how-rails-sessions-work/)
- [RubyOnRails 공식 가이드 - Session](https://guides.rubyonrails.org/action_controller_overview.html#session)

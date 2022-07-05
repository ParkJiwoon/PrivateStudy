# Rails 프로젝트 히스토리 정리

# Overview

Rails 연습을 위해 이것저것 적용해보면서 시간 흐름에 따라 정리합니다.

<br>

# Skills

- Ruby 2.6.2
- Rails 5.2.8
- [Grape API](https://github.com/ruby-grape/grape)

~~Rails 6+ 에서 Grape API 사용 시 `Unknown NameError` 발생해서 Rails 5.2.8 로 다운그레이드 했습니다.~~
Grape 관련된 이슈였고 Rails 버전과는 관계 없었습니다만 그냥 실무에서 사용중인 버전과 맞춰서 Rails 5 를 사용합니다.

<br>

# ERD

간단한 게시판 예제입니다.

<br>

# 1. Install Ruby

```sh
# 1. asdf 설치 (rbenv 같은 버전 관리 툴)
$ brew install asdf

# 2. ruby 설치 (XCode12 버전부터 install 하면 에러가 발생해서 컴파일 옵션 변경 필요)
$ asdf plugin add ruby
$ export CFLAGS="-Wno-error=implicit-function-declaration"
$ asdf install ruby 2.6.2

# 설치 확인
$ asdf list
$ asdf global ruby 2.6.2
$ asdf current
```

[asdf](https://asdf-vm.com/) 를 사용해 설치합니다.

<br>

# 2. Install MySQL

로컬에서 직접 실행시키지 말고 Docker 를 사용합니다.

[Apple Silicon 에서 MySQL Docker 실행](https://github.com/ParkJiwoon/PrivateStudy/blob/master/ci-cd/docker-mysql.md) 을 참고하여 설치, 실행합니다.

혹시 3306 포트로 실행이 안되면 로컬 MySQL 이 실행된거니 `brew services stop mysql` 명령어로 중지해줍니다.

`ps -ef | grep mysql` 명령어를 사용했을 때 아무것도 나오지 않아야 로컬 DB 충돌이 없습니다.

혹시라도 `mysqld -v` 같은 프로세스가 떠있다면 kill 해줍니다.

<br>

# 3. Start Rails

![](images/screen_2022_07_04_11_23_10.png)

RubyMine 에서 New Project 로 생성합니다.

아무것도 건들지 않은 상태로 `(asdf exec) rails s` 를 사용해서 서버를 띄워봅니다.

실패나는 경우는 보통 DB 연결이 제대로 안되어 있어서 그런거니 [✨ first commit](https://github.com/ParkJiwoon/practice-rails/commit/16fd5ad541a9c149ca6c7fe5470ac2748bee87f4) 의 `config/database.yml` 을 보고 맞춰줍니다.

<br>

![](images/screen_2022_07_04_11_57_22.png)

http://127.0.0.1:3000 접속 시 위 사진처럼 서버가 정상적으로 뜨면 [gitignore](https://www.toptal.com/developers/gitignore) 를 사용해서 `.gitignore` 파일을 수정해줍니다.

저는 `Rails`, `RubyMine+all` 을 입력해서 생성했습니다.

<br>

# 4. Grape API

Grape 는 Ruby 에서 사용할 수 있는 REST-like API 프레임워크 입니다.

Rails 에서 간단한 DSL 만 추가하면 쉽게 사용할 수 있습니다.

[ruby-grape/grape github](https://github.com/ruby-grape/grape) 참고해서 적용했습니다.

<br>

## 4.1. Installation

Ruby 버전이 2.4 이상이어야 합니다.

아래 명령어로 gem 을 추가하면 바로 사용 가능합니다.

```sh
bundle add grape
```

<br>

## 4.2. Basic Usage

Grape API 를 추가하는 건 기존 Rails 와 크게 다르지 않습니다.

기존 Rails 에서는 Controller 를 추가하고 `config/routes.rb` 에서 라우팅 해줬습니다.

마찬가지로 Grape 역시 API 를 추가하고 `config/routes.rb` 에서 연결해주면 되는데 여기서는 `mount` 라는 걸 사용합니다.

<br>

### app/api/v1/hello.rb

```rb
module V1
  class Hello < Grape::API
    version 'v1', using: :path  # /v1/... prefix
    format :json                # json response

    desc 'Return success message'
    get '/hello' do
      { res: "success" }
    end

    desc 'Return message with ID'
    params do
      requires :id, type: Integer, desc: 'ID'
    end
    get '/hello/:id' do
      { res: "hello #{params[:id]}" }
    end
  end
end
```

우선 API 파일을 하나 작성합니다.

Grape 는 `api` 디렉터리 하위에 API 파일 (Controller) 을 작성하는 것이 관례입니다.

api 버전 관리를 위해 `V1` 이라는 모듈 하위에 작성했고 클래스는 `Grape::API` 를 상속 받아야 합니다.

위 코드를 보면 알수 있듯이 별다른 설명이 없어도 알아볼 수 있을 정도로 굉장히 친절합니다.

<br>

### config/routes.rb

```rb
Rails.application.routes.draw do
  mount V1::Hello => '/api'
end
```

`mount` 키워드를 사용해서 매핑해줍니다.

이제 우리는 `http://localhost:3000/api/v1/hello` URL 로 요청하면 `{ res: "success" }` 응답을 받을 수 있습니다.

<br>

## 4.3. Mounting

비슷한 API 끼리 한 파일에 모아둘 수도 있습니다.

예를 들어 `/v1` prefix 를 가지며 항상 json 응답을 주는 API 들끼리 한 곳에 모아두려면 이렇게 작성하면 됩니다.

<br>

### app/api/v1/mount.rb

```rb
module V1
  class Mount < Grape::API
    version 'v1', using: :path
    format :json

    mount V1::Hello
  end
end
```

별도의 클래스를 하나 선언한 후 공통으로 사용할 `version`, `format` 을 담고 `mount` 키워드를 사용합니다.

<br>

```rb
Rails.application.routes.draw do
  mount V1::Mount => '/api'
end
```

그리고 `config/routes.rb` 파일을 수정하면 `V1::Mount` 내부에서 마운트되어 있는 `V1::Hello` 내용들도 전부 `/api` 에 매핑됩니다.

<br>

# 5. Grape Entity

Grape Entity 는 Grape API 를 보조하는 라이브러리입니다.

Grape Entity 는 API 의 Request, Response Model 을 쉽게 만들어 주는 일종의 DTO 입니다.

가장 큰 특징은 특정 Model 에서 원하는 필드값만 노출할 수 있게 도와주는 기능입니다.

마치 `@JsonIgnore` 처럼 노출하고 싶지 않은 필드값은 제외할 수 있습니다.

자세한 내용은 [ruby-grape/grape-entity github](https://github.com/ruby-grape/grape-entity)에서 볼 수 있습니다.

<br>

## 5.1. Installation

grape 와 별개로 grape-entity 라는 gem 을 추가해야 합니다.

```sh
bundle add grape-entity
```

<br>

## 5.2. Exposure

- `expose`: 노출할 필드값
- `documentation`: swagger 와 같은 문서화 툴에 노출할 설명
- `if`: 특정 조건이 True 일 때 노출
- `as`: 설정된 필드명을 다른 값으로 노출하고 싶을 때 사용

<br>

## 5.3. Basic Usage

간단한 예시를 통해 사용법을 알아봅니다.

<br>

### app/api/v1/entities/simple_response_entity.rb

```rb
module V1
  module Entities
    class SimpleResponseEntity < Grape::Entity
      expose :code, documentation: { type: "Integer", desc: "Status code" }
      expose :message, documentation: { type: "String", desc: "Response message" }
    end
  end
end
```

`Grape::Entity` 를 상속하는 클래스를 선언해줍니다.

Grape 관련된 파일들은 module 과 path 를 정확히 일치시켜줘야 합니다. ([Ruby Grape 사용 시 NameError (uninitialized constant {Class}) 가 발생하는 이유](https://bcp0109.tistory.com/375) 참고)

API 응답으로 `code`, `message` 만 내려줍니다.

<br>

### app/api/v1/hello.rb

```rb
module V1
  class Hello < Grape::API
    desc 'Return Simple Response'
    get '/simple' do
      response = { code: 200, message: "OK", success: true }
      present response, with: V1::Entities::SimpleResponseEntity
    end
  end
end
```

사용법은 간단합니다.

`return` 대신 `present` 키워드를 사용하여 우리가 내려줄 Model 을 지정하고, `with` 키워드로 Grape Entity 를 지정합니다.

`response` 는 `code`, `message`, `success` 세 개의 필드값을 갖고 있지만 `SimpleResponseEntity` 에 의해 실제 응답은 `code`, `message` 만 내려갑니다.


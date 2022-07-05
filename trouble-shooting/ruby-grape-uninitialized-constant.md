# Ruby Grape 사용 시 NameError (uninitialized constant {Class}) 가 발생하는 이유

# Issue

Ruby Grape 와 Grape Entity 를 사용할 때 클래스를 참조하지 못하고 자꾸 `NameError (uninitialized constant {Class})` 에러가 발생했습니다.

처음에는 Rails 버전과 Grape 버전이 호환되지 않아서 발생하는 이슈인가 해서 다운그레이드도 해봤지만 해결되지 않았습니다.

<br>

# Solution

제가 이런 오류를 겪게된 이유는 크게 두가지였는데요.

첫번째는 제가 Ruby 에 익숙하지 않아서 발생한 일이고 다른 하나는 Grape 문서를 제대로 읽지 않아서 발생했습니다.

<br>

## 1. Ruby File 형식을 준수하고 파일 이름과 Class 이름이 일치해야 함

`SimpleResponseEntity` 라는 클래스를 사용한다고 하면 파일 이름을 `simple_response_entity.rb` 로 만들어야 합니다.

무심코 Java 에서의 버릇처럼 Camel Case 로 작성했던게 문제였습니다.

<br>

## 2. Grape 프레임워크는 Module 과 파일의 경로가 일치해야 함

[Ruby Grape Docs > Mounting > Rails](https://github.com/ruby-grape/grape#rails) 파트를 보면 다음과 같은 문장이 있습니다.

> Place API files into app/api. Rails expects a subdirectory that matches the name of the Ruby module and a file name that matches the name of the class. In our example, the file name location and directory for Twitter::API should be app/api/twitter/api.rb.

<br>

즉, Grape 관련된 기능을 사용하기 위해선 파일을 `app/api` 하위에 만들어야 하며 subdirectory 와 module 이름까지 일치해야 한다는 뜻입니다.

Rails 자체도 파일이나 경로를 굉장히 중요시하는 것처럼 Grape 도 비슷한 특징을 갖고 있는 것 같습니다.

<br>

# Reference

- https://github.com/ruby-grape/grape#rails
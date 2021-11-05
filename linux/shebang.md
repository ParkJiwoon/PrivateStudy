# SheBang(#!) 이란? #!/usr/bin/env 를 사용하는 이유

# Overview

쉘 스크립트 파일을 보면 파일 최상단에 `#!/usr/bin/env bash` 가 있는 걸 많이 봤을겁니다.

잘 모를때는 그냥 주석으로 실행 환경을 의미하는 건 줄 알았는데 사실은 굉장히 중요한 문장이었습니다.

<br>

# 1. SheBang (#!)

우선 `#!` 은 주석이 아니라 이 자체가 하나의 기호입니다.

어원은 위키피디아에 의하면 어원은 Hash(#) 또는 Sharp(#) 과 Bang(!) 의 합성어에서 유래한 것 같다고 합니다.

`#!` 은 2 Byte 의 매직 넘버 (Magic Number) 로 스크립트의 맨 앞에서 이 파일이 어떤 명령어 해석기의 명령어 집합인지를 시스템에 알려주는 역할을 합니다.

`#!` 바로 뒤에 나오는 것은 경로명으로, 명령어들을 해석할 프로그램의 위치를 나타냅니다.

가장 일반적으로 사용되는건 `#!/bin/bash` 이며 앞으로 나올 명령어들을 주석을 제외하고 순서대로 실행시킵니다.

만약 경로가 정확하지 않다면 `bad interpreter` 가 발생하고 다른 인터프리터를 지정하면 문법 오류로 실패합니다.

<br>

## 1.1. 문법

```html
#!<interpreter> [optional-arg]
```

문법은 위와 같으며 `#!` 뒤에 공백 (Space) 이 하나 있어도 동작합니다.

`<Interpreter>` 에는 프로그램의 절대경로가 입력되어야 합니다.

<br>

## 1.2. 예시

```html
#!/bin/sh
#!/bin/bash
#!/usr/bin/pwsh
#!/usr/bin/env python3
```

<br>

# 2. #!/usr/bin/env 란?

위 설명에서 `<interpreter>` 에는 프로그램의 절대 경로가 와야 한다고 했습니다.

하지만 절대경로는 시스템에 따라 달라질 수도 있습니다.

파이썬을 예로 들면 시스템에 따라 `/bin/local/python` 또는 `/usr/bin/python` 에 위치할 수도 있고 버전 또한 python2 와 python3 둘다 설치되어 있을 수 있습니다.

이럴 때 `#!/usr/bin/env` 로 설정하면 **절대경로에 상관 없이 인터프리터의 위치를 찾아서 실행**해줍니다.

그러니 여러 환경에서 실행되야할 스크립트라면 `#!/usr/bin/env` 를 사용하는 게 좋습니다.

<br>

# 3. Test

간단하게 테스트를 해봅시다.

터미널을 켜서 `vi test` 로 파일 하나를 작성합니다.

```sh
#! /usr/bin/env bash

echo "Hello This is Bash"
```

<br>

위 파일에 실행 권한을 추가하고 실행하면 정상적으로 동작합니다.

```sh
# 권한 추가
$ chmod +x test

# 실행
$ ./test
Hello This is Bash
```

<br>

이번엔 다시 파일을 열어 인터프리터를 파이썬으로 지정합니다.

```sh
#! /usr/bin/env python

echo "Hello This is Bash"
```

<br>

권한은 아까 주었으니 다시 파일만 실행시키면 에러가 발생합니다.

파이썬 문법에 맞지 않아 오류가 발생한 겁니다.

```sh
$ ./test
  File "./test", line 3
    echo "Hello This is Bash"
                            ^
SyntaxError: invalid syntax
```

<br>

다시 파일을 열어 파이썬 문법으로 고쳐줍시다.

```sh
#! /usr/bin/env python

print "Hello This is Python"
```

<br>

파일을 실행시키면 정상적으로 실행됩니다.

```sh
$ ./test
Hello This is Python
```

<br>

# Conclusion

SheBang(`#!`) 문법은 스크립트 파일에서 어떤 프로그램으로 해당 파일을 실행시킬 지 결정합니다.

`#!/usr/bin/env` 를 사용하면 인터프리터의 절대 경로를 지정하지 않아도 알아서 경로를 찾아주기 때문에 여러 시스템 환경에서 사용할 때 유용합니다.

테스트를 보면서 알 수 있었던 한가지 특징은 확장자를 입력하지 않아도 `#!` 으로 인터프리터를 지정하면 정상적으로 실행된다는 사실입니다.

<br>

# Reference

- [Wikipedia - Shebang (Unix)](https://en.wikipedia.org/wiki/Shebang_(Unix))
- [StackOverflow - What is the difference between "#!/usr/bin/env bash" and "#!/usr/bin/bash"?](https://stackoverflow.com/questions/16365130/what-is-the-difference-between-usr-bin-env-bash-and-usr-bin-bash)
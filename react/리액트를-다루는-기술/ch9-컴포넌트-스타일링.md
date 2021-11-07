# 9장 컴포넌트 스타일링

# Overview

리액트의 컴포넌트 스타일링은 다양한 방식이 있습니다.

딱히 정해진 방식은 없고 회사마다 개발자마다 취향에 따라 선택합니다.

- 일반 CSS: 가장 기본적인 방식
- Sass: 확장된 CSS 문법으로 CSS 코드를 더욱 쉽게 작성하게 해줌
- CSS Module: 스타일을 작성할 때 CSS 클래스가 다른 CSS 클래스의 이름과 절대 충돌하지 않도록 파일마다 고유한 이름을 자동으로 생성해주는 옵션
- styled-components: CSS-in-Js 방식으로 자바스크립트 파일 내부에 스타일 존재

<br>

# 1. 일반 CSS

대부분의 프로젝트가 일반 CSS 방식으로 만들어져 있습니다.

기존 스타일링 방식이 딱히 불편하지 않다면 그대로 사용해도 됩니다.

```css
.App {
  text-align: center;
}

.App-logo {
  height: 40vmin;
}
```

<br>

## 1.1. CSS Selector

CSS 는 이름이 겹치지 않는게 중요합니다.

CSS Selector 를 사용하면 특정 클래스 내부에 있는 경우에만 스타일을 적용할 수 있습니다.

```css
/* .App 클래스 내부의 .logo 클래스에만 스타일 적용 */
.App .logo {
  height: 40vmin;
}

/* .App 클래스 내부의 a 태그에만 스타일 적용 */
.App a {
  color: #61afb;
}
```

<br>

# 2. Sass

Sass(Syntactically Awesome Style Sheets) (문법적으로 매우 멋진 스타일시트) 는 CSS 전처리기입니다.

복잡한 작업을 쉽게 해주고 스타일 코드의 재활용성을 높여주며 가독성도 향상시킵니다.

Sass 는 두가지 확장자 `.scss` 와 `.sass` 를 지원합니다.

```sass
<!-- .sass -->
body
  font: 100%
  color: white
```

```scss
// .scss
body {
  font: 100%;
  color: white;
}
```

보통 .scss 문법이 더 자주 사용됩니다.

<br>

## 2.1. .scss 문법

```scss
// $ 을 붙여서 변수 사용
$red: #fa5252;

// mixin: 재사용 되는 스타일을 함수처럼 사용 가능
@mixin square($size) {
  $calculated: 32px * $size;
  width: $calculated;
  height: $calculated;
}

.SassComponent {
  
  // 클래스 내부의 클래스
  .box {
    
    // .red 클래스가 .box 와 함께 사용 ex) className="box red"
    &.red {
      background: $red;
      @include square(1);
    }

    // .box 에 마우스를 올렸을 때
    &:hover {
      background: black;
    }
  }
}
```

위에서 사용되는 `$red` 와 같은 변수나 `@mixin` 과 같은 함수를 여러 곳에서 재사용 한다면 별도의 파일로 분리 후 `@import` 하여 사용할 수 있습니다.

<br>

## 2.2. node_modules 에서 라이브러리 불러오기

```scss
@import "~library/styles";
```

물결 기호 (`~`) 를 사용한다면 node_modules 에서 쉽게 스타일 라이브러리를 불러올 수 있습니다.

<br>

## 2.3. 반응형 CSS

```sh
$ yarn add open-color include-media
```

반응형을 지원해주는 라이브러리 `include-media` 와 색 라이브러리 `open-color` 를 추가합니다.

<br>

```scss
@import "~include-media/dist/include-media";
@import "~open-color/open-color";

.SassComponent {
  display: flex;
  background: $oc-gray-2;
  @include media("<768px") {
    background: $oc-gray-9;
  }
  // ...
}
```

위 코드는 화면 가로 크기가 768px 미만이 되면 배경색을 검정색으로 바꿔줍니다.
# 2장 JSX

- [2장 JSX](#2장-jsx)
- [1. JSX 란?](#1-jsx-란)
- [2. JSX 의 장점](#2-jsx-의-장점)
- [3. JSX 문법](#3-jsx-문법)
  - [3.1. 감싸인 요소](#31-감싸인-요소)
  - [3.2. 자바스크립트 표현](#32-자바스크립트-표현)
  - [3.3. if 문 대신 조건부 연산자](#33-if-문-대신-조건부-연산자)
  - [3.4. AND 연산자 (&&) 를 사용한 조건부 렌더링](#34-and-연산자--를-사용한-조건부-렌더링)
  - [3.5. undefined 렌더링하지 않기](#35-undefined-렌더링하지-않기)
  - [3.6. 스타일 적용할때는 camelCase 로](#36-스타일-적용할때는-camelcase-로)
  - [3.7. class 대신 className](#37-class-대신-classname)
  - [2.8. 닫아야 하는 태그](#28-닫아야-하는-태그)

# 1. JSX 란?
JSX 는 자바스크립트의 확장 문법입니다.

```jsx
function App() {
  return (
    <div>
      Hello <b>react</b>
    </div>
  );
}
```

위 코드는 아래처럼 변환됩니다.

```jsx
function App() {
  return React.createElement("div", null, "Hello ", React.createElement("b", null, "react"));
}
```

<br>

# 2. JSX 의 장점

1. 일반 자바스크립트보다 보기 쉽고 익숙합니다.
2. HTML 태그와 혼용할 수 있기 때문에 컴포넌트들도 각각 JSX 안에 작성 가능합니다.

<br>

# 3. JSX 문법

JSX 를 사용하기 위해선 몇 가지 규칙을 준수해야 합니다.

<br>

## 3.1. 감싸인 요소

컴포넌트에 여러 요소가 있다면 반드시 부모 요소 하나로 감싸야 합니다.

```jsx
import React, { Fragment } from 'react';

function App() {
  return (
    // <Fragment></Fragment> 대신 <div></div> 또는 <></> 로도 표현 가능
    <Fragment>
      <h1>리액트 안녕!</h1>
      <h2>잘 작동하니?</h2>
    </Fragment>
  );
}
```

<br>

## 3.2. 자바스크립트 표현

JSX 안에서 자바스크립트 표현을 쓰려면 `{ }` 로 감싸면 됩니다.

```jsx
function App() {
  const name = '리액트';
  return (
    <div>
      <h1>{name} 안녕!</h1>
      <h2>잘 작동하니?</h2>
    </div>
  );
}
```

<br>

## 3.3. if 문 대신 조건부 연산자

JSX 내부에서는 `if` 문을 사용할 수 없습니다.

만약 조건에 따른 분기를 추가해야 한다면 조건부 연산자 (삼항 연산자) 를 활용하면 됩니다.

```jsx
function App() {
  const name = '리액트';
  return (
    <div>
      {name === '리액트' ? (
        <h1>리액트입니다.</h1>
      ) : (
        <h2>리액트가 아닙니다.</h2>
      )}
    </div>
  );
}
```

<br>

## 3.4. AND 연산자 (&&) 를 사용한 조건부 렌더링

특정 조건을 만족할 때 내용을 보여주고, 만족하지 않으면 아예 아무것도 렌더링 하지 않는 경우도 있습니다.

이럴 때는 삼항 연산자 또는 AND 연산자를 사용해서 구현할 수 있습니다.

```jsx
function App() {
  const name = '리액트';
  return (
    <div>
      {/* 1. 삼항 연산자 활용 (null 을 렌더링하면 아무것도 보여주지 않음) */}
      {name === '리액트' ? <h1>리액트입니다.</h1> : null}
      
      {/* 2. AND 연산자 (&&) 활용 */}
      {name === '리액트' && <h1>리액트입니다.</h1>}
    </div>
  );
}
```

<br>

## 3.5. undefined 렌더링하지 않기

```jsx
function App() {
  const name = undefined;
  return name;
}
```

위 코드처럼 `undefined` 를 바로 렌더링하면 에러가 발생합니다.

<br>

```jsx
function App() {
  const name = undefined;
  return name || '값이 undefined 입니다';
}
```

이렇게 기본값을 OR 연산자 (`||`) 로 지정해야 합니다.

<br>

```jsx
function App() {
  const name = undefined;
  return <div>{name || '리액트'}</div>;
}
```

이렇게 js 문법 내부에서도 사용할 수 있습니다.

<br>

## 3.6. 스타일 적용할때는 camelCase 로

원래 HTML 에서 스타일을 적용할 때는 `background-color` 처럼 `-` 을 사용합니다.

하지만 리액트 DOM 요소에 스타일을 적용할 때는 객체 형태로 넣어야 하기 때문에 `backgroundColor` 로 작성합니다.

```jsx
function App() {
  const name = '리액트';
  const style = {
    backgroundColor: 'black',
    color: 'aqua',
    fontSize: '48px',
    fontWeight: 'bold',
    padding: 16 // 단위를 생략하면 px로 지정
  };

  return <div style={style}>{name}</div>;
}
```

<br>

`style` 객체를 미리 선언하지 않고 바로 지정할 수도 있습니다.

```jsx
function App() {
  const name = '리액트';
  return (
    <div 
      style={{
        backgroundColor: 'black',
        color: 'aqua',
        fontSize: '48px',
        fontWeight: 'bold',
        padding: 16 // 단위를 생략하면 px로 지정
      }}
    >
      {name}
    </div>
  )
}
```

<br>

## 3.7. class 대신 className

HTML 에서 class 를 지정할 때는 `class="myClass"` 같은 형식으로 지정합니다.

하지만 JSX 에서는 `className` 으로 설정해줘야 합니다.

`className` 대신 `class` 를 사용해도 스타일이 적용되기는 하지만 브라우저 개발자 도구 Console 에 경고가 나타납니다.

우선 CSS 파일을 먼저 작성해봅니다.

```css
/* App.css */

.react {
  background: aqua;
  color: black;
  font-size: 48px;
  font-weight: bold;
  padding: 16px;
}
```

<br>

그리고 JSX 파일에서 해당 CSS 를 적용하면 됩니다.

```jsx
import './App.css';

function App() {
  const name = '리액트';
  return <div className="react">{name}</div>
}
```

<br>

## 2.8. 닫아야 하는 태그

HTML 에서는 가끔 태그를 닫지 않은 상태로 코드를 작성할 수 있습니다.

예를 들면 `<input></input>` 대신 `<input>` 이라고만 입력해도 작동합니다.

하지만 JSX 에서는 태그를 닫지 않으면 오류가 발생합니다.

`<input />` 처럼 태그 사이에 별 내용이 없는 경우에 사용할 수 있는 self-closing 태그는 정상적으로 동작합니다.

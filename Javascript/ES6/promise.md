# 프로미스 (Promise)

프로미스는 비동기 상태를 값으로 다룰 수 있는 객체다.

프로미스가 사용되기 전에는 콜백 패턴이 많이 사용되었다.

그러다가 여러 가지 프로미스 라이브러리가 등장하면서 널리 사용되었고 ES6 에서는 프로미스가 자바스크립트 언어에 포함되었다.

<br>

## 1. 콜백(Callback) 패턴의 문제

자바스크립트에서는 비동기 프로그래밍의 한 가지 방법으로 콜백(callback) 패턴을 많이 사용했다.

하지만 콜백 패턴은 조금만 중첩되어도 코드가 복잡해지는 콜백 지옥(Callback Hell)을 만들어낸다.

```js
function display(data) {
  console.log("result: " + data);
}

function conactData(data) {
  parsedData = data.concat(" parsing");
  console.log(parsedData);
  display(parsedData);
}

function getData(callback) {
  data = "This is new Data";
  console.log(data);
  callback(data, display);
}

getData(parsing);
// This is new Data
// This is new Data parsing
// result: This is new Data parsing
```
<br>

위 코드는 프로미스를 사용하면 아래와 같이 좀더 보기 쉽게 바꿀 수 있다.

```js
const getData = new Promise((resolve, reject) => {
  data = "This is new Data";
  console.log(data);
  resolve(data);
})

getData
  .then(data => {
    // concatData
    parsedData = data.concat(" parsing");
    console.log(parsedData);
    return parsedData;
  })
  .then(data => {
    // display
    console.log("result: " + data);
    return data;
  })
// This is new Data
// This is new Data parsing
// result: This is new Data parsing
```

<br>

## 2. 프로미스의 세가지 상태

프로미스는 다음 세 가지 상태 중 하나로 존재한다.

상태 | 설명
:--: | --
대기 중 (pending)  | 결과를 기다리는중
이행됨 (fulfilled) | 수행이 정상적으로 끝났고 결과값을 갖고 있음
거부됨 (rejected)  | 수행이 비정상적으로 끝났음

그리고 이행됨, 거부됨 상태를 처리됨(settled) 상태라고 한다.

프로미스는 처리됨 상태가 되면 더 이상 다른 상태로 변경되지 않는다.

대기 중 상태일때만 이행됨 또는 거부됨 상태로 변할 수 있다.

<br>

## 3. 프로미스 생성

### 3.1. new 키워드로 생성

```js
// 1. new 키워드로 생성
const promise = new Promise((resolve, reject) => {
  // ..
  // resolve() or reject('error')
})
```

`new` 키워드로 생성한 프로미스는 대기 중 상태가 된다.

생성자에 입력되는 `resolve` 와 `reject` 는 콜백 함수이다.

`resolve` 를 호출하면 `promise` 는 이행됨 상태가 된다.

반대로 `reject` 를 호출하면 거부됨 상태가 된다.

만약 생성자에 입력된 함수 안에서 예외가 발생하면 거부됨 상태가 된다.

`new` 키워드로 생성된 프로미스의 내부는 즉시 실행된다.

만약 API 요청을 보내는 비동기 코드가 있다면 프로미스가 생성되는 순간에 요청을 보낸다.

<br>

### 3.2. Promise.reject 생성

```js
// Promise.reject 로 생성
const promise = Promise.reject('error');
```

거부됨 상태인 프로미스를 생성한다.

<br>

### 3.3. Promise.resolve 생성

```js
// Promise.resolve 로 생성
const promise = Promise.resolve(params);

const p1 = Promise.resolve(123);
const p2 = Promise.resolve(p1);   // return p1
console.log(p2 === p1);           // true
```

입력값이 프로미스라면 그 객체가 그대로 반환되고, 프로미스가 아니라면 이행됨 상태인 프로미스가 반환된다.

<br>

## 4. 프로미스 사용

### 4.1. then

```js
getData().then(onResolve, onReject);
```

`then` 은 처리됨 상태가 된 프로미스를 처리할 때 사용되는 메소드다.

프로미스가 처리됨 상태가 되면 `then` 메소드의 파라미터로 전달된 함수가 호출된다.

`then` 메소드는 항상 프로미스를 반환하기 때문에 하나의 프로미스에서 연속적으로 `then` 메소드를 호출할 수 있다.


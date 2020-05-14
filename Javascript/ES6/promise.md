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

`then` 은 처리됨 상태가 된 프로미스를 처리할 때 사용되는 메소드다.

프로미스가 처리됨 상태가 되면 `then` 메소드의 파라미터로 전달된 함수가 호출된다.

`then` 메소드는 항상 프로미스를 반환하기 때문에 하나의 프로미스에서 연속적으로 `then` 메소드를 호출할 수 있다.

```js
getData().then(onResolve, onReject);
```

<br>

만약에 함수 수행 중에 예외가 발생해서 거부됨 상태가 되면 `onReject` 함수가 존재하는 `then` 까지 이동한다.

`onReject` 함수가 실행되고 나면 프로미스는 다시 이행됨 상태가 되어 4 다음에는 5 를 출력한다.

```js
// print: 4 5
Promise.reject('error')
  .then(() => console.log('1'))
  .then(() => console.log('2'))
  .then(() => console.log('3'), () => console.log('4'))
  .then(() => console.log('5'));
```

<br>

### 4.2. catch

`catch` 는 프로미스 수행 중에 발생한 예외를 처리한다.

`then` 함수의 `onReject` 와 같은 역할을 한다.

예외 처리는 `then` 보다 `catch` 함수를 사용하는 게 가독성이 더 좋다.

```js
Promise.reject('error').then(null, error => {
  console.log(error);
});

Promise.reject('error').catch(error => {
  console.log(error);
})
```

<br>

`then` 은 `onResolve` 에서 에러가 발생했을 때 같은 함수 내에 있는 `onReject` 함수에서 처리할 수 없다.

하지만 `catch` 를 사용하면 처리가능하고 함수도 좀 더 직관적이다.

`catch` 도 마찬가지로 프로미스를 반환하기 때문에 계속해서 체이닝을 이어나갈 수 있다.

```js
// onResolve 에서 에러가 발생해도 onReject 가 아닌 뒤에서 처리해야됨
Promise.resolve().then(onResolve, onReject);

// print: this is Error: then error
// print: success then 2
Promise.resolve()
  .then(() => {
    throw new Error('then error');
  })
  .catch(error => {
    console.log('this is ' + error);
    return 2;
  })
  .then(data => {
    console.log('success then: ' + data);
  });
```

<br>

### 4.3. finally

`finally` 는 프로미스가 처리됨(settled) 상태일 때 호출되는 함수이다.

프로미스 체인의 가장 마지막에 사용된다.

`finally` 는 이전에 사용된 프로미스를 그대로 반환하기 때문에 처리됨 상태인 프로미스의 데이터를 건드리지 않고 추가 작업을 할 수 있다.

```js
// print: 123
// print: finish promise
Promise.resolve(123)
  .then(data => {
    console.log(data);
    return data;
  })
  .catch(error => {
    console.log(error);
    return error;
  })
  .finally(() => {
    console.log('finish promise');
  });
```

<br>

## 5. 프로미스 활용

다음 예제를 통해 `Promise` 를 활용하는 방법들을 알아본다.

```js
const getData1 = Promise.resolve(1);
const getData2 = new Promise((resolve, reject) => 
  setTimeout(function() {
    console.log('3');
    resolve(2);
  }, 5000));
```

<br>

### 5.1. Promise.all: 병렬 처리

`then` 함수를 체인으로 연결하면 각각의 비동기 로직이 병렬로 처리되지 않고 순차적으로 실행된다.

서로 간의 의존성이 없다면 병렬로 처리하는 게 더 빠르다.

`Promise.all` 함수는 여러 개의 프로미스를 동시에 실행하며 프로미스가 모두 처리되면 처리됨 상태가 되고 하나라도 거부된다면 거부됨 상태가 된다.

```js
// print: 3   (5초 뒤 출력)
// print: 1 2
Promise
  .all([getData1, getData2])
  .then(([data1, data2]) => {
    console.log(data1, data2);
  });
```

<br>

### 5.2. Promise.race: 가장 빨리 처리된 프로미스

`Promise.race` 는 여러 개의 프로미스 중 가장 빨리 처리된 프로미스를 반환한다.

여러 개의 프로미스 중 하나라도 처리되면 처리됨 상태가 된다.

프로미스 간의 격차가 커서 먼저 `Promise.race` 가 실행 되더라도 다른 프로미스의 작업은 중지되지 않는다.

```js
// print: 1
// print: 3   (5초 뒤 출력)
Promise
  .race([getData1, getData2])
  .then(data => {
    console.log(data);
  });
```
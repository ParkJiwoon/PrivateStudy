# 프로미스 (Promise)

프로미스는 비동기 상태를 값으로 다룰 수 있는 객체다.

프로미스가 사용되기 전에는 콜백 패턴이 많이 사용되었다.

그러다가 여러 가지 프로미스 라이브러리가 등장하면서 널리 사용되었고 ES6 에서는 프로미스가 자바스크립트 언어에 포함되었다.

<br>

## 1. 콜백(Callback) 패턴의 문제

자바스크립트에서는 비동기 프로그래밍의 한 가지 방법으로 콜백(callback) 패턴을 많이 사용했다.

하지만 콜백 패턴은 조금만 중첩되어도 코드가 복잡해지는 콜백 지옥(Callback Hell)을 만들어낸다.

```js
function getData(callback) {
  // .. request and get data
  callback(data);
}

function parsing(data, callback) {
  // .. data parse and call callback
  callback(parsedData);
}

function display(data) {
  console.log(data);
}

getData(parsing);
```

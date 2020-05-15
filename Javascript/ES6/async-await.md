# async await

`Promise` 를 사용해도 어쩔 수 없는 한계가 존재한다.

콜백 함수에 비해서 조금 나아졌다고는 하나 중첩해서 쓰거나 return 으로 함수 호출하면서 로직이 복잡해지면 가독성이 떨어진다.

`async await` 을 사용하면 `then` 체이닝보다 가독성이 좋아진다.

하지만 `async await` 도 프로미스를 활용하는 개념이기 때문에 프로미스를 완전히 대체할 수는 없다.

<br>

## 1. 사용하기

### 1.1. async

`async` 키워드를 이용해서 정의된 함수는 항상 프로미스를 반환한다.

따라서 함수 호출 뒤에 `then` 으로 체이닝이 가능하다.

리턴값이 프로미스라면 그대로 반환된다.

```js
async function getData1() {
  return 111;
}

const getData2 = async () => {
  throw new Error(222);
}

async function getData3() {
  return Promise.resolve(333);
}

getData1().then(data => console.log(data));     // 111
getData2().catch(err => console.log('getData2: ' + err));   // getData2: Error: 222
getData3().then(data => console.log(data));     // 333
```

<br>

### 1.2. await

`await` 키워드는 `async` 함수 내부에서만 사용된다.

`await` 키워드를 붙여서 프로미스를 사용하면 해당 프로미스가 처리됨 상태가 될 때까지 기다린다.

```js
function printAfter(seconds) {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log('print ' + seconds);
      resolve(seconds);
    }, seconds * 1000)
  },);
}

async function log() {
  const data1 = await printAfter(3);    // 3 초 뒤 출력
  const data2 = await printAfter(5);    // 8 초 뒤 출력
  console.log(data1, data2);
}

log();
```

<br>

## 2. 활용하기

### 2.1. 비동기 함수 병렬 실행

`await` 키워드가 붙으면 해당 함수가 끝날때까지 기다리게 된다.

하지만 여러 개의 `await` 함수를 호출할 때 함수들 간의 의존성이 없다면 굳이 순차적으로 실행할 필요가 없다.

프로미스는 생성과 동시에 실행된다는 점을 활용하여 프로미스 생성을 먼저 하고 `await` 키워드를 나중에 호출하는 방법을 사용하면 된다.

```js
async function log() {
  const p3 = printAfter(3);     // 3초 뒤 출력
  const p5 = printAfter(5);     // 5초 뒤 출력

  const data1 = await p3;
  const data2 = await p5;
  
  console.log(data1, data2);
}
```

<br>

아니면 `Promise.all` 을 사용한다면 더 간단하게 표현 할 수 있다.

```js
async function log() {
  const [data1, data2] = await Promise.all([printAfter(3), printAfter(5)]);

  console.log(data1, data2);
}
```

<br>

### 2.2. 예외 처리

`try catch` 문으로 감싸면 async / sync 함수 구분하지 않고 발생하는 예외들을 모두 잡는다.

```js
async function log() {
  try {
    const data = printAfter(3);
    console.log(data);
  } catch (error) {
    console.log(error);
  }
}
```
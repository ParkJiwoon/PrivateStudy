# 변수 정의: const, let

ES5 까지는 var 키워드로 변수를 정의했다.

ES6 에서는 const 와 let 을 이용하는 새로운 변수 정의 방법이 생겼다.

<br>

## 1. var 문제점

### 1.1. 함수 스코프

스코프(scope) 란 변수가 사용될 수 있는 영역을 말한다.

var 는 함수 스코프라서 함수 밖에서 사용하면 에러가 발생한다.

```js
function error() {
    var a = 1;
}
console.log(a);     // ReferenceError: a is not defined
```

<br>

var 키워드 없이 변수를 선언하면 전역 변수가 된다.

```js
function global() {
    a = 1;
}
function local() {
    console.log(a);
}
global();
local();    // 1 출력
```

<br>

for 문에서 선언된 변수가 사라지지 않는다.

for 문 뿐만 아니라 while, switch, if 문 등 함수 내부에서 작성되는 모든 코드가 같은 문제를 안고 있다.

```js
for (var i = 0; i < 10; i++) {
    console.log(i);     // 0 ~ 9 출력
}
console.log(i);     // 10 출력
```

<br>

### 1.2. 호이스팅(hoisting)

var 로 정의된 변수는 해당 스코프의 최상단으로 끌어올려진다.

변수를 선언 없이 사용하면 참조 에러가 발생한다.

```js
console.log(a);     // ReferenceError: a is not defined
```

<br>

하지만 다음 코드는 에러가 발생하지 않는다.

```js
console.log(a);     // undefined
var a = 1;
```

호이스팅에 의해 위 코드는 아래와 같이 취급된다.

```js
var a = undefined;
console.log(a);     // undefined
a = 1;
```

<br>

심지어 변수 선언 전에 값을 할당할 수도 있다.

```js
console.log(a);     // undefined
a = 2;
console.log(a);     // 2
var a = 1;
```

이런 호이스팅 개념은 코드를 직관적이지 않게 만들며, 버그가 발생하여도 정확한 원인을 찾기 힘들게 만든다.

<br>

### 1.3. 항상 재할당 가능

var 는 재할당 가능한 변수로만 만들 수 있다.

따라서 상수처럼 고정된 값을 변수로 선언해서 이용해야 할 때에도 변경될 위험이 존재하고 있다.

<br>

## 2. var 문제점 해결: const, let

### 2.1. 블록 스코프

대부분의 언어에서 블록 스코프를 사용한다.

블록 스코프에서는 블록을 벗어나면 변수를 사용할 수 없다.

블록을 벗어나도 살아있던 var 변수와 달리 const, let 변수는 블록을 벗어나면 관리해줄 필요가 없다.

```js
if (true) {
    const a = 0;
}
console.log(a);     // ReferenceError: a is not defined
```

<br>

### 2.2. 호이스팅(hoisting)

const, let 도 호이스팅 된다.

하지만 변수를 선언하기 전에 사용하려고 하면 에러가 발생한다.

```js
console.log(a);      // ReferenceError: Cannot access 'a' before initialization
const a = 1;
```

이 때문에 const, let 은 호이스팅 되지 않는다고 생각하기 쉽다.

const, let 은 서로 다른 스코프에서 같은 이름의 변수를 사용할 때 실수를 방지해준다.

<br>

아래 코드에서 a 는 다른 블록 스코프지만 상위에 선언되었기 때문에 1 이 출력된다.

```js
const a = 1;
{
    console.log(a);     // 1
}
```

<br>

하지만 아래 코드에서는 참조 에러가 발생한다.

하위 블록 스코프에서 호이스팅으로 인해 `const a = 2` 로 할당 되기 전까지는 사용이 불가능하다.

이처럼 같은 이름의 변수를 사용할 때의 실수를 방지해준다.

```js
const a = 1;
{
    console.log(a);
    const a = 2;
}
```

<br>

### 2.3. 재할당 불가능 const

const 로 정의된 변수는 재할당이 불가능하다.

let 으로 선언된 변수는 재할당 가능하다.

대신 const 로 정의된 객체의 내부 속성값은 수정 가능하다.

```js
const arr = [1, 2];
arr[0] = 3;
arr.push(4);
console.log(arr);   // [3, 2, 4]
```

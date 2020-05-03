# 객체와 배열의 사용성 개선

## 1. 간편한 생성 및 수정

### 1.1. 단축 속성명

단축 속성명 (shorthand property names) 로 객체 리터럴 코드를 간편하게 작성할 수 있다.

```js
const name = 'alice';
const obj = {
    age: 21,
    name,
    getName() { return this.name; },
}
console.log(obj);   // { age: 21, name: "alice", getName: ƒ getName() }
```

새로 만드려는 객체의 속성명이 이미 변수로 존재하면 변수 이름만 적어주면 된다.

이때 속성명은 변수 이름과 같아진다.

속성값이 함수이면 function 키워드 없이 함수명만 적으면 된다.

마찬가지로 속성명은 함수명과 같아진다.

<br>

### 1.2. 계산된 속성명

계산된 속성명 (computed property names) 으로 객체의 속성명을 동적으로 결정할 수 있다.

```js
function create1(key, value) {
  const obj = {};
  obj[key] = value;
  return obj;
}

create2 = (key, value) {}

function create2(key, value) {
  return { [key]: value };
}

console.log(create1('key1', 'value1'));       // { key1: 'value1' }
console.log(create2('key2', 'value2'));       // { key2: 'value2' }
```

계산된 속성명을 사용하면 create2 처럼 간결하게 코드를 짤 수 있다.

key 를 대괄호 [ ] 로 감싸는 이유는 `return { key: value }` 처럼 하면 속성명으로 변수값이 아닌 __key__ 자체가 되어버리기 때문이다.

<br>

## 2. 속성값 간편하게 가져오기

### 2.1. 전개 연산자

전개 연산자(spread operator)는 배열이나 객체의 모든 속성을 풀어놓을 때 사용한다.

<br>

#### 2.1.1. 매개변수 여러개 전달

코드의 첫 번째 줄처럼 전개 연산자를 사용하지 않는다면 매개변수의 갯수가 4 개로 고정된다.

하지만 전개 연산자를 사용한다면 numbers 배열의 갯수가 몇개든 전부 전달할 수 있다.

```js
Math.max(1, 2, 3, 4);

const numbers = [1, 2, 3, 4];
Math.max(...numbers);
```

<br>

#### 2.1.2. 배열과 객체 복사

전개 연산자를 이용하여 간단하게 배열과 객체를 복사할 수 있다.

복사된 배열이나 객체는 새로운 값이기 대문에 수정해도 기존 배열이나 객체에 영향을 주지 않는다.

배열의 경우 전개 연산자를 사용하면 그 순서가 유지된다.

```js
const arr1 = [1, 2, 3];
const obj1 = { age: 23, name: 'alice' };

const arr2 = [...arr1];
const obj2 = { ...obj1 };
arr2.push(4);
obj2.age = 80;

console.log(arr1);      // [1, 2, 3]
console.log(arr2);      // [1, 2, 3, 4]
console.log(obj1);      // { age: 23, name: 'alice' }
console.log(obj2);      // { age: 80, name: 'alice' }
```

<br>

전개 연산자를 사용하면 서로 다른 두 배열이나 객체를 쉽게 합칠 수 있다.

```js
const obj1 = { age: 21, name: 'alice' };
const obj2 = { address: 'seoul' };
const obj3 = { ...obj1, ...obj2 };      // { age: 21, name: 'alice', address: 'seoul' }

const arr1 = [1, 3, 5];
const arr2 = [2, 4, 6];
const arr3 = [...arr1, ...arr2];        // [1, 3, 5, 2, 4, 6]
```

<br>

ES5 까지는 중복된 속성명으로 합치면 에러가 발생했지만 ES6 부터는 허용된다.

마지막에 입력된 값이 최종값이 된다.

```js
const obj1 = { age: 21, name: 'alice' };
const obj2 = { name: 'bob' };
const obj3 = { ...obj1, ...obj2 };      // { age: 21, name: 'bob' }
const obj4 = { ...obj2, ...obj1 };      // { name: 'alice', age: 21 }
```

<br>

### 2.2. 비구조화

#### 2.2.1. 배열 비구조화

배열 비구조화(array destructuring)를 사용하면 배열의 여러 속성값을 변수로 쉽게 할당할 수 있다.

배열의 속성값이 왼쪽 변수에 순서대로 들어간다.

```js
const arr = [1, 2];
const [a, b] = arr;     // a: 1, b: 2
```

<br>

배열 비구조화 정의 시 기본값을 설정할 수 있다.

만약 기본값도 없고 할당되는 값도 없다면 undefined 가 된다.

```js
const arr = [1];
const [a = 10, b = 20] = arr;       // a: 1, b: 20
```

<br>

두 값을 교환할 수도 있다.

```js
let a = 1;
let b = 2;
[a, b] = [b, a];        // a: 2, b: 1
```

<br>

일부 속성값을 무시할 수도 있다.

```js
const arr = [1, 2, 3];
const [a, , c] = arr;       // a: 1, c: 3
```

<br>

나머지 값을 별도의 배열로 만들 수도 있다.

```js
const arr = [1, 2, 3];
const [first, ...rest] = arr;       // first: 1, rest: [2, 3]
const [a, b, c, ...empty] = arr;    // a: 1, b: 2, c: 3, empty: []
```

<br>

#### 2.2.2. 객체 비구조화

객체 비구조화(object destructuring)를 사용하면 여러 속성값을 변수로 쉽게 할당할 수 있다.

순서대로 들어가는 배열과 달리 Object 의 키에 맞춰서 들어간다.

그리고 Object 에 존재하는 키와 동일한 이름의 변수명을 사용해야 한다.

```js
const obj = { age: 21, name: 'alice' };
const { age, name } = obj;      // age: 21, name: 'alice'
const { name, age } = obj;      // age: 21, name: 'alice'
const { a, b } = obj;           // a: undefined, b: undefined
```

<br>

임의로 다른 변수명에 할당할 수도 있다.

```js
const obj = { age: 21, name: 'alice' };
const { age: age2, name } = obj;        // age: not defined error, age2: 21, name: 'alice'
```

<br>

기본값을 정의할 수 있다.

들어오는 값이 undefined 인 경우에만 기본값으로 세팅된다.

기본값 세팅과 다른 변수명에 할당을 동시에 사용할 수도 있다.

```js
const obj = { age: undefined, name: null, grade: 'A' };
const { age: age2 = 0, name = 'noName', grade = 'F', address = 'seoul' } = obj;  
// age: not defined error, age2: 0, name: null, grade: 'A', address: 'seoul'
```

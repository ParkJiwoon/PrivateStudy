# 강화된 함수의 기능

ES6 에서는 함수의 기능을 온전하게 완성했다고 볼 수 있다.

<br>

## 1. 매개변수에 추가된 기능

### 1.1. 매개변수 기본값 (default parameter)

ES6 부터 함수 매개변수에 기본값을 줄 수 있다.

```js
function ex(a = 1) {
    console.log({ a });
}

ex();      // { a: 1 }
```

<br>

기본값으로 함수 호출을 넣어줄 수도 있다.

```js
function getDefault() {
    return 1;
}

function ex(a = getDefault()) {
    console.log({ a });
}

ex();       // { a: 1 }
```

<br>

### 1.2. 나머지 매개변수 (rest parameter)

입력된 매개변수 중에서 특정 매개변수 외의 나머지는 배열로 만들어줄 수 있다.

매개변수 개수가 가변적일 때 유용하다.

```js
function ex(a, ...rest) {
    console.log({ a, rest });
}

ex(1, 2, 3);        // { a: 1, rest: [2, 3] }
```

<br>

### 1.3. 명명된 매개변수 (named parameter)

객체 비구조화를 이용하여 매개변수의 이름을 명시적으로 사용하며 함수를 호출할 수 있다.

매개변수의 이름과 값을 동시에 적을 수 있기 때문에 가독성이 높다.

명명된 매개변수를 사용하면 함수를 호출할 때마다 객체를 생성하기 때문에 비효율적이라고 생각할 수도 있다.

하지만 자바스크립트 엔진이 최적화를 통해 새로운 객체를 생성하지 않으므로 안심하고 사용해도 된다.

```js
function getValues1(numbers, greaterThan, lessThan) {
    console.log({ numbers, greaterThan, lessThan });
}

function getValues2({ numbers, greaterThan, lessThan }) {
    console.log({ numbers, greaterThan, lessThan });
}

const numbers = [10, 20, 30, 40];

getValues1(numbers, 5, 25);                             // { numbers, greaterThan: 5, lessThan: 25 }
getValues2({ numbers, greaterThan: 5, lessThan: 25 });  // { numbers, greaterThan: 5, lessThan: 25 }
```

<br>

### 1.4. 선택적 매개변수 (optional parameter)

명명된 매개변수를 응용하면 선택적 매개변수를 사용할 수도 있다.

`getValues1` 함수에서는 필요없는 매개변수가 있어도 undefined 로 값을 넣어주어야 한다.

매개변수의 값이 많아지면 일일히 undefined 를 넣어주어야 하고 가독성도 굉장히 떨어지게 된다.

하지만 `getValues2` 함수에서는 필요없는 매개변수는 코드로 적지 않고 필요한 매개변수만 넣어주면 된다.

```js
getValues1(numbers, undefined, 25);
getValues2({ numbers, greaterThan: 5 });
getValues2({ numbers, lessThan: 25 });
```

<br>

## 2. 화살표 함수 (arrow function)

ES6 에서는 화살표를 이용하여 함수를 정의하는 방법이 새로 추가되었다.

화살표 함수를 사용하면 함수를 간결하게 작성할 수 있다.

<br>

### 2.1. 한줄 사용

화살표 함수는 한줄로도 간단하게 정의할 수 있다.

중괄호 블록을 사용하지 않고 바로 오른쪽에 정의하며, return 키워드를 명시적으로 정의하지 않아도 오른쪽에 있는 값이 리턴된다.

매개변수가 하나라면 소괄호도 생략 가능하다.

반환하는 값이 Object 라면 반드시 소괄호로 감싸야 한다.

```js
const add = (a, b) => a + b;
const add5 = a => a + 5;                                        // 매개변수가 하나면 소괄호를 생략 가능하다.
const addAndReturnObject = (a, b) => ({ result: a+b });         // 반환값이 Object 라면 소괄호로 감싸준다.
const print = () => console.log("print");
```

<br>

### 2.2. 여러줄 사용

화살표 함수에 코드가 여러줄이라면 전체를 중괄호로 묶고 return 키워드를 사용한다.

```js
const add = (a, b) => {
    if (a <= 0 || b <= 0) {
        throw new Error('must be positive number');
    }
    return a + b;
}
```

<br>

### 2.3. this 와 arguments 가 바인딩 되지 않음

화살표 함수에서는 this 와 arguments 가 바인딩 되지 않는다.

만약 arguments 가 필요하다면 나머지 매개변수 (rest parameter) 를 사용한다.

```js
const print = (...rest) => console.log(rest);
print(1, 2);        // [1, 2]
```

<br>

### 2.4. this 바인딩 차이점

일반 함수는 호출되었을 때, 호출한 대상에 바인딩된다.

아래 코드를 통해 this 가 각각 어디를 바라보고 있는지 알 수 있다.

앞에 `f.` 를 붙여서 호출하면 this 가 func() 함수를 가리키고 있다.

하지만 다른 변수에 할당한 다음에 아무것도 붙이지 않고 호출하면 this 는 전역객체를 참조한다. (브라우저에서는 window)

```js
function func() {
  this.value = 1;

  this.increase = function() {
    this.value++;
  };

  this.print = function() {
    console.log(this);
  };
}

const f = new func();
f.increase();
console.log(f.value);       // 2
f.print();                  // func { value: 2, increase: ƒ, print: ƒ }

const inc = f.increase;
const print = f.print;
inc();
console.log(f.value);       // 2
print();                    // Window { parent: Window, ... }
```

<br>

기존 ES5 에서는 이런 문제점을 우회하기 위해 클로저(closure) 라는 개념을 사용했다.

```js
function func() {
  this.value = 1;
  that = this;
  
  this.increase = function() {
    that.value++;
  };

  this.print = function() {
    console.log(that);
  };
}
```

<br>

일반 함수는 호출할 당시의 객체에 this 바인딩 되는 대신 화살표 함수는 가장 가까운 일반 함수를 참조한다.

따라서 함수를 어디에 재할당 하던지 항상 생성되었을 당시의 일반함수를 참조하게 된다.


```js
function func() {
  this.value = 1;

  this.increase = () => {
    this.value++;
  };

  this.print = () => {
    console.log(this);
  };
}

const f = new func();
f.increase();
console.log(f.value);       // 2
f.print();                  // func { value: 2, increase: ƒ, print: ƒ }

const inc = f.increase;
const print = f.print;
inc();
console.log(f.value);       // 3
print();                    // func { value: 2, increase: ƒ, print: ƒ }
```

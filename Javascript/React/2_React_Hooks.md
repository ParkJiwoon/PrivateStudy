# React Hooks

리액트 훅은 16.8 버전부터 추가되었습니다.

함수형 컴포넌트에서 상태값을 사용할 수 있고 자식 요소에 접근할 수 있는게 가장 크며, 리액트 팀에서도 적극적으로 밀고 있습니다.

새로 만드는 컴포넌트는 클래스형보다는 함수형으로 작성하는게 좋습니다.

<br>

## useState

`useState` 함수를 사용하면 함수형 컴포넌트에서 상태값을 사용할 수 있습니다.

<br>

#### 기본 사용

```js
import React, { useState } from 'react';

function App() {
  const [count, setCount] = useState(0);

  function increment() {
    setCount(count + 1);
  }
  
  function decrement() {
    setCount(count - 1);
  }

  return (
    <div>
      <h1>count: {count}</h1>
      <button onClick={increment}>증가</button>
      <button onClick={decrement}>감소</button>
    </div>
  );
}

export default App;
```

가장 기본적인 `useState` 사용 코드입니다.

`useState` 를 사용하면 `[상태값, 상태를 변화시키는 메소드]` 배열을 리턴합니다.

파라미터로는 `상태의 기본값 (defaultState)` 을 넘겨받습니다.

따라서 `const [count, setCount] = useState(0)` 라인은 `count` 라는 상태값을 가지며 `setCount` 로 상태값을 변화시킬 수 있고 `count` 의 기본값은 0 을 나타냅니다.

<br>

#### 상태값 변경은 batch 로 처리

```js
function increment() {
  setCount(count + 1);
  setCount(count + 1);
}
```

리액트는 상태값 변경을 배치로 처리하기 때문에 위와 같이 `setCount` 를 여러번 사용하여도 한개만 동작합니다.

상태값이 변경될 때마다 렌더링을 하면 비효율적이므로 `increment` 함수를 호출했을 때 실제로 렌더링 되는 횟수는 한 번입니다.

<br>

#### 함수를 파라미터로

```js
function increment() {
  setCount(count => count + 1);
  setCount(count => count + 1);
}
```

상태값 변경 함수의 파라미터로 함수를 넘기면 호출되기 직전의 상태값을 파라미터로 받습니다.

첫번째 `setCount` 에서 `count` 의 값이 변경되었고

두번째 `setCount` 가 호출되면 바로 직전의 `count` 를 매개변수로 받기 때문에

최종적으로 2 가 증가합니다.


<br>

## useEffect

`useEffect` 함수는 컴포넌트가 **실제 돔에 렌더링 되었을 때**와 **렌더링이 끝난 직후**에 호출됩니다.

아래와 같이 컴포넌트를 만들면 Increment 버튼을 누를 때마다 `count` 값이 1 증가하여 새로 렌더링 되고 그때마다 `count is ${count}` 가 콘솔에 출력됩니다.

```js
import React, { useState, useEffect } from 'react';

function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log(`count is ${count}`);
  })

  function increment() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>count: {count}</h1>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default App;
```

<br>

#### 특정 값의 변경에만 반응

컴포넌트에 여러 개의 값이 있을 때, 아무런 상관이 없는 값이 변경 되었을 때에도 매번 렌더링이 새로 된다면 비효율적입니다.

`useEffect` 함수는 특정 값들에 대해서만 반응하게 설정할 수 있습니다.

```js
useEffect(() => {
  console.log(`count is ${count}`);
}, [count])
```

`useEffect` 의 두번째 파라미터로 배열을 넘기면 배열 안의 값들이 변경될 때만 함수가 동작합니다.

만약 빈 배열 `[ ]` 을 입력한다면 최초 렌더링 시에만 동작합니다.

<br>

#### 렌더링이 사라질 때 반응

위에서 설명한 `useEffect` 함수가 렌더링이 되는 순간 동작합니다.

그리고 렌더링이 사라지는 순간에도 동작하게 설정할 수 있습니다.

```js
useEffect(() => {
  console.log(`count is ${count}`);

  return () => console.log(`count is ${count} when disagree`);
}, [count])
```

`return` 으로 넘기는 함수는 렌더링이 사라지는 순간에 동작합니다.

배열에 `count` 가 있기 때문에 `count` 값이 변할 때마다 기존의 렌더링은 끝나고 새롭게 렌더링 됩니다.

그래서 렌더링이 사라지는 순간에 한번, 다시 렌더링 되는 순간에 한번 이렇게 총 두개의 로그가 찍힙니다.

```html
count is 0 when agree
count is 1
```

<br>

## Context API

상위 컴포넌트에서 하위 컴포넌트로 데이터를 전달할 때 보통 `props` 를 사용합니다.

전달할 컴포넌트 간의 거리가 가깝다면 충분히 전달-전달을 할 수 있지만 굉장히 멀리 떨어져있거나 많은 하위 컴포넌트들에게 전달할 때는 `props` 를 반복적으로 내려주어야 합니다.

또, 사용되지 않는 중간 컴포넌트들이 의미 없이 `props` 를 계속 받아야 합니다.

이럴 때 **Context API** 를 사용하면 상위 컴포넌트에서 하위에 있는 모든 컴포넌트에 직접 데이터를 전달 할 수 있습니다.

<br>

#### Context API 를 사용하지 않는 코드

```js
function App() {
  return (
    <div>
      <h1> React </h1>
      <Child comment="hello react" />
    </div>
  )
}

function Child({ comment }) {
  return (
    <div>
      <GrandChild comment={comment} />
    </div>
  )
}

function GrandChild({ comment }) {
  return <p>{ `${comment}, hello world`}</p>
}
```

위 코드는 `App => Child => GrandChild` 순으로 `comment` 값을 전달합니다.

사실 `Child` 컴포넌트에서는 `comment` 를 사용하지 않지만 불필요하게 받아서 하위 컴포넌트에 넘겨주어야 합니다.
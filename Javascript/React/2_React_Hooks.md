# React Hooks

리액트 훅은 16.8 버전부터 추가되었습니다.

함수형 컴포넌트에서 상태값을 사용할 수 있고 자식 요소에 접근할 수 있는게 가장 크며, 리액트 팀에서도 적극적으로 밀고 있습니다.

새로 만드는 컴포넌트는 클래스형보다는 함수형으로 작성하는게 좋습니다.

<br>

## useState

`useState` 함수를 사용하면 함수형 컴포넌트에서 상태값을 사용할 수 있습니다.

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

#### 상태값 변경은 batch 로 처리

```js
function increment() {
  setCount(count + 1);
  setCount(count + 1);
}
```

리액트는 상태값 변경을 배치로 처리하기 때문에 위와 같이 `setCount` 를 여러번 사용하여도 한개만 동작합니다.

상태값이 변경될 때마다 렌더링을 하면 비효율적이므로 `increment` 함수를 호출했을 때 실제로 렌더링 되는 횟수는 한 번입니다.

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


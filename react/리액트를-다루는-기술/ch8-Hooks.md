# 8장 Hooks

Hooks 는 리액트 v16.8 에 새로 도입된 기능으로 함수형 컴포넌트에서도 상태 관리를 하고 라이프 사이클을 관리할 수 있게 해줍니다.

<br>

# 1. useState

```jsx
/* const [상태 변수, Setter 함수] = useState(초기값) */
const [state, setState] = useState(0);
```

가장 기본적인 Hook 이며 함수형 컴포넌트에서도 가변적인 상태를 지닐 수 있게 해줍니다.

함수형 컴포넌트에서 상태 관리를 해주는 Hook 입니다.

`useState` 는 생성자로 초기값을 받으며 리턴값으로 상태 변수와 Setter 함수를 리턴합니다.

상태를 변경할때는 항상 같이 넘겨받은 Setter 함수를 사용해야 합니다.

<br>

```jsx
import { useState } from "react";

function UseStateSample() {
  const [value, setValue] = useState(0);
  const [name, setName] = useState('');

  return (
    <div>
      <p>현재 카운터 값은 <b>{value}</b>입니다.</p>
      <button onClick={() => setValue(value + 1)}>+1</button>
      <button onClick={() => setValue(value - 1)}>-1</button>

      <p>--------------------------------</p>

      <input value={name} onChange={(e) => setName(e.target.value)} />
      <p>이름: <b>{name}</b></p>
    </div>
  )
}

export default UseStateSample;
```

`value` 는 초기값이 0 이고 `name` 은 초기값이 '' 입니다.

button 과 input 에 Setter 메소드를 매핑시켰기 때문에 버튼을 누르거나 input text 에 글자를 입력할 때마다 실시간으로 `value`, `name` 값이 바뀌는 걸 확인할 수 있습니다.

이렇게 `useState` 를 여러개 선언해서 각각의 상태를 관리할 수도 있지만 관련있는 상태는 하나의 객체로 묶어서 사용할 수도 있습니다.

<br>

# 2. useEffect

```jsx
useEffect(fn, [deps])
```

`useEffect` 는 리액트 컴포넌트가 렌더링 될 때마다 특정 작업을 수행하도록 설정할 수 있는 Hook 입니다.

두번째 파라미로 특정 변수를 넣으면, 해당 변수가 변경될 때만 실행되도록 설정할 수 있습니다.

만약 컴포넌트를 처음 렌더링, 즉 마운트할 때만 실행하고 싶다면 빈 배열을 넣어주면 됩니다.

정리하자면 두번째 파라미터에 따라 다음과 같이 동작합니다.
- 빈 배열 `[]` : 처음 마운트 될 때만 실행
- 상태 추가 `[a]` : `a` 라는 상태가 변경될 때마다 실행
- 빈 값 : default 값으로 렌더링 될때마다 항상 실행

<br>

```jsx
import { useEffect, useState } from "react";

export default function UseEffectSample() {
  const [plus, setPlus] = useState(0);
  const [minus, setMinus] = useState(0);

  useEffect(() => {
    console.log('컴포넌트가 처음 생성되었습니다.');
  }, []);


  useEffect(() => {
    console.log('plus 변경');
  }, [plus]);


  useEffect(() => {
    console.log('minus 변경');
  }, [minus]);


  useEffect(() => {
    console.log('plus 또는 minus 변경');
  }, [plus, minus]);

  
  useEffect(() => {
    console.log('컴포넌트가 리렌더링 되었습니다.')
  })

  return (
    <div>
      <p>plus: {plus}, minus: {minus}</p>
      <button onClick={() => setPlus(plus + 1)}>Plus</button>
      <button onClick={() => setMinus(minus + 1)}>Minus</button>
    </div>
  )
}
```

<br>

## 2.1. Cleanup (뒷정리)

`useEffect` 의 또다른 사용법 중 하나는 컴포넌트가 업데이트 되기 직전에 실행하는 뒷정리(cleanup) 함수입니다.

`useEffect` 내부에서 `retrun` 키워드를 사용하면 리렌더링 직전에 함수를 수행합니다.

```jsx
import { useEffect, useState } from "react"

export default function UseEffectCleanupSample() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log('effect: 컴포넌트가 처음 생성됩니다.')
    
    return () => {
      console.log('cleanup: 컴포넌트가 이제 사라집니다.')
    }
  }, []);


  useEffect(() => {
    console.log('렌더링 직후 count: ' + count);

    return () => {
      console.log('렌더링 직전 count: ' + count);
    }
  });

  
  return (
    <div>
      <p>count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  )
}
```

<br>

# 3. useReducer

`useReducer` 는 `useState` 보다 더 다양한 컴포넌트 상황에 따라 다양한 상태를 다른 값으로 업데이트 해주고 싶을 때 사용합니다.

```jsx

// state: 현재 상태
// action: 업데이트를 위해 필요한 정보를 담은 액션
function reducer(state, action) {
  return { ... }; // 불변성을 지키면서 업데이트한 새로운 상태를 반환
}

function Component() {
  const [state, dispatch] = useReducer(reducer, initialArg, init);
  ...
}
```

<br>

액션값의 예시이며 객체가 아니라 문자열이나 숫자여도 상관 없습니다.

```jsx
{
  type: 'INCREMENT',
  ...
}
```

<br>

## 3.1. 카운터 구현하기

```jsx
import { useReducer } from "react";

function reducer(state, action) {
  // action.type 에 따라 다른 작업 수행
  switch (action.type) {
    case 'INCREMENT':
      return { value: state.value + 1 };
    case 'DECREMENT':
      return { value: state.value - 1 };
    default:
      return state;
  }
}

export default function ReducerCounter() {
  const [state, dispatch] = useReducer(reducer, { value: 0 });

  return (
    <div>
      <p>현재 카운터 값은 <b>{state.value}</b>입니다.</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+1</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-1</button>
    </div>
  )
}
```

`useReducer` 를 사용해서 `state` 와 `dispatch` 를 받습니다.

얼핏 보면 `useState` 와 사용법이 비슷한데 `dispatch` 함수는 `action` 을 넘겨줘야 한다는 것이 다릅니다.

참고로 `dispatch` 함수는 리렌더링 시에도 포함되지 않아 `useCallback` 으로 감싸지 않아도 된다고 합니다.

<br>

## 3.2. Input 상태 관리하기

```jsx
import {useReducer} from "react";

function reducer(state, action) {
  return {
    ...state,
    [action.name]: action.value
  };
}

export default function ReducerInfo() {
  const [state, dispatch] = useReducer(reducer, {
    name: '',
    nickname: '',
  });

  const { name, nickname } = state;
  const onChange = (e) => {
    dispatch(e.target)
  };

  return (
    <div>
      <div>
        <input name="name" value={name} onChange={onChange} />
        <input name="nickname" value={nickname} onChange={onChange} />
      </div>
      <div>
        <p>이름: {name} </p>
        <p>닉네임: {nickname} </p>
      </div>
    </div>
  )
}
```

`state` 를 여러개 사용할 때 원래는 `useState` 를 여러 개 호출해서 각각 관리했었습니다.

하지만 상태 갯수가 너무 많아지면 함수도 많아지고 관리하기 불편해질 수 있는데 이럴 때 `useReducer` 를 사용하면 편리하게 관리 가능합니다.

위에서 말했듯이 `action` 값으로는 어떤 것이든 상관 없기 때문에 `e.target` 을 넘겼습니다.

<br>

# 4. useMemo

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

`useMemo` 는 함수형 컴포넌트 내부에서 발생하는 연산을 최적화 할 수 있게 해줍니다.

특정 값을 메모이제이션 하여 기억하고 있습니다.

<br>

예를 들어 다음과 같은 컴포넌트가 있습니다.

```jsx
import { useState } from "react";

export default function UseMemoSample() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  const calculateSquare = (n) => {
    // useMemo 로 감싸지 않으면 text 값이 변할 때도 매번 콘솔에 출력됨
    console.log("제곱 계산");
    return n * n;
  }

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <p>Count 10 제곱 : {calculateSquare(count)}</p>
    </div>
  )
}
```

`calculateSquare` 함수는 `count` 값이 변할 때마다 제곱을 구해서 출력합니다.

컴포넌트는 `state` 값이 변할 때마다 리렌더링 되는 특징이 있습니다.

컴포넌트가 리렌더링 된다는 뜻은 컴포넌트 내부에 있는 함수들도 전부 리렌더링 된다는 뜻입니다.

위 컴포넌트의 문제점은 `calculateSquare` 와 관련 없는 상태값인 `text` 가 바뀔 때마다 함수가 계속 실행된다는 점입니다.

만약 단순한 제곱 계산이 아니라 무거운 연산이었다면 성능적인 문제를 피하기 힘들겁니다.

이 때, `useMemo` 를 사용하여 값을 기억해둔다면 다른 상태값 변화에 영향을 받지 않습니다.

<br>

```jsx
import { useMemo, useState } from "react";

export default function UseMemoSample() {
  //...
  const square = useMemo(() => calculateSquare(count), [count])
  //...

  return (
    //...
      <p>Count 10 제곱 : {square}</p>
    //...
  )
}
```

이렇게 코드를 고치면 `count` 값이 변할 때만 `calculateSquare` 메소드를 실행시켜서 불필요한 계산을 막을 수 있습니다.

<br>

# 5. useCallback

```jsx
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

`useCallback` 은 `useMemo` 와 상당히 유사합니다.

`useMemo` 가 값을 기억한다면 `useCallback` 은 함수를 기억합니다.

`useCallback(fn, deps)` 은 `useMemo(() => fn, deps)` 와 같습니다.

마찬가지로 컴포넌트가 리렌더링 될 때 불필요하게 새로 렌더링 되는 함수들을 감싸서 필요할 때만 새로 렌더링 하게 변경해줍니다.

<br>

# 6. Custom Hooks 만들기

여러 컴포넌트에서 비슷한 기능을 공유할 경우, 이를 Hook 으로 작성해서 로직을 재사용할 수 있습니다.

`useReducer` 예제에서 만들었던 Input 로직을 `useInputs` 라는 Hook 으로 분리해보겠습니다.

```jsx
import {useReducer} from "react";

function reducer(state, action) {
  return {
    ...state,
    [action.name]: action.value
  };
}

export default function useInputs(initialForm) {
  const [state, dispatch] = useReducer(reducer, initialForm);

  const onChange = (e) => {
    dispatch(e.target);
  }

  return [state, onChange];
}
```

<br>

위 훅을 원하는 컴포넌트에서 호출하여 사용하면 됩니다.

코드가 훨씬 깔끔해졌습니다.

```jsx
import useInputs from "./useInputs";

export default function CustomHooksSample() {
  const [state, onChange] = useInputs({
    name: '',
    nickname: ''
  });

  const { name, nickname } = state;

  return (
    <div>
      <div>
        <input name="name" value={name} onChange={onChange} />
        <input name="nickname" value={nickname} onChange={onChange} />
      </div>
      <div>
        <p>이름: {name} </p>
        <p>닉네임: {nickname} </p>
      </div>
    </div>
  )
}
```

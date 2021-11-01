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


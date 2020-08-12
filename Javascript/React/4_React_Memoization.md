# React.memo, useMemo, useCallback - 리액트 메모이제이션

리액트가 실행될 때 CPU 리소스를 가장 많이 사용하는 곳은 렌더링입니다.

리액트는 데이터와 컴포넌트 함수로 화면을 그리는데, 여기서 데이터는 컴포넌트의 `props` 와 `state` 입니다.

`props` 또는 `state` 가 변경되면 리액트가 자동으로 컴포넌트 함수를 이용해서 화면을 다시 그립니다.

최초 렌더링 이후에 데이터 변경 시 리렌더링을 하게 되면 다음과 같은 과정을 거칩니다.

1. 이전 렌더링 결과를 재사용할 지 판단한다.
2. 컴포넌트 함수를 호출한다.
3. 가상 돔끼리 비교해서 변경된 부분만 실제 돔에 반영한다.

첫 번째 단계에서는 속성값과 상태값의 이전 이후 값을 비교하고, 변경이 없다고 판단된다면 이후 단계를 생략할 수 있습니다.

대부분의 웹 페이지는 성능을 고민하지 않아도 문제없이 잘 돌아가기 때문에 성능 이슈가 생기면 최적화 할 수 있는 방법을 알아봅니다.

<br>

## React.memo 로 렌더링 결과 재사용하기

컴포넌트의 속성값이나 상태값이 변경되면 리액트는 그 컴포넌트를 다시 그릴 준비를 합니다.

만약 `React.memo` 함수로 감싼 컴포넌트라면 `props` 비교 함수가 호출됩니다.

이 함수는 이전 이후 `props` 매개 변수로 받아서 참 또는 거짓을 반환한다.

참을 반환하면 렌더링을 멈추고, 거짓을 반환하면 컴포넌트 함수를 실행해서 가상 돔을 업데이트 후 변경된 부분만 실제 돔에 반영합니다.

`React.memo` 를 사용하지 않아도 `props` 값이 변경되지 않으면 실제 돔도 변경되지 않기 때문에 대부분 문제가 되지 않으므로 정말 필요한 경우에만 사용합니다.

```js
// React.memo 사용 예
function MyComponent(props) {
  // ...
}

function isEqual(prevProps, nextProps) {
  // true 또는 false 를 반환
}

React.memo(MyComponent, isEqual);
```

<br>

## 함수의 값이 변하지 않도록 관리하기

부모 컴포넌트에서 정의된 함수는 리렌더링 될 때 재정의 되기 때문에 내부 로직이 같아도 다른 함수로 인식됩니다.

```js
// App.js
import React, { useState } from 'react';
import Name from './Name';

function App() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("kakao");

  return (
    <div>
      <h1>{ count }</h1>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <Name name={ name } onChange={ (newName) => setName(newName) } />
    </div>
  )
}

export default App;


// Name.js
import React, { useEffect } from 'react';

function Name({ name, onChange }) {
  useEffect(() => {
    console.log("start Name useEffect");
    console.log(name);
    console.log(onChange);
  })

  return (
    <div>
      <h1>{ name }</h1>
    </div>
  )
}

export default React.memo(Name);
```

`Name` 컴포넌트에서 `React.memo` 로 감싸주었고, `props` 값이 변하지 않았기 때문에 리렌더링이 발생하지 않는다고 생각할 수 있습니다.

그러나 `onChange` 속성값은 항상 새로운 함수를 생성하기 때문에 `Name` 컴포넌트에서는 `props` 값이 변했다고 인식합니다.

이 문제를 해결하기 위해 `useState`, `useReducer` 의 상태값 변경 함수인 `setState`, `dispatch` 함수는 불변이라는 점을 이용하면 됩니다.

코드를 다음과 같이 변경하면 `onChange` 속성값에는 항상 같은 값이 입력됩니다.

```js
// before
<Name name={ name } onChange={ (newName) => setName(newName) } />

// after
<Name name={ name } onChange={ setName } />
```

<br>

만약 `onChange` 에서 `setName` 뿐만 아니라 다른 로직도 필요하다면 `useCallback` 을 사용할 수 있다.

`deps` 로 빈 배열을 입력했으므로 이 함수는 항상 고정된 값을 가진다.

```js
const onChangeName = useCallback(name => {
    setName(name);
    console.log("on change name");
  }, [])

<Name name={ name } onChange={ onChangeName } />
```

<br>

## 객체의 값이 변하지 않도록 관리하기

함수와 마찬가지로 객체도 내부의 값이 변하지 않았는데도 속성값이 변경되었다고 인식할 수 있습니다.

이를 해결하기 위해선 두가지 방법이 있습니다.

1. 컴포넌트 밖에서 상수 변수로 관리
2. `useMemo` 사용

변하지 않는 값이거나 속성값, 상태값과 무관한 값이라면 컴포넌트 외부에 상수로 선언하면 됩니다.

```js
// befoer
function App() {
  return (
    <div>
      <Select options={[
        { name: "apple", price: 500 },
        { name: "banana", price: 1000 }
      ]} />
    </div>
  )
}

// after
function App() {
  return (
    <div>
      <Select options={FRUITS} />
    </div>
  )
}

const FRUITS = [
  { name: "apple", price: 500 },
  { name: "banana", price: 1000 }
]
```

<br>

만약 속성값이나 상태값을 이용해야 하는 경우 `useMemo` 훅을 사용한 메모이제이션 기법을 사용할 수 있습니다.

```js
const memoSum = useMemo(() => sum(a, b), [a, b]);
```

<br>

# Reference
- [실전 리액트 프로그래밍 (개정판)](http://www.yes24.com/Product/Goods/90873270?OzSrank=2)

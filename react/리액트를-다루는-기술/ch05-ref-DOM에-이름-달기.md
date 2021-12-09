# 5장 ref: DOM 에 이름 달기와 로컬 변수

- [5장 ref: DOM 에 이름 달기와 로컬 변수](#5장-ref-dom-에-이름-달기와-로컬-변수)
- [1. ref 는 어떤 상황에서 사용해야 할까?](#1-ref-는-어떤-상황에서-사용해야-할까)
- [2. ref 로 DOM 지정](#2-ref-로-dom-지정)
- [3. forwardRef: 자식 컴포넌트의 요소 접근](#3-forwardref-자식-컴포넌트의-요소-접근)
  - [3.1. useImperativeHandle: 자식 컴포넌트의 메소드 접근](#31-useimperativehandle-자식-컴포넌트의-메소드-접근)
- [4. useRef 의 또다른 사용법 로컬 변수](#4-useref-의-또다른-사용법-로컬-변수)
- [5. 정리](#5-정리)

# 1. ref 는 어떤 상황에서 사용해야 할까?

HTML 에서 DOM 요소에 이름을 달 때는 `id` 를 사용합니다.

하지만 리액트 컴포넌트에서는 `id` 대신 `ref` (reference) 를 사용합니다.

JSX 안에서 DOM 에 `id` 를 달면 컴포넌트를 재활용할 때 `id` 값이 겹칩니다.

`ref` 는 전역적으로 동작하지 않고 컴포넌트 내부에서만 동작하기 때문에 이런 문제가 발생하지 않습니다.

`ref` 는 **DOM 을 꼭 직접적으로 건드려야 할 때** 사용합니다.

<br>

# 2. ref 로 DOM 지정

원래 책에서는 클래스형 컴포넌트로 예제를 만들었으나 여기서는 함수형 컴포넌트로 작성하고 `useRef` 를 사용합니다.

<br>

```jsx
import { useRef } from "react"

const RefSample = () => {
  const inputRef = useRef();

  const handleFocus = () => {
    inputRef.current.focus();
  }

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={handleFocus}>Focus the input</button>
    </div>
  )
}

export default RefSample
```

`useRef` 를 사용해서 멤버 변수로 `inputRef` 를 선언합니다.

지정하고자 하는 요소에 `ref` 속성으로 해당 변수를 넣어주고 `.current` 로 접근하면 됩니다.

<br>

# 3. forwardRef: 자식 컴포넌트의 요소 접근

부모 컴포넌트에서 자식 컴포넌트에 접근하는 것은 컴포넌트의 캡슐화를 파괴하기 때문에 권장되지는 않습니다.

하지만 때때로 직접 접근해야할 상황이 존재합니다.

이럴 때 `forwardRef` 를 사용하면 자식 컴포넌트의 DOM 요소에 접근 가능합니다.

<br>

```jsx
import React, { forwardRef, useRef } from 'react';

/* 자식 컴포넌트: forwardRef 로 ref 를 넘겨받아 DOM 요소에 지정 */
const InputComponent = forwardRef((props, ref) => {
  return <input type="text" ref={ref} />
})

/* 부모 컴포넌트: useRef 로 ref 를 만들어서 자식 컴포넌트에 넘겨줌 */
function App() {
  const inputRef = useRef()

  return (
    <div>
      <InputComponent ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>
        Focus
      </button>
    </div>
  )
}

export default App;
```

사용방법은 크게 어렵지 않습니다.

자식 컴포넌트를 만들 때 `forwardRef` 로 만들어서 `(props, ref)` 를 파라미터로 받은 후 `ref` 를 내부 요소에 매칭시켜줍니다.

그리고 부모 컴포넌트에서 `useRef` 로 변수 하나를 생성 후 자식 컴포넌트의 `ref` 속성으로 내려주면 됩니다.

<br>

## 3.1. useImperativeHandle: 자식 컴포넌트의 메소드 접근

`forwardRef` 의 또다른 기능 중 하나는 자식 컴포넌트에 존재하는 메소드도 부모 컴포넌트에서 사용 가능하다는 점입니다.

리액트는 기본적으로 부모 -> 자식 으로 데이터가 흐르는 것을 거슬러서 자식에서 부모로 메소드를 전달해서 사용할 수 있게 합니다.

<br>

```jsx
import React, { forwardRef, useImperativeHandle, useRef } from 'react';

/* 자식 컴포넌트: forwardRef 로 ref 를 넘겨받아 useImperativeHandle 로 메소드 설정 */
/* useImperativeHandle(ref, createHandle, [deps]) */
const InputComponent = forwardRef((props, ref) => {

  useImperativeHandle(ref, () => ({
    alertWarning: () => {
      alert('warning!')
    }
  }))

  return <input type="text" />
})

/* 부모 컴포넌트: useRef 로 ref 를 만들어서 자식 컴포넌트에 넘겨줌 */
function App() {
  const childRef = useRef()

  return (
    <div>
      <InputComponent ref={childRef} />
      <button onClick={() => childRef.current.alertWarning()}>
        Warning
      </button>
    </div>
  )
}

export default App;
```

사용법은 크게 어렵지 않으며 부모 컴포넌트로부터 넘겨받은 `ref` 에 `useImperativeHandle` 를 사용해서 메소드를 지정해주면 됩니다.

그리고 부모 컴포넌트는 일반적인 사용법과 마찬가지로 `.current` 로 해당 메소드에 접근할 수 있습니다.

메소드를 여러개 넘길 수도 있습니다.

<br>

# 4. useRef 의 또다른 사용법 로컬 변수

기본적으로 `ref` 는 특정 DOM 을 조작해야 할 때 사용합니다.

그리고 함수형 컴포넌트에서는 `ref` 를 편하게 사용하기 위해 `useRef` 라는 Hook 을 사용합니다.

하지만 `useRef` 는 또다른 용도로 사용할 수 있는데요, 컴포넌트 내부의 변수를 관리할 때 사용할 수 있습니다.

일반적으로 컴포넌트 내부의 변수는 `state` 를 사용합니다.

하지만 두 변수의 가장 큰 차이점은 **`state` 는 변경될 때마다 컴포넌트가 리렌더링 되지만 `useRef` 변수는 변경되어도 컴포넌트가 리렌더링 되지 않습니다.**

<br>

```jsx
import { useRef, useState } from "react"

function RefCounter() {
  const [state, setState] = useState(0);
  const ref = useRef(0);

  console.log("rendering")

  return (
    <div>
      <h1>state: {state}, ref: {ref.current}</h1>
      <button onClick={() => setState(state + 1)}>Plus State</button>
      <button onClick={() => ref.current += 1}>Plus Ref</button>
    </div>
  )
}

export default RefCounter;
```

테스트를 위한 코드를 작성했고 기능은 다음과 같습니다.

- 초기값이 0 인 `state`, `ref` 변수 존재
- 컴포넌트가 렌더링 될때마다 브라우저 콘솔에 "rendering" 출력
- "Plus State" 버튼을 누르면 `state` 값 1 증가
- "Plus Ref" 버튼을 누르면 `ref` 값 1 증가

<br>

위 코드를 브라우저에서 확인해보면 "Plus State" 버튼을 누를 때마다 `state` 값이 1 씩 올라가고 콘솔에 "rendering" 이 출력되는 걸 볼 수 있습니다.

그러나 "Plus Ref" 버튼을 눌러도 페이지는 아무 변화가 없습니다.

하지만 "Plus Ref" 버튼을 누르다가 "Plus State" 을 눌러서 컴포넌트를 리렌더링 하게 되면 지금까지 "Plus Ref" 버튼을 누른 만큼 `ref` 값이 증가한 것을 확인할 수 있습니다.

<br>

위 테스트를 통해 우리는 한가지 사실을 알 수 있었습니다.

**`useRef` 로 선언된 변수는 컴포넌트의 렌더링에 영향을 주지도 받지도 않는다**

렌더링과 관계 없는 로컬 변수가 필요할 때 사용하면 됩니다.

<br>

# 5. 정리

지금까지 useRef 의 사용법과 응용에 대해서 알아봤습니다.

사실 책에서는 클래스형 컴포넌트를 기준으로 설명하기 때문에 위 내용과는 조금 다른 부분도 있고 로컬 변수 사용 내용도 없습니다.

# 6장 컴포넌트 반복

# 1. 자바스크립트 배열의 map() 함수

자바스크립트 배열 객체의 내장 함수인 `map` 함수를 사용하면 반복되는 컴포넌트를 렌더링 할 수 있습니다.

`map` 함수는 파라미터로 전달된 함수를 사용해서 배열 내 각 요소를 원하는 규칙에 따라 변환한 후 그 결과로 새로운 배열을 생성합니다.

<br>

## 1.1. 문법

```js
arr.map(callback, [thisArg])
```

- `callback` : 새로운 배열의 요소를 생성하는 함수로 파라미터는 세가지를 갖고 있음
  - `currentValue` : 현재 처리하고 있는 요소
  - `index` : 현재 처리하고 있는 요소의 index 값
  - `array` : 현재 처리하고 있는 원본 배열
- `thisArg` (optional) : callback 함수 내부에서 사용할 this 레퍼런스


<br>

## 1.2. 예제

```js
var numbers = [1, 2, 3, 4, 5];

var processed = numbers.map(function(num) {
    return num * num;
})

console.log(processed);  // [1, 4, 9, 16, 25]


// ES6 버전
const numbers = [1, 2, 3, 4, 5];
const result = numbers.map(num => num * num);
console.log(result);  // [1, 4, 9, 16, 25]
```

위 코드는 `numbers` 배열의 각 요소를 제곱한 새로운 배열 `processed` 를 생성하는 코드입니다.

<br>

# 2. 데이터 배열을 컴포넌트 배열로 변환

위에서 배열에 있는 값들을 활용해서 새로운 배열을 만든 것처럼 컴포넌트도 똑같이 만들 수 있습니다.

다음의 반복되는 코드를 컴포넌트를 한번 수정해봅니다.

```jsx
const IterationSample = () => {
  return (
    <ul>
      <li>눈사람</li>
      <li>얼음</li>
      <li>눈</li>
      <li>바람</li>
    </ul>
  );
};

export default IterationSample;
```

<br>

## 2.1. 컴포넌트 수정하기

```jsx
const IterationSample = () => {
  const names = ['눈사람', '얼음', '눈', '바람'];
  const nameList = names.map(name => <li>{name}</li>);
  return <ul>{nameList}</ul>
};

export default IterationSample;
```

문자열로 구성된 배열을 선언하고 `map` 함수를 사용해서 `<li></li>` 태그의 JSX 코드로 변환합니다.

변환 결과는 DOM 요소를 사용해도 되고 컴포넌트를 사용해도 됩니다.

<br>

## 2.2. App 컴포넌트에서 예제 컴포넌트 렌더링

```jsx
import IterationSample from './IterationSample';

function App() {
  return <IterationSample />
}

export default App;
```

`App` 컴포넌트에서 `IterationSample` 컴포넌트를 불러와서 렌더링하면 됩니다.

`localhost:3000` 에 접속하면 정상적인 화면을 볼 수 있습니다.

<br>

그러나 개발자 도구의 콘솔을 보면 다음과 같은 에러가 있습니다.

`key` prop 이 없다는 뜻인데 `key` 가 무엇일까요?

```sh
index.js:1 Warning: Each child in a list should have a unique "key" prop.

Check the render method of `IterationSample`. See https://reactjs.org/link/warning-keys for more information.
    at li
    at IterationSample
    at App
```

<br>

## 3. key

리액트에서 key 는 컴포넌트 배열을 렌더링 했을 때 어떤 원소에 변동이 있었는지 알아내려고 사용합니다.

예를 들어 유동적인 데이터를 다룰 때는 원소를 생성, 제거, 수정이 가능합니다.

key 가 없을 때는 Virtual DOM 을 비교하는 과정에서 리스트를 순차적으로 비교하면서 변화를 감지합니다.

하지만 key 가 있다면 이 값을 사용하여 어떤 변화가 일어났는지 더욱 빠르게 알아낼 수 있습니다.

<br>

## 3.1. 재조정 (Reconciliation)

key 속성의 필요성에 대해 이해하기 위해서는 리액트의 [재조정 (Reconciliation)](https://ko.reactjs.org/docs/reconciliation.html) 에 대해 먼저 알아야 합니다.

리액트는 `state`, `props`, `element` 등이 변경되었을 때 새롭게 UI 를 갱신합니다.

렌더링 페이지의 바뀐 점을 확인하기 위해 기존 트리와 새로운 트리 (Virtual DOM) 의 루트 (root) 엘리먼트부터 비교를 시작합니다.

여기서 루트 엘리먼트가 같은지 아닌지에 따라 동작이 달라집니다.

<br>

**3.1.1. 엘리먼트 타입이 다른 경우**

```jsx
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

위 코드처럼 루트 엘리먼트가 다른 경우 (`div`, `span`) 에는 자식 노드를 전부 교체합니다.

<br>

**3.1.2. 엘리먼트 타입이 같은 경우**

```jsx
// 1. className 만 바뀜
<div className="before" title="stuff" />

<div className="after" title="stuff" />


// 2. color 속성만 바뀜
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

루트 엘리먼트가 같은 경우에는 변경된 속성만 갱신합니다.

이후에는 같은 방식으로 자식 노드들을 재귀적으로 모두 처리합니다.

<br>

## 3.2. 그렇다면 key 의 필요성은?

리액트가 DOM 노드들을 처리할 때는 기본적으로 동시에 두 리스트 (기존 트리, 새로운 트리) 를 순회하면서 차이점이 있으면 변경합니다.

이 순서는 위에서부터 시작하므로 변경된 부분이 어디에 위치하느냐에 따라 렌더링 성능이 달라집니다.

<br>

**3.2.1. 변경이 마지막에 발생했을 때**

```jsx
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

위 코드처럼 마지막에 새로운 요소가 추가되면 리액트는 `first`, `second` 를 먼저 비교해서 일치하는 걸 확인하고 마지막에 `third` 를 추가합니다.

<br>

**3.2.2. 변경이 처음 또는 중간에 발생했을 때**

```jsx
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

새로 추가된 요소가 처음에 위치합니다.

리액트는 첫 요소인 `Duke` 와 `Connecticut` 부터 비교를 시작합니다.

만약 불일치하면 이후 요소들을 전부 교체해버립니다.

<br>

## 3.3. key 속성을 사용해서 비교

```jsx
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

위에서 나온 문제점을 해결하기 위해 리액트는 `key` 속성을 지원합니다.

비교하려는 요소들이 key 를 갖고 있다면, 리액트는 key 값을 통해 기존 트리와 새로운 트리의 요소들이 일치하는지 확인합니다.

따라서 전부 교체하는 대신 변경되거나 새로 추가된 부분만 반영되어 성능 문제를 해결할 수 있습니다.

<br>

## 3.4. key 설정

```jsx
const articleList = articles.map(article => (
  <Article 
    title={article.title}
    writer={article.writer}
    key={article.id}
  />
));
```

key 값은 컴포넌트의 props 를 설정하듯이 지정하면 됩니다.

key 값은 언제나 유일해야 하기 때문에 데이터의 고유값으로 지정해야 합니다.

<br>

```jsx
const IterationSample = () => {
  const names = ['눈사람', '얼음', '눈', '바람'];
  const nameList = names.map((name, index) => <li key={index}>{name}</li>);
  return <ul>{nameList}</ul>
};

export default IterationSample;
```

이 예제에서 사용하는 배열에는 `id` 값이 없으므로 배열의 `index` 를 사용해서 지정합니다.

하지만 배열이 변경되는 경우 `index` 값이 달라질 수 있기 때문에 일반적으로는 사용하지 않는 것이 좋습니다.

<br>

# 4. 응용

이제 위에서 배운 내용들로 실습을 해봅니다.

실습은 아래 순서로 진행됩니다.

1. 초기 상태 설정하기
2. 데이터 추가 기능 구현하기
3. 데이터 제거 기능 구현하기

<br>

## 4.1. 초기 상태 설정하기

```jsx
import { useState } from 'react';

const IterationSample = () => {
  const [names, setNames] = useState([
    { id: 1, text: '눈사람' },
    { id: 2, text: '얼음' },
    { id: 3, text: '눈' },
    { id: 4, text: '바람' },
  ]);
  const [inputText, setInputText] = useState('');
  const [nextId, setNextId] = useState(5);  // 새로운 항목을 추가할 때 사용할 id

  const namesList = names.map((name) => <li key={name.id}>{name.text}</li>);
  return <ul>{namesList}</ul>
};

export default IterationSample;
```

- `names`
  - 이름 목록입니다.
  - `index` 대신 사용할 `id` 필드도 추가합니다.
- `inputText`
  - 새로운 이름을 추가할 때 사용할 입력 필드 값입니다.
- `nextId`
  - 새로 추가할 이름에 붙일 다음 `id` 값입니다.
  - 기존에 4 개의 이름이 있기 때문에 5 부터 시작합니다.

<br>

## 4.2. 데이터 추가 기능 구현하기

```jsx
import { useState } from 'react';

const IterationSample = () => {
  const [names, setNames] = useState([
    { id: 1, text: '눈사람' },
    { id: 2, text: '얼음' },
    { id: 3, text: '눈' },
    { id: 4, text: '바람' },
  ]);
  const [inputText, setInputText] = useState('');
  const [nextId, setNextId] = useState(5);  // 새로운 항목을 추가할 때 사용할 id

  const onChange = e => setInputText(e.target.value);
  const onClick = () => {
    const nextNames = names.concat({  // push: 기존 배열을 변경, concat: 새로운 배열 생성
      id: nextId, // nextId 값을 id 로 설정
      text: inputText
    });
    setNextId(nextId + 1);  // nextId 에 1을 더함
    setNames(nextNames);    // names 값을 업데이트
    setInputText('');       // inputText 를 비움
  };

  const namesList = names.map((name) => <li key={name.id}>{name.text}</li>);
  return (
    <div>
      <input value={inputText} onChange={onChange} />
      <button onClick={onClick}>추가</button>
      <ul>{namesList}</ul>
    </div>
  );
};

export default IterationSample;
```

- `onChange`
  - `input` 태그에 있는 값이 바뀔 때마다 `setInputText()` 를 호출합니다.
- `onClick`
  - "추가" 버튼을 누르면 `inputText` 에 있는 값으로 새로운 이름 객체를 추가합니다.
  - 새로 만든 이름 객체는 `names` 배열에 추가되는데, 이때 기존 배열을 변경하지 않고 새로운 배열을 만드는 방법을 사용합니다.
  - 이렇게 하는 이유는 불변성을 유지해줘야 리액트 컴포넌트의 성능을 최적화 할 수 있기 때문입니다. 자세한 내용은 추후에 알아봅니다.

<br>

## 4.3. 데이터 제거 기능 구현하기

```jsx
import { useState } from 'react';

const IterationSample = () => {
  const [names, setNames] = useState([
    { id: 1, text: '눈사람' },
    { id: 2, text: '얼음' },
    { id: 3, text: '눈' },
    { id: 4, text: '바람' },
  ]);
  const [inputText, setInputText] = useState('');
  const [nextId, setNextId] = useState(5);  // 새로운 항목을 추가할 때 사용할 id

  const onChange = e => setInputText(e.target.value);
  const onClick = () => {
    const nextNames = names.concat({  // push: 기존 배열을 변경, concat: 새로운 배열 생성
      id: nextId, // nextId 값을 id 로 설정
      text: inputText
    });
    setNextId(nextId + 1);  // nextId 에 1을 더함
    setNames(nextNames);    // names 값을 업데이트
    setInputText('');       // inputText 를 비움
  };

  const onRemove = id => {
    const nextNames = names.filter(name => name.id !== id);
    setNames(nextNames);
  };

  const namesList = names.map((name) => (
    <li key={name.id} onDoubleClick={() => onRemove(name.id)}>
      {name.text}
    </li>
  ));
  return (
    <div>
      <input value={inputText} onChange={onChange} />
      <button onClick={onClick}>추가</button>
      <ul>{namesList}</ul>
    </div>
  );
};

export default IterationSample;
```

- `onRemove`
  - 선택된 `id` 값의 이름을 제거하는 기능입니다.
  - 위에서 말한것과 마찬가지로 기존 배열 자체를 수정하는 대신 `filter` 로 새로운 배열을 만듭니다.
- `onDoubleClick`
  - HTML 요소를 더블클릭할 때 사용하는 이벤트 이름입니다.

<br>

# 5. 정리

- 반복되는 데이터 (배열) 은 `map` 함수를 사용해서 DOM, 컴포넌트로 변환할 수 있다.
- key 값을 설정해야 성능이 향상된다.
- key 값은 유일해야 한다. (중복되면 렌더링 할 때 오류 발생)
- 상태(state) 를 변경할 때는 배열을 직접 수정하지 말고 concat, filter 등을 사용하여 새로운 배열을 만들어야 한다. (불변성)
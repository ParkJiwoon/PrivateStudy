# 12장 immer 를 사용하여 더 쉽게 불변성 유지하기

# Overview

11 장에서 잠깐 왜 불변성을 유지하면서 상태를 업데이트 하는것이 중요한지 배웠습니다.

간단한 배열이나 객체면 쉽게 값을 복사하거나 덮어쓸 수 있지만, 복잡한 구조를 갖게 되면 불변성을 유지하기가 쉽지 않습니다.

이럴 때 `immer` 라는 라이브러리를 사용하면 구조가 복잡한 객체도 매우 쉽고 짧게 업데이트 가능합니다.

<br>

## 1. 설치

```sh
$ yarn add immer
```

<br>

## 2. 사용법

```jsx
import produce from "immer";
const nextState = produce(originalState, draft => {
  // 구조가 복잡한 객체도 쉽게 바꿈
  draft.somewhere.deep.inside = 5;
})
```

`produce` 함수는 두가지 파라미터를 받습니다.

첫 번째는 수정 대상, 두 번째 파라미터는 상태를 어떻게 업데이트 할 지 정의하는 함수입니다.

위 코드처럼 함수를 변경하면 `produce` 함수가 불변성을 대신 유지해주면서 새로운 상태를 만들어줍니다.

다음은 좀더 복잡한 예시입니다.

```jsx
import produce from "immer";

const originalState = [
  {
    id: 1,
    todo: "hello",
    checked: true,
  },
  {
    id: 2,
    todo: "world",
    checked: false,
  }
];

const nextState = produce(originalState, draft => {
  const todo = draft.find(t => t.id === 2);
  todo.checked = true;

  // 배열에 새로운 데이터 추가
  draft.push({
    id: 3,
    todo: "react",
    checked: false,
  });

  // id 가 1 인 항목 제거
  draft.splice(draft.findIndex(t => t.id === 1), 1);
});
```

<br>

# 3. Immer 를 적용한 App 컴포넌트 생성

```jsx
import { useCallback, useRef, useState } from "react";
import produce from "immer";

export default function ImmerApp() {
  const nextId = useRef(1);
  const [form, setForm] = useState({ name: '', username: '' });
  const [data, setData] = useState({
    array: [],
    uselessValue: null
  });

  const onChange = useCallback(e => {
    const { name, value } = e.target;

    setForm(
      produce(form, draft => {
        draft[name] = value;
      })
    );
  }, [form]);

  const onSubmit = useCallback(e => {
    e.preventDefault();

    const info = {
      id: nextId.current,
      name: form.name,
      username: form.username
    };

    // array 에 새 항목 등록
    setData(
      produce(data, draft => {
        draft.array.push(info);
      })
    );

    // form 초기화
    setForm({
      name: '',
      username: ''
    });

    nextId.current += 1;
  }, [form.name, form.username]);

  const onRemove = useCallback(id => {
    setData(
      produce(draft => {
        draft.array.slice(draft.array.findIndex(info => info.id === id), 1);
      })
    );
  }, []);

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input name="username" placeholder="아이디" value={form.username} onChange={onChange} />
        <input name="name" placeholder="이름" value={form.name} onChange={onChange} />
        <button type="submit">등록</button>
      </form>
      <div>
        <ul>
          {data.array.map(info => (
            <li key={info.id} onClick={() => onRemove(info.id)}>
              {info.username} ({info.name})
            </li>
          ))}
        </ul>
      </div>
    </div>
  )
}
```


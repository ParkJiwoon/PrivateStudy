# useEffect 실전 활용법

```js
useEffect(fn, deps);
```

`useEffect` 의 기본 문법은 위와 같습니다.

`deps` 는 `fn` 을 호출하기 위해 비교하는 배열인데, 이 배열을 관리하는 방법을 알아봅니다.

대부분의 경우에는 `deps` 배열을 사용하지 않는 것이 좋습니다.

<br>

## useEffect 에서 API 를 호출하기

```js
function Profile({ userId }) {
  const [user, setUser] = useState();

  useEffect(() => {
    fetchUser(userId).then(data => setUser(data));
  })
}
```

위 코드는 컴포넌트가 렌더링 될 때마다 `fetchUser` 를 호출하므로 비효율적입니다.

`userId` 가 변경될 때마다 호출하도록 변경하기 위해 `deps` 에 `userId` 를 추가합니다.

<br>

```js
useEffect(() => {
  fetchUser(userId).then(data => setUser(data));
}, [userId])
```

하지만 `userId` 외에 다른 변수가 `useEffect` 내에서 추가된다면 의존성 배열에 추가해줘야합니다.

한두개면 상관없지만 `useEffect` 가 많아지면 관리할 배열이 많아지고 사람의 실수를 유발합니다.

의존성 배열을 검사를 위해 eslint 에서 **exhaustive-deps** 규칙을 제공해 주기도 합니다.

<br>

## 비동기 함수 호출하기

`useEffect` 의 첫번째 파라미터로 비동기 함수를 넣으면 에러가 발생합니다.

함수를 받아야 하는데 `async await` 는 프로미스를 리턴하기 때문입니다.

비동기 함수를 사용하려면 이펙트 내부에 선언하고 바로 호출하는 방법으로 해야 합니다.

```js
useEffect(() => {
  async function fetchAndSetUser() {
    const data = await fetchUser(userId);
    setUser(data);
  }

  fetchAndSetUser();
}, [userId])
```

<br>

## useCallback 으로 함수 재사용하기

```js
function Profile() {
  const [count, setCount] = useState(0);

  async function fetchAndSetUser() {
    console.log("fetch and set user");
  }

  useEffect(() => {
    fetchAndSetUser();
  }, [fetchAndSetUser])

  return (
    <div>
      <h1>This is Profile component</h1>
      <h1>{ count }</h1>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  )
}
```

`fetchAndSetUser` 함수를 `useEffect` 외부에서도 써야 한다면 함수 선언을 외부에 하면 됩니다.

`deps` 에는 훅 내부에서 사용되는 값들이 들어가야 하기 때문에 `fetchAndSetUser` 을 추가합니다.

그런데 리액트 컴포넌트는 렌더링 할 때마다 함수를 새로 정의합니다.

따라서 `fetchAndSetUser` 메소드의 내부 로직은 같아도 함수 자체는 다른 것으로 인식되어 `useEffect` 가 계속 호출됩니다.

`count` 를 증가시키는 버튼을 누를 때마다 관계 없는 `fetchAndSetUser` 함수가 호출되어 콘솔에 찍힙니다.

`useCallback` 훅을 이용하면 메모이제이션 기법으로 함수를 저장하여 변경되지 않게 합니다.

<br>

```js
function Profile({ userId }) {
  const [count, setCount] = useState(0);

  const fetchAndSetUser = useCallback(
    async function fetchAndSetUser() {
      console.log("fetch and set user");
    },
    [userId]
  )

  useEffect(() => {
    fetchAndSetUser();
  }, [fetchAndSetUser])

  return (
    <div>
      <h1>This is Profile component</h1>
      <h1>{ userId }</h1>
      <h1>{ count }</h1>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  )
}
```

이제 `fetchAndSetUser` 함수는 `userId` 가 변경된 경우에만 새로 호출됩니다.

<br>

# deps 를 없애는 방법

앞에서도 언급했지만 `deps` 는 가능하면 사용하지 않는게 좋습니다.

`deps` 를 관리하는 데 생각보다 시간과 노력이 많이 들기 때문입니다.

<br>

## 1. 이펙트 함수 내에서 분기 처리하기

```js
useEffect(() => {
  if (!user || user.id !== userId) {
    fetchAndSetUser();
  }
})
```

`deps` 를 입력하지 않는 대신에 이펙트 함수 내에서 실행 시점을 조절할 수 있습니다.

<br>

## 2. useReducer 활용하기

여러 상태값을 참조하면서 값을 변경할 때는 `useReducer` 훅을 사용하는게 좋습니다.

```js
function Timer({ initialTotalSeconds }) {
  const [hour, setHour] = useState(Math.floor(initialTotalSeconds / 3600));
  const [minute, setMinute] = useState(Math.floor((initialTotalSeconds % 3600) / 60));
  const [second, setSecond] = useState(initialTotalSeconds % 60);

  useEffect(() => {
    console.log("start use Effect");

    const id = setInterval(() => {
      if (second) {
        setSecond(second - 1);
      } else if (minute) {
        setMinute(minute - 1);
        setSecond(59);
      } else if (hour) {
        setHour(hour - 1);
        setMinute(59);
        setSecond(59);
      }
    }, 1000)

    return () => clearInterval(id);
  }, [hour, minute, second])

  return (
    <div>
      <h1>{ hour } { minute } { second }</h1>
    </div>
  )
}
```

`hour`, `minute`, `second` 세 가지 상태값을 사용하고 1 초마다 타이머의 시간을 차감하는 컴포넌입니다.

렌더링 시마다 클린업 되고 다시 호출되어 `clearInterval`, `setInterval` 이 1초마다 반복해서 호출됩니다.

`useReducer` 를 사용하면 한번만 호출하게 변경할 수 있습니다.

<br>

```js
function Timer({ initialTotalSeconds }) {
  const [state, dispatch] = useReducer(reducer, {
    hour: Math.floor(initialTotalSeconds / 3600),
    minute: Math.floor((initialTotalSeconds % 3600) / 60),
    second: initialTotalSeconds % 60
  });

  const { hour, minute, second } = state;

  useEffect(() => {
    console.log("start use Effect");
    const id = setInterval(dispatch, 1000);
    return () => clearInterval(id);
  }, [dispatch]); 

  return (
    <div>
      <h1>{ hour } { minute } { second }</h1>
    </div>
  )
}

function reducer(state) {
  const { hour, minute, second } = state;
  
  if (second) {
    return { ...state, second: second - 1 };
  } else if (minute) {
    return { ...state, minute: minute - 1, second: 59 };
  } else if (hour) {
    return { hour: hour - 1, minute: 59, second: 59 };
  } else {
    return state;
  }
}
```

리액트는 `dispatch`, `setState`, `useRef` 컨테이너 값은 항상 고정되어 있다는 것을 보장합니다.

따라서 `deps` 에서 `dispatch` 를 빼도 상관 없지만 명시한다고 해도 나쁠 것은 없습니다.

`useReducer` 를 사용하면 다양한 액션과 상태값을 관리하기 용이하고, 상태값 변경 로직을 여러 곳에서 재사용하기에도 좋습니다.

<br>

# Reference
- [[번역] useEffect 완벽 가이드](https://rinae.dev/posts/a-complete-guide-to-useeffect-ko#tldr-too-long-didnt-read---%EC%9A%94%EC%95%BD)
- [실전 리액트 프로그래밍 (개정판)](http://www.yes24.com/Product/Goods/90873270?OzSrank=2)

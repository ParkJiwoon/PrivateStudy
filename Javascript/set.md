# Set

JavaScript 에서 `Set` 자료 구조는 ES6 에서 추가되었다.

`Set` 은 중복을 허용하지 않고 순서가 없는 리스트다.

<br>

## 생성자 Constructor

```jsx
const a = new Set()
// Set { }

const b = new Set([1, 2, 3])
// Set { 1, 2, 3 }

const c = new Set([1, 1, 1])
// Set { 1 }
```

<br>

## add

값을 추가한다.

값을 추가할 때 `Object.is()` 메서드를 사용해서 값을 비교한다.

```jsx
const a = new Set()
// Set { }

a.add(1)
// Set { 1 }

a.add(2)
// Set { 1, 2 }

a.add(1)
// Set { 1, 2 }
```

<br>

## size

크기를 알려준다.

`size()` 가 아니라 `size` 다.

```jsx
const a = new Set([1, 2, 3])
// Set { 1, 2, 3 }

a.size
// 3
```

<br>

## has

값이 이미 있는 지 검사한다.

```jsx
const a = new Set([1])
// Set { 1 }

a.has(1)
// true

a.has(2)
// false
```

<br>

## delete

값을 지운다.

만약 값이 있어서 지우는데 성공하면 `true` 를 리턴하고 아니면 `false` 를 리턴한다.

```jsx
const a = new Set[1, 2, 3])
// Set { 1, 2, 3 }

a.delete(1)
// true
// Set { 2, 3 }

a.delete(4)
// false
// Set { 2, 3 }
```

<br>

## clear

`Set` 에 있는 모든 값을 지운다.

```jsx
const a = new Set([1, 2, 3])
// Set { 1, 2, 3 }

a.clear()
// Set { }
```

<br>

## forEach

`forEach` 를 사용하여 `Set` 을 순회할 수 있다.

`forEach` 는 콜백 함수를 파라미터로 받으며 콜백 함수는 세 가지 파라미터를 받는다.

1. 값
2. 키 (index)
3. 현재 배열 (여기서는 Set)

`Set` 은 키값이 따로 없기 때문에 1번 2번이 같은 값을 가진다.

```jsx
const a = new Set([1, 2, 3, 4, 5])
// Set { 1, 2, 3, 4, 5 }

a.forEach((value) => {
    console.log(value)
})
// 1
// 2
// 3
// 4
// 5

a.forEach((key, value) => {
    console.log(key, value)
})
// 1 1
// 2 2
// 3 3
// 4 4
// 5 5

a.forEach((key, value, currentSet) => {
    console.log(key value, currentSet)
})
// 1 1 Set { 1, 2, 3, 4, 5 }
// 2 2 Set { 1, 2, 3, 4, 5 }
// 3 3 Set { 1, 2, 3, 4, 5 }
// 4 4 Set { 1, 2, 3, 4, 5 }
// 5 5 Set { 1, 2, 3, 4, 5 }
```

<br>

## Set ⇒ Array 또는 Array ⇒ Set

전개 연산자 (spread) 를 사용하면 간단하게 Set 을 Array 로, Array 를 Set 으로 변경할 수 있다.

```jsx
const a = new Set([1, 2, 3]) 
// Array => Set { 1, 2, 3}

const b = [...a]
// Set => [1, 2, 3]
```

<br>

# 문자열 한글 포함 여부 확인

정규식을 사용하면 한글의 포함 여부를 알 수 있습니다.

```jsx
const koRegex = /[ㄱ-ㅎ|ㅏ-ㅣ|가-힣]/;

koRegex.check("hello world"); // false
koRegex.test("안녕"); // true
koRegex.test("반가워 Hi"); // true
koRegex.test("ㅏㅏㅁㄹㄹㄲ"); // true
```

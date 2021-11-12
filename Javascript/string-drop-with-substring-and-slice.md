# JavaScript 에서 문자열 마지막을 자르는 방법 (feat. substring, slice)

# Overview

JavaScript 에서는 마지막 부분을 잘라내는 방법 (`drop`) 이 여러 가지 있습니다.

그 중에서 가장 일반적으로 사용하는 게 `substring` 과 `slice` 인데 둘의 사용법과 차이점을 알아봅니다.

<br>

# 1. substring

```js
string.substring(start, end);
```

`substring` 은 이름 그대로 문자열의 일부를 구하는 함수이며 사용법은 위와 같습니다.

잘라내고자 하는 문자열의 시작 (`start`) 과 끝 (`end`) 인덱스를 입력합니다.

가장 헷갈릴 만한 점은 **`start` 는 자르는 대상에 포함되고 `end` 는 포함되지 않습니다.**

<br>

**Example**

```js
// 마지막 문자 n 개 버리기
string.substring(0, string.length - n);
```

마지막 문자들만 버릴 예정이므로 `start` 는 무조건 0 으로 하고 자를 문자의 갯수만큼 `n` 을 입력하면 됩니다.

<br>

# 2. slice

```js
string.slice(start, end);
```

`slice` 는 `substring` 과 사용법과 문법이 완전히 같습니다.

하지만 단 하나의 차이가 있다면 파라미터로 **음수**값을 넘길 수 있다는 점입니다.

음수값은 쉽게 이해하자면 `-n == string.length - n` 으로 생각하면 됩니다.

<br>

**Example**

```js
// 마지막 문자 n 개 버리기
string.slice(0, -n);
```

음수를 사용할 수 있다는 점 때문에 `substring` 보다 훨씬 간결합니다.

<br>

# Conclusion

사용법은 비슷하지만 음수 파라미터의 사용이 가능한 **`slice` 가 훨씬 사용하기 편한** 것 같습니다.

StackOverflow 에서는 `substring` 이 속도가 더 빠르다는 결과도 있었던 것 같은데, 과거의 이야기고 현재는 비슷하다고 하네요.

실제로 벤치마크 가능한 사이트에서 [slice vs substr vs substring](https://www.measurethat.net/Benchmarks/Show/2335/1/slice-vs-substr-vs-substring-with-no-end-index) 을 테스트 해보면 비슷하게 나옵니다.

<br>

# Reference

- [2 Methods to Remove Last Character from String in JavaScript](https://tecadmin.net/remove-last-character-from-string-in-javascript/)
- [StackOverflow - How do I chop/slice/trim off last character in string using Javascript?](https://stackoverflow.com/questions/952924/how-do-i-chop-slice-trim-off-last-character-in-string-using-javascript)
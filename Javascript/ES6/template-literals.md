# 템플릿 리터럴

템플릿 리터럴 (template literals)은 변수를 이용해서 동적으로 문자열을 생성할 수 있다.

기존에는 문자열에 변수를 추가하려면 더하기 기호를 반복해서 사용해야했다.

```js
const year = 22;
const name = 'alice';
const text = 'Hi! My name is ' + name + ' and I am ' + year * 2 + ' years old';
console.log(text);      // Hi! My name is alice and I am 22 years old
```

<br>

이런식으로 코드를 작성할 경우 시간도 너무 오래걸리고 따옴표로 인한 가독성도 굉장히 떨어지게 된다.

ES6 에서는 템플릿 리터럴로 바꾸어 표현할 수 있다.

템플릿 리터럴은 백틱(\` \`) 을 사용하며 변수나 식은 `${variable} ${expression}` 으로 입력한다.

```js
const text = `Hi! My name is ${name} and I am ${year * 2} years old`
```

<br>

여러 줄을 입력할 때는 엔터를 입력하면 된다.

```js
// 기존: \n 을 사용
const text = 'Hi! My name is ' + name + '\n and I am ' + year * 2 + ' years old';

// ES6: 엔터 추가
const text = `Hi! My name is ${name} 
and I am ${year * 2} years old`
```
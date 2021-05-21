# Textarea 내부에 HTML Tag 를 넣고 싶을 때

# Overview

`textarea` 내부에는 `<b>` 와 같은 HTML Tag 적용이 되지 않습니다.

구글링 해보니 `textarea` 내부에 부분적으로 변화를 주는 건 불가능하고 `textarea` 를 사용하지 않고 `<div>` 태그를 사용해서 꼼수 부리는 방식들이 나와있었습니다.

<br>

# 1. contenteditable 로 편집 기능 추가

[contenteditable](https://developer.mozilla.org/ko/docs/Web/HTML/Global_attributes/contenteditable) 이라는 값을 이용하면 다른 태그에도 편집 기능을 추가할 수 있습니다.

<br>

## 1.1. HTML

```html
<div class="editable" contenteditable="true"></div>
```

HTML 옵션으로 `contenteditable` 값을 주면 편집기능이 추가됩니다.

<br>

## 1.2. JavaScript or JQuery 로 편집 기능 추가

```js
$('.editable').each(function(){
    this.contentEditable = true;
});
```

JavaScript 로도 `contentEditable` 옵션을 사용해서 내부에 text 를 입력가능하게 할지 말지 설정할 수 있습니다.

만약 값을 `false` 로 준다면 `readonly` 처럼 동작합니다.

<br>

# 2. CSS 로 textarea 처럼 변경

```css
div.editable {
    width: 300px;
    height: 200px;
    border: 1px solid #dcdcdc;
    overflow-y: auto;
}
```

`textarea` 처럼 보이기 위한 방법입니다.

<br>

# Reference

- [Is it possible to display bold and non-bold text in a textarea?](https://stackoverflow.com/questions/17456295/is-it-possible-to-display-bold-and-non-bold-text-in-a-textarea)
- [Rendering HTML inside textarea](https://stackoverflow.com/questions/4705848/rendering-html-inside-textarea)
- [Format text in a textarea?](https://stackoverflow.com/questions/12831101/format-text-in-a-textarea)
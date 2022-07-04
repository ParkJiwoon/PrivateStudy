원래 티스토리에 포스팅 할 때 Github 글을 그대로 복사해서 넣으면 제대로 들어갔는데 언제부터인가 티스토리 전체 설정이 바뀌었는지 적용이 잘 안되어서 새로 적용하게 되었습니다.

<br>

# 1. 티스토리 자체 플러그인 해제

아래 플러그인이 사용중이 아니어야 합니다.

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Tip/images/tistory.png?raw=true)

<br>

# 2. 스킨편집 > HTML 코드 추가

HTML 코드에서 `Head` 태그 안에 다음 스크립트를 추가합니다.

첫번째 `link` 태그에서 원하는 테마를 추가합니다.

저는 인텔리제이에서도 사용중인 `atom-one-dark` 테마를 추가하였고 아래 예시에 나와있는 것처럼 `styles` 와 `min.css` 사이에 추가하면 됩니다.

맨 아래의 `initHighlightingOnLoad` 를 사용해서 추가한 스크립트를 즉시 반영합니다.

```html
<link rel="stylesheet"
      href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/10.4.1/styles/atom-one-dark.min.css">

<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/10.4.1/highlight.min.js"></script>
	
<script>hljs.initHighlightingOnLoad();</script>
```

<br>

# 3. 스킨편집 > CSS 코드 추가

테마를 추가해도 맘에 안드는 부분이 좀 있을 수 있습니다.

이럴땐 CSS 를 직접 수정해야 합니다.

원래 존재하던 `pre` 태그와 `code` 태그를 주석 처리하고 아래 코드를 추가합니다.

CSS 설정은 기호에 맞게 수정해서 사용하시면 됩니다.

```css
/* Code block style */
code {
    padding: 0.25rem;
    background-color: #F1F1F1;
    border-radius: 5px;
    font-family: "D2Coding", "Hack", "Sans Mono", "Courier";
    font-size: 0.9rem;
}

pre > code {
    margin: 1rem auto;
    white-space: pre;
    overflow:auto !important; /* scroll setting */
}
```

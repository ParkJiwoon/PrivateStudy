# 이미지를 넣는 방법: HTML img tag vs CSS background image

# Overview

이미지는 HTML 에서 넣거나 CSS 에서 넣을 수 있습니다.

둘다 이미지가 노출된다는 사실은 같으나 약간의 차이점이 있습니다.

<br>

# 1. HTML 에서 <img> 태그 사용

```html
<img src="/temp/image">
```

- `<img>` 태그를 사용하면 이미지 업로드 실패 시 "깨진 이미지 아이콘" 과 "alt" 가 함께 노출된다.
- SEO 나 성능 등에서 이점이 많다.

<br>

# 2. CSS 에서 background-image 속성 사용

```css
background-image: url(image.jpg);
```

- 순전히 디자인 목적이라면 CSS 를 이용해도 된다.
- CSS 는 이미지 사이즈가 큰 경우 로딩하는데 시간이 더 걸린다
- 이미지 업로드 실패 시 아무것도 노출되지 않는다.

<br>

# 3. 각각 언제 사용하는게 좋을까?

만약 배경 이미지가 있어도 그만 없어도 문제 없는 상황이라면 실패했을 경우 아예 이미지가 노출되지 않는 편이 좋을 수도 있습니다.

이미지가 없어도 컨텐츠를 이해하는 데 무리가 없기 때문에 사용자에게 굳이 에러 상황을 알려줄 필요가 없습니다.

`img` 태그는 이미지가 컨텐츠와 관련이 깊고 검색 엔진에 노출이 필요한 경우에 사용하고 `background-image` 속성은 순수하게 디자인을 위한 목적인 경우에 사용합니다.

제 개인적인 생각으로는 웹 접근성도 고려해서 웬만하면 HTML 태그를 사용하는 게 좋다고 생각합니다.

<br>

# Reference

- [HTML img tag vs CSS background-image](https://blog.px-lab.com/html-img-tag-vs-css-background-image/)

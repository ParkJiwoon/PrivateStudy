# iOS 웹뷰에서 스크린 리더 (VoiceOver) 가 실시간 타이머 초점 못잡는 이슈

# Issue

웹 페이지를 만들 때 고려해야 할 점 중 하나가 시각 장애인을 위한 웹 접근성입니다. ([웹 접근성 포스트](../html-css-javascript/aria.md))

웹 접근성을 적용할 때 iOS 웹뷰에서 실시간으로 변하는 타이머에 초점이 잡히지 않는 이슈가 있었습니다.

처음에는 `aria-live` 속성을 적용해서 실시간 보이스 낭독을 기대했지만 문장을 다 읽기 전에 시간이 계속 바뀌어서 말이 끊기는 이슈가 있었습니다.

그래서 처음 초점 잡혔을 때의 시간만 안내하길 기대하면서 `role="timer"` 를 적용했지만 아예 초점이 잡히지 않았습니다.

<br>

# Solution

답은 `role="text"` 를 적용하는 거였습니다.

`role="text"` 는 WAI-ARIA 공식 홈페이지에서도 설명이 없는 걸로 보아 정식 role 은 아닌 것 같습니다.

다만 이 role 만 적용하면 어떤 역할인지 보이스 안내만으론 알기 힘들기 때문에 `aria-describedby` 속성까지 추가하는 것을 추천합니다.

```html
<span class="screen_out" id="timer_label" aria-hidden="true">남은 시간</span>
<div role="text" aria-describedby="timer_label">10:00</div>
```
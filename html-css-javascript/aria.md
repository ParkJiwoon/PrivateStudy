# 웹 접근성과 WAI-ARIA

# Overview

HTML 페이지를 만들 때 고려해야 하는 것 중 하나가 웹 접근성입니다.

웹 접근성이란 시각장애인들이 웹 페이지를 원활하게 이용할 수 있도록 알려주는 가이드라인이라고 생각하면 됩니다.

대표적으로 input 과 label 이 있는데요.

일반 사람들은 별다른 설명 없이 input, button, a 등등의 태그만 있어도 어느정도 이용할 수 있지만 시각장애인들은 직접 클릭할 수도 없이 탭 같은 걸로 각 요소를 이동하는 수밖에 없습니다.

따라서 탭을 눌렀을 때 초점이 어느 순서로 이동하는지, 초점이 잡혔을 때의 안내 메시지가 제대로 나오는지 여부는 굉장히 중요합니다.

이런 설정들을 HTML native 요소만으로는 처리하기 어렵기 때문에 W3C 는 WAI-ARIA 라는 걸 정의했습니다.

<br>

# 1. WAI-ARIA

WAI-ARIA (Web Accessibility Initiative – Accessible Rich Internet Applications) 는 W3C 에서 정의한 기술로 웹 접근성을 위해 지원되는 여러 가지 특성들을 의미합니다.

일반 사용자가 보기에 정상인 화면들도 HTML 요소에 따라서 스크린 리더 등의 보조기기에서 제대로 읽히지 않을 수 있습니다.

WAI-ARIA 는 이를 개선하기 위해 웹 애플리케이션에 역할(Role), 속성(Property), 상태(State) 정보를 추가할 수 있습니다.

<br>

## 1.1. Role

`Role` 은 HTML 요소의 역할을 정의합니다.

일반적으로는 HTML native 요소만으로 이를 처리하는게 가장 이상적입니다.

버튼은 button, 링크는 a, 체크박스는 input checkbox 등등 이미 존재하는 요소들로도 충분히 표현할 수 있습니다.

하지만 이미지에 버튼 클릭 이벤트를 준다던가 좀더 세세하고 다양한 설정이 하고 싶을 때는 native 요소만으로 부족할 수 있습니다.

이럴 때 `role` 을 사용해서 해당 요소들의 역할을 명시합니다.

Role 의 종류는 W3C WAI ARIA 의 [5.3 Categorization of Roles](https://www.w3.org/TR/wai-aria/#roles_categorization) 에서 확인할 수 있습니다.

```html
<!-- role example -->
<li role="menuitem">Open file…</li>
```

<br>

## 1.2. State and Property

속성 (Property) 는 해당 요소의 특징이나 상황을 정의하며 `aria-` 라는 접두사를 사용합니다.

상태 (State) 는 요소의 현재 상태를 나타냅니다.

```html
<!-- property state example 1 -->
<li role="checkbox" aria-checked="true">체크박스 아이템</li>

<!-- property state example 2 -->
<div role="alert" aria-live="assertive">올바르지 않은 입력입니다.</div>
```

<br>

# 2. ARIA 속성들

공부한 속성들에 대해서 정리합니다.

앞으로도 생길때마다 조금씩 추가할 예정입니다.

<br>

## 2.1. role="checkbox" aria-checked="true"

```html
<span role="checkbox" aria-checked="false" tabindex="0" aria-labelledby="chk1-label"></span> 
<label id="chk1-label">Remember my preferences</label>

<!-- native 요소로 사용할 수 있는 경우 -->
<input type="checkbox" id="chk1-label">
<label for="chk1-label">Remember my preferences</label>
```

- [w3c - Checkbox Example](https://www.w3.org/TR/wai-aria-practices/examples/checkbox/checkbox-1/checkbox-1.html)
- [ARIA: checkbox role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/checkbox_role)

특정 요소에 체크박스 역할을 부여합니다.

`aria-checked` 를 사용해서 체크됨, 체크안됨 여부를 판단할 수 있습니다.

focus 되지 않는 요소일 경우 `tabindex` 를 사용하기도 합니다.

가능하면 HTML native checkbox 를 사용하는게 권장되지만 사용할 수 없는 경우에 ARIA 속성을 사용합니다.

<br>

## 2.2. aria-label, aria-labelledby, aria-describedby

- [ARIA Label](https://developers.google.com/web/fundamentals/accessibility/semantics-aria/aria-labels-and-relationships?hl=ko)

특정 요소를 설명하는 데 사용되는 ARIA 속성들입니다.

비슷해보이지만 조금씩 차이가 있습니다.

<br>

```html
<button aria-label="menu" class="button"></button>
```

**aria-label** 은 우리가 흔히 알고 있는 Label 목적을 위한 속성입니다.

특정 요소에 대한 설명을 그대로 적습니다.

텍스트 대신 그래픽을 사용하는 경우처럼 추가 설명이 필요한 경우 사용할 수 있습니다.

<br>

```html
<span id="rg-label">음료수 옵션</span>
<div role="radiogroup" aria-labelledby="rg-label"></div>
```

**aria-labelledby** 은 `aria-label` 과 비슷하지만 조금 다릅니다.

`aria-label` 은 어떤 요소에 대한 설명을 직접 적는 반면, `aria-labelledby` 은 다른 요소의 ID 값을 매칭시킵니다.

`aria-labelledby` 은 Label 자체를 재정의하기 때문에 다른 모든 Label 속성들, `aria-label` 또는 HTML native label 과 함께 쓰여도 항상 `aria-labelledby` 을 우선합니다.

<br>

```html
<label for="pw">Password:</label>
<input type="password" id="pw" aria-describedby="pw-help">
<div id="pw-help">비밀번호는 12 자 이상으로 이루어져야 합니다</div>
```

**aria-describedby** 은 `aria-labelledby` 와 같은 방식으로 다른 요소의 ID 를 매칭하여 현재 요소에 대한 설명을 나타냅니다.

둘의 사용법과 쓰임새가 굉장히 비슷하여 헷갈릴 수도 있지만 다른 목적으로 사용됩니다.

[MDN Web Docs - Using the aria-describedby attribute](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute) 에는 다음과 같이 나와있습니다.

**This is very similar to aria-labelledby: a label describes the essence of an object, while a description provides more information that the user might need.**

`label` 관련 속성들과의 가장 큰 차이점은 Label 이 요소의 필수 설명이라면 `aria-describedby` 은 어디까지나 부연 설명이라는 점입니다.

그래서 위 예시 코드에서처럼 `label` 태그와 함께 사용할 수 있습니다.

<br>

## 2.3. aria-live

- [ARIA live regions](https://developer.mozilla.org/ko/docs/Web/Accessibility/ARIA/ARIA_Live_Regions)
- [콘텐츠 숨기기 및 업데이트](https://developers.google.com/web/fundamentals/accessibility/semantics-aria/hiding-and-updating-content?hl=ko)

JavaScript 를 사용하면 페이지를 새로 로드하지 않고 일부만 동적으로 변경하는게 가능합니다.

동적인 변경은 페이지를 볼 수 있는 사용자들은 알 수 있지만, 시각 장애인들은 알아채기 어렵습니다.

그래서 이런 동적인 (실시간) 변경들을 알려줄 수 있는 `aria-live` 라는 속성이 제공됩니다.

<br>

```html
<input type="text" name="email">
<div role="alert" aria-live="assertive">이메일 형식이 올바르지 않습니다.</div>
```

회원가입 또는 로그인 페이지에서 사용하는 Email Input 은 실시간으로 Email 형식 검사를 하여 올바른 형식이 아니면 빨간색 에러 메시지를 사용자에게 노출합니다.

페이지를 볼 수 있는 사용자는 그 정보를 실시간으로 확인할 수 있지만 시각장애인은 에러 메시지가 노출되었다는 사실을 모를 수 있습니다.

이럴 때 `role="alert"` 로 경고 역할을 갖고 있는 요소를 만들고 실시간 변화를 감지해서 알려주는 `aria-live`  속성을 추가할 수 있습니다.

`aria-live` 의 값으로는 다음 값들이 올 수 있습니다.

- off (default)
- polite : 현재 진행중인 음성 또는 타이핑 이후에 알림
- assertive : 현재 진행중인 알림을 중단하고 즉시 알림

`assertive` 값은 사용자의 현재 작업을 방해할 수 있기 때문에 신중하게 적용해야 합니다.

<br>

## 2.4. role="alert"

- [ARIA: alert role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/Alert_Role)

`alert` 역할은 사용자에게 동적인 변화를 알려줄 때 사용합니다.

스크린 리더는 `alert` 역할이 붙은 요소가 업데이트 되면 바로 읽기 시작합니다.

`role="alert"` 로 설정한다는 건 `aria-live="assertive" aria-atomic="true"` 와 동일합니다.

<br>

# Reference

- [W3C - 웹 접근성이란?](https://www.w3.org/WAI/fundamentals/accessibility-intro/ko)
- [WAI-ARIA 웹퍼블리싱](https://www.biew.co.kr/entry/WAI-ARIA-%EC%9B%B9%ED%8D%BC%EB%B8%94%EB%A6%AC%EC%8B%B1)
- [W3C - WAI ARIA](https://www.w3.org/TR/wai-aria)
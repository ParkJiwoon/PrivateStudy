# 13장 리액트 라우터로 SPA 개발하기

# 1. SPA 란?

SPA 는 Single Page Application 의 약어입니다.

말 그대로 한 개의 페이지로 이루어진 애플리케이션이라는 의미입니다.

전통적인 웹 페이지는 사용자가 다른 페이지로 이동할 때마다 서버로부터 새로운 html 을 받아와서 화면에 보여줬습니다.

요즘은 웹에서 제공되는 정보가 많아서 서버 측에서 새로운 화면을 모두 보여준다면 성능상의 문제가 발생할 수 있습니다.

그래서 리액트 같은 라이브러리 또는 프레임워크를 사용해서 뷰 렌더링을 클라이언트 쪽에 맡깁니다.

페이지 변화가 필요하면 서버 API 를 호출해서 데이터만 받아온 뒤 필요한 부분만 자바스크립트로 업데이트 해줍니다.

<br>

SPA 의 경우 서버에서 사용자에게 제공하는 페이지는 한 종류지만, 해당 페이지에서 로딩된 자바스크립트와 현재 사용자 브라우저의 주소 상태에 따라 다양한 화면을 보여줄 수 있습니다.

다른 주소에 다른 화면을 보여 주는 걸 라우팅이라고 합니다.

리액트 자체에는 이 기능이 없기 때문에 브라우저의 API 를 사용합니다.

리액트 라우팅 라이브러리는 리액트 라우터 (react-router), 리치 라우터 (reach-router), Next.js 등 여러 가지가 있습니다.

이 장에서는 **리액트 라우터**에 대해서 알아봅니다.

<br>

# 1.1. SPA 의 단점

SPA 의 단점은 앱 규모가 커지면 **자바스크립트 파일이 너무 커진다**는 겁니다.

페이지 로딩 시 사용자가 실제로 방문하지 않을 수도 있는 페이지의 스크립트도 전부 불러오기 때문이죠.

이를 해결하기 위해 라우트 별로 파일을 나누는 코드 스플리팅 (code splitting) 을 학습합니다.

또한 검색 엔진 이슈가 있는데 이 또한 나중에 배울 서버 사이드 렌더링 (SSR, Server Side Rendering) 을 통해 해결할 수 있습니다.

<br>

# 2. React Router 적용한 프로젝트 생성

## 2.1. 설치

```sh
$ yarn add react-router-dom@5 
```

현재 `react-router-dom` 으로 하면 6.0.0 버전이 설치되는데 이러면 `Routes` 컴포넌트가 제대로 동작하지 않아서 5 버전대로 서치합니다.

그리고 5 버전대로 설치하면 `Routes` 컴포넌트를 사용할 필요가 없습니다.

<br>

## 2.2. 프로젝트에 라우터 적용

```jsx
// src/index.js

import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import { BrowserRouter } from "react-router-dom";

ReactDOM.render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>,
  document.getElementById('root')
);

reportWebVitals();
```

리액트 라우터를 적용하기 위해선 `src/index.js` 파일에서 `react-router-dom` 에 내장되어 있는 `BrowserRouter` 라는 컴포넌트를 사용하여 감싸야 합니다.

이 컴포넌트는 웹 애플리케이션의 HTML5 History API 를 사용하여 페이지를 새로고침하지 않고도 주소를 변경하고, 현재 주소에 관련된 정보를 `props` 로 쉽게 조회하거나 사용할 수 있도록 해줍니다.

<br>

## 2.3. 페이지 만들기

```jsx
// Home.js
export default function Home() {
  return(
    <div>
      <h1>홈</h1>
      <p>홈, 그 페이지는 가장 먼저 보여지는 페이지</p>
    </div>
  )
}

// About.js
export default function About() {
  return (
    <div>
      <h1>소개</h1>
      <p>이 프로젝트는 리액트 라우터 기초를 실습해보는 예제 프로젝트입니다.</p>
    </div>
  )
}
```

<br>

## 2.4. Route 컴포넌트로 특정 주소에 컴포넌트 연결

```jsx
<Route path="주소규칙" component={보여줄 컴포넌트} />
```

`Route` 컴포넌트를 사용해서 다른 컴포넌트를 연결해줄 수 있습니다.

사용법은 위와 같습니다.

<br>

```jsx
import { Route } from "react-router-dom";
import Home from "./Home";
import About from "./About";

export default function RouteApp() {
  return (
    <div>
      <Route path="/" component={Home} />
      <Route path="/about" component={About} />
    </div>
  )
}
```

위 코드르 짜고 페이지에 진입하면 글씨가 뜹니다.

6.0.0 버전에서는 `Routes` 컴포넌트를 사용해야 하는 것 같은데 5 버전대로 설치하면 없어도 됩니다.

`/about` 경로로 들어가면 예상과 다르게 `Home` 컴포넌트도 등장합니다.

`/about` 경로가 `/` 에도 일치하기 때문이라서 이를 수정하려면 `exact` props 를 true 로 설정하면 됩니다.

```jsx
<Route path="/" component={Home} exact={true} />
```

<br>

## 2.5. Link 컴포넌트를 사용해서 다른 주소로 이동하기

```jsx
<Link to="주소">내용</Link>
```

`Link` 컴포넌트는 클릭하면 다른 주소로 이동시켜 주는 컴포넌트입니다.

일반 웹 애플리케이션에서는 페이지 전환 시 `a` 태그를 사용하는데 리액트 라우터를 사용할 땐 `a` 태그를 직접 사용하면 안됩니다.

이 태그는 페이지를 아예 새로 불러오기 때문에 애플리케이션이 들고 있던 상태를 모두 날려버립니다.

`Link` 컴포넌트를 사용해서 페이지를 전환하면, 페이지를 새로 불러오지 않고 페이지의 주소만 변경해줍니다.

```jsx
import {Link, Route} from "react-router-dom";
import Home from "./Home";
import About from "./About";

export default function RouteApp() {
  return (
    <div>
      <ul>
        <li><Link to="/">홈</Link></li>
        <li><Link to="/about">소개</Link></li>
      </ul>
      <hr />
      <Route path="/" component={Home} exact={true} />
      <Route path="/about" component={About} />
    </div>
  )
}
```

<br>

# 3. Route 하나에 여러 개의 path 설정하기

```jsx
<Route path={["/about", "/info"]} component={About} />
```

배열로 감싸주면 됩니다.

<br>

# 4. URL 파라미터와 쿼리

페이지 주소를 정의할 때 유동적인 값을 전달하는 방법은 두 가지입니다.

- 파라미터 : `/profiles/{username}`
- 쿼리 : `/about?details=true`

둘 중 어떤 방식을 사용해야 할지 정해진 것은 없지만 일반적으로 파라미터를 많이 사용하고 쿼리는 검색 필터, 페이징 옵션 등 추가 옵션을 위해 사용합니다.

<br>

## 4.1. URL 파라미터

```jsx
const data = {
  vue: {
    name: "뷰",
    description: "SPA 를 위한 뷰 프레임워크"
  },
  react: {
    name: "리액트",
    description: "SPA 를 위한 뷰 라이브러리"
  }
}

export default function Profile ({ match }) {
  const { username } = match.params;
  const profile = data[username];

  if (!profile) {
    return <div>존재하지 않는 사용자입니다.</div>
  }

  return (
    <div>
      <h3>{username}({profile.name})</h3>
      <p>{profile.description}</p>
    </div>
  )
}
```

컴포넌트로 넘어오는 `match` 라는 객체 안의 `params` 값을 참조합니다.

사용할 때는 다음과 같이 `path` 에 콜론(:) 과 함께 넣어주면 됩니다.

```jsx
<Route path="/profile/:username" component={Profile} />
```

<br>

## 4.2. URL 쿼리

쿼리는 `?detail=true&another=1` 과 같은 문자열 형태로 이루어져 있습니다.

이 값을 사용하기 위해선 문자열을 객체 형태로 변환해주어야 합니다.

쿼리 문자열을 객체로 변환할 때는 `qs` 라는 라이브러리를 사용합니다.

```sh
$ yarn add qs
```

<br>

`About` 컴포넌트 코드를 아래와 같이 변경합니다.

```jsx
import qs from "qs";

export default function About({ location }) {
  const query = qs.parse(location.search, {
    ignoreQueryPrefix: true // 이 설정을 통해 문자열 맨 앞의 ? 를 생략함
  })

  // query 는 Integer, Boolean 값을 넣어도 무조건 String 형태로 받아짐
  const showDetail = query.detail === "true"

  return (
    <div>
      <h1>소개</h1>
      <p>이 프로젝트는 리액트 라우터 기초를 실습해보는 예제 프로젝트입니다.</p>
      {showDetail && <p>detail 값을 true 로 설정하셨군요!</p>}
    </div>
  )
}
```

이제 `http://localhost:3000/about?detail=true` 로 들어가면 새로운 문구가 하나 더 추가되서 보입니다.

<br>

# 5. 서브 라우트

서브 라우트란 라우트 내부에 또 라우트를 정의하는 것을 의미합니다.

별로 어렵진 않고 그냥 라우트로 사용되는 컴포넌트 내부에 또 `Route` 컴포넌트를 사용하면 됩니다.

`Profiles` 컴포넌트를 새로 만들어서 `Profile` 컴포넌트를 옮깁니다.

```jsx
import { Link, Route } from "react-router-dom";
import Profile from "./Profile";

export default function Profiles () {
  return (
    <div>
      <h3>사용자 목록:</h3>
      <ul>
        <li><Link to="/profiles/vue">vue</Link></li>
        <li><Link to="/profiles/react">react</Link></li>
      </ul>

      <Route path="/profiles" exact render={() => <div>사용자를 선택해 주세요.</div>} />
      <Route path="/profiles/:username" component={Profile} />
    </div>
  )
}
```

`Route` 컴포넌트에 `component` props 가 없는 대신 `render` props 를 넣어서 보여주고 싶은 JSX 만 간단하게 표현합니다.

<br>

```jsx
import {Link, Route} from "react-router-dom";
import Home from "./Home";
import About from "./About";
import Profiles from "./Profiles";

export default function RouteApp() {
  return (
    <div>
      <ul>
        <li><Link to="/">홈</Link></li>
        <li><Link to="/about">소개</Link></li>
        <li><Link to="/profiles">프로필</Link></li>
      </ul>
      <hr />
      <Route path="/" component={Home} exact={true} />
      <Route path={["/about", "/info"]} component={About} />
      <Route path="/profiles" component={Profiles} />
    </div>
  )
}
```

그리고 `RouteApp` 컴포넌트에 `Profile` 대신 `Profiles` 컴포넌트를 매핑해주면 됩니다.
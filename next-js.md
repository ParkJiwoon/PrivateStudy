# Customizing Babel Config

- [NextJS 공식 홈페이지의 Customizing Babel Config](https://nextjs.org/docs/advanced-features/customizing-babel-config) 를 번역한 글입니다.

<br>

Next.js 는 React 애플리케이션과 서버사이드 코드를 컴파일 하기 위한 모든 사전 설정이 `next/babel` 에 포함되어 있습니다.

하지만 기본 Babel 설정을 확장하는 것도 가능합니다.

`.babelrc` 파일을 프로젝트 가장 상단(루트) 에 생성하기만 하면 됩니다.

```json
{
  "presets": ["next/babel"],
  "plugins": []
}
```

<br>

presets/plugins 에 대한 설정이 없다면 아래와 같이 작성할 수 있습니다.

```json
{
  "presets": ["next/babel"],
  "plugins": ["@babel/plugin-proposal-do-expressions"]
}
```

<br>

presets/plugins 에 대한 커스텀 설정을 추가하려면 `next/babel` preset 을 다음과 같이 작성합니다.

```json
{
  "presets": [
    [
      "next/babel",
      {
        "preset-env": {},
        "transform-runtime": {},
        "styled-jsx": {},
        "class-properties": {}
      }
    ]
  ],
  "plugins": []
}
```

<br>

각 설정에 대해 가능한 옵션은 해당 document 페이지에 방문해서 배울 수 있습니다.

- NextJS 는 서버사이드 컴파일을 위해 최신 노드 버전을 사용합니다.
- `"preset-env"` 의 `modules` 옵션은 `false` 로 설정되어야 합니다. 그렇지 않으면 webpack 의 코드 분할이 꺼집니다.

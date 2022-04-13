---
layout: post
title: 웹팩 모듈 페더레이션으로 마이크로 프론트엔드 시작하기
date: 2022-04-13 20:23:00 +0900
description: Micro Front-End with Webpack Module Federation
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags:
  [micro front-end, webpack5, module-federation, craco, craco-module-federation]
---

마이크로 프론트엔드 아키텍처를 구성하는 방법으로는 여러가지 방법들이 존재하는데, 아래 블로그 글에서 잘 정리되어 있으니 살펴보면 좋을 것 같다.
https://mobicon.tistory.com/572

해당 포스트에서는 Webpack5의 Module Federation 기능을 사용해 runtime 시점에서의 통합하는 방식을 사용한 마이크로 프론트엔드 구성 방법을 살펴본다.

## Module Federation?

모듈 페더레이션은 웹팩5에 추가된, 개별적으로 빌드된 번들 파일을 통합할 수 있는 기술이다. 기본적으로 모듈을 제공하는 `remote`와 제공된 모듈을 사용하는 `host`가 존재하고, 이는 양방향(Bidirectional)으로도 가능하다. 즉 `remote`이면서 `host`인 것도 가능하다.
![image]({{site.baseurl}}/assets/img/2022-04-13/module-federation-bidirectional.png)

웹팩 모듈 페더레이션은 기본적으로 노출된 모듈을 비동기로 로드하고, 동기적으로 평가한다.
예시로 remote에서 제공하는 리액트 컴포넌트를 host에서 가져올 때는 대략 아래와 같다.

```typescript
const ModuleA = React.lazy(() => import("remoteA/ComponentA"));
```

## Federation 시작해보기

이번 예제에서는 2개의 CRA 프로젝트를 생성하고, 하나의 프로젝트를 `host`, 다른 하나를 `remote`로 사용할 예정이다. 최신 CRA 버전으로 프로젝트를 생성하였을 경우에는 기본적으로 webpack5를 사용하므로 걱정할 필요가 없지만, 현재 개발중인 프로젝트를 대상으로 모듈 페더레이션을 적용할 경우, webpack5를 사용하도록 마이그레이션 해주어야한다. 하지만 webpack5를 사용하기 위해서 CRA 프로젝트를 `eject`한다는건 어느정도 리스크 아닌 리스크(다시 되돌릴 수 없고, 프로젝트 폴더 구조가 지저분해진다.)가 있으므로 CRA에서 사용하는 `react-scripts`의 버전을 최신버전으로 올려주는 것을 추천하고, 이로인한 사이드 이펙트는 개인적인 경험으로 비추어볼때 크지 않거나 없었다.
자 이제 2개의 CRA 프로젝트를 생성해보자

```
// host용 CRA 프로젝트 생성
npx create-react-app module-federation-host --template typescript

// remote용 CRA 프로젝트 생성
npx create-react-app module-federation-remote --template typescript
```

위 명령어로 프로젝트를 생성했다면 이제 모듈 페더레이션을 위한 추가적인 웹팩 설정을 해줄 차례인데, webpack 설정을 건드리기 위해서는 `eject`시키거나 다른 방법을 사용해야하는데, 여기서는 `eject`를 시키지 않을 예정이므로 webpack 설정을 오버라이딩 하기위한 라이브러리로 `craco`를 사용할 예정이다. `craco`와 모듈 페더레이션을 위한 플러그인 `craco-module-federation`을 설치해주자

```
// package manager는 원하는걸 사용하면 된다.

// module-federation-host
yarn add @craco/craco
yarn add craco-module-federation

// module-federation-remote
yarn add @craco/craco
yarn add craco-module-federation
```

위 내용까지 설치를 마쳤다면, 설치한 플러그인 설정을 해주기 위해서 `craco.config.js` 파일을 생성하고 아래와 같이 설정해주자 (craco 실행을 위한 package.json 명령어 수정은 스킵한다.)

```js
const CracoModuleFederation = require("craco-module-federation");
module.exports = {
  plugins: [
    {
      plugin: CracoModuleFederation,
    },
  ],
  devServer: {
    port: 3000, // port 넘버는 host와 remote가 다르게 되도록 설정해주자, 이 글에서는 host:3000, remote: 3001 번을 사용한다.
  },
};
```

자 이제 host에 제공할 간단한 컴포넌트를 remote에서 만들어보자, 여기서는 간단하게 Button을 만들어서 모듈로 내보낼 예정이다.

host에 제공할 컴포넌트를 만들었다면, 이제 module을 expose하고, host에서는 이를 받기 위한 설정을 해야한다. `modulefederation.config.js` 파일을 host와 remote에 각각 생성해주고 아래와 같이 설정해주자

```js
// host
const { dependencies } = require("./package.json");

module.exports = {
  name: "hostApp",
  remotes: {
    remoteApp: "remoteApp@http://localhost:3001/remoteEntry.js",
  },
  shared: {
    ...dependencies,
    react: {
      singleton: true,
      requiredVersion: dependencies["react"],
    },
    "react-dom": {
      singleton: true,
      requiredVersion: dependencies["react-dom"],
    },
  },
};
```

```js
// remote
const { dependencies } = require("./package.json");

module.exports = {
  name: "remoteApp",
  exposes: {
    "./Button": "./src/Button",
  },
  filename: "remoteEntry.js",
  shared: {
    ...dependencies,
    react: {
      singleton: true,
      requiredVersion: dependencies["react"],
    },
    "react-dom": {
      singleton: true,
      requiredVersion: dependencies["react-dom"],
    },
  },
};
```

모듈로 내보낼 컴포넌트도 만들었고, 이를 내보내고 받을 설정까지 모두 마쳤다. 아마 이대로 remote의 모듈을 host에서 가져다 사용하려고 하면 에러가 발생할텐데, 아직 한가지더 설정 해주어야 할 것이 남아있다. CRA를 기준으로 `src/index.tsx`를 보면 아래와 같은 형태로 코드가 작성되어져있는데,

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";

const root = ReactDOM.createRoot(
  document.getElementById("root") as HTMLElement
);
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

해당 파일의 이름을 `bootstrap.tsx`로 변경하고 새로운 `index.tsx` 파일을 만들어 아래와 같이 작성해주자. 이 설정은 `host`와 `remote` 모두 해주어야한다.

```tsx
// index.tsx
import("./bootstrap");

export default {};
```

그리고 `host`에서 `remote`에서 만들어온 컴포넌트를 비동기로 가져와 렌더링 해보자.

```tsx
// host의 App.tsx
import React from "react";
import logo from "./logo.svg";
import "./App.css";

// @ts-ignore
const Button = React.lazy(() => import("remoteApp/Button"));

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.tsx</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
        <Button> remote button </Button>
      </header>
    </div>
  );
}

export default App;
```

이제 `host`와 `remote`를 모두 로컬서버 실행시킨후, `host`의 로컬서버 url로 접속해보면 remote에서 exposes한 모듈을 잘 가져와 렌더링하고있음을 확인할 수 있다.
![image]({{site.baseurl}}/assets/img/2022-04-13/module-federation-result.png)

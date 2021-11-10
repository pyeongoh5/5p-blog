---
layout: post
title: webpack bundle analyzer로 내 프로젝트 상태 파악하기
date: 2021-09-08 20:23:00 +0900
description: webpack bundle analyzer로 내 프로젝트 상태 파악하기
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [webpack-bundle-analyzer, webpack]
---

외주 프로젝트는 조심스럽게 잘 진행되어야 한다. 만약 이벤트 페이지처럼 일회성 프로젝트라면 크게 문제될 것은 없지만 외주 이후에 인수인계를 받아 이어서 진행하는 프로젝트라면 무자비한 레거시 요소에 당황하지 않도록 많은 부분을 챙겨야만한다. 컨벤션이나 기술스택등을 엄격히 제한하고, 함께 논의할 수 있어야한다. 또 외주 맡겼다고 일정 데드라인이 다가올 때까지 손을 놓고 있어서도 안되며 주기적으로 진행상황을 검토하고 모니터링 해야한다. 그리고 맡게된 프로젝트가 딱 그렇게 하지 못했던 그런 케이스였다. 당시엔 어떤 요구사항을 제시하고 관리해야하는지 알지 못했었던것 같다. 좋지 않은 결과물에 대한 핑계로 외주사 입장에서야 많은 업무량과 짧은 일정이라는 사정이 있었을 수 있지만 이를 인계받아서 앞으로의 프로젝트를 진행해나가야 하는 입장에서는 쉽게 납득되지는 않았다. scss 파일은 물론이고, tsx, assets파일 중에서 사용하지 않는 파일들이 넘쳐났고, git repository를 통한 소스코드를 넘겨받는것 이외에 별도의 인수인계 절차도 없었다.

<br>

뭔가 블로그 제목과 어긋나는 이야기를 하는 것 같아 보일 수 있겠지만, 이 이야기가 이 글을 쓰게된 원인이 됐다. 프로젝트를 인계받고 이것 저것 확인하는 과정에서 많은 문제점들이 있었다. 빌드 시간, 로컬 서버 실행시간은 이상할 정도로 너무나 느렸으며, inspector에서 css 속성을 껏다 끼면, rendering이 멈췄다..(**????** 한번도 보지 못한 처음 겪는 현상이라 적잖이당황했었다.)
그래서 프로젝트를 개선하기 위해서는 먼저 어떤 문제가 있는 지 파악했어야 했다. 이런 문제 상황을 공유하고 플랫폼 리더분과 이야기 하다가 프로젝트의 구성요소들을 분석할 수 있는 라이브러리를 알게 되었고 이를 통해 분석을 할 수 있었다.

# webpack-bundle-analyzer

바로 `webpack-bundle-analyzer`이다. webpack plugin으로 번들러를 통해서 빌드하는 과정에서 나오는 아웃풋 파일들을 분석해서 이를 시각화 해주는 라이브러리이다.
![image]({{site.baseurl}}/assets/img/2021-11-10/webpack-bundle-analyzer.gif)

이 라이브러리를 통해서 프로젝트에서 과도하게 사용되고 있는 부분이나 필요하지 않은 부분들을 파악하려고 시도했다.

# webpack-bundle-analyzer 설치

```
# NPM
npm install --save-dev webpack-bundle-analyzer
# Yarn
yarn add -D webpack-bundle-analyzer
```

라이브러리를 설치한 이후에는 webpack plugin이기 때문에, webpack에 plugin 설정을 추가해주어야한다. 최근에 많이 사용하는 `CRA(create react app)`를 통해서 구성된 프로젝트의 경우 webpack 설정을 오버라이딩 할 수 있는 라이브러리를 사용하거나 `eject`하여서 설정할 수 있다. 이 글에서는 `CRA` 환경에서, `eject`시키지 않고 `craco`를 통해서 오버라이딩했다.

```js
// craco.config.js
const BundleAnalyzerPlugin =
  require("webpack-bundle-analyzer").BundleAnalyzerPlugin;
module.exports = {
  webpack: {
    mode: "production",
    plugins: [
      new BundleAnalyzerPlugin({
        analyzerMode: "static", // 분석결과를 파일로 저장
        reportFilename: "bundle-report.html", // 분설결과 파일을 저장할 경로와 파일명 지정
        openAnalyzer: true, // 웹팩 빌드 후 보고서파일을 자동으로 열지 여부
        generateStatsFile: true, // 웹팩 stats.json 파일 자동생성
        statsFilename: "stats.json", // stats.json 파일명 rename
      }),
    ],
  },
};
```

stats.json파일을 생성해주어야하는데, 공식사이트에서는 아래와 같은 Command를 가이드해주고 있다.

```
webpack --profile --json > stats.json
```

하지만, webpack4에서는(5에서도 가능한지는 확인해보지 않았다.) 아래와 같이 실행해야된다.

```
webpack-cli --stats=detailed --json > stats.json
```

이제 웹팩이 수행되도록 아래 커맨드를 실행시켜주면 분석된 파일이 생성된 것을 확인할 수 있다.

```
yarn start
yarn build
```

![image]({{site.baseurl}}/assets/img/2021-11-10/output.png)

# 결과

생성된 HTML 파일을 열어보자. 아래사진은 HTML 파일에 시각화된 내용이다.
![image]({{site.baseurl}}/assets/img/2021-11-10/before.png)
![image]({{site.baseurl}}/assets/img/2021-11-10/before_zoomed.png)

`index.scss`의 크기가 무려 12메가였다.. 빌드시간이 오래걸리는 것도, 로컬서버 실행속도가 느린것도, inspector에서 css 속성을 껏다 키면 rendering이 멈추는 이유도 뭔가 납득이 가는것 같았다. 다른 개선사항이 더 많겠지만(lodash를 통채로 사용하고 다운받아 사용하고 있어서, 사용하고있는 기능대비 차지하고 있는 용량이 컷고, 사용하고 있는 editor 역시 용량이 많이 큰 라이브러리였다.), 우선순위가 높아보이는 이것 먼저 해결하기로 했다. 사용하지 않는 scss 파일들을 찾아서 import에서 제외하고, 중첩되는 scss 속성들을 삭제했다. 그리고 다시 빌드를 돌려서 결과를 확인해봤다.

![image]({{site.baseurl}}/assets/img/2021-11-10/after.png)

무려 10메가의 사이즈를 줄였다. 빌드 시간은 180초에서 80초 수준으로 줄어들었고, 이제는 inspector에서 css속성을 건들여도 여전히 느리긴하지만 rendering이 멈추진 않았다. 아직 scss파일의 import 구조가 얽히고 섥혀있어 완벽히 쓸모없는 부분을 제거하지는 못해서 아마 더 개선할 부분이 있을것 같다. 추후 node_module의 용량을 많이 차지하고 있는 부분과 scss 부분을 추가적으로 개선하면 지금 보다 더 좋은 결과가 나올것으로 기대된다.

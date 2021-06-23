---
layout: post
title: 안드로이드 구버전 Webview 대응하기
date: 2021-06-23 19:21:20 +0900
description: 안드로이드 구버전 Webview 대응하기
img: "/Chrome-OS-Android.png"
fig-caption: # Add figcaption (optional)
tags: [javascript, webview, polyfill]
---

웹 프론트엔드 개발을 해오면서 최신 브라우저 환경에서 개발하는 일이 잦고, 크롬과 같은 특정 브라우저에서만 개발하는 케이스도 많아지면서 오래된 브라우저 대응에 대한 감이 많이 사라졌었는데, 이런 상태로 모바일 웹뷰를 개발하다보니 구버전의 웹뷰가 발목을 잡았다. 오늘은 이에 대한 느낀점과 해결 방법을 적어보려고 한다.

| Typescript + Next.js 환경을 기준으로 작성되었습니다.

## 문제의 발단: Android webview

문제의 발단은 개발을 진행하면서 서비스가 지원해야하는 최소버전을 확인하면서부터였다. 안드로이드 6.0 버전을 대응하려고 보니, 웹뷰의 버전이 크롬 44버전이었고 지원하지 않는 기능들로 인해 페이지는 렌더링되지 않았다.

<figure class="image">
<img src="{{site.baseurl}}/assets/img/chrome_version.png" alt="안드로이드 6버전 시뮬레이터의 크롬버전">
<figcaption>안드로이드 6버전 시뮬레이터의 크롬버전</figcaption>
</figure>

안드로이드 6.0버전을 사용한다고 해도 사용하는 웹뷰를 최신버전으로 업데이트 하면 문제가 해결되지만, 업데이트하지 않은 사용자는 구버전에서 지원하지 않는 기능을 사용한 소스코드들로 인해 화면을 볼 수 없어 서비스를 이용할 수 없으니 꽤나 큰 문제다.

## 문제들

44버전의 크롬에서 꽤 많은 문제가 발생했는데 이는 아래와 같다.

- use of const in strict mode
- attachShadow 함수 undefined 이슈
- Mobx 5+ 에서의 Symbol, Proxy 문제
- React dangerouslySetInnerHTML 에서의 arrow function 이슈
- css 미지원 이슈

<br>

### 1.use of const in strict mode

기존에 작성된 script코드가 strict mode로 되어 있지 않은 상태였고, 아래의 사진에서처럼 strict mode가 아닌 환경에서 `const` 키워드는 크롬 49버전에서 부터 지원한다.

![image]({{site.baseurl}}/assets/img/Const.png)

이 문제는 strict mode가 아니여서 발생하는 문제이기 때문에 babel plugin을 통해서 해결할 수 있다. babel이 코드를 트랜스파일 할 때에 `@babel/plugin-transform-strict-mode` 플러그인을 추가함으로써, 해결 했다.

```
// .babelrc
{
  "presets": ["next/babel"],
  "plugins": ["@babel/plugin-transform-strict-mode", ...others]
}
```

<br>

### 2.attachShadow 함수 undefined 이슈

`attachShadow` 함수도 마찬가지로 크롬 44에서는 지원하지 않으며, 무려 63이상의 크롬 버전부터 지원한다.

![image]({{site.baseurl}}/assets/img/AttachShadow.png)

`attachShadow`의 경우에는 크롬 44버전의 엔진에서 해당 함수를 지원하지 않는 문제이기 때문에, polyfill을 추가함으로써 해결할 수 있다.

위 문제를 해결해줄 `@webcomponents/shadydom` 모듈을 설치해주자

```
yarn add @webcomponents/shadydom
```

이후에 해당 모듈을 포함시킬 polyfills 파일을 하나 만든후에 `@webcomponents/shadydom`를 import한다.

```javascript
// polyfills.js
import "@webcomponents/shadydom";
```

webpack 설정에서 해당 파일을 번들링에 포함될 수 있도록 entry로 지정한다.( 번들링 entry파일에서 직접 import 시켜서 번들링에 포함되도록 구성하여도 될 듯 하다.)

```javascript
// next.config.js
  ...
  webpack: (config, options) => {
    const originalEntry = config.entry;
    config.entry = async () => {
      const entries = await originalEntry();
      if (entries['main.js'] && !entries['main.js'].includes('./polyfill.js')) {
        entries['main.js'].unshift('./polyfills.js');
      }
    }
  }
  ...
```

<br>

### 3.Mobx 5+ 에서의 Symbol, Proxy 문제

모바일 웹뷰를 개발하면서 상태관리 라이브러리로 Mobx를 메이저 5버전대로 사용하고 있었는데, 해당 버전은 Symbol과 Proxy 기능이 지원가능한 환경에서 사용 가능했다. 우선 Symbol의 지원 버전을 살펴보자

![image]({{site.baseurl}}/assets/img/Symbol.png)

다행이 우리의 타겟으로 하는 44버전보다 이전 버전에서부터 사용이 가능하다!!!

그럼 이제 Proxy의 지원 버전을 확인해보자

![image]({{site.baseurl}}/assets/img/Proxy.png)

<br>

<p style="font-size:30px; font-weight: bold; text-align:center">지원되지 않는다.</p>

<figure class="image" style="text-align: center;">
  <img src="{{site.baseurl}}/assets/img/despair.jpeg" alt="">
</figure>

<br>

여기서 더 큰 문제는 `Symbol`과 `Proxy`와 같은 syntax는 attachShadow와 같은 함수와는 다르게 Polyfill로써 해당 기능을 대체할 수 없다. 실제로 에러를 뱉고 있는 mobx 라이브러리의 콘솔 로그를 살펴보면, 위 두 기능이 지원되지 않을 경우, mobx 4버전으로 다운그레이드 하라고 가이드한다.

그래서 버전을 내렸다...

<br>

### 4.React dangerouslySetInnerHTML 에서의 arrow function 이슈

기본적으로 구버전에서 지원하지 않는 기능들은 `core-js`와 `react-app-polyfill`을 통해서 해결 할 수 있고, arrow function과 같은 문제는 babel 플러그인을 통해서 해결 할 수 있다. 그럼에도 arrow function에 관한 에러가 발생하고 있었는데, 알고보니 `dangerouslySetInnerHTMl` 을 통해서 주입되는 script 코드에서 arrow function이 사용되고 있었고, 이는 문자열로 작성되어 있기 때문에 babel의 영향을 받지 않았기 때문이었다. 문제는 arrow function이 정의되어 있고 문자열로 감싸져있어서 babel을 통한 트랜스파일이 되지 못하는 이슈였기 때문에, 직접 일반적은 function의 형태로 수정하여 해결 했다.

<br>

### 5.css 미지원 이슈

위 문제들을 해결하고 나니 페이지가 렌더링 되기 시작했다. 그런데 무언가 내가 기대했던 다른 모습이었는데, 그 이유는 해당 버전(크롬 44)에서 지원하지 않는 css 속성의 사용 때문이었다. 대표적으로는 `var()`과 `grid`의 사용이었다.

![image](<{{site.baseurl}}/assets/img/var().png>)

![image]({{site.baseurl}}/assets/img/grid.png)

둘 다 지원하지 않는다. var() 같은 경우에는, var()함수로 사용하는 것이 아닌, 필요한 variable을 정의하고, 해당 css 파일을 import하여 사용하는 형태로 변경되어야 한다.

가령 아래와 같이 작성되어 있었다고 가정을 해보면

```css
// A.scss
:root {
  --primary: #fff;
}

// B.scss
.container {
  background-color: var(--primary);
}
```

위 코드는 아래와 같이 변경함으로써 대응할 수 있다.

```css
// A.scss
$--primary: #fff

// B.scss
@import "A.scss"
.container: {
  background-color: $--primary;
}
```

`grid` 또한 사용할 수 없기 때문에, 다른 형태로 재구성하여야만 한다.

<br>

## 결론

웹뷰 개발을 하기 이전까지만해도, 많은 케이스에서 최신 브라우저를 사용했었고 구버전을 대응한다는건 IE에 대한 크로스 브라우징과 IE8 ~ 11 버전에 대한 대응 정도였다. 특히 크롬 브라우저만을 사용한다고 생각하였을때는 이러한 고민은 전혀하지 않았었었는데, 웹뷰를 개발하게 되면서 생각보다 오래된 디바이스를 사용하는 사용자는 많았으며 이 사용자들에게도 서비스를 원할히 제공하기 위해서는 개발을 진행할때 넓은 범위의 버전 호환성을 챙겨가야했고 새로운 기술을 도입할때도 매우 신중해야한다는 것을 다시 한번 느끼는 계기가 됬다. (좋다고 막 가져다 쓰는게 마냥 좋은게 아니였다...)

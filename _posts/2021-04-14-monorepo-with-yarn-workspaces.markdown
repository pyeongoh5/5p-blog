---
layout: post
title: Monorepo With Yarn Workspaces
date: 2021-04-14 10:12:20 +0300
description: Monorepo가 무엇인지 알아보고, yarn의 workspaces 기능을 통해 Monorepo를 설정하는 방법에 대해 정리하였습니다. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Monorepo, yarn, workspaces]
---

# 1. Monorepo

`Monorepo`는 `Monolithic Repositories`의 약자로, 직역하면 `단단히 하나로 짜여진 저장소`를 의미한다. 일반적으로 보통 하나의 프로젝트는 하나의 레포지토리와 매칭이 되고 이런 형태의 프로젝트 구성을 `Multi-repo` 혹은 `PolyRepo`라고 하는데,

```
A 프로젝트 -> A 레포지토리
B 프로젝트 -> B 레포지토리
```

이와 반대로 여러 프로젝트(패키지)가 하나의 레포지토리에 저장되는 형태를 `Monorepo`라고 한다.

```
A 프로젝트
B 프로젝트  -> Mono 레포지토리
C 프로젝트
```

이해를 돕기 위해서 아래와 같이 그림을 그려 보았는데, 이해가 잘 될지는 모르겠다.

![image]({{site.baseurl}}/assets/img/monorepo-scheme.png)
<br><br>

# 2. Monorepo를 사용하는 이유

개인적으로 Monorepo를 통해 얻을 수 있는 가장 큰 장점은 하나의 IDE에서 여러개의 프로젝트를 관리하고 개발할 수 있는 것인데, 무엇이든 장점이 있으면 단점도 존재하기 마련이기 때문에 상황에 맞추어서 사용하면 좋을 것 같다. 아래에 Monorepo가 가질 수 있는 장점과 단점에 대해서 작성해 보았다.

### 2.1. 장점

- **하나의 레포지토리로 여러개의 프로젝트를 관리할 수 있다.**

  하나의 레포지토리가 여래개의 프로젝트(패키지)를 포함하고 있는것은 굉장히 큰 편의성을 가질 수 있다. 코드를 짜는 입장에서도 여러개의 IDE를 열거나, IDE에서 프로젝트를 스위치해가며 개발할 필요없이 하나의 IDE에서 하위폴더로 구분된 여러 패키지들의 코드를 작성할 수 있기 때문이다.

- **중첩되는 코드를 공통화할 수 있다.**

  `A, B, C, D` 패키지로 구성된 Monorepo가 있고, 여러 프로젝트들이 공통으로 사용해야하는 로직이 있을 때, 이를 쉽게 추가적인 `E` 패키지로 분리하고 `A~D` 패키지에서 `import` 하여 사용할 수 있다.

- **중첩되는 모듈은 하나만 설치하여 사용합니다.**

  `A, B, C, D` 패키지 모두 `react 16.11.0` 버전을 사용한다고 가정하면, 각각의 패키지에서 react 16.11.0 버전을 설치하는것이 아닌 root에 `react 16.11.0` 버전 하나를 설치하고 `A~D`에서 끌어다 사용한다.

### 2.2. 단점

- **의존성의 충돌이 생길경우, 정상동작 하던 패키지들도 동작하지 않을 수 있다.**

  특정 패키지가 다른 버전의 모듈을 필요로 하는 경우, 다른 버전의 모듈을 사용하는 패키지와 의존성 충돌이 발생할 수 있다.

- **여러 프로젝트를 하나의 레포지토리로 관리하기 때문에 오히려 관리가 어려울 수 있다.**

  Monorepo로 관리하는 패키지가 많지 않을 경우에는 해당되지 않지만, 관리하는 패키지가 증가함에 따라 오히려 가독성이나 여러가지 측면에서 비효율적이게 될 수 있다.

- **초기 프로젝트 설정이 오래걸린다.**

  Monorepo로 포함되는 모든 프로젝트를 사용한다면 상관없지만, 그 중 일부만 필요한 경우에도, Monorepo가 포함하는 모든 패키지의 전체적인 node_module 설치가 이루어 져야한다.
  <br><br>

# 3. Monorepo를 구성하는 여러가지 방법

Monorepo를 구성하는 방법은 아래와 같이 여러가지 방법들이 있으며, 현재 나는 yarn workspace 기능만을 사용하고 있다.

- **yarn workspace**

  `node package manager`중에 하나인 `yarn`에서는 (`npm`엔 없음!!) workspace 기능을 통해서 monorepo를 가능하게 해준다.

- **[Lerna](https://github.com/lerna/lerna)**

  yarn의 workspace와 마찬가지로 monorepo를 가능하게 해주는 기능을 제공함과 동시에, 설치된 의존성을 제거해주는 `clean` 기능이나 monorepo로 구성한 package를 npm 배포할 수 있는 기능들을 제공한다.

- **git submodule**

  기존 multi-repo로 관리되는 것들을 하나로 합쳐서, monorepo를 구성하려는 경우 git의 submodule을 사용하여 구성 할 수 있다. 단 git submodule은 위의 방법들과는 다르게 다수의 레포지토리를 관리하여야 하고 submodule이라는 것의 개념과 사용방법을 익혀야하기 때문에 어느정도 러닝 커브가 있다.
  <br><br>

# 4. yarn workspace로 Monorepo 구성하기

yarn의 workspace 기능을 통해서 monorepo를 구성하는 방식은 아주 간단한데, 우선

```
yarn init
```

을 통해서 프로젝트 초기화를 해주고, package.json에 아래와 같이 `private`과 `workspaces` 프로퍼티 값을 설정해주면 된다.

```json
// package.json
{
  ...
  "private": true,
  "workspaces": [
    "packages/*"
  ],
  ...
}
```

### 4.1. 의존성 관리 방식

yarn workspaces 기능은 포함된 모든 패키지들의 의존성을 분석하여 공통된 모듈을 root 경로의 node_modules 폴더 안에 설치하고 [symlink](https://ko.wikipedia.org/wiki/%EC%8B%AC%EB%B3%BC%EB%A6%AD*%EB%A7%81%ED%81%AC)를 통해 각각의 패키지로 연결시킨다. 따라서 100개의 패키지에서 같은 버전의 `A` 모듈을 사용한다고 해도, 100개의 `A` 모듈을 설치하는게 아닌 1개만 설치하고 symlink로 연결하여 사용한다.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vyozch6lxr5vzf6ed0fz.png)
<br><br>

# 5. yarn workspace를 사용한 이유

위에 설명했엇던 방식들(yarn workspace, lerna, git submodule)로 비교를 해보자면, lerna는 npm 배포와 관련된 유용한 유틸 기능들을 제공해주지만 나는 npm 배포가 필요없고, git submodule 또한 기존의 multi-repo로 관리되던 프로젝트를 monorepo로 구성하는 것이 아닌 시작부터 monorepo로 시작하였기 때문에, monorepo를 구성하는데만 필요한 기능인 yarn의 workspace 기능을 사용하였다. 만약 npm배포를 통해서 라이브러리를 관리하고자 한다면 lerna를 사용하는 것이 여러모로 유리한 측면이 있다.
<br><br>

# 6. 결론

Monorepo를 구성하는 형태는 다양할 수 있다. 예를들어 Client 코드와 Server 코드를 한번에 같이 개발 하고 싶은 경우, Client와 Server 프로젝트를 하나의 Monorepo로 구성하여 개발할 수 도 있고, 오픈소스 UI 라이브러리를 만든다고 할 경우, 전체 UI 요소를 담고있는 패키지로도 배포가능하면서, 특정 단위 (버튼, 차트, 그리드 등등)을 Monorepo의 패키지로 구성하여 개별적으로 설치하여 사용하게끔 구성하는 것도 가능하다. 그리고 이는 하나의 IDE만 실행시켜 구성된 많은 패키지들을 관리할 수 있다는 측면에서 매우 매력적이고 편리하지만 포함된 패키지의 갯수가 많아질 수 록, 또 패키지의 맥락(예를 들어 하나의 Monorepo안에 Client, Server, UI Component, Uitls 등 다른 맥락을 띈 패키지들)이 다양해 질 수록 코드는 다시 역으로 관리가 불편하고 힘들어 지는 경향이 있었다. 그럼에도 여전히 목적과 필요에 맞게 구성한 Monorepo는 개발의 편의성과 효율성을 가져다 주는 것 같다.

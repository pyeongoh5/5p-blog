---
layout: post
title: Micro Front-End 살펴보기
date: 2021-09-08 20:23:00 +0900
description: Micro Front-End
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [micro front-end]
---

최근에 리액트로 구성된 하나의 서비스 `A`를 다른 하나의 서비스 `B`로 이관해야 하는 작업을 해야할 일이 생겼다. 각각의 서비스는 별도의 레파지토리로 관리되고 별도의 배포 파이프라인을 통해 배포되고 서비스 되고 있다. 단순히 `A`의 코드를 `B`로 옮겨 `A`를 포함하는 커다란 `B`가 되도록 구성해야하는 것인지 아니면 `A`와 `B`를 모놀리식으로 구성하여 코드베이스를 공유할 수 있도록 해야 할 지 고민을 하고 리서치를 하다가 마이크로 프론트 엔드 아키텍처에 대해서 알게되었고, 해당 방법이 해결책이 될 수 있을 지 궁금해서 찾아보기 시작했다. (아직 해당 아키텍처를 적용해보진 못했다.) 이번 포스트에서는 마이크로 프론트엔드 아키텍처를 구성함으로써 위 문제를 해결 할 수 있는지 검토할 겸, 해당 아키텍처에서 조사해보는 시간을 가졌다.

## 마이크로 프론트엔드?

이전부터 백엔드에서 흔히 사용하는 아키텍처인 MSA(Micro Service Architecture)는 들어본 적은 있었지만, 해당 개념을 프론트엔드의 영역으로 확장시켜 생각해보지는 못했었다(이미 2016년 부터 해당 개념이 나오기 시작했었다..). 마이크로 프론트엔드는 마이크로 서비스처럼 전체 화면을 작동할 수 있는 단위로 나누어 개발한 뒤 서로 조립하는 방식이다.

## 멀티레포, 모놀리식, 그리고 마이크로

이전에는 하나의 서비스를 하나의 레포지토리, 하나의 배포 환경과 공간으로 구성하여 독립적으로 개발하여 서비스했었다. 그러다 보니, 서비스간 존재하는 공통 코드들임에도 불구하고 각각의 서비스별로 중복되어 가지고 있는 경우가 생기고, 많은 서비스를 제공하는 경우 관리 포인트도 늘어나게 되었다.
이후 이런 문제점을 해결하기 위해서 하나의 레파지토리에서 이러한 서비스들을 한 곳에 두어 관리하는 모놀리식이 관심을 받기 시작했다. 공통된 코드들을 별도의 패키지로 구분하고 이를 가져다 사용할 수 있으니 관리측면에서 유용했다. 실제로 모놀리식으로 구성해 사용하는데서 오는 편리함은 분명한 이점이 있었다. 하지만 모놀리식에 포함되는 패키지들이 늘어날수록 이전에는 보이지 않던 단점들이 생겨나기 시작했다. 설치한 라이브러리들에 대한 의존성이 새로 추가하려는 서비스에 맞지 않아 충돌이 발생하기도 하고, 너무 거대해져버린 모놀리식 구성은 점점 모듈 설치와 빌드 타임을 길어지게 만들었다. 또한 여러 패키지들이 서로 단단히 얽혀있기 때문에, 추후 필요에 의해 별도의 레파지토리로 다시 분리해내기는 거의 불가능에 가까웠다.
모놀리식 아키텍처에 포함되어 있는 서비스 `A`를 멀티레포로 운영되던 `B`서비스로 이관하기 위해서 방법을 조사하다가 `마이크로 프론트엔드 아키텍처`를 알게 되었다. 각각의 서비스로서 개발하여 하나의 서비스로 통합하는 형태의 해당 아키텍처가 현재 내가 겪고있는 문제의 해결점이 될 수 있을 거라고 생각했다.

## 그래서 마이크로 프론트엔드 아키텍처가 뭔데?

마이크로 프론트엔드 아키텍처는 마이크로 서비스 아키텍처가 가지는 장점처럼, 개발과 배포할 수 있는 단위를 더욱 더 작은 덩어리로 나누어, 각 개발팀(각각의 쪼갠 서비스들을 담당하는 팀들)이 서로간의 영역에 간섭없이 개발 할 수 있도록 하는 아키텍처이다.
![image]({{site.baseurl}}/assets/img/2021-09-08/Micro-Front-End.png)
위 그림에서 처럼 각각의 코드베이스는 다른 베이스들에 영향을 받지 않고 독자적으로 개발되고 빌드되어, 마지막에 하나의 프로덕션으로 통합되어 배포된다.

## 통합

각각의 서비스를 독립적으로 개발할 수 있다는 점은 매력적인것 같다. 하지만 여기서 풀어야할 한가지 숙제가 있다. 작은 단위로 나누어 개발한 서비스를 하나의 서비스로 통합해야 한다는 것인데 여기에는 여러가지 방법이있다. 서버 사이드에서 통합하는 방법도 있고, 자바스크립트를 통해서 런타임 시점에 통합하는 방법도 있으며, 이는 통합을 위한 앱 컨테이너를 필요로 한다.
![image]({{site.baseurl}}/assets/img/2021-09-08/Micro-Front-End-Composed.png)

## 결론

마이크로 프론트 엔드 아키텍처를 구성하기 위해서는, 각각의 서비스를 최종적으로 어떻게 합성해서 하나의 결과물을 도출할 것인지에 대해 결정되어 있어야 하고, 구현되어 있어야한다.
기존의 모놀리식 아키텍처가 가지는 의존성 충돌 문제나, 빌드 타임의 문제들을 효과적으로 해결할 수 있는 아키텍처가 될 수 있을것이라고 생각되고, 하나의 큰 통합서비스를 잘게 쪼개어 독립적으로 개발 할 수 있는 환경이 될 수 있을것 같다.
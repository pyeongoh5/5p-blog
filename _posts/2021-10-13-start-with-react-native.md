---
layout: post
title: React Native 시작해보기
date: 2021-09-08 20:23:00 +0900
description: React Native 개발환경 설정하기
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [react-native, react, Expo]
---

# React-Native 시작하기

이번 글은 React-Native 개발 환경을 세팅하는 방법에 관한 내용이다. 사실 단순히 공식 레퍼런스를 번역하는 수준이긴 하겠지만, 공부하는 겸해서 작성하게 되었다.

## 왜 React-Native?

`React-Native`는 페이스북에서 개발한 오픈소스 모바일 어플리케이션 프레임워크다. 한번의 코드 작성으로 Android, iOS등의 멀티 플랫폼을 지원한다. 물론 이런 프레임워크는 React-Native 하나 뿐만은 아닌데, 대표적으로 구글에서 개발한 [Flutter](https://flutter.dev/?gclid=Cj0KCQjw5JSLBhCxARIsAHgO2SfmDuJt-19KkNCa75bxB-ExKZjFqvRYdA0x39V1W9O44PXxNJjJ4zYaAtMhEALw_wcB&gclsrc=aw.ds)가 있다.
여러 선택지 가운데 React-Native를 선택해본건, 이미 React를 할 줄 알기 때문에 빠르게 습득이 가능할 것 같아서이다. 바로 시작해보자.

## 2가지 방법

공식 레퍼런스 사이트에서는 크게 2개의 환경 구성 방법을 제공한다. `Expo cli`, `React-Native cli`를 이용하는 방법이다. 해당 글에서는 `React-Native cli` 를 이용하며 `MacOS`에서 `Android`를 타겟으로 한다.

## 개발환경 설정

우선 개발환경을 세팅하기 위해선 `Node`, `Watchman`, `React-Native CLI`, `JDK`, `Android Studio`가 필요하다.

### 1. Node와 Watchman 설치

`Homebrew`를 통한 설치를 추천하며, `Homebrew`를 설치한 이후에 아래 커맨드를 통해 `Node`와 `Watchman`을 설치한다. 만약 `Node`가 이미 설치되어 있다면 `12버전 이상`이 설치되어 있는지 확인하자.
`watchman`은 파일시스템의 변경사항을 감지하기 위한 `Facebook`에서 개발한 도구이다.

```console
brew install node
brew install watchman
```

### 2. JDK(Java Development Kit) 설치

역시 `Homebrew`를 통해서 설치해주자

```console
brew install --cask adoptopenjdk/openjdk/adoptopenjdk8
```

만약 JDK가 이미 설치되어 있다면, `8버전 이상`인지 확인해주자.

### 3.Android 개발 환경

**3-1. Android Studio 설치**

[링크](https://developer.android.com/studio)를 통해서 Android Studio를 설치해주자.
설치 위자드에서 `아래 항목`을 체크하고 진행해야한다고 가이드되어 있으나 실제로 설치해봤을 때에는 별다른 체크항목없이 `Next` 버튼을 통해 설치하면 자동으로 잘 설치 되는듯 했다.

- Android SDK
- Android SDK Platform
- Android Virtual Device

**3-2. Android SDK**

Android SDK는 Android Studio를 설치하는 과정에서 같이 자동으로 설치된다. 하지만 네이티브 코드로 React Native 앱을 빌드하려면, `Android 10(Q) SDK`가 필요하고, Studio에 `SDK Manager`를 통해서 추가로 설치할 수 있다.

| SDK Manager는 다음 경로에서 찾을 수 있다. `Appearance & Behavior` → `System Settings` → `Android SDK`

Android Studio 설치 이후에,`SDK Manager`를 실행하면 아래와 같은 화면을 볼 수 있다.
![image]({{site.baseurl}}/assets/img/2021-10-13/sdk_manager.png)
좌측 하단의 `Show Package Details` 버튼을 눌러서 `Android 10 (Q)` 엔트리를 확장시켜 아래 항목들을 선택해준다.

- `Android SDK Platform 29`
- `Intel x86 Atom_64 System Image` 혹은 `Google APIs Intel x86 Atom System Image`

이후 `Apply` 버튼을 통해서 설치를 진행해주자.

**3-3. ANDROID_HOME 환경변수 설정하기**

React Native 도구는 네이티브 코드로 앱을 빌드하기 위해 몇 가지 환경 변수를 설정해야 한다.
사용하고 있는 shell의 종류에 따라 알맞은 설정파일을 사용하면된다. 만약 bash shell을 사용한다면 `~/.bash_profile` 혹은 `~/.bashrc`를, zshell을 사용한다면 `~/.zprofile` 혹은 `~/.zshrc`를 열어 아래 내용을 기입해주자

```
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

**3-4. React-Native CLI**

React-Native에는 CLI(Command Line Interface)가 내장되어 있는데, 이를 통해서 새로운 프로젝트를 생성할 수 있다. `npx`를 통해서 별도로 React-Native 설치 없이 사용 할 수 있다. 아래 명령어를 통해서 React-Native 프로젝트를 생성해보자.

## 새 어플리케이션 만들기

```c
// javscript
npx react-native init AwesomeProject
// typescript
npx react-native init AwesomeTSProject --template react-native-template-typescript
// 특정 버전으로 생성
npx react-native init AwesomeProject --version X.XX.X
```

위 커맨드라인을 통해서 프로젝트를 생성하게되면 기존 어플리케이션으로의 별도 통합과정을 필요로 하지 않는다.

## Android 기기 준비하기

개발을에 사용할 Android 기기가 필요한데, 실제기기를 사용해도되고, Android Studio의 Virtual Device를 사용해도 되며, 여기서는 Virtual Device를 사용한다.

Virtual Device는 AVD Manager를 통해서 사용할 수 있다.

![image]({{site.baseurl}}/assets/img/2021-10-13/avd_manager.png)

원하는 디바이스 종류를 고르고 `Next` 버튼을 누른뒤 `Q API Level 29` 이미지를 선택하자 (아래 이미지 참고)

![image]({{site.baseurl}}/assets/img/2021-10-13/android_image.png)

## React-Native 어플리케이션 실행하기

### 1. Metro 시작하기

우선 `Metro`를 실행시킬 필요가 있다. `Metro`는 React-Native와 함께 제공되는Javascript Bundler로, 다양한 옵션과 엔트리 파일들을 하나아ㅢ javascript 파일로 번들링 한다. `Metro`는 아래 코드로 실행할 수 있다.

```
  npx react-native start
```

실행하고 난 뒤 모습은 아래와 같다.

![image]({{site.baseurl}}/assets/img/2021-10-13/metro.png)

### 어플리케이션 실행하기

프로젝트의 내부 파일들을 보면 `package.json`가 있음을 확인할 수 있는데, 해당 파일에는 아래와 같은 `scripts` 코드를 확인 할 수 있으며, 각 플랫폼(android, ios)별로 어플리케이션을 실행할 수 있는 코드가 작성되어 있다.

```json
  "scripts": {
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "start": "react-native start",
    "test": "jest",
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx"
  },
```

여기서는 안드로이드를 대상으로 했기 때문에, `npx react-native run-android` script code를 실행했다.

## 실행결과

<span style="display: inline-block; width: 50%;">
![image]({{site.baseurl}}/assets/img/2021-10-13/run_application.png)
</span>

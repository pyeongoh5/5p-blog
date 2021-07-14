---
layout: post
title: Typescript, Any의 위험성
date: 2021-07-14 19:19:00 +0900
description: Typescript의 Any 타입의 위험성에 대해서 다루어 보았습니다.
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [typescript, any, 동적타입, 정적타입]
---

## 동적 타입과 정적 타입 그리고 javascript와 typescript

Javascript는 `동적 타입` 언어이다. `동적 타입` 언어는 런타임시에 타입이 결정되기 때문에 프로그램이 실행되는 런타임에 변수에 할당된 값이 자신의 타입이 된다. 즉 어떤 타입도 될 수 있다는 의미다. 코에 걸면 코걸이 귀에 걸면 귀걸이가 되는 셈이다.

우선 `동적 타입`의 큰 장점은 매우 편리하다는 것인데, 데이터를 다른 타입으로 가공한다고 하더라도 다시 자기 자신에게 할당 할 수 있기 때문이다. (타입을 신경쓰며 프로그래밍하지 않아도 된다.)

```javascript
var price = 3000;
price = `${price.toLocaleString()}원`;
```

위 예제에서 처음에 `number` 타입으로 초기화된 price 변수는 다음 라인 코드에 의해서 `string` 타입이 되었다.

위 코드를 정적타입 언어(여기서는 `Typescript`)를 사용해서 변경해보면 아래처럼 표현해 볼 수 있다.

```typescript
var price: number = 3000;
var priceString: string = `${price.toLocaleString()}원`;
```

물론 위에처럼 타입을 선언해주지 않아도 타입 추론에 의해서 자동적으로 number와 string 타입이 되기는 한다. 하지만 정적 타입에 비해 빠르게 작성할 수 있다. 또 정적 타입에 비해, 동적 타입은 변수를 다시 선언할 필요가 없어 네이밍을 추가로 하지 않아도 되는 아주 강력한(?) 장점도 있다...

상대적으로 편하지만, 반대로 위험하다. 그 이유는 동적 타입언어가 편한 이유와 동일한데, 어떤 타입이든 될 수 있기 때문에 타입 추론이 어려워지고 언어 차원에서의 타입 체크가 불가능하다 보니 예기치 못한 error를 발생시키기 쉽다.

```javascript
function IncreaseNumber(number) {
  return number + 1;
}
var number = 1000;
number = number.toLocaleString();
IncreaseNumber(number);
// "10001" 값이 반환됨
```

<p style="text-align: center; font-size: 30px">...???</p>

number를 왜 굳이 string으로 변환해서 넘겼을까? 코드를 보고 의도를 파악해본다면 당연히 number 타입을 `IncreaseNumber`함수의 인자로 넘겨야 하는것이 아닌가 싶을수 있다. 하지만 Javascript는 할당된 값에 의해 타입이 결정되기 때문에, 해당 변수의 타입이 number에서 string으로 변경된 것인지 확인 할 방법도 없고(console을 통해 찍어보는 것 이외에는..) 문제도 되지 않는다. 그렇기 때문에 코드를 작성하다보면 나도 모르게 실수로 타입이 변경되었을 수 있고, 잘못된 타입이 함수의 인자로 넘어갈 수도 있다.(심지어 에러도 발생하지 않고 "1000" + 1 이 되어, "10001"이 반환된다.) 위와 같은 상황은 Javascript를 사용하는 환경에서 매우 쉽고 빈번하게 발생할 수 있으며 application의 규모가 커지면 커질 수록 더 자주 발 생할 것이다.

위와 같은 위험성은 동적 타입의 언어가 주는 편리함에 대비해서는 너무나 큰 단점이다. 이런 문제를 해결하기 위해 여러 시도들이 있었으며, flow와 Typescript가 대표적이다.(여기서는 Typescript만을 다룬다.)
Typescript는 동적타입 언어가 가지는 여러가지 문제들을 해결하기 위해 등장했는데, 대표적으로 우리는 이제 Typescript의 타입 체크의 도움알 받으면서 코드를 작석 할 수 있다.

![image]({{site.baseurl}}/assets/img/2021-07-14/type-check.png)

하지만 정작 타입을 가능하게 해주는 Typescript를 사용하더라도 위에서 살펴본 동적 타입의 위험이 나타는 순간이 있다. 이제 그것에 대해 살펴보자.

## Any

이제야 본론으로 들어오는 것 같지만.. 동적 타입의 위험을 발생시키는 것은 바로 `any` 타입이다. Typescript 공식 Handbook에서는 아래와 같이 `any` 타입에 대해 설명하고 있다.

> In some situations, not all type information is available or its declaration would take an inappropriate amount of effort. These may occur for values from code that has been written without TypeScript or a 3rd party library. In these cases, we might want to opt-out of type checking. To do so, we label these values with the any type:

일부 상황에서는 모든 타입을 사용할수 없거나, 필요한 타입 선언에 너무 많은 부적절한 양의 노력을 필요로 하는 경우가 있는데, 예를 들어 타입스크립트 없이 작성된 코드나 3rd party 라이브러리를 사용하는 경우가 그러하며, 이런 경우 `any` 타입을 사용할 수 있다고 작성되어 있다.

`any` 타입은 모든 타입을 수용할 수 있는 특징을 가지기 때문에, 필연적으로 정작타입에서의 위험성을 동일하게 수반한다. 아래 예시를 보자:

```typescript
async function fetchData(url: string) {
  try {
    const response = await axios.get(url);
    return response.data;
  } catch (e) {
    throw e;
  }
}

function getPriceWon(price: number) {
  return `${price.toLocaleString()}원`;
}

const data = await fetchData("http://localhost:3000/prices");
const price = getPriceWon(data);
```

위 코드를 살펴보자 fetchData함수의 인자값인 `http://localhost:3000/prices`은 Mock server를 만들어 아래와 같은 값을 응답하도록 했다.

```json
// http://localhost:3000/prices 의 응답값
[
  { "id": 1, "price": 3000 },
  { "id": 2, "price": 5000 }
]
```

따라서 data 변수는 배열이고, 이 변수를 인자로 넘겨 받는 `getPriceWon` 함수의 파라미터 타입은 number 타입이기 때문에 TypeChecking에 의해서 에러를 뱉어낼 것 같지만, 실제로 에러는 감지되지 않는다. 실제로 위 코드에서는 에러가 발생하지도 않지만, 원하는 값을 얻을 수 없기 때문에 에러와 같다고 볼 수 있다.

무엇이 문제일까? 어느 부분에서 타입체크의 영향범위를 벗어났을까? 정답은 axios를 통한 비동기 함수 호출에 있다.

![image]({{site.baseurl}}/assets/img/2021-07-14/axios-return-type.png)

axios.get 함수의 반환 타입을 살펴보면 AxiosResponse<T>로 제네릭으로 되어 있다. 위 코드에서는 어떤 타입이 반환될 것인지 명시해주지 않았기 때문에 `any`타입으로 자동으로 지정되었다. 따라서 이를 반환 받은 data 변수 또한 자동적으로 `any`로 타입 추론된다.

![image]({{site.baseurl}}/assets/img/2021-07-14/axios-return-any.png)

위 경우처럼 위 같은 상황이나 써드파티 라이브러리의 함수 호출 등으로 인해 타입이 `any`로 추론되는 경우가 존재하며, 이를 인지하지 못하고 그대로 사용할 경우, 타입 추론이 제대로 되지 않기 때문(`any`로 추론되기 때문)에 타입 체크라는 Typescript의 장점이 사라지고 정적 타입의 위험성이 그 자리에 남게 된다. 위 코드를 타입적으로 안전하게 수정한다면 아래와 같이 수정할 수 있다.

```typescript
interface PriceInfo {
  id: number;
  price: number;
}
async function fetchData(url: string) {
  try {
    const response = await axios.get<PriceInfo[]>(url);
    return response.data;
  } catch (e) {
    throw e;
  }
}

function getPriceWon(price: number) {
  return `${price.toLocaleString()}원`;
}

const data = await fetchData("http://localhost:3000/prices");
const price = getPriceWon(data);
```

![image]({{site.baseurl}}/assets/img/2021-07-14/axios-return-right-type.png)

위와 같이 작성하면 보이는 것과 같이 올바르게 타입이 추론되고, 맞지않는 타입을 인자로 넘기고 있는 `getPriceWon` 함수에 대해 타입 체크 오류가 발생한 것을 확인할 수 있다.

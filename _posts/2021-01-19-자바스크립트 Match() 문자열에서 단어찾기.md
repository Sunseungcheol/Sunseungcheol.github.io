---
layout: post
categories: React.js
tags: [JavaScript, match]
---
알고리즘 문제를 풀다가 알게된 메서드인데 기억해두려고 다시 한번 정리한다.

match() 는 함수의 파라미터가 문자열과 일치하는 부분이 있는지 확인해 볼 수 있다.

공식 문서를 보면 아래와 매개변수에 대해 아래와 같이 설명하고 있다.

```
`regexp`
정규식 개체입니다.  RegExp가 아닌 객체 obj가 전달되면, new RegExp(obj)를 사용하여 암묵적으로 `RegExp`로 변환됩니다. 매개변수를 전달하지 않고 match()를 사용하면, 빈 문자열:[""]이 있는 `Array`가 반환됩니다.

```

결과값으로는 문자열이 파라미터와 일치하는 부분이 있다면 일치하는 문자들을 전부 배열 형식으로 반환하고 없으면 null 을 반환한다.

다만 정규식이 아닌 문자열인 경우 파라미터로 넣은 값이 문자열에 있다면 그 문자열을 반환한다.

이를 활용해 여러가지를 할 수 있다.

```js
const color = "color and color";

if (color.match("color")) {
  console.log("ok");
} else {
  console.log("no");
}

//ok
```

위와 같이 특정 문자열에 무언가가 포함되는지를 구분해 각기 다른 작업을 해줄 수 있다. 예를 들어 location.href 를 이용해 현재 주소에 따라 다른 작업을 해줄 수 도 있다.

또 위에 나온대로 정규식을 활용할 수 도 있다. 이를 이용해 특정 문자열에 원하는 특정 문자열이 몇개나 있는지 확인할 수 있다.

```js
const color = "color and color";

console.log(color.match(/color/gi));
//[ 'color', 'color' ]
```
---
layout: post
categories: React.js
tags: [JavaScript, padStart, padEnd]
---
`padStart()` 는 문자열의 시작부분을 기준으로 원하는 문자열의 길이만큼 문자열을 채워넣어준다. 첫번째 파라미터로 원하는 길이를, 두번째 파라미터로 원하는 문자열을 넣어주면 된다.

아래의 예시를 보면 이해가 쉽다.

```js
const str = "1001";
const str2 = str.padStart(6, "0");

console.log(str2);//001001

const str3 = "가나다";
const str4 = str3.padStart(10, "라라마마");

console.log(str4); //라라마마라라마가나다
```

`padEnd()` 메서드는 `padStart()` 와는 반대로 시작부분이 아닌 끝부분을 기준으로 문자열을 채워넣어 준다.

```js
const str = "1001";
const str2 = str.padEnd(6, "0");

console.log(str2);//100100
const str3 = "가나다";
const str4 = str3.padEnd(10, "라라마마");

console.log(str4); //가나다라라마마라라마
```
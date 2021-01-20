---
layout: post
categories: React.js
tags: [JavaScript, include]
---
`include` 메서드는 배열이 특정요소를 가지고 있는지 판별하는데 `boolean` 값을 리턴한다.

`include` 는 두개의 파라미터를 받는데 첫번째는 탐색할 요소, 두번째는 검색을 시작할 위치(index) 이다. 이때 두번째 값을 보내지 않으면 기본값은 0 으로 설정된다.

또 만약 두번째 값으로 음수를 보내면 array.length + 보낸 값을 asc(아스키코드) 로 검색한다.

간단한 예제를 보자.

```js
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false 
[1, 2, 3].includes(3, 3);  // false index 3 부터 검색하기 때문에 false
[1, 2, 3].includes(3, -1); // true  array.length + (-1) 이기 때문에 true
[1, 2, NaN].includes(NaN); // true
```

위에서 두번째 값으로 음수를 보내는 경우 array.length + 보낸 값 으로 계산한다고 했는데 만약 이때 `계산된 값 <= -1 * array.length` 이라면 전체배열을 검색한다.

---
layout: post
categories: React.js
tags: [JavaScript, Date]
---
자바스크립트에서 오늘 날짜를 불러오기 위해서 new Date() 를 사용한다.

```js
let toDay = new Date();
console.log(toDay);
//Tue Feb 09 2021 23:23:39 GMT+0900 (대한민국 표준시)
```

이를 이용해 각 년도, 월, 날짜, 요일등을 가져올 수 있는데 이때 월은 +1 을 해줘야 한다.

```js
let year = toDay.getFullYear();
let month = toDay.getMonth() +1;
let date = toDay.getDate();
let day = today.getDay();
```

또 toLocaleDateString() 을 이용해서 더 보기 쉽게 가져올 수 있다.

```js
console.log(toDay.toLocaleDateString());
//2021. 2. 9.
console.log(toDay.toLocaleTimeString())
//오후 11:23:39
console.log(toDay.toLocaleString())
//2021. 2. 9. 오후 11:23:39
```

예전에 이걸 모르고 replace 를 사용하던가 따로 붙여 넣었었다..
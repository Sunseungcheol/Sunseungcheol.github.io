---
layout: post
categories: javascript
tags: [javascript]
---

es6부터 사용된 `const`와 `let`은 `var`과 비교했을 때 변수의 유효범위에 차이가 있다.
기존 `var`이 함수단위 스코프였다면 `const`와 `let`은 {} 단위이다.

```javascript
//블럭
{
    const name = "sun";
    console.log(name);//sun 출력
}
console.log(name);//not defined

//밖에서 안으로
let age = 28;
{
    age++;
    console.log(age);//29
}
console.log(age);//29

```

이와 같이 es6문법에서의 `const`와 `let`은 {} 을 벗어나면 인식하지 못하는 모습을 보인다.
하지만 `var`의 경우 함수단위 스코프를 갖고 있기 때문에 `const`, `let`과는 달리 단순 {} 만으로는 영향을 받지 않는다.

```javascript
//var 함수 스코프

var c = 0;
{
    c++;
    console.log(c);//1
}

{
    var d=0;
    console.log(d);//0
}

console.log(d);//0
```

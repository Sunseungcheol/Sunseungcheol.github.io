---
layout: post
categories: React.js
tags: [JavaScript, this]
---

자바스크립트에서 함수를 호출할 때 arguments 객체와 this 가 함수 내부에 전달된다. 
이때 this 바인딩 ( this 가 가리키는 값 ) 은 함수 호출방식에 의해 동적으로 결정된다.

기본적으로 함수에서 this 는 전역객체 즉, window 가 바인딩된다.

```js
function thisA() {
  console.log(this);
}

thisA();//window
```

하지만 객체의 경우에는 그 객체가 바인딩된다.

```js
const thisB = {
  a: 100,
  b: function () {
    console.log(this);
  },
};

thisB.b();//{a: 100, b: ƒ}
```

다만 화살표 함수를 사용시에는 조금 다른 결과를 보여주는데 화살표 함수의 this 는 한단계 위의 상위객체를 가리킨다. 즉 위와 같은 객체에서 화살표 함수로 this 를 호출하면 window 가 표시된다.

```js
const thisB = {
  a: 100,
  b:() => {
    console.log(this);
  },
};

thisB.b();//window
```

어떤 변수안에 위의 thisB.b() 를 담은 후 변수를 호출하면 window 가 나오게된다. 이미 그 변수에 담는 시점에 객체가 아닌 함수로써 담아지기 때문이다.

```js
let b = thisB.b;
b();//window
```


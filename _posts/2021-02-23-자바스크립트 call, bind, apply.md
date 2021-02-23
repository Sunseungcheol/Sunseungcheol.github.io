---
layout: post
categories: React.js
tags: [JavaScript, call, bind, apply]
---
call, bind, apply 는 javascript 함수의 기본 메소드 중 하나이다.
보통 함수를 호출할 때는 함수 뒤에 괄호 ()를 붙이거나 call, apply, bind 를 붙이는 방법이 있는데 먼저 call 과 apply 를 확인해보자.

```js
let ex = function (a, b, c) {
  return a + b + c;
};

ex(1,2,3);//6
ex.call(null,1,2,3)//6
ex.apply(null, [1,2,3])//6
```

위의 call 과 apply 를 보면 둘다 첫번째 인자로 null 이 들어가는데 null 은 this 를 대체해준다.
name 과 그 name 을 콘솔로 찍어주는 함수를 가지고 있는 객체를 만들어 확인해보자. 

```js
let ex = {
  name: "sun",
  callName: function () {
    console.log(this.name);
  },
};

let ex2 = { name: "kim" };

ex.callName();//sun
ex.callName.call(ex2);//kim
ex.callName.apply(ex2);//kim
```

call 이나 apply 를 쓰는 예 중 하나로 함수의 arguments 를 조작할 때 사용한다.
arguments 는 함수에 받은 인자를 유사배열 형식으로 반환하는데 유사배열 형식이기 때문에 배열의 메소드를 사용할 수 없다. 
이때 call 이나 apply 를 사용하면 배열의 메소드를 사용할 수 있다.


```js
function ex() {
  console.log(arguments);
}

ex(1, 2, 3);//Arguments(3) [1, 2, 3, callee: ƒ, Symbol(Symbol.iterator): ƒ]
```

```js
function ex2() {
  console.log(arguments.join());
}

ex2(1, 2, 3);//Uncaught TypeError: arguments.join is not a function
```

위와 같이 join() 을 사용시 에러가 발생한다.
이때 prototype 을 이용해 call, aplly를 써주면 사용이 가능하다.

```js
function ex3() {
  console.log(Array.prototype.join.call(arguments));
}

ex3(1, 2, 3);//1,2,3
```


마지막으로 bind 함수는 함수가 가리키는 this 만 바꿔주고 호출 하지는 않는다.

```js
let ex = {
  name: "sun",
  callName: function () {
    console.log(this.name);
  },
};

let ex2 = { name: "kim" };
let ex3 = ex.callName.bind(ex2);

ex3();//kim
```

bind 를 사용할 때는 변수에 담는 형식으로 사용해야 되는 것 같다.
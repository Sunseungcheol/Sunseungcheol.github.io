---
layout: post
categories: React.js
tags: [JavaScript, 호이스팅, hoisting]
---
호이스팅이 어떤 건지는 알겠는데 정리해서 말하려니 잘 안되 정리해보려고 한다.

자바스크립트에서 호이스팅은 코드에 있는 변수 또는 함수를 코드 상단으로 끌어올리는 것을 말한다. 
다만 여기서 주의해야 할점은 실제로 위로 옮겨지는 것이 아니라는 점이다. MDN 문서를 보면 아래와 같이 설명한다.

```
예를 들어, 호이스팅을 변수 및 함수 선언이 물리적으로 작성한 코드의 상단으로 옮겨지는 것으로 가르치지만, 실제로는 그렇지 않습니다. 변수 및 함수 선언은 컴파일 단계에서 메모리에 저장되지만, 코드에서 입력한 위치와 정확히 일치한 곳에 있습니다.
```

예제를 확인해보자

```js
console.log(a);//undefined
var a = 1;
console.log(a);//1
```

위와 같이 첫번째 콘솔은 ReferenceError 가 아닌 undefined 가 나온다. 이유는 변수의 선언부를 자바스크립트 내부에서 위로 끌어올렸기 때문이다. 위의 예제는 결국 아래와 같이 동작한다.

```js
var a;
console.log(a);//undefined
a = 1;
console.log(a);//1
```

다만 var 이 아닌 let, const 의 경우 호이스팅이 되지 않는다.


함수 또한 hoisting 된다.
```js
a();//a

function a() {
    console.log('a');
}

a();//a


///호이스팅
function a() {
    console.log('a');
}

a();
a();
```

다만 함수표현식 - 변수에 함수를 할당해주는 표현식 의 경우와  new Function 을 이용한 객체 생성의 경우 호이스팅이 되지 않는다.
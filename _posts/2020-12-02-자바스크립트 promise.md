---
layout: post
categories: javaScript
tags: [javaScript]
---

es6에 들어오며 promise 객체가 JavaScript 표준내장객체로 추가 되었다.
MDN 문서를 보면 `Promise 객체는 비동기 작업이 맞이할 미래의 완료 또는 실패와 그 결과 값을 나타냅니다` 라고 설명되 있다.

promise 객체는 생성자를 통해 만들 수 있는데 이때 생성자의 인자로 executor 함수를 이용해야한다. executor 함수는 resolve 와 reject를 인자로 가지는데 이 둘은 모두 함수이다.
생성자를 통해 promise 객체를 만드는 순간 대기 -pending 상태로 돌입하게 되고, pending 상태로 돌입 후 executor 함수 인자 중 하나인 resolve를 호출 했을시에는 이행 -fulfilled 상태가, reject 함수를 호출 시에는 거부 -rejected 상태가 된다.

그 후 생성한 promise 객체가 fulfilled 상태가 되는 시점에 객체.then() 안에 설정한 callback 함수가 실행된다.

```javascript
const p = new Promise((resolve, reject)=>{
  // pending 상태
  setTimeout(()=>{
    resolve();//fulfilled 상태
  },1000)
});

p.then(()=>{
  console.log('1000ms 후 fulfilled 되었습니다.');//콜백
})

```

위에서는 promise 객체와 then을 바로 밑에 둬서 큰 상관이 없지만 보통 실무에서 사용시에는 promise 객체가 필요한 부분에서 바로 생성하고 then과 묶어서 사용하는 방식으로 사용한다.
이는 then을 설정하는 시점을 정확히 하고, 함수 실행과 동시에 pending이 시작하도록 하기 위함으로, 아래와 같이 promise 객체를 생성하면서 리턴하는 함수 p()를 만들어 함수 p() 실행과 동시에 then을 설정한다. 

```javascript
function p() {
  return new Promise((resolve, reject)=>{
    // pending 상태
    setTimeout(()=>{
      resolve();//fulfilled 상태
    },1000)
  });
}

p().then(() => {
  console.log('1000ms 후 fulfilled 되었습니다.');//콜백
})
```

then과 마찬가지로 promise 객체가 rejected 되는 시점에는 객체.catch() 안에 설정한 callback 함수가 실행된다. 이때 catch()는 체이닝 방식으로 설정이 가능하다.

```javascript
function p() {
  return new Promise((resolve, reject) => {
    // pending 상태
    setTimeout(() => {
      reject(); //rejected 상태
    }, 1000);
  });
}

p()
  .then(() => {
    console.log("1000ms 후 fulfilled 되었습니다."); //콜백
  })
  .catch(() => {
    console.log("1000ms 후 rejected 되었습니다."); //콜백
  });
```

또한 resolve와 reject 함수를 실행할 때 인자를 보내면, then과 catch에서 인자를 사용 가능한다. 
보통 Try/Catch로 예외 처리를 할때와 같이 promoise의 catch부분에서 또한 오류 메시지를 인자로 보내는 경우가 많다. 이때는 주로 Error 객체를 만들어 보낸다.

```javascript
function p() {
  return new Promise((resolve, reject) => {
    // pending 상태
    setTimeout(() => {
      reject(new Error("error")); //rejected 상태
    }, 1000);
  });
}

p().catch((error) => {
  console.log(`${error} 1000ms 후 fulfilled 되었습니다.`); //Error: error 1000ms 후 fulfilled 되었습니다.
});
```
---
layout: post
categories: javaScript
tags: [javaScript]
---
Promise 로 연속된 처리를 할 때와 async 로 처리할 때의 차이점을 알아보자.
먼저 Promise 객체를 리턴하는 함수를 하나 생성해줬다.

```javascript
function p(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(ms);
    }, ms);
  });
}
```

먼저 앞서 공부했던 promise 를 이용한 연속된 처리를 살펴보면 체이닝 형식으로 진행된다.

```javascript
p(1000)
  .then(() => p(1000))
  .then(() => p(1000))
  .then(() => {
    console.log("3000ms 후에 실행");
  });
```

async await를 사용하여 연속된 처리를 할때는 아래와 같은 모습을 보이는데 결과는 위와 같다.

```javascript
(async function main() {
  await p(1000);
  await p(1000);
  await p(1000);
  console.log("3000ms 후에 실행");
})();
```

await의 경우 첫번째 줄이 끝나야 두번째 줄이 실행되고 두번째줄이 끝나면 세번째 줄이 실행하게 된다. 움직이는 로직은 같지만 사용하는 방식에서 조금 차이가 있는데 개인적으로 await 가 조금 더 가독성이 좋아보인다.

다음으로 Promise.all 과 Promise.race 를 async 를 이용하는 방법을 보면 promise 를 사용했을 때와 크게 다르지 않다.

```javascript
(async function main() {
  const results = await Promise.all([p(1000), p(3000), p(3000)]);
  console.log(results);
})(); //[ 1000, 3000, 3000 ] 3초후 실행

(async function main() {
  const results = await Promise.race([p(1000), p(3000), p(3000)]);
  console.log(results);
})(); //1000 1초후 실행
```

비동기 처리를 하는데 있어서 promise, async await 는 매우 중요하다. 잊지말고 계속 공부하자.
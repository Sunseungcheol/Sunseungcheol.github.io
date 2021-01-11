---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, redux-saga, Generator 문법]
---
redux-saga 는 redux-thunk 다음으로 많이 사용되는 라이브러리이다.

리덕스 사가는 액션을 모니터링 하다가 특정 액션이 발생하면 이에 따라 특정 작업을 하도록 해준다. 

리덕스 사가를 사용하면 thunk 를 사용했을 때 하기 어려운 작업들도 가능하다.

1. 비동기 작업을 진행 할 때 기존 요청 취소
 - ex
 - 기존에 작업하던 비동기 작업을 특정 액션에 따라 중지
 - 동일한 비동기 작업 시 첫번째 실행되거나 마지막 실행된 작업만 하도록 설정

2. 특정 액션 발생시 다른 액션 디스패치, 자바스크립트 코드 실행
 - thunk 는 이와 같은 작업을 위해 함수타입의 값을 디스패치, saga 에서는 순수 액션 객체를 사용하면서 수행 가능

3. 웹소켓 사용시 channel 기능 사용하여 효율적 코드 관리
4. 비동기 작업 실패 시 재시도 기능 구현


리덕스 사가에서는 자바스크립트의 <a href='https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Generator'>Generator</a> 라는 문법을 사용한다.

Generator 를 사용하면 여러 작업들을 할 수 있는데 예를 들어


1. 함수의 흐름을 특정 구간에서 멈춰놓았다가 다시 실행 가능
2. 결과값을 여러번 내보낼 수 있음

```javascript
function testFunction() {
  return 1;
  return 2;
  return 3;
  return 4;
  return 5;
  return 6;
}
```

예를 들어 위와 같은 함수는 무조건 `1`만을 반환한다.
하지만 Generator 를 사용하면 

```javascript
//제너레이터 문법 사용시 * 을 붙인다.
function* generatorFunction() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;

  return 5;
}
```

위와 같이 yield 값을 여러 번 내보내거나 yield 를 기준으로 함수를 멈춰놨다가 다시 실행하는 기능을 사용할 수 있다.

조금 더 자세히 보자면 아래와 같은데

```javascript
function* generatorFunction() {
  console.log("하이");
  yield 1;
  console.log("두번째");
  yield 2;
  console.log("세번째");
  yield 3;
  return 4;
}
const generator = generatorFunction()

generator;
generatorFunction {<suspended>}

generator.next();
하이
{value: 1, done: false}
generator;
generatorFunction {<suspended>}

generator.next();
두번째
{value: 2, done: false}

generator.next();
세번째
{value: 3, done: false}

generator.next();
{value: 4, done: true}

generator
generatorFunction {<closed>}

generator.next()
{value: undefined, done: true}
```

위의 내용을 보면 이해가 쉬운데 generator 를 호출했을 때 `{<suspended>}` 라고 나온다 이는 함수가 잠시 멈춰있다는 것을 의미하고 `generator.next()` 를 호출하자 처음에는 value : 1, done: false 가 나오게 된다. 

이후 다시 generator 를 호출해 상태를 확인해 보면 여전히 `{<suspended>}` 가 나오고 계속 해서 `.next()` 를 사용하면 yield 를 기준으로 콘솔이 찍힌 후 yield 값이 리턴되는 것을 알 수 있다.

yield 값을 모두 지나고 마지막에 return 값 까지 온후에는 done: true 로 변경되어있고 generator 호출해 상태를 확인해보면 `{<closed>}` 라고 적혀있다. 그 후에는 계속 호출해봐도 value: undefinde, done: true 가 나오는 것을 확인할 수 있다.

또 다른 예시로는

```javascript
function* testGeneratorFunction() {
  console.log("제너레이터 시작");
  //.next() 를 호출할 때 넣어주는 파라미터를 a 변수에 담을 수 있음
  let a = yield;
  console.log("a 값을 받음");
  let b = yield;
  console.log("b 값을 받음");
  return a + b;
}
const test = testGeneratorFunction()

test.next()
제너레이터 시작
{value: undefined, done: false}

test.next(3)
a 값을 받음
{value: undefined, done: false}

test.next(5)
b 값을 받음
{value: 8, done: true}
```

위와 같이 되는 것을 볼 수 있는데 이전의 예시와 비교했을 때 yield 가 바로 value 로 리턴된 반면 지금의 예시는 값을 저장하고 반환하지는 않는 것을 볼 수 있다.

```javascript
function* testGeneratorFunction() {
 let result =0;
 //result 에 next 로 받아온 값을 더해주고 결과값으로 result를 주겠다.
 while(true){
   result += yield result;
 }
}

const test = testGeneratorFunction()
//먼저 함수 시작
test.next()
{value: 0, done: false}

test.next(10)
{value: 10, done: false}

test.next(3)
{value: 13, done: false}

test.next(12)
{value: 25, done: false}
```

또 위와 같은 작업도 가능하다. 잘 고민해보면 여러가지 응용이 가능할 것 같다.
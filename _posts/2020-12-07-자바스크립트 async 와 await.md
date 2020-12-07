---
layout: post
categories: javaScript
tags: [javaScript]
---

async await 는 es2017에서 추가되었는데 MDN 문서를 살펴보면 async 에 대해 이렇게 설명하고 있다.

`async function 선언은 AsyncFunction객체를 반환하는 하나의 비동기 함수를 정의합니다. 비동기 함수는 이벤트 루프를 통해 비동기적으로 작동하는 함수로, 암시적으로 Promise를 사용하여 결과를 반환합니다. 그러나 비동기 함수를 사용하는 코드의 구문과 구조는, 표준 동기 함수를 사용하는것과 많이 비슷합니다.`

async 는 JavaScript에서 비동기 방식을 처리하기 위한 방법 중 하나이며 `async function 함수명(){}`
`const 함수명 = async() => {}`
와 같은 방식으로 사용한다.

다음 예시를 통해 알아보자.
먼저 Promise 객체를 리턴하는 함수를 생성한다.

{% highlight ruby %}
function p(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(ms);
    }, ms);
  });
}
{% endhighlight %}

Promise 객체를 이용해 비동기 로직을 수행할 때는 아래와 같이 사용했었다.

{% highlight ruby %}
p(1000).then((ms) => {
  console.log(`${ms} ms 후에 실행된다.`);
});

{% endhighlight %}

위와 같이 resolve 됐을 때 then 으로 넘어가 resolve 함수의 인자를 받아 콘솔로 사용이 가능했었는데, 아래와 같이 await 를 사용하면 비동기 처리가 끝날 때 까지 로직이 넘어가지 않고 기다렸다가 resolve 이후 인자값을 return 받게 된다.
단, await를 사용하기 위해서는 반드시 async 함수 안에서 사용되어야 한다.

{% highlight ruby %}
async function main() {
  const ms = await p(1000);
  console.log(`${ms} ms 후에 실행된다.`);
}
main();
//1000 ms 후에 실행된다.
{% endhighlight %}

Promise 객체가 resolve 가 아닌 reject 가 된 경우에는 try catch 문을 사용한다. try catch 를 사용할 때도 동작이 끝난 후 무언가를 실행해야 한다면 finally 를 사용하면 된다.

{% highlight ruby %}
function p(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(new Error("reason"));
    }, ms);
  });
}

async function main() {
  try {
    const ms = await p(1000);
    console.log(`${ms} ms 후에 실행된다.`);
  } catch (error) {
    console.log(error); 
  } finally {
    console.log('끝');
  }
}

main()//Error: reason 끝
{% endhighlight %}

Promise 객체를 new 로 생성하지 않고 async function 자체를 사용하는 방법도 가능한데 이때 async function 에서 return 되는 값은 Promise.resolve 함수로 감싸서 리턴되게 된다. 앞서 배웠듯이 Promise 객체에 값이 들어오면 바로 resolve 된다.

아래의 asyncP() 함수 또한 async 함수이기 때문에 위 예제의 p() 함수를 await 로 사용할 수 있다. p() 함수의 reject 부분은 resomlve(ms) 로 다시 바꾼 후 아래 예제를 확인해보자.

{% highlight ruby %}
async function asyncP() {
  const ms = await p(1000);
  return "Sun" + ms;
}

async function main() {
  try {
    const name = await asyncP();
    console.log(name);
  } catch (error) {
    console.log(error); 
  } finally {
    console.log('끝');
  }
}

main(); //Sun : 1000 끝 (1초 후 실행된다.)
{% endhighlight %}

만약 resolve 가 아닌 reject 가 된다면 위의 asyncP() 의 return 부분까지 가지 못하고 main() 의 catch 부분에서 잡히게 된다. 
이를 이용해 에러처리를 어디서 할지에 대해 결정할 수 있다. 위와 같은 방법으로 main() 함수에서 처리할 수도 있지만 asyncP() 의 `const ms = await p(1000);` 부분을 try catch 로 감싸 사용할 수도 있다.


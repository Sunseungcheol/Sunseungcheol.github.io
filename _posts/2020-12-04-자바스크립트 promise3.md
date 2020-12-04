---
layout: post
categories: javaScript
tags: [javaScript]
---

promise 객체 사용시 new 를 사용하지 않고 만드는 방법들이 몇가지 있다.
그 중 처음은 Promise.resolve(value) 를 사용하는 방법인데 resolve의 인자로는 일반 객체와 promise 객체 두가지를 모두 보낼 수 있다.
먼저 promise 객체를 넣는 방법부터 알아보자.

{% highlight ruby %}
Promise.resolve(
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve();
    }, 1000);
  })
);
{% endhighlight %}

위와 같이 Promise.resolve 의 객체로 새로운 promise 객체를 만들어 넣었다. 이 경우 then 은 인자값의 resolve 가 호출되는 순간 불리게 된다.

{% highlight ruby %}
Promise.resolve(
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("foo");
    }, 1000);
  })
).then((data) => {
  console.log(
    "프로미스 객체인 경우, resolve 된 결과를 받아 then 이 실행된다.",
    data
  );
});
{% endhighlight %}

다음은 일반 객체를 넣었을 경우인데 이 경우에는 then 이 바로 실행된다는 점이 다르다.

{% highlight ruby %}
Promise.resolve("bar").then((data) => {
  console.log("then 메서드가 없는 경우, 바로 fulifilled 된다", data);
});
{% endhighlight %}

즉, 받는 객체가 promise 객체인지 일반객체인지 정확히 모를 때 Promise.resolve() 를 사용하면 유용하게 사용할 수 있다.

위의 resolve와 마찬가지로 reject 또한 Promise.reject도 같은 방식으로 사용할 수 있다.

{% highlight ruby %}
Promise.reject(new Error("reason"))
  .then((error) => {})
  .catch((error) => {
    console.log(error);
  });
{% endhighlight %}

Promise 객체가 하나가 아니고 여러 개일 때는 배열로 만들어 쓸 수 있는데, 이때는 Promise.all() 을 사용한다.  Promise.all() 는 인자로 받은 배열의 모든 promise 객체들이 fulfilled 되었을 때, then 함수를 실행하는데, then 함수의 인자로 배열의 promise 객체들의 resolve 값을 다시 배열로 돌려준다.

{% highlight ruby %}
function p(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(ms);
    }, ms);
  });
}

Promise.all([p(1000), p(2000), p(3000)]).then((messages) => {
  console.log("all clear", messages);//all clear [ 1000, 2000, 3000 ]
});

{% endhighlight %}

위의 로그를 보면 Promise.all 의 then 은 Promise.all 의 인자 중 가장 늦게 실행되는 p(3000) 이 끝난 3초 후 실행된다. 이때 앞서말했던 것과 같이 then 에서 받은 인자가 배열 형식으로 들어와 있는 것을 볼 수 있다. 
비동기 작업을 사용할 때, 모두 실행된 후 무언가를 해야될 때가 있을 때 많이 사용하는 방법이다.

Promise.all 과 비슷하지만 전부 실행된 다음이 아닌 가장 먼저 실행된 객체를 기준으로 then 함수가 실행되는 Promise.race 가 있다. Promise.race 는 then 함수의 인자로 가장 먼저 fulfilled 된 promise 객체의 resolve 값을 돌려준다.

{% highlight ruby %}
function p(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(ms);
    }, ms);
  });
}

Promise.race([p(1000), p(2000), p(3000)]).then((message) => {
  console.log("clear", message);//clear 1000
});
{% endhighlight %}

위의 Promise.race 의 로그를 보면 가장 빨리 완료된 p(1000)의 인자가 들어와 있는 것을 확인할 수 있다. 당연히 1초 후 로그가 찍힌다.
Promise 개념은 JavaScript 의 비동기 작업을 할 때 매우 중요하다. 계속 숙지하자
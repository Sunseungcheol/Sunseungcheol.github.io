---
layout: post
categories: javaScript
tags: [javaScript]
---

promise 객체 사용시 fulfilled 되거나 rejected 된 후 최종적으로 실행할 것이 있다면, .finally() 를 이용하는데 이때 인자로 함수를 넣어준다.

```javascript
function p() {
  return new Promise((resolve, reject) => {
    // pending 상태
    setTimeout(() => {
      reject(new Error("error")); //rejected 상태
    }, 1000);
  });
}

p()
  .then((message) => {
    console.log(`${message} 1000ms 후 fulfilled 되었습니다.`); //콜백
  })
  .catch((error) => {
    console.log(`${error} 1000ms 후 rejected 되었습니다.`); //콜백
  })
  .finally(() => {
    console.log("end");
  });

```

위와 같이 모든 로직이 끝난 후에는 finally()로 들어와 실행하게 된다.


es6 이전까지는 비동기 작업을 할 때 callback 함수를 인자로 넣어 로직이 끝난 후 callback 함수를 호출하는 방식으로 사용했다.

```javascript
function c(callback) {
  setTimeout(() => {
    callback();
  }, 1000);
}

c(() => {
  console.log("1000ms 후 callback"); //1초후
});

c(() => {
  c(() => {
    console.log("2000ms 후 callback"); //2초후
  });
});

c(() => {
  c(() => {
    c(() => {
      console.log("3000ms 후 callback"); //3초후
    });
  });
});
```

위와 같이 콜백함수 안에 콜백함수를 넣는 방법으로 사용했는데 이는 가독성면에서 조금 떨어지게 된다. 
promise를 사용하면 체이닝방식이 가능해 조금 더 알아보기 쉽게 사용할 수 있다. 아래는 es6 이전에 사용하던 방법이다.

```javascript
function c(callback) {
  setTimeout(() => {
    callback();
  }, 1000);
}

c(() => {
  console.log("1000ms 후 callback"); //1초후
});

c(() => {
  c(() => {
    console.log("2000ms 후 callback"); //2초후
  });
});

c(() => {
  c(() => {
    c(() => {
      console.log("3000ms 후 callback"); //3초후
    });
  });
});
```

위와 같이 콜백함수 안에 콜백함수가 들어가며 가독성이 떨어진다. 이제 promise를 사용한 방법을 확인해보자.

```javascript
function p() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve();
    }, 1000);
  });
}

p().then(() => {
  return p();
});
```

위의 함수를 보면 .then()안에서 p()를 다시 리턴한다. 즉 1초후 resolve가 실행되는 순간 다시 한번 p를 호출하는 것이다. 다시 호출된 p가 끝날 때 다시한번 then을 넣어줄 수 있다.

```javascript
function p() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve();
    }, 1000);
  });
}

p()
  .then(() => {
    return p(); //2초후
  })
  .then(() => p()) //3초후
```

위와 같이 체이닝 방식을 사용하기 때문에 가독성면에서 훨씬 좋다.

```javascript
p()
  .then(() => {
    return p(); //2초후
  })
  .then(() => p()) //3초후
  .then(p)
  .then(() => {
    console.log("4000ms 후 fulfulled");
  }); //4초후
```

위의 .then 뒤의 방식들은 모두 같은 결과를 나타낸다. 쓰는 방법의 차이일 뿐이다.


---
layout: post
categories: React.js
tags: [React.js, useState]
---

useState 를 통해 가장 기본이 되는 버튼을 눌렀을때 값이 변경되는 것을 만들어 본다.

```javascript
//App.js
import React, { Component } from "react";
import Counter from "./Counter";

function App() {
  return <Counter></Counter>;
}

export default App;

//Counter.js
import React, { useState } from "react";

function Counter() {
  const [number, setNumber] = useState(0);
  const onIncrease = () => {
    setNumber(number + 1);
  };
  const onDecrease = () => {
    setNumber(number - 1);
  };
  return (
    <div>
      <h1>{number}</h1>
      <button onClick={onIncrease}>+1</button>
      <button onClick={onDecrease}>-1</button>
    </div>
  );
}

export default Counter;
```

useState 를 사용하기 위해서는 먼저 `import React, { useState } from "react";` 를 통해 useState 를 불러와야 한다. 
먼저 `const [number, setNumber] = useState(0);` 부분을 보면 `useState()` 의 괄호 안의 값은 첫번째 원소 number 의 기본값을 나타낸다. 즉, 현재 number 의 값은 '0' 이다. 두번째 원소인 setNumber 는 괄호안의 값을 변경해주는 함수다.

두번째로 함수 onIncrease, onDecrease 를 보면 함수가 호출될 때 setNumber 를 통해 number 의 값을 변경하도록 되어있다. 
결과적으로 h1 은 {number} 의 현재값을 보여주고 있고, 두개의 버튼을 누를 때 각각 setNumber 를 통해 number의 값이 변경되도록 하는 것이다.

정리하자면 useState 를 통해 바뀌는 값을 관리할 수 있는데 useState() 의 파라미터로 기본값을 넣어주고, useState 는 배열을 반환하게 되는데 배열의 첫번째 원소는 현재 상태, 두번째 원소는 현재 상태를 바꾸는 함수이다.


### 함수형 업데이트

useState 를 사용할 때 함수형 업데이트를 사용할 수 도 있다. 위의 onIncrease, onDecrease 함수의 setNumber 의 경우 다음 업데이트 하고 싶은 값을 넣어준 상태인데 `ex) setNumber(number + 1);` , 그 대신에 값을 어떻게 할지에 대한 로직을 정의하는 함수를 넣어줄 수 도 있다.

```javascript
  const onIncrease = () => {
    setNumber((prevNumber) => prevNumber + 1);//prevNumber 의 이름은 무엇으로 하든 상관없지만 보통 prev를 많이 이용한다.
  };
  const onDecrease = () => {
    setNumber((prevNumber) => prevNumber - 1);
  };
```
---
layout: post
categories: React.js
tags: [React.js, useReducer]
---
상태를 관리하는 방법에는 이전에 공부한 useState 말고도 useReducer 라는 것이 있다. 

useState 에서 상태를 업데이트 하고 싶은 값을 직접 지정해 줬다면 `ex) setValue(5)` useReducer 는 action 객체를 기반으로 상태를 업데이트 해준다. 

action 객체란 업데이트 시 참조하는 객체인데 `ex) dispatch({ type: 'INCREMENT', diff: 4})` 예시에 type 값을 참조해서 어떤 업데이트를 진행할지 명시 할 수 있고, diff 객체에서 업데이트에 필요한 참조하고 싶은 다른 값을 넣어줄 수 있다.

useReducer 을 사용하면 컴포넌트의 상태 업데이트 로직을 컴포넌트에서 분리시킬 수 있는데, 상태 업데이트 로직을 컴포넌트 바깥에 작성 할 수도 있고, 다른 파일에 작성 후 불러와서 사용 할 수도 있다.

reducer 는 현재 상태와 액션 객체를 파라미터로 받아와서 새로운 상태를 반환해주는 함수인데 아래와 같은데 액션의 타입에 따라 state 의 값을 설정해준다.

```javascript
function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    default:
      return state;
  }
}
```

useReducer 은 다음과 같이 사용한다.

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

state 는 컴포넌트에서 사용할 수 있는 상태를 가리키고, dispatch 는 액션을 발생시키는 함수이다. useReducer 의 첫번째 파라미터로 reducer 함수를 넣어주고 두번째 파라미터에는 초기값을 넣어주면 된다.

아래는 이전에 useState 를 사용해 만든 버튼을 누르면 값이 바뀌는 컴포넌트인데 useReducer 을 사용하여 변경해 보려한다.

```javascript
import React, { useState } from "react";

function Counter() {
  const [number, setNumber] = useState(0);
  const onIncrease = () => {
    setNumber((prevNumber) => prevNumber + 1);
  };
  const onDecrease = () => {
    setNumber((prevNumber) => prevNumber - 1);
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

먼저 상단에서 useReducer 을 불러온 후 reducer 함수를 작성해준다.

```javascript
import React, { useReducer } from "react";

function reducer(state, action) {
  switch (action.type) {
    case "PLUS":
      return state + 1;
    case "MINUS":
      return state - 1;
    default:
      throw new Error('Unhandled action');
  }
}
```

두번째 파라미터로 받은 action 의 타입에 따라 다른 로직을 실행하기 위해 switch 문을 사용해주는데 default 값으로는 에러를 줘도 되고 그냥 state 값을 줘도 된다.

이후 Counter 컴포넌트 안에서 useReducer 을 사용해준다.

```javascript
  const [number, dispatch] = useReducer(reducer, 0);
  //number = 현재의 상태, dispatch = action
```

위에서 말했듯이 useReducer 의 첫번째 파라미터로 위에서 만든 reducer 함수를 넣어주고 두번째 파라미터에 초기값을 넣어주면 된다.

마지막으로 onIncrease, onDecrease 함수에서 dispatch 의 type 을 변경해주도록 하면 끝난다.

```javascript
function Counter() {
  const [number, dispatch] = useReducer(reducer, 0);
  const onIncrease = () => {
    dispatch({
      type: "PLUS",
    });
  };
  const onDecrease = () => {
    dispatch({
      type: "MINUS",
    });
  };
```

이전과 똑같이 작동하는 것을 볼 수 있다.

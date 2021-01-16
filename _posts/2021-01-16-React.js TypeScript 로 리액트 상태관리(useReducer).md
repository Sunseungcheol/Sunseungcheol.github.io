---
layout: post
categories: React.js
tags: [React.js, TypeScript]
---

useState 를 사용했던 걸 useReducer 로 바꿔보자.

```tsx
//Counter.tsx
import React, { useReducer, useState } from "react";

//사용할 모든 액션 넣어줄것
type Action = { type: "INCREASE" } | { type: "DECREASE" };

function reducer(state: number, action: Action): number {
  switch (action.type) {
    case "INCREASE":
      return state + 1;
    case "DECREASE":
      return state - 1;
    default:
      return state;
  }
}

function Counter() {
  const [count, dispatch] = useReducer(reducer, 0);
  const onIncrease = () => dispatch({ type: "INCREASE" });
  const onDecrease = () => dispatch({ type: "DECREASE" });

  return (
    <div>
      <h1>{count}</h1>
      <div>
        <button onClick={onIncrease}>+1</button>
        <button onClick={onDecrease}>-1</button>
      </div>
    </div>
  );
}

export default Counter;
```

action 에 대한 타입을 하나의 타입에다가 다 정리해놓고 action 넣어주는 것 외에는 큰 차이가 없다.

조금 더 복잡한 예제를 확인해보자.

```tsx
import React, { useReducer } from "react";

type Color = "red" | "green" | "blue";

type State = {
  count: number;
  text: string;
  color: Color;
  isGood: boolean;
};

type Action =
  | { type: "SET_COUNT"; count: number }
  | { type: "SET_TEXT"; text: string }
  | { type: "SET_COLOR"; color: Color }
  | { type: "TOGGLE_GOOD" };

//state 값이 객체등일 경우 interface, TypeAlias 를 사용
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "SET_COUNT":
      return {
        ...state,
        count: action.count,
      };
    case "SET_TEXT":
      return {
        ...state,
        text: action.text,
      };
    case "SET_COLOR":
      return {
        ...state,
        color: action.color,
      };
    case "TOGGLE_GOOD":
      return {
        ...state,
        isGood: !state.isGood,
      };
    default:
      throw new Error("error");
  }
}

function ReducerSample() {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
    text: "hello",
    color: "red",
    isGood: true,
  });

  const setCount = () => dispatch({ type: "SET_COUNT", count: 5 });
  const setText = () => dispatch({ type: "SET_TEXT", text: "안녕안녕" });
  const setColor = () => dispatch({ type: "SET_COLOR", color: "green" });
  const toggleGood = () => dispatch({ type: "TOGGLE_GOOD" });

  return (
    <div>
      <p>
        <code>count: </code>
        {state.count}
      </p>
      <p>
        <code>text: </code>
        {state.text}
      </p>
      <p>
        <code>color: </code>
        {state.color}
      </p>
      <p>
        <code>isGood: </code>
        {/* boolean 타입은 바로 출력안됨 */}
        {state.isGood ? "true" : "false"}
      </p>
      <div>
        <button onClick={setCount}>SET_COUNT</button>
        <button onClick={setText}>SET_TEXT</button>
        <button onClick={setColor}>SET_COLOR</button>
        <button onClick={toggleGood}>TOGGLE_GOOD</button>
      </div>
    </div>
  );
}

export default ReducerSample;
```

이것도 크게 다른 건 없다. 그냥 타입스크립트에 익숙해지면 크게 어려운건 없는 것 같다.
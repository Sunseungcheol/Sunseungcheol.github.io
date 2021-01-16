---
layout: post
categories: React.js
tags: [React.js, TypeScript, ContextAPI]
---
타입스크립트에서 ContextAPI 를 사용해보자.
ContextAPI 를 사용할 때도 Context 안에서 관리할 값을 위한 타입을 설정해줘야 한다.

큰 차이는 없고 몇가지만 주의 해주면 된다. 

1. context 를 만들때 제너릭 설정
2. 커스텀 훅을 만들 때 에러처리(null 처리)

```tsx
//App.tsx
import React from "react";
import Counter from "./Counter";
import MyForm from "./MyForm";
import ReducerSample from "./ReducerSample";
import { SampleProvider } from "./SampleContext";

const App: React.FC = () => {
  return (
    <SampleProvider>
      <ReducerSample />
    </SampleProvider>
  );
};

export default App;

//SampleContext.tsx
import React, { createContext, Dispatch, useContext, useReducer } from "react";

//context 안에서 사용할 값 타입지정
type FooValue = {
  foo: number;
};

const FooContext = createContext<FooValue | null>(null);

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

//상태를 다루는 컨택스트
const SampleStateContext = createContext<State | null>(null);
//액션을 다루는 컨택스트
//type SampleDispatch = Dispatch<Action>;
const SampleDispatchContext = createContext<Dispatch<Action> | null>(null);

//provider 컴포넌트
type SampleProviderProps = {
  children: React.ReactNode;
};

export function SampleProvider({ children }: SampleProviderProps) {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
    text: "hello",
    color: "red",
    isGood: true,
  });

  return (
    <SampleStateContext.Provider value={state}>
      <SampleDispatchContext.Provider value={dispatch}>
        {children}
      </SampleDispatchContext.Provider>
    </SampleStateContext.Provider>
  );
}

//커스텀훅
export function useSampleState() {
  const state = useContext(SampleStateContext);
  if (!state) throw new Error("Cannor find SampleProvider");
  return state;
}

export function useSampleDispatch() {
  const dispatch = useContext(SampleDispatchContext);
  if (!dispatch) throw new Error("Cannot find SampleProvider");
  return dispatch;
}

//ReducerSample.tsx
import React, { useReducer } from "react";
import { useSampleDispatch, useSampleState } from "./SampleContext";

function ReducerSample() {
  const state = useSampleState();
  const dispatch = useSampleDispatch();

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



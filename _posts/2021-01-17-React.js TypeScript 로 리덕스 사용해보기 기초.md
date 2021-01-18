---
layout: post
categories: React.js
tags: [React.js, TypeScript, 리덕스]
---
라이브러리에서 타입스크립트를 자체적으로 지원하는지, 아니면 따로 무언가를 설치해줘야 하는지 알아보려면 node_modules -> 설치한 라이브러리 -> src 안에 `index.d.ts` 라는 파일이 있는지 확인해 보면 된다. 

예를 들어 redux 안에는 `index.d.ts` 파일이 존재하는데 react-redux 안에는 존재하지 않는다. 이런 경우 따로 설치를 해줘야 한다.

```
yarn add @types/react-redux
```

#### 리덕스 사용해보기

먼저 src 폴더에 modules/counter.ts 파일을 만들자.

```ts
//액션 타입
const INCREASE = "counter/INCREASE";

export const increase = () => ({ type: INCREASE });//{type : string}
```

위와 같이 액션 타입을 만든 후 액션 생성 함수를 작성할 때 increase 부분에 마우스를 가져다 대보면 액션 생성함수의 결과부분에 type 이 string 으로 나온다. 

이때 액션타입 뒤에 as const 를 추가해주면 타입이 우리가 작성한 counter/INCREASE 값, 즉 실제 문자열 값이 들어가 있게 된다.

```ts
const INCREASE = "counter/INCREASE" as const;

export const increase = () => ({ type: INCREASE });//{type : "counter/INCREASE"}
```

리듀서를 작성할 때도 action 에 대한 타입을 지정해줘야 하는데 이때 ReturnType 을 사용해주면 조금 더 편리하게 사용가능하다. ReturnType 은 특정 함수에 대한 결과물을 반환해준다.


```ts
//액션에 대한 타입
//ReturnType = 특정 함수에 대한 결과물
type CounterAction =
  | ReturnType<typeof increase>
  | ReturnType<typeof decrease>
  | ReturnType<typeof increaseBy>;

//리듀서
function counter(state: CounterState = initialState, dispatch: CounterAction):CounterState {}
```

또 리듀서에도 타입을 지정해줘야 후에 실수를 방지해줄 수 있다.


```ts
//moudles/counter.ts
//액션 타입
const INCREASE = "counter/INCREASE" as const;
const DECREASE = "counter/DECREASE" as const;
const INCREASE_BY = "counter/INCREASE_BY" as const;

//액션 생성 함수
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });
export const increaseBy = (diff: number) => ({
  type: INCREASE_BY,
  payload: diff,
});

//초기상태
//초기 상태에 대한 기본값 잊지 말것.
type CounterState = {
  count: number;
};

const initialState: CounterState = {
  count: 0,
};

//액션에 대한 타입
//ReturnType = 특정 함수에 대한 결과물
type CounterAction =
  | ReturnType<typeof increase>
  | ReturnType<typeof decrease>
  | ReturnType<typeof increaseBy>;

//리듀서
function counter(
  state: CounterState = initialState,
  action: CounterAction
): CounterState {
  switch (action.type) {
    case INCREASE:
      return { count: state.count + 1 };
    case DECREASE:
      return { count: state.count - 1 };
    case INCREASE_BY:
      return { count: state.count + action.payload };
    default:
      return state;
  }
}

export default counter;
```

이제 모듈을 적용해보자

```ts
//modules/index.ts
import { combineReducers } from "redux";
import counter from "./counter";

const rootReducer = combineReducers({
  counter,
});

export default rootReducer;
//리덕스에서 관리하는 상태에 대한 타입
export type RootState = ReturnType<typeof rootReducer>;

//index.tsx
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import { createStore } from "redux";
import rootReducer from "./modules";
import { Provider } from "react-redux";

const store = createStore(rootReducer);

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

이제 프리젠테이셔널 컴포넌트와 컨테이너 컴포넌트를 만들어보자.

```ts
//components/Counter.tsx
import React from "react";

//props 타입
type CounterProps = {
  count: number;
  onIncrease: () => void;
  onDecrease: () => void;
  onIncreaseBy: (diff: number) => void;
};

export default function Counter({
  count,
  onIncrease,
  onDecrease,
  onIncreaseBy,
}: CounterProps) {
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={onIncrease}>+1</button>
      <button onClick={onDecrease}>-1</button>
      <button onClick={() => onIncreaseBy(5)}>+5</button>
    </div>
  );
}


//containers/CounterContainer.tsx
import React from "react";
import { useDispatch, useSelector } from "react-redux";
import Counter from "../components/Counter";
import { RootState } from "../modules";
import { decrease, increase, increaseBy } from "../modules/counter";

function CounterContainer() {
  const count = useSelector((state: RootState) => state.counter.count);
  const dispatch = useDispatch();

  const onIncrease = () => {
    dispatch(increase());
  };
  const onDecrease = () => {
    dispatch(decrease());
  };
  const onIncreasBy = (diff: number) => {
    dispatch(increaseBy(diff));
  };
  return (
    <Counter
      count={count}
      onIncrease={onIncrease}
      onDecrease={onDecrease}
      onIncreaseBy={onIncreasBy}
    />
  );
}

export default CounterContainer;
```

App.tsx 에 적용 후 확인해보면 잘된다.

#### 정리

타입스크립트를 사용하는 프로젝트에서 리덕스를 사용할 때

1. 액션 타입 뒤에 `as const` 를 붙이고 액션에 대한 타입을 지정할 때 ReturnType 을 사용하면 나중에 사용할 때 쉽게 유추해낼 수 있다.

```ts
//액션 타입
const INCREASE = "counter/INCREASE" as const;
const DECREASE = "counter/DECREASE" as const;
const INCREASE_BY = "counter/INCREASE_BY" as const;

//액션 생성 함수
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });
export const increaseBy = (diff: number) => ({
  type: INCREASE_BY,
  payload: diff,
});
//액션에 대한 타입
type CounterAction =
  | ReturnType<typeof increase>
  | ReturnType<typeof decrease>
  | ReturnType<typeof increaseBy>;
```

2. 리듀서를 사용할 때 타입 지정 잊지 말것

```ts
type CounterState = {
  count: number;
};

const initialState: CounterState = {
  count: 0,
};

//리듀서
function counter(
  state: CounterState = initialState,
  action: CounterAction
): CounterState {
  switch (action.type) {
    case INCREASE:
      return { count: state.count + 1 };
    case DECREASE:
      return { count: state.count - 1 };
    case INCREASE_BY:
      return { count: state.count + action.payload };
    default:
      return state;
  }
}
```

3. rootReducer 을 만들때 리덕스 스토어에서 관리하는 모든 상태를 위한 타입을 지정할 때도 ReturnType 을 쓰면 편하다.

```ts
export type RootState = ReturnType<typeof rootReducer>;
```
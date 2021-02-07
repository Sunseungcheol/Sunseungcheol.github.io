---
layout: post
categories: React.js
tags: [React.js, TypeScript, ToDoList, typesafe-action]
---
<a href="https://www.npmjs.com/package/typesafe-actions" target="_blank">typesafe-actions</a> 는 리덕스를 사용한 프로젝트에서 액션 생성 함수와 리듀서를 더욱 쉽고 깔끔하게 작성 할 수 있도록 도와주는 라이브러리이다.

먼저 사용을 위해 설치를 해야한다.

```
yarn add typesafe-actions
```

#### counter

이제 기존에 타입스크립트를 이용해 작성했던 counter.ts 를 하나씩 고쳐보려고 한다.

먼저 typesafe-actions 를 사용하면 기존에 액션 타입을 설정할 때 했던 `as const` 가 필요없다.

```ts
//전
const INCREASE = "counter/INCREASE" as const;
const DECREASE = "counter/DECREASE" as const;
const INCREASE_BY = "counter/INCREASE_BY" as const;

//후
const INCREASE = "counter/INCREASE";
const DECREASE = "counter/DECREASE";
const INCREASE_BY = "counter/INCREASE_BY";
```

액션 생성 함수도 더욱 간단하게 사용가능한데, 이를 위해 `createAction` 을 사용하거나 `createStandardAction` 을 사용하기 위해서는 아래와 같이 사용하면 불러오면 된다. 강의를 보면 `createStandardAction` 을 불러오는 방법이 다른데 버전이 업데이트 되면서 바뀐 것 같다.

```ts
import { deprecated, ActionType, createReducer } from 'typesafe-actions';
const { createAction, createStandardAction } = deprecated;
//or
import { ActionType, createAction, createReducer } from "typesafe-actions";
```

```ts
//전
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });
export const increaseBy = (diff: number) => ({
  type: INCREASE_BY,
  payload: diff,
});

//후
export const increase = createAction(INCREASE)();
export const decrease = createAction(DECREASE)();
//받아오는 payload 의 타입을 제너릭으로 설정해줘야 한다.
export const increaseBy = createAction(INCREASE_BY)<number>();
```

액션에 대한 타입은 `ActionType` 을 이용해 바꿔줄 수 있다. 이때 사용을 위해 각 타입을 객체형태로 담아줘야 한다.

```ts
//전
type CounterAction =
  | ReturnType<typeof increase>
  | ReturnType<typeof decrease>
  | ReturnType<typeof increaseBy>;

//후
//액션 타입을 위한 객체 생성
const actions = { increase, decrease, increaseBy };
type CounterAction = ActionType<typeof actions>;
```

리듀서는 `createReducer` 을 이용한다.

```ts
//전
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

//후
const counter = createReducer<CounterState, CounterAction>(initialState, {
  //각 액션에 대한 함수
  [INCREASE]: (state) => ({ count: state.count + 1 }),
  [DECREASE]: (state) => ({ count: state.count - 1 }),
  [INCREASE_BY]: (state, action) => ({ count: state.count + action.payload }),
});
```

또한 메서드 체이닝 방식으로도 이용이 가능한다. 
취향에 따라 어떤 방법을 사용하든 상관은 없지만 메서드체이닝 방식은 코드를 조금 더 간결하게 바꿀 수 있다는 장점이 있다. 

액션 생성 함수를 바로 넣어줄 수 있기 때문에 굳이 액션 타입을 따로 선언하지 않고 액션 생성 함수를 만들 때 바로 넣어주면 된다.

```ts
//필요없음
//const INCREASE = "counter/INCREASE";
//const DECREASE = "counter/DECREASE";
//const INCREASE_BY = "counter/INCREASE_BY";

//액션 생성 함수
export const increase = createAction("counter/INCREASE")();
export const decrease = createAction("counter/DECREASE")();
//받아오는 payload 의 타입을 제너릭으로 설정해줘야 한다.
export const increaseBy = createAction("counter/INCREASE_BY")<number>();

//필요없음
//액션 타입을 위한 객체 생성
//const actions = { increase, decrease, increaseBy };
//type CounterAction = ActionType<typeof actions>;

const counter = createReducer<CounterState, CounterAction>(initialState)
  .handleAction(increase, (state) => ({ count: state.count + 1 }))
  .handleAction(decrease, (state) => ({ count: state.count - 1 }))
  .handleAction(increaseBy, (state, action) => ({
    count: state.count + action.payload,
  }));
```


#### 투두리스트 

이번에는 기존에 작성했던 todoList 를 변경해 보려고 한다.

액션 생성함수부터 변경해보자.

```ts
//전
export const addTodo = (text: string) => ({
  type: ADD_TODO,
  payload: {
    id: nextId++,
    text,
  },
});

export const toggleTodo = (id: number) => ({
  type: TOGGLE_TODO,
  payload: id,
});

export const removeTodo = (id: number) => ({
  type: REMOVE_TODO,
  payload: id,
});

//후
//파라미터로 가져오는 값과 payload 가 다른 경우 createAction 을 사용하지 않는 것이 더 깔끔할 수 있다.
export const addTodo = createAction(ADD_TODO, (text: string) => ({
  id: nextId++,
  text: text,
}))<Todo>();
export const toggleTodo = createAction(TOGGLE_TODO)<number>();

export const removeTodo = createAction(REMOVE_TODO)<number>();
```

다만 위에 주석으로 적어놨듯이 파라미터로 가져오는 값과 payload 가 다른 경우 createAction 을 사용하지 않는 것이 더 깔끔할 수 있다. 다만 이때는 잊지말고 as const 를 사용해줘야 한다.

```ts
//전
type TodosAction =
  | ReturnType<typeof addTodo>
  | ReturnType<typeof toggleTodo>
  | ReturnType<typeof removeTodo>;
//후
const actions = { addTodo, toggleTodo, removeTodo };
type TodosAction = ActionType<typeof actions>;
```

마지막으로 리듀서를 변경해보자.

```ts
//전
function todos(
  state: TodosState = initialState,
  action: TodosAction
): TodosState {
  switch (action.type) {
    case ADD_TODO:
      return state.concat({
        id: action.payload.id,
        text: action.payload.text,
        done: false,
      });
    case TOGGLE_TODO:
      return state.map((todo) =>
        todo.id === action.payload ? { ...todo, done: !todo.done } : todo
      );
    case REMOVE_TODO:
      return state.filter((todo) => todo.id !== action.payload);
    default:
      return state;
  }
}
//후
const todos = createReducer<TodosState, TodosAction>(initialState, {
  [ADD_TODO]: (state, action) =>
    state.concat({
      ...action.payload,
      done: false,
    }),
  [TOGGLE_TODO]: (state, action) =>
    state.map((todo) =>
      todo.id === action.payload ? { ...todo, done: !todo.done } : todo
    ),
  [REMOVE_TODO]: (state, action) =>
    state.filter((todo) => todo.id !== action.payload),
});
```
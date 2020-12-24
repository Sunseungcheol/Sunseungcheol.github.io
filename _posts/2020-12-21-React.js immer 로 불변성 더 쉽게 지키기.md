---
layout: post
categories: React.js
tags: [React.js, Hooks]
---
 immer 를 사용하면 불변성을 해치는 코드를 작성해도 대신 불변성을 유지해준다.
 
 React 에서 배열, 객체를 업데이트 할 때 아래와 같이 값을 직접 바꾸는 행위는 불변성을 깨드리는 것이다.

 ```javascript
const object = {
  a: 1,
  b: 2,
};

object.b = 3;
 ```

 이 대신 아래와 같이 스프레드 연산자를 이용해 새로운 객체를 생성하고 값을 덮어씌우는 방식으로 사용해야 한다.

 ```javascript
const nextObject = {
  ...object,
  b: 3,
};
 ```

 이런 방식을 사용해야 후에 컴포넌트 최적화, 리렌더링을 제대로 할 수 있다.

 배열 또한 push, slice 로 배열을 직접적으로 수정하는 것이 아닌 concat, filter, map 등의 함수를 이용해 새로운 배열을 만들어야 한다.

 immer 를 사용하기 위해서는 먼저 프로젝트 디렉토리에 설치해야 한다.

```
 yarn add immer
```

그 후 immer 를 불러온다.

```javascript 
import produce from "immer";
```

먼저 간단하게 사용법을 알아보자.
produce 의 첫번째 파라미터에는 변경하고자 하는 객체, 배열등을 넣어주면 되고 두번째 파라미터에는 어떻게 바꿀지에 대한 함수를 넣어주면 되는데 draft 라는 값을 파라미터로 받아와 내부에서 하고자 하는 작업을 하면된다.

```javascript
const state = {
  number: 1,
  dontChangeMe: 2
};

const nextState = produce(state, draft => {
  draft.number += 1;
});

console.log(nextState);
// { number: 2, dontChangeMe: 2 }
console.log(state);
// { number: 1, dontChangeMe: 2 }
```

배열도 같은 방법으로 사용하면 되는데

```javascript
const array = [
  { id: 1, text: "a" },
  { id: 2, text: "b" },
  { id: 3, text: "c" },
];

const nextArray = produce(array, (draft) => {
  draft.push({ id: 4, text: "d" });
  draft[0].text = draft[0].text + "aaa";
});
```

nextArray 를 만들어 array 배열을 넣고 새로운 값을 넣고, 배열의 첫번째 값을 변경했다.

nextArray 와 기존의 array 를 확인해보면 아래와 같이 나온다.

```javascript
//nextArray
(4) [{…}, {…}, {…}, {…}]
0: {id: 1, text: "aaaa"}
1: {id: 2, text: "b"}
2: {id: 3, text: "c"}
3: {id: 4, text: "d"}
length: 4
__proto__: Array(0)

//array
(3) [{…}, {…}, {…}]
0: {id: 1, text: "a"}
1: {id: 2, text: "b"}
2: {id: 3, text: "c"}
length: 3
__proto__: Array(0)
```

기존의 배열은 그대로 유지한 채로 nextArray 만 새로 생성된 것을 볼 수 있다.

이제 이전에 작성했던 App.js 의 reducer 을 immer 를 사용해 바꿔보려 한다.

여기서 주의해야 할점은 immer 을 쓴다고 무조건 코드가 깔끔해지는 것은 아니라는 것이다. 아래 switch 문의 TOGGLE_USER 의 경우 immer 를 사용하면 코드가 더욱 깔끔해지겠지만 CREATE_USER, REMOVE_USER 는 굳이 바꿔줄 필요가 없다.

```javascript
//App.js
function reducer(state, action) {
  switch (action.type) {
    case "CREATE_USER":
      return produce(state, (draft) => {
        draft.users.push(action.user);
      });
    // return {
    //   inputs: initialState.inputs,
    //   users: state.users.concat(action.user),
    // };
    case "TOGGLE_USER":
      return produce(state, (draft) => {
        //특정 user 검색
        const user = draft.users.find((user) => user.id === action.id);
        user.active = !user.active;
      });
    // return {
    //   ...state,
    //   users: state.users.map((user) =>
    //     user.id === action.id ? { ...user, active: !user.active } : user
    //   ),
    // };
    case "REMOVE_USER":
      return produce(state, (draft) => {
        const index = draft.users.findIndex((user) => user.id !== action.id);
        draft.users.splice(index, 1);
      });
    // return {
    //   ...state,
    //   users: state.users.filter((user) => user.id !== action.id),
    // };
    default:
      throw new Error("Unhandled action");
  }
}
```
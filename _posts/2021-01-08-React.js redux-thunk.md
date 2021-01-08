---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, 미들웨어, redux-thunk]
---
redux-thunk 는 액션 객체가 아닌 함수를 디스패치 할 수 있도록 도와준다.

```javascript
const thunk = (store) => (next) => (action) => {
  typeof action === "function"
    ? action(store.dispatch, store.getState)
    : next(action);
};
```

thunk 는 위와 같은 로직으로 움직이는데 action 이 함수인 경우에는 액션에 store.dispatch 와 store.getState 파라미터로 넣어 실행한다. 

즉, thunk 안에서 액션을 디스패치하거나 getState 를 통해 현재 상태를 조회할 수 있다.

```javascript
//ex
const getComments = () => (dispatch, getState) => {
  //현재 가지고 있느 아이디값 조회
  const id = getState().post.activeId;

  // 요청 시작을 알리는 액션
  dispatch({ type: "GET_COMMENTS" });

  // 댓글을 조회하는 프로미스를 반환하는 getComments 가 있다고 가정
  api
    .getComments(id) // 요청
    .then((comments) =>
      dispatch({ type: "GET_COMMENTS_SUCCESS", id, comments })
    ) // 성공시
    .catch((e) => dispatch({ type: "GET_COMMENTS_ERROR", error: e })); // 실패시
};

//async 사용시
const getComments = () => async (dispatch, getState) => {
  const id = getState().post.activeId;
  dispatch({ type: 'GET_COMMENTS' });
  try {
    const comments = await api.getComments(id);
    dispatch({ type:  'GET_COMMENTS_SUCCESS', id, comments });
  } catch (e) {
    dispatch({ type:  'GET_COMMENTS_ERROR', error: e });
  }
}
```

위와 같이 디스패치와 겟스테이드를 받아와 함수내부에서 사용할 수 가 있다. 

한번 사용해보자. 먼저 redux-thunk 를 설치해주자

```
yarn add redux-thunk
```

그 후 상단에서 불러와 applyMiddleweare 에 파라미터로 보내주면 되는데 이때 logger 와 다른 미들웨어를 같이 사용할 경우 logger 가 맨 마지막에 와야한다.

이제 counter 모듈에서 특정 thunk 함수가 디스패치 되면 x 초 후에 increase, 혹은 decrease 되도록 해볼 것이다.

```javascript
//modules/counter.js
//thunk 함수(dispatch, getState 중 필요한 것만 가져와도 된다.)
export const increaseAsync = () => (dispatch) => {
  setTimeout(() => {
    dispatch(increase());
  }, 1000);
};
export const decreaseAsync = () => (dispatch) => {
  setTimeout(() => {
    dispatch(decrease());
  }, 1000);
};
```

그 후 CounterContainer.js 에서 기존에 dispatch 되던 increase, decrease 를 조금 전 만든 thunk 함수로 변경해보자.

```javascript
  const onIncrease = () => {
    dispatch(increaseAsync());
  };
  const onDecrease = () => {
    dispatch(decreaseAsync());
  };
```

1초 후에 동작하는 것을 볼 수 있다.

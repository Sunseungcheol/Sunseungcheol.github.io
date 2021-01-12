---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, redux-saga, Generator 문법]
---
#### 비동기 카운터 구현

이전에 thunk 를 공부하기 위해 만들었던 비동기 카운터를 리덕스 사가로 구현해 보자. 먼저 설치하자

```
yarn add redux-saga
```

아래는 기존의 카운터 모듈이다.

```javascript
//modules/counter.js
//액션타입
const INCREASE = "counter/INCREASE";
const DECREASE = "counter/DECREASE";

//액션 생성 함수
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });

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

//초기 상태
const initialState = 0;

//리듀서
export default function counter(state = initialState, action) {
  switch (action.type) {
    case INCREASE:
      return state + 1;
    case DECREASE:
      return state - 1;
    default:
      return state;
  }
}

```

먼저 기존의 thunk 함수를 아래와 같이 바꿔준다.

```javascript
import { delay, put } from "redux-saga/effects";

//액션타입
const INCREASE = "counter/INCREASE";
const DECREASE = "counter/DECREASE";
const INCREASE_ASYNC = "counter/INCREASE_ASYNC";
const DECREASE_ASYNC = "counter/DECREASE_ASYNC";

//액션 생성 함수
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });

//순수 객체 반환
export const increaseAsync = () => ({ type: INCREASE_ASYNC });
export const decreaseAsync = () => ({ type: DECREASE_ASYNC });

//saga
function* increaseSaga() {
  //리덕스 미들웨어에 특정 작업을 명령하려면 effects(dalay, put 등) 를 yield 하면 된다.
  yield dalay(1000);
  //increase() 호출 -> increase 액션개체 생성 -> 미들웨어에 디스패치 명령(put)
  yield put(increase());
}

function* decreaseSaga() {
  yield dalay(1000);
  yield put(decrease());
}
```

이제 액션을 모니터링해 INCREASE_ASYNC, DECREASE_ASYNC 액션이 디스패치 된 경우 사가함수에 있는 코드들이 실행되게 해야 되는데 이를 위해 먼저 상단에서 `takeEvery` 와 `takeLatest` 를 불러와야 한다.

`takeEvery` 는 첫번째 파라미터의 액션이 디스패치되면 두번째 파라미터로 받은 함수를 호출 해준다.

`takeLatest` 는 가장 마지막에 들어온 것만 처리해주는데 예를 들어 delay 가 있는 동안 다시 명령이 들어오면 기존의 것은 무시하고 마지막것만 처리하도록한다.

```javascript
import { delay, put, takeEvery, takeLatest } from "redux-saga/effects";

//...
//후에 rootReducer 을 만들듯이 rootSaga 를 만들기 위해 export
export function* counterSaga() {
  //INCREASE_ASYNC 가 디스패치 되면 increaseAsync() 실행
  yield takeEvery(INCREASE_ASYNC, increaseSaga);
  //디스패치된 DECREASE_ASYNC 중 가장 마지막에 디스패치된 것만 실행
  yield takeLatest(DECREASE_ASYNC, decreaseSaga);
}
```

이제 modules/index.js 에서 rootSaga 를 만들어주자.

```javascript
//modules/index.js
import { combineReducers } from "redux";
import counter, { counterSaga } from "./counter";
import posts from "./posts";
import { all } from "redux-saga/effects";

const rootReducer = combineReducers({ counter, posts });
export function* rootSaga() {
  //다른 사가를 추가할 경우 배열 내부에 계속 등록해주면 된다.
  yield all([counterSaga()]);
}

export default rootReducer;
```

이제 루트의 index.js 에서

```javascript
//index.js
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import { Provider } from "react-redux";
import { createStore, applyMiddleware } from "redux";
import rootReducer, { rootSaga } from "./modules";
import logger from "redux-logger";
import { composeWithDevTools } from "redux-devtools-extension";
import ReduxThunk from "redux-thunk";
import { Router } from "react-router-dom";
import { createBrowserHistory } from "history";
import createSagaMiddleware from "redux-saga";

const customHistory = createBrowserHistory();

const sagaMiddleware = createSagaMiddleware();

const store = createStore(
  rootReducer,
  //logger 와 다른 미들웨어를 같이 사용시 logger 가 마지막에 와야한다.
  composeWithDevTools(
    applyMiddleware(
      //withExtraArgument : Thunk 함수에서 3번째 파라미터로 넣은 값을 불러올 수 있도록 도와준다.
      ReduxThunk.withExtraArgument({ history: customHistory }),
      sagaMiddleware,
      logger
    )
  )
);

//store 밑에서
sagaMiddleware.run(rootSaga);

ReactDOM.render(
  <React.StrictMode>
    <Router history={customHistory}>
      <Provider store={store}>
        <App />
      </Provider>
    </Router>
  </React.StrictMode>,
  document.getElementById("root")
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

위와 같이 설정해주면 된다.

```javascript
//createSagaMiddleware 를 불러와 주고 
import createSagaMiddleware from "redux-saga";

//변수에 담고
const sagaMiddleware = createSagaMiddleware();

//스토어에 넣어준후
const store = createStore(
  rootReducer,
  //logger 와 다른 미들웨어를 같이 사용시 logger 가 마지막에 와야한다.
  composeWithDevTools(
    applyMiddleware(
      //withExtraArgument : Thunk 함수에서 3번째 파라미터로 넣은 값을 불러올 수 있도록 도와준다.
      ReduxThunk.withExtraArgument({ history: customHistory }),
      //사가와 덩크의 순서는 상관 없다.
      sagaMiddleware,
      logger
    )
  )
);

//Run!
sagaMiddleware.run(rootSaga);
```

이제 App.js 에 컴포넌트를 넣고 확인해 보자.

+를 누르면 1초후에 작동하고 연속으로 누르면 누른 시점 기준으로 1초후에 계속해서 올라가는 것을 볼 수 있다.

-를 여러번 연속으로 눌렀을 때는 마지막 것만 작동하고 그 이전에 delay 중이던 작업들은 작동되지 않는 것을 확인할 수 있다.
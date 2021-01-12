---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, redux-saga, Promise]
---
thunk 를 사용해 비동기 작업을 할 때 특정 파라미터가 필요하다면 thunk 를 생성하는 함수에서 파라미터로 받아와 사용했다. 

그리고 먼저 시작에 대한 액션을 디스패치한 후 성공과 실패에 따라 다른 액션을 디스패치 해줬다.

마지막으로 비동기 작업을 시작할 때는 thunk 함수 자체를 디스패치 해줌으로써 작업을 시작했다.


```javascript
export const getPost = (id) => async (dispatch) => {
  //meta 값을 아이디로
  dispatch({ type: GET_POST, meta: id });
  try {
    const payload = await postsAPI.getPostById(id);
    dispatch({
      type: GET_POST_SUCCESS,
      payload,
      meta: id,
    });
  } catch (e) {
    dispatch({
      type: GET_POST_ERROR,
      payload: e,
      error: true,
      meta: id,
    });
  }
};

//비동기 작업 시작
dispatch(getPost(1));
```

리덕스 사가에서는 작동 방식이 조금 다른데 비동기 작업을 시작하기 위해 thunk 함수가 아닌, 순수 액션 생성 함수 (type 값을 가진 객체) 를 만든다. 

```javascript
export const getPost = (id) => ({
  type: GET_POST,
  //saga 에서 API 호출시 파라미터로 사용할 아이디
  payload: id,
  //리듀스에서 사용할 아이디
  meta: id,
});
```

위의 액션이 디스패치 되면 액션을 모니터링해 작성한 saga 가 작동하게 해준다. 

saga 에서는 action 에 대한 정보를 파라미터로 받아올 수 있다. 여기서 action 은 위의 getPost 와 같은 액션 생성 함수를 말한다.

```javascript
//modules/posts.js
import * as postsAPI from "../api/posts";
import {
  createPromiseThunk,
  reducerUtils,
  handleAsyncAction,
  createPromiseThunkId,
  handleAsyncActionById,
} from "../lib/asyncUtils";

import { call, put, takeEvery } from "redux-saga/effects";
//getPosts
//특정 요청 시작
const GET_POSTS = "posts/GET_POSTS";
//요청 성공
const GET_POSTS_SUCCESS = "posts/GET_POSTS_SUCCESS";
//요청 실패
const GET_POSTS_ERROR = "posts/GET_POSTS_ERROR";

//getPOSTById
//특정 요청 시작
const GET_POST = "posts/GET_POST";
//요청 성공
const GET_POST_SUCCESS = "posts/GET_POST_SUCCESS";
//요청 실패
const GET_POST_ERROR = "posts/GET_POST_ERROR";

//뒤로 갈 때 포스트 내용 비워버리기
const CLEAR_POST = "posts/CLEAT_POST";

//saga 액션 생성
export const getPosts = () => ({ type: GET_POSTS });
export const getPost = (id) => ({
  type: GET_POST,
  //saga 에서 API 호출시 파라미터로 사용할 아이디
  payload: id,
  //리듀스에서 사용할 아이디
  meta: id,
});


function* getPostsSaga() {
  try {
    //특정 함수 호출 이팩트 call
    //yield call 을 통해 프로미스가 끝날 때 까지 기다린 후 posts 에 담김
    const posts = yield call(postsAPI.getPosts);
    yield put({
      type: GET_POSTS_SUCCESS,
      payload: posts,
    });
  } catch (e) {
    yield put({
      type: GET_POSTS_ERROR,
      payload: e,
      error: true,
    });
  }
}

//파라미터를 확인하기 위해 디스패치된 액션의 정보를 봐야 하는데 방법은 파라미터로 action 을 보내면 된다.
//여기서 action 은 getPost
function* getPostSaga(action) {
  //상단의 액션생성함수 getPost 에서 사용하는 id
  const id = action.payload;
  try {
    const post = yield call(postsAPI.getPostById, id);
    yield put({
      type: GET_POST_SUCCESS,
      payload: post,
      meta: id,
    });
  } catch (e) {
    yield put({
      type: GET_POST_ERROR,
      payload: e,
      error: true,
      meta: id,
    });
  }
}

//리덕스 모듈을 위해 사가를 모니터링하는 함수
export function* postsSaga() {
  yield takeEvery(GET_POSTS, getPostsSaga);
  yield takeEvery(GET_POST, getPostSaga);
}

// //thunk 생성함수
// export const getPosts = createPromiseThunk(GET_POSTS, postsAPI.getPosts);

// //export const getPost = createPromiseThunk(GET_POST, postsAPI.getPostById);
// export const getPost = createPromiseThunkId(GET_POST, postsAPI.getPostById);

//홈으로 가는 Thunk 함수 (history 이용)
export const goHome = () => (dispatch, getState, { history }) => {
  history.push("/");
};

export const clearPost = () => ({ type: CLEAR_POST });

//기본상태
const initialState = {
  posts: reducerUtils.initial(),
  //post: reducerUtils.initial(),
  post: {},
};

//리듀서
const getPostsReducer = handleAsyncAction(GET_POSTS, "posts", true);
//const getPostReducer = handleAsyncAction(GET_POST, "post");
const getPostReducer = handleAsyncActionById(GET_POST, "post", true);
export default function posts(state = initialState, action) {
  switch (action.type) {
    case GET_POSTS:
    case GET_POSTS_SUCCESS:
    case GET_POSTS_ERROR:
      return getPostsReducer(state, action);
    case GET_POST:
    case GET_POST_SUCCESS:
    case GET_POST_ERROR:
      return getPostReducer(state, action);
    //클리어 포스트인 경우 post 값을 기본 값으로
    case CLEAR_POST:
      return {
        ...state,
        post: reducerUtils.initial(),
      };
    default:
      return state;
  }
}

//modules/index.js
import { combineReducers } from "redux";
import counter, { counterSaga } from "./counter";
import posts, { postsSaga } from "./posts";
import { all } from "redux-saga/effects";

const rootReducer = combineReducers({ counter, posts });
export function* rootSaga() {
  //다른 사가를 추가할 경우 배열 내부에 계속 등록해주면 된다.
  yield all([counterSaga(), postsSaga()]);
}

export default rootReducer;
```


#### 리팩토링

thunk 함수를 리팩토링 할 때랑 거의 비슷하다. 

```javascript
//lib/asyncUtils.js
import { call, put } from "redux-saga/effects";

export const createPromiesSaga = (type, promiseCreator) => {
  const [SUCCESS, ERROR] = [`${type}_SUCEESS`, `${type}_ERROR`];

  //제너레이터 함수 리턴
  return function* (action) {
    try {
      const result = yield call(promiseCreator, action.payload);
      yield put({
        type: SUCCESS,
        payload: result,
      });
    } catch (e) {
      yield put({
        type: ERROR,
        error: true,
        payload: e,
      });
    }
  };
};

export const createPromiseSagaById = (type, promiseCreator) => {
  const [SUCCESS, ERROR] = [`${type}_SUCEESS`, `${type}_ERROR`];

  //제너레이터 함수 리턴
  return function* (action) {
    const id = action.meta;
    try {
      const result = yield call(promiseCreator, action.payload);
      yield put({
        type: SUCCESS,
        payload: result,
        meta: id,
      });
    } catch (e) {
      yield put({
        type: ERROR,
        error: true,
        payload: e,
        meta: id,
      });
    }
  };
};

//modules/posts.js

const getPostsSaga = createPromiesSaga(GET_POSTS, postsAPI.getPosts);
const getPostSaga = createPromiseSagaById(GET_POST, postsAPI.getPostById);
```
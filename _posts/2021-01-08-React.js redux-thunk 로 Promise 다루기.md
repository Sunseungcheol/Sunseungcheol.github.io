---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, 미들웨어, redux-thunk]
---
먼저 api 를 가짜로 만들어 받아와서 연습을 해보려고 파일을 하나 작성했다.

```javascript
//api/posts.js
// n 초 후 프로미스 객체를 만들어 리졸브
const sleep = (n) => new Promise((resolve) => setTimeout(resolve, n));

//{ id, title, body}
const posts = [
  {
    id: 1,
    title: "리덕스 미들웨어",
    body: "리덕스 어렵다",
  },
  {
    id: 2,
    title: "redux-thunk",
    body: "thunkthunk",
  },
  {
    id: 3,
    title: "리덕스 사가도 공부해야지",
    body: "사가사가",
  },
];

//getPosts 호출 시 Promise 객체가 만들어지고 500ms 후에 posts 리턴
export const getPosts = async () => {
  await sleep(500);
  return posts;
};

export const getPostById = async (id) => {
  await sleep(500);
  return posts.find((post) => post.id === id);
};
```

이제 포스트에 관련된 상태를 관리할 모듈을 만든다.

```javascript
//modules/posts.js
import * as postsAPI from "../api/posts";

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

//thunk 생성함수
export const getPosts = () => async (dispatch) => {
  //요청 시작
  dispatch({ type: GET_POSTS });
  //API 호출(async await 은 try catch 로)
  try {
    const posts = await postsAPI.getPosts;
    //성공
    dispatch({ type: GET_POSTS_SUCCESS, posts });
  } catch (e) {
    //실패
    dispatch({ type: GET_POSTS_ERROR, error: e });
  }
};

export const getPost = (id) => async (dispatch) => {
  dispatch({ type: GET_POST });
  try {
    const post = postsAPI.getPostById(id);
    dispatch({ type: GET_POST_SUCCESS, post });
  } catch (e) {
    dispatch({ type: GET_POST_ERROR, error: e });
  }
};

//기본상태
const initialState = {
  posts: {
    loading: false,
    data: null,
    error: null,
  },
  post: {
    loading: false,
    data: null,
    error: null,
  },
};

//리듀서
export default function posts(state = initialState, action) {
  switch (action.type) {
    case GET_POSTS:
      return {
        ...state,
        posts: {
          loading: true,
          data: null,
          error: null,
        },
      };
    case GET_POSTS_SUCCESS:
      return {
        ...state,
        posts: {
          loading: false,
          data: action.posts,
          error: null,
        },
      };
    case GET_POSTS_ERROR:
      return {
        ...state,
        posts: {
          loading: false,
          data: null,
          error: action.error,
        },
      };
    case GET_POST:
      return {
        ...state,
        post: {
          loading: true,
          data: null,
          error: null,
        },
      };
    case GET_POST_SUCCESS:
      return {
        ...state,
        post: {
          loading: false,
          data: action.posts,
          error: null,
        },
      };
    case GET_POST_ERROR:
      return {
        ...state,
        post: {
          loading: false,
          data: null,
          error: action.error,
        },
      };
    default:
      return state;
  }
}
```

만들고 보니 중복되는 코드가 매우 많은 것을 알 수 있다. 보기싫다. 리팩토링 해주자. 정리하기 위해 유틸함수를 몇가지 정의한다.

```javascript
//lib/asyncUtils.js
export const reducerUtils = {
  //기본상태
  initial: (data = null) => ({
    data,
    loading: false,
    error: null,
  }),

  //상태
  loading: (prevState = null) => ({
    data: prevState,
    loading: true,
    error: null,
  }),
  success: (data) => ({
    data,
    loading: false,
    error: null,
  }),
  error: (error) => ({
    data: null,
    loading: false,
    error,
  }),
};

//modulse/posts.js
//기본상태
const initialState = {
  posts: reducerUtils.initial(),
  post: reducerUtils.initial(),
};

//리듀서
export default function posts(state = initialState, action) {
  switch (action.type) {
    case GET_POSTS:
      return {
        ...state,
        posts: reducerUtils.loading(),
      };
    case GET_POSTS_SUCCESS:
      return {
        ...state,
        posts: reducerUtils.success(action.posts),
      };
    case GET_POSTS_ERROR:
      return {
        ...state,
        posts: reducerUtils.error(action.error),
      };
    case GET_POST:
      return {
        ...state,
        post: reducerUtils.loading(),
      };
    case GET_POST_SUCCESS:
      return {
        ...state,
        post: reducerUtils.success(action.post),
      };
    case GET_POST_ERROR:
      return {
        ...state,
        post: reducerUtils.error(action.error),
      };
    default:
      return state;
  }
}
```

기본 상태와 리듀서 부분을 reducerUtils 함수 안에서 만들어 사용했다. 훨씬 깔끔해진 것을 알 수 있다. 이렇게 했는데도 거의 비슷한 코드가 계속 해서 중복된다. 

Promise에 기반한 Thunk를 만들어주는 함수를 만들어보자.

```javascript
//lib/asyncUtils.js
export const createPromiseThunk = (type, promiseCreator) => {
  //type = GET_POST or GET_POSTS , promiseCreator = promise 를 만들어주는 함수 (ex postAPI.getPosts, postAPI.getPost)
  const [SUCCESS, ERROR] = [`${type}_SUCCESS`, `${type}_ERROR`];

  return (param) => async (dispatch) => {
    //API 호출
    const payload = await promiseCreator(param);
    dispatch({ type });
    //FSA 규칙, Flux Standard Action
    try {
      dispatch({
        type: SUCCESS,
        payload,
      });
    } catch (e) {
      dispatch({
        type: ERROR,
        payload: e,
        error: true,
      });
    }
  };
};
```

이제 modules/posts.js 의 기존 getPosts, getPost 함수를 한줄로 바꿔줄 수 있다.

```javascript
//thunk 생성함수
export const getPosts = createPromiseThunk(GET_POSTS, postsAPI.getPosts);

export const getPost = createPromiseThunk(GET_POST,postsAPI.getPostById);
```

변경 후 아래 리듀서 부분에서도 변경해줘야 할 것이 있는데 기존에 action.posts, action.error 등으로 작성했던 것들을 payload 로 바꿔줘야 한다.

```javascript
//ex
 case GET_POSTS_ERROR:
      return {
        ...state,
        posts: reducerUtils.error(action.payload),
      };
```

이제 바꾸고 나니 리듀서 부분도 거의 다른게 거의 없다. 줄여보자.

```javascript
//lib/asyncUtils.js
export const handleAsyncAction = (type, key) => {
  //key 는 각 상태에서 관리하는 값. ex posts, post
  const [SUCCESS, ERROR] = [`${type}_SUCCESS`, `${type}_ERROR`];

  return (state, action) => {
    switch (action.type) {
      case type:
        return {
          ...state,
          [key]: reducerUtils.loading(),
        };
      case SUCCESS:
        return {
          ...state,
          [key]: reducerUtils.success(action.payload),
        };
      case ERROR:
        return {
          ...state,
          [key]: reducerUtils.error(action.payload),
        };
      default:
        return state;
    }
  };
};

//modules/posts.js
//리듀서
const getPostsReducer = handleAsyncAction(GET_POSTS, "posts");
const getPostReducer = handleAsyncAction(GET_POST, "post");
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
    default:
      return state;
  }
}
```

매우 짧아졌다. 솔직히 아직 머리에 완전히 들어오진 않는다. 그냥 계속 작성하면서 중복된다 싶으면 어떻게 줄일 수 있을지 고민하면서 연습해봐야 할 것 같다.
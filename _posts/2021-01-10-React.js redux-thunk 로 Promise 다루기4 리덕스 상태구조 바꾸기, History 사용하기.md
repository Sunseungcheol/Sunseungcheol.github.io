---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, 미들웨어, redux-thunk, think 에서 history]
---
이전에 만들었던 post 리덕스 모듈에서는 아래와 같은 방식으로 사용하고 있었다.

```javascript
const postData =
 {
    posts: {
      data,
      loading,
      error,
    },
    post: {
      data,
      loading,
      error,
    },
  };
```

post 는 특정 아이디를 조회해서 사용하는 방식으로 다른 아이디를 호출하면 기존 데이터를 덮어씌우고 불러오는 방식이었다. 데이터를 재사용하기가 어려웠었다.

```javascript
const postData = 
  {
    posts: {
      data,
      loading,
      error,
    },
    post: {
      1: {
        data,
        loading,
        error,
      },
      2: {
        data,
        loading,
        error,
      },
      [id]: {
        data,
        loading,
        error,
      },
    },
  };
```

그래서 위와 같은 방식으로 post 객체 안에 각 아이디를 키로 사용해 특정 키가 정보를 가지고 있게끔 변경해주려고 한다.

구조를 변경하기 위해서는 기존의 `thunk` 와 `rudecer` 을 다시 작성해줘야 한다.

```javascript
//modules/posts.js
//export const getPost = createPromiseThunk(GET_POST, postsAPI.getPostById);
export const getPost = (id) => async (dispatch) => {
  //meta 값을 아이디로
  dispatch({ type: GET_POST, meta: id });
  try {
    const payload = await postsAPI.getPostById(id);
    dispatch({ type: GET_POST_SUCCESS, payload, meta: id });
  } catch (e) {
    dispatch({ type: GET_POST_ERROR, payload: e, error: true, meta: id });
  }
};

export const clearPost = () => ({ type: CLEAR_POST });

//기본상태
const initialState = {
  posts: reducerUtils.initial(),
  //post: reducerUtils.initial(),
  post: {},
};

//리듀서
//const getPostReducer = handleAsyncAction(GET_POST, "post");
const getPostReducer = (state, action) => {
  const id = action.meta;
  switch (action.type) {
    case GET_POST:
      return {
        ...state,
        post: {
          ...state.post,
          //초기 값이 없다면 loading 에 null 이 들어가도록
          [id]: reducerUtils.loading(state.post[id] && state.post[id].data),
        },
      };
    case GET_POST_SUCCESS:
      return {
        ...state,
        post: {
          ...state.post,
          [id]: reducerUtils.success(action.payload),
        },
      };
    case GET_POST_ERROR:
      return {
        ...state,
        post: {
          ...state.post,
          [id]: reducerUtils.error(action.payload),
        },
      };
    default:
      return state;
  }
};
```

컨테이너 컴포넌트도 그에 맞게 바꿔줘야 한다.

```javascript
//container/PostContainers.js
function PostContainer({ postId }) {
  //비구조화 할당 중 undefinded 오류처리 해줄 것.
  const { data, loading, error } = useSelector(
    (state) => state.posts.post[postId] || reducerUtils.initial()
  );
  const dispatch = useDispatch();

  useEffect(() => {
    //데이터가 있는 경우 그냥 바로 리턴
    if (data) return;
    dispatch(getPost(postId));
  }, [postId, dispatch, data]);

  if (loading && !data) return <div>로딩중</div>;
  if (error) return <div>에러발생</div>;
  if (!data) return null;

  return <Post post={data} />;
}

export default PostContainer;
```

이제 불러온 값들을 저장시켜놓는 것을 볼 수 있다.

정리하자면 먼저 Thunk 생성 함수에서는 id 를 파라미터로 받아와 디스패치 할 때 id 값을 같이 보냈다. 

그 후 리듀서에서 받아온 id 값을 가지고 각 post 객체에 id 값을 키값으로 각 객체를 만들어 저장 시켰다.

그리고 컨테이너 컴포넌트에서는 받아온 id 값을 기준으로 비구조화 할당시 id 에 해당하는 키 값이 있다면 그 값을, 없다면 기본값을 들고오도록 설정해줬다.


#### 유틸함수 만들기

이제 다 만들었으니 이전과 같이 깔끔하게 정리하기 위해 유틸함수를 작성해보자.

```javascript
//modules/posts.js
export const getPost = createPromiseThunkId(GET_POST, postsAPI.getPostById);

const getPostReducer = handleAsyncActionById(GET_POST, "post", true);

//lib/asyncUtils.js
//파라미터는 파라미터 그대로( 현재는 ID 만 보내기 때문)
const defaultIdSelector = (param) => param;

export const createPromiseThunkId = (
  type,
  promiseCreator,
  idSelector = defaultIdSelector
) => {
  //idSelector 은 추후 아이디값만 보내는게 아닌 객체로 보낼 경우를 대비
  const [SUCCESS, ERROR] = [`${type}_SUCCESS`, `${type}_ERROR`];

  return (param) => async (dispatch) => {
    const id = idSelector(param);
    //API 호출
    const payload = await promiseCreator(param);
    dispatch({ type, meta: id });
    try {
      dispatch({
        type: SUCCESS,
        payload,
        meta: id,
      });
    } catch (e) {
      dispatch({ type: ERROR, payload: e, error: true, meta: id });
    }
  };
};

export const handleAsyncActionById = (type, key, keepData) => {
  //key 는 각 상태에서 관리하는 값. ex posts, post
  const [SUCCESS, ERROR] = [`${type}_SUCCESS`, `${type}_ERROR`];
  //key 안에 있는 id 객체를 업데이트
  return (state, action) => {
    const id = action.meta;
    switch (action.type) {
      case type:
        return {
          ...state,
          [key]: {
            ...state[key],
            [id]: reducerUtils.loading(
              keepData ? state[key][id] && state[key][id].data : null
            ),
          },
        };
      case SUCCESS:
        return {
          ...state,
          [key]: {
            ...state[key],
            [id]: reducerUtils.success(action.payload),
          },
        };
      case ERROR:
        return {
          ...state,

          [key]: {
            ...state[key],
            [id]: reducerUtils.error(action.payload),
          },
        };
      default:
        return state;
    }
  };
};
```

코드는 기존에 작성했던 유틸함수와 매우 흡사하다.

후에 리액트와 리덕스를 사용할 때 API 연동을 하게 되면 비슷한 코드가 매우 자주 중복될 것이다. 유틸함수를 만드는 걸 계속 연습해 체화시켜야 될 것 같다.


#### Thunk 함수에서 라우터 History 사용하기

`Thunk` 에서 History 는 보통 `Thunk` 에서 특정 주소로 이동하는 로직을 구현하고 싶을 때 사용한다.

예를 들어 로그인 시 성공과 실패에 따라 다른 페이지로 이동하고 싶은 경우 사용하면 편리하다.

`Thunk` 에서 History 를 사용하고 싶은 경우 먼저 index.js 를 수정해줘야 한다.

```javascript
//index.js
import { Router } from "react-router-dom";
import { createBrowserHistory } from "History";

const customHistory = createBrowserHistory();
const store = createStore(
  rootReducer,
  //logger 와 다른 미들웨어를 같이 사용시 logger 가 마지막에 와야한다.
  composeWithDevTools(
    applyMiddleware(
      //withExtraArgument : Thunk 함수에서 3번째 파라미터로 넣은 값을 불러올 수 있도록 도와준다.
      ReduxThunk.withExtraArgument({ history: customHistory }),
      logger
    )
  )
);

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
```

먼저 상단에서 Router 와 createBrowserHistory 를 불러와 주고 Provider 을 Router 로 감싸주면서 props 로 history 를 보내주면 된다.

그후 store 에 withExtraArgument 를 이용해 히스토리 값을 보내준다. withExtraArgument 는 위에 주석으로 써놨는데 Thunk 함수에서 3번째 파라미터를 사용할 수 있게 해준다.

이제 posts 모듈에 홈으로 갈 수 있도록 `Thunk` 함수를 하나 작성하고 컨테이너에서 버튼을 추가해보자.

```javascript
//modules/posts.js
//홈으로 가는 Thunk 함수 (history 이용)
export const goHome = () => (dispatch, getState, { history }) => {
  history.push("/");
};

//container/PostContainer.js
return (
    <>
      <button onClick={() => dispatch(goHome())}>홈으로 이동</button>
      <Post post={data} />
    </>
  );
```

잘 작동한다.
현재는 디스패치 되면 바로 홈으로 이동하게 해놨지만 나중에는 `getState` 를 사용해 조건부로 경로를 이동하게 하거나 비동기작업 후 결과물에 따라 조건부로 이동하게 하는 등, 여러 작업을 할 수 있다.
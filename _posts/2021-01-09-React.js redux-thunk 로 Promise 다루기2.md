---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, 미들웨어, redux-thunk]
---
먼저 만들어 놓은 것들을 modules/index.js 의 rootReducer 에 적용해주자.

```javascript
 //modules/index.js
 const rootReducer = combineReducers({ counter, posts });
```

그 후 리덕스 안의 상태들과 연동할 컴포넌트를 만들어주자.

```javascript
//프리젠테이셔널 컴포넌트  = 리덕스 스토어에 직접적으로 접근하지 않고 필요한 값, 함수를 props 로 받아와 사용
//components/PostList.js
import React from "react";

function PostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id} id={post.id}>
          {post.title}
        </li>
      ))}
    </ul>
  );
}

export default PostList;

//컨테이너 컴포넌트 = 리덕스 스토어의 상태조회, 액션 디스패치를 할 수 있는 컴포넌트, HTML 태그를 이용하지 않고 다른 프리젠테이셔널 컴포넌트 불러와 사용
//containers/PostListContainers.js
import React, { useEffect } from "react";
import { useSelector, useDispatch } from "react-redux";
import PostList from "../components/PostList";
import { getPosts } from "../modules/posts";

function PostListContainers() {
  //modules/index.js 안의 rootReducer 의 두번째 파라미터 posts 안의 posts 객체
  const { data, loading, error } = useSelector((state) => state.posts.posts);
  const dispatch = useDispatch();

  //처음 렌더링 될 때만
  useEffect(() => {
    dispatch(getPosts());
  }, [dispatch]);

  if (loading) return <div>로딩중</div>;
  if (error) return <div>에러발생</div>;
  if (!data) return null;

  return <PostList posts={data} />;
}

export default PostListContainers;
```


### 라우터 적용하기

먼저 `react-router-dom` 설치 후 index.js 에서 Provider 을 감싸주고, post 를 조회를 위한 프리젠테이셔널 컴포넌트와 컨테이너 컴포넌트를 작성해주자

```javascript
//index.js
import { BrowserRouter } from "react-router-dom";


ReactDOM.render(
  <React.StrictMode>
    <BrowserRouter>
      <Provider store={store}>
        <App />
      </Provider>
    </BrowserRouter>
  </React.StrictMode>,
  document.getElementById("root")
);

//components/Post.js
import React from "react";

function Post({ post }) {
  const { title, body } = post;
  return (
    <div>
      <h1>{title}</h1>
      <p>{body}</p>
    </div>
  );
}

export default Post;

//containers/PostContainer.js
import React, { useEffect } from "react";
import { useSelector, useDispatch } from "react-redux";
import Post from "../components/Post";
import { getPost } from "../modules/posts";

function PostContainer({ postId }) {
  const { data, loading, error } = useSelector((state) => state.posts.post);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(getPost(postId));
  }, [postId, dispatch]);

  if (loading) return <div>로딩중</div>;
  if (error) return <div>에러발생</div>;
  if (!data) return null;

  return <Post post={data} />;
}

export default PostContainer;
```

이후 라우트를 설정해주기 위해 두개의 파일을 더 작성하고 App.js 에서 적용해줬다.

```javascript
//pages/PostListPage.js
import React from 'react';
import PostListContainers from '../containers/PostListContainers';

function PostListPage() {
  return <PostListContainers />;
}

export default PostListPage;

//pages/PostPage.js
import React from "react";
import PostContainer from "../containers/PostContainer";

function PostPage({ match }) {
  const { id } = match.params; // URL파라미터 조회
  //파라미터는 무조건 문자열로 들어옴 항상 주의할 것.
  const postId = parseInt(id, 10);
  return <PostContainer postId={postId} />;
}

export default PostPage;

//App.js
import { Route } from "react-router-dom";
import PostListPage from "./pages/PostListPage";
import PostPage from "./pages/PostPage";

function App() {
  return (
    <>
      <Route path="/" component={PostListPage} exact />
      <Route path="/:id" component={PostPage} />
    </>
  );
}

export default App;
```

이제 기존의 components/PostList.js 파일도 변경해줘야 하는데 li 클릭시 다른 주소로 보내기 위해 Link 태그를 넣어줘야 한다.

```javascript
//components/PostList.js
import React from "react";
import { Link } from "react-router-dom";

function PostList({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id} id={post.id}>
          <Link to={`/${post.id}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  );
}

export default PostList;
```

굳이 페이지 파일을 만들지 않고 기존의 PostContainer 과 PostListContainer 에 바로 연결해도 정상적으로 작동되는데 강의에서 설명 중 편의를 위해 그렇게 한건지, 아니면 정해진 규칙인건지 잘모르겠다.


### 문제해결

작동은 정상적으로 잘 되는데 몇가지 문제점이 있다. 특정 포스트를 보고 다른 포스트를 볼 때 순간적으로 이전의 데이터가 남아있거나, 뒤로 가기를 했을 때 재로딩되는 문제점 등이 있다. 

#### 포스트 재로딩 해결

```javascript
//container/PosiLostContainer.js
  //처음 렌더링 될 때만
  useEffect(() => {
    //data 가 존재한다면 바로 리턴
    if (data) return;
    dispatch(getPosts());
  }, [dispatch, data]);

  if (loading) return <div>로딩중</div>;
  if (error) return <div>에러발생</div>;
  if (!data) return null;
```

data 가 존재하면 바로 리턴하는 방식으로 변경해주면 된다. 또 다른 방법이 있는데 새로 불러는 오지만 만약 데이터가 있는 경우 `로딩중` 을 보여주지 않는 방법이 있다.

이 방법을 사용하면 로딩중이 보이지 않으면서 사용자가 항상 최신 상태를 받을 수 있다는 장점이 있다.

기존의 asyncUtils.js 에서 설정한 상태를 보면 로딩중일때 받는 파라미터가 없으면 null 이 들어가도록 설정했었다.

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
```

해결을 위해 `handleAsyncAction` 에서 파라미터를 하나 더 받아와 데이터에 따라 로딩의 상태를 처리해줬다.

```javascript
//lib/asyncUtils.js
export const handleAsyncAction = (type, key, keepData) => {
  //key 는 각 상태에서 관리하는 값. ex posts, post
  const [SUCCESS, ERROR] = [`${type}_SUCCESS`, `${type}_ERROR`];

  return (state, action) => {
    switch (action.type) {
      case type:
        return {
          ...state,
          [key]: reducerUtils.loading(keepData ? state[key].data : null),
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
const getPostsReducer = handleAsyncAction(GET_POSTS, "posts", true);

//containers/PostListContainers.js

  //처음 렌더링 될 때만
  useEffect(() => {
    dispatch(getPosts());
  }, [dispatch]);

  //로딩중이거나 데이터가 없을 때
  if (loading && !data) return <div>로딩중</div>;
```

이제 데이터가 있는 경우에도 API 를 호출하여 최신 상태를 들고오지만 필요 없는 로딩중이라는 글은 나오지 않게 되었다.


#### 포스트를 다시 클릭했을 때 잠시 이전 내용이 보이는 문제

해결방식이 여러가지가 있는데 그 중 하나는 포스트에서 뒤로 갈 때 상태를 비워버리는 방법이다.

먼저 modules/posts.js 에서 새로운 상태와 thunk 생성함수, 그리고 리듀서를 작성해주고 container/PostContainer.js 에서 클린업 함수로 호출해주면 된다.

```javascript
//modules/posts.js
//뒤로 갈 때 포스트 내용 비워버리기
const CLEAR_POST = "posts/CLEAT_POST";

export const clearPost = () => ({ type: CLEAR_POST });

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

//container/PostContainer.js
  useEffect(() => {
    dispatch(getPost(postId));
    //클린업 함수 컴포넌트가 언마운트 되거나, postId 바뀌어 위의 dispatch 가 호출되기 직전
    return () => {
      dispatch(clearPost());
    };
  }, [postId, dispatch]);
```
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
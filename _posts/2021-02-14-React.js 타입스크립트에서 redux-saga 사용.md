---
layout: post
categories: React.js
tags: [React.js, TypeScript, redux-saga]
---
saga 사용을 위해 먼저 설치해준다.

```
yarn add redux-saga
```

먼저 사가함수를 작성해주자.

```ts
//modules/github/saga.ts
import { getUserProfileAsync, GET_USER_PROFILE } from "./actions";
//call = 특정 함수 호출, put = 액션 디스패치, takeEvery = 특정액션 모니터링 중 원하는 액션 들어올 시 정해놓은 사가함수 호출
import { call, put, takeEvery } from "redux-saga/effects";
import { getUserPeofile, GithubProfile } from "../../api/guthub";

function* getUserProfileSaga(
  action: ReturnType<typeof getUserProfileAsync.request>
) {
  try {
    const userProfile: GithubProfile = yield call(
      getUserPeofile,
      action.payload
    );
    //성공시
    yield put(getUserProfileAsync.success(userProfile));
  } catch (e) {
    //실패시
    yield put(getUserProfileAsync.failure(e));
  }
}

export function* githubSage() {
  //GET_USER_PROFILE 액션 감지시 사가함수 호출
  yield takeEvery(GET_USER_PROFILE, getUserProfileSaga);
}

//modules/github/index.ts
export { default } from "./reducer";
export * from "./actions";
export * from "./types";
export * from "./thunk";
export * from "./saga";
```

`getUserProfileSaga` 를 보면 액션의 타입을 가져오기 위해 ReturnType<typeof 액션생성함수> 를 사용해 가져왔다.

이제 루트 사가를 만들어주자.

```ts
//modules/index.ts
import { combineReducers } from "redux";
import counter from "./counter";
import todos from "./todos";
import github, { githubSage } from "./github";
import { all } from "redux-saga/effects";

const rootReducer = combineReducers({
  counter,
  todos,
  github,
});

export default rootReducer;
//리덕스에서 관리하는 상태에 대한 타입
export type RootState = ReturnType<typeof rootReducer>;

//루트 사가
export function* rootSaga() {
  yield all([githubSage()]);
}

```

이후 루트의 index.tsx 에 사가미들웨어를 설정해주고 기존의 컴포넌트에서 덩크함수를 디스패치 해주는 부분을 바꿔주면 된다.

```tsx
//index.tsx
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import { createStore, applyMiddleware } from "redux";
import rootReducer, { rootSaga } from "./modules";
import { Provider } from "react-redux";
import createSagaMiddleware from "redux-saga";

//사가 미들웨어
const sagaMiddleware = createSagaMiddleware();

const store = createStore(rootReducer, applyMiddleware(sagaMiddleware));

sagaMiddleware.run(rootSaga);

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);

reportWebVitals();

//containers/GithubProfileLoader.tsx
import React from "react";
import { useDispatch, useSelector } from "react-redux";
import { RootState } from "../modules";
import GithubUsernameForm from "../components/GithubUsernameForm";
import GithubProfileInfo from "../components/GithubProfileInfo";
import { getUserProfileAsync, getUserProfileThunk } from "../modules/github";

function GithubProfileLoader() {
  const { data, loading, error } = useSelector(
    (state: RootState) => state.github.userProfile
  );
  const dispatch = useDispatch();

  const onSubmitUsername = (username: string) => {
    //dispatch(getUserProfileThunk(username));
    dispatch(getUserProfileAsync.request(username));
  };

  return (
    <>
      <GithubUsernameForm onSubmitUsername={onSubmitUsername} />
      {loading && <p style={{ textAlign: "center" }}>로딩중..</p>}
      {error && <p style={{ textAlign: "center" }}>에러 발생!</p>}
      {data && (
        <GithubProfileInfo
          bio={data.bio}
          blog={data.blog}
          name={data.name}
          thumbnail={data.avatar_url}
        />
      )}
    </>
  );
}

export default GithubProfileLoader;
```

정상적으로 작동하는 것을 볼 수 있다. 
사가는 아직도 눈에 잘 안들어온다. 계속 사용해보는 수 밖에 없을 것 같다.
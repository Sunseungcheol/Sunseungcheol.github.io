---
layout: post
categories: React.js
tags: [React.js, TypeScript, redux-thunk]
---
깃 api 를 통해 특정 사용자의 정보를 받아오는 작업을 typescript 로 해보려고 한다.

깃헙 사용자 정보는 아래와 같이 요청하면 된다.

```
GET https://api.github.com/users/:username
```

먼저 axios 와 redux-thunk 를 설치하자.

```
yarn add axios redux-thunk
```

두 라이브러리다 공식적으로 typescript 를 지원한다.
그 후 thunk 를 적용해주자.

```ts
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import { createStore, applyMiddleware } from "redux";
import rootReducer from "./modules";
import { Provider } from "react-redux";
import Thunk from "redux-thunk";

const store = createStore(rootReducer, applyMiddleware(Thunk));

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);

reportWebVitals();
```

깃헙에서 유저정보를 요청하면 매우 많은 정보가 있는데 응답한 내용의 타입을 매우 쉽게 정리할 수 있는 도구가 있다.

<a href="quicktype.io">quicktype</a> 를 사용하면 된다.

좌측에 응답내용을 적어주면 아래와 같이 변환해준다.

```
export interface GithubProfile {
    login:               string;
    id:                  number;
    node_id:             string;
    avatar_url:          string;
    gravatar_id:         string;
    url:                 string;
    html_url:            string;
    followers_url:       string;
    following_url:       string;
    gists_url:           string;
    starred_url:         string;
    subscriptions_url:   string;
    organizations_url:   string;
    repos_url:           string;
    events_url:          string;
    received_events_url: string;
    type:                string;
    site_admin:          boolean;
    name:                null;
    company:             null;
    blog:                string;
    location:            null;
    email:               null;
    hireable:            null;
    bio:                 null;
    twitter_username:    null;
    public_repos:        number;
    public_gists:        number;
    followers:           number;
    following:           number;
    created_at:          Date;
    updated_at:          Date;
}
```

이제 위의 타입들을 이용해 데이터를 요청하는 api 요청 함수를 만들자

```ts
//api/github.ts
import axios from "axios";

export async function getUserPeofile(username: string) {
  //response 의 데이터가 GithubProfile 인 경우 get 뒤에서 제네릭으로 설정해주면 된다.
  const response = await axios.get<GithubProfile>(
    `https://api.github.com/users/${username}`
  );

  return response.data;
}

export type GithubProfile = {
  login: string;
  id: number;
  node_id: string;
  avatar_url: string;
  gravatar_id: string;
  url: string;
  html_url: string;
  followers_url: string;
  following_url: string;
  gists_url: string;
  starred_url: string;
  subscriptions_url: string;
  organizations_url: string;
  repos_url: string;
  events_url: string;
  received_events_url: string;
  type: string;
  site_admin: boolean;
  name: null;
  company: null;
  blog: string;
  location: null;
  email: null;
  hireable: null;
  bio: null;
  twitter_username: null;
  public_repos: number;
  public_gists: number;
  followers: number;
  following: number;
  created_at: Date;
  updated_at: Date;
};
```

받아오는 데이터의 타입을 지정할 경우 get 뒤에 제네릭으로 설정해주면 된다.

이제 깃헙에 관련된 리덕스 코드들을 작성하자.


액션 생성함수를 만들때 typesafe 의 `createAsyncAction` 을 사용해주면 더욱 쉽게 만들 수 있다. 
파라미터로 각 액션타입들을 넣어주고 제너릭으로 각 타입들의 페이로드 타입들을 넣어주면 된다. 만약 없는 경우 undefined 를 넣어주면 된다.

```ts
//moudle/github/actions.ts
import { AxiosError } from "axios";
import { createAction, createAsyncAction } from "typesafe-actions";
import { GithubProfile } from "../../api/guthub";

//액션타입
export const GET_USER_PROFILE = "github/GET_USER_PROFILE";
export const GET_USER_PROFILE_SUCCESS = "github/GET_USER_PROFILE_SUCCESS";
export const GET_USER_PROFILE_ERROR = "github/GET_USER_PROFILE_ERROR";

//액션 생성함수
//AxiosError = axios 에서 에러객체를 담는 타입
export const getUserProfileAsync = createAsyncAction(
  GET_USER_PROFILE,
  GET_USER_PROFILE_SUCCESS,
  GET_USER_PROFILE_ERROR
)<undefined, GithubProfile, AxiosError>();
```

이제 액션에 대한 타입, 리듀서에서 사용할 타입등, 타입들을 만들어주자.

```ts
//modules/github/types.ts
import * as actions from "./actions";
import { ActionType } from "typesafe-actions";
import { GithubProfile } from "../../api/guthub";

export type GithubAction = ActionType<typeof actions>;
export type GithubState = {
  userProfile: {
    loading: boolean;
    data: GithubProfile | null;
    error: Error | null;
  };
};
```

이제 덩크함수와 리듀서를 만들자

```ts
//modules/github/thunks.ts
import { Dispatch } from "redux";
import { getUserPeofile } from "../../api/guthub";
import { getUserProfileAsync } from "./actions";

export function getUserProfileThunk(username: string) {
  return async (dispatch: Dispatch) => {
    const { request, success, failure } = getUserProfileAsync;
    dispatch(request());
    try {
      const userProfile = await getUserPeofile(username);
      dispatch(success(userProfile));
    } catch (e) {
      dispatch(failure(e));
    }
  };
}

//modules/github/reducer.ts
import { createReducer } from "typesafe-actions";
import {
  GET_USER_PROFILE,
  GET_USER_PROFILE_ERROR,
  GET_USER_PROFILE_SUCCESS,
} from "./actions";
import { GithubAction, GithubState } from "./types";

//기본상태
const initialState: GithubState = {
  userProfile: {
    loading: false,
    data: null,
    error: null,
  },
};

const github = createReducer<GithubState, GithubAction>(initialState, {
  [GET_USER_PROFILE]: (state) => ({
    ...state,
    userProfile: {
      loading: true,
      error: null,
      data: null,
    },
  }),
  [GET_USER_PROFILE_SUCCESS]: (state, action) => ({
    ...state,
    userProfile: {
      loading: false,
      error: null,
      data: action.payload,
    },
  }),
  [GET_USER_PROFILE_ERROR]: (state, action) => ({
    ...state,
    userProfile: {
      loading: false,
      error: action.payload,
      data: null,
    },
  }),
});

//modeules/github/index.ts
export { default } from "./reducer";
export * from "./actions";
export * from "./thunk";

```

이제 준비는 끝났으니 루트의 index.ts 의 루트리듀서에 넣어주면 된다.

```ts
//index.ts
import { combineReducers } from "redux";
import counter from "./counter";
import todos from "./todos";
import github from "./github";

const rootReducer = combineReducers({
  counter,
  todos,
  github,
});

export default rootReducer;
//리덕스에서 관리하는 상태에 대한 타입
export type RootState = ReturnType<typeof rootReducer>;

```
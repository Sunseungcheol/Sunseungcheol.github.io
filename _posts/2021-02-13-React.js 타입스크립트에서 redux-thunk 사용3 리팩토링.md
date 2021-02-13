---
layout: post
categories: React.js
tags: [React.js, TypeScript, redux-thunk]
---
이번에는 조금 더 깔끔하게 리팩토링을 해보자.

```ts
//lib/createAsyncThunk.ts
import { Dispatch } from "redux";
//AsyncActionCreatorBuilder=createAsyncAction 을 호출 했을 때 나타나는 결과
import { AsyncActionCreatorBuilder } from "typesafe-actions";

type AnyAsyncActionCreator = AsyncActionCreatorBuilder<any, any, any>;
type AnyPromiseCreator = (...params: any[]) => Promise<any>;

//extends = 제너릭으로 어떤 것이든 가져올 수 있지만 뒤의 타입에 만족되어야 한다.
export default function createAsyncThunk<
  A extends AnyAsyncActionCreator,
  F extends AnyPromiseCreator
>(asyncActionCreator: A, promiseCreator: F) {
  //F 의 파라미터들을 Params 로 추출(아래는 AnyPromiseCreator 타입의 파라미터들)
  type Params = Parameters<F>;
  return function thunk(...params: Params) {
    return async (dispatch: Dispatch) => {
      const { request, success, failure } = asyncActionCreator;
      //payload 가 없는 경우 에러가 나기 때문에 undefinde
      dispatch(request(undefined));
    };
  };
}
```

이제 만들어 놓은 `createAsyncThunk` 를 이용해 조금 더 쉽게 Thunk 를 이용할 수 있다.

```ts
//modules/github/thunk.ts
import { Dispatch } from "redux";
import { getUserPeofile } from "../../api/guthub";
import createAsyncThunk from "../../lib/createAsyncThunk";
import { getUserProfileAsync } from "./actions";

// export function getUserProfileThunk(username: string) {
//   return async (dispatch: Dispatch) => {
//     const { request, success, failure } = getUserProfileAsync;
//     dispatch(request());
//     try {
//       const userProfile = await getUserPeofile(username);
//       dispatch(success(userProfile));
//     } catch (e) {
//       dispatch(failure(e));
//     }
//   };
// }

export const getUserProfileThunk = createAsyncThunk(
  getUserProfileAsync,
  getUserPeofile
);
```

이제 리듀서를 리팩토링해보자.

```ts
//lib/reducerUtils.ts
//E 의 타입을 지정하지 않는 경우 자동으로 any 로 지정
export type AsyncState<T, E = any> = {
  loading: boolean;
  data: T | null;
  error: E | null;
};

export const asyncState = {
  //화살표 함수에서 제너릭 사용 시 앞부분에
  //리턴 타입은 AsyncState<T,E>
  initial: <T, E>(initialData?: T): AsyncState<T, E> => ({
    loading: false,
    data: initialData || null,
    error: null,
  }),
  load: <T, E>(data?: T): AsyncState<T, E> => ({
    loading: true,
    data: data || null,
    error: null,
  }),
  success: <T, E>(data: T): AsyncState<T, E> => ({
    data,
    loading: false,
    error: null,
  }),
  error: <T, E>(error: E): AsyncState<T, E> => ({
    error,
    loading: false,
    data: null,
  }),
};
//modules/github/types.ts
import * as actions from "./actions";
import { ActionType } from "typesafe-actions";
import { GithubProfile } from "../../api/guthub";
import { AsyncState } from "../../lib/reducerUtils";

export type GithubAction = ActionType<typeof actions>;
export type GithubState = {
  //첫번째 제너릭 = 성공시, 두번째 제너릭 = 실패시
  userProfile: AsyncState<GithubProfile, Error>;
};

//modules/github/reducer.ts
import { createReducer } from "typesafe-actions";
import { asyncState } from "../../lib/reducerUtils";
import {
  GET_USER_PROFILE,
  GET_USER_PROFILE_ERROR,
  GET_USER_PROFILE_SUCCESS,
} from "./actions";
import { GithubAction, GithubState } from "./types";

//기본상태
const initialState: GithubState = {
  userProfile: asyncState.initial(),
};

const github = createReducer<GithubState, GithubAction>(initialState, {
  [GET_USER_PROFILE]: (state) => ({
    ...state,
    userProfile: asyncState.load(),
  }),
  [GET_USER_PROFILE_SUCCESS]: (state, action) => ({
    ...state,
    userProfile: asyncState.success(action.payload),
  }),
  [GET_USER_PROFILE_ERROR]: (state, action) => ({
    ...state,
    userProfile: asyncState.error(action.payload),
  }),
});

export default github;
```

위의 reducer 로직을 따로 분리시키면 후에 사용하기가 더욱 편해진다.

리듀서 로직 분리코드를 보면 제네릭에 keyof 를 사용하는데 keyof 는 아래와 같다.

```ts
const state = {
  a: 1,
  b: 2,
  c: 3,
};

type key = keyof typeof state;
//key = a || b || c
```

```ts
//lib/reducerUtils.ts
import {
  ActionType,
  AsyncActionCreatorBuilder,
  getType,
} from "typesafe-actions";

//E 의 타입을 지정하지 않는 경우 자동으로 any 로 지정
export type AsyncState<T, E = any> = {
  loading: boolean;
  data: T | null;
  error: E | null;
};

export const asyncState = {
  //화살표 함수에서 제너릭 사용 시 앞부분에
  //리턴 타입은 AsyncState<T,E>
  initial: <T, E>(initialData?: T): AsyncState<T, E> => ({
    loading: false,
    data: initialData || null,
    error: null,
  }),
  load: <T, E>(data?: T): AsyncState<T, E> => ({
    loading: true,
    data: data || null,
    error: null,
  }),
  success: <T, E>(data: T): AsyncState<T, E> => ({
    data,
    loading: false,
    error: null,
  }),
  error: <T, E>(error: E): AsyncState<T, E> => ({
    error,
    loading: false,
    data: null,
  }),
};

type AnyAsyncActionCreator = AsyncActionCreatorBuilder<any, any, any>;
export function createAsyncReducer<
  S,
  AC extends AnyAsyncActionCreator,
  K extends keyof S
>(asyncActionCreator: AC, key: K) {
  return (state: S, action: ActionType<AC>) => {
    //getType 해당 액션의 타입을 추출
    const [request, success, failure] = [
      asyncActionCreator.request,
      asyncActionCreator.success,
      asyncActionCreator.failure,
    ].map(getType);
    switch (action.type) {
      case request:
        return {
          ...state,
          [key]: asyncState.load(),
        };
      case success:
        return {
          ...state,
          [key]: asyncState.success(action.payload),
        };
      case failure:
        return {
          ...state,
          [key]: asyncState.error(action.payload),
        };
      default:
        return state;
    }
  };
}

//modules/github/reducer.ts
import { createReducer } from "typesafe-actions";
import { asyncState, createAsyncReducer } from "../../lib/reducerUtils";
import {
  getUserProfileAsync,
  GET_USER_PROFILE,
  GET_USER_PROFILE_ERROR,
  GET_USER_PROFILE_SUCCESS,
} from "./actions";
import { GithubAction, GithubState } from "./types";

//기본상태
const initialState: GithubState = {
  userProfile: asyncState.initial(),
};

const github = createReducer<GithubState, GithubAction>(
  initialState
).handleAction(
  [
    getUserProfileAsync.request,
    getUserProfileAsync.success,
    getUserProfileAsync.failure,
  ],
  createAsyncReducer(getUserProfileAsync, "userProfile")
);

export default github;
```

위의 handleAction 을 보면 getUserProfileAsync 가 중복되는 것을 볼 수 있다. 저것도 줄여보자.

```ts
//lib/reducerUtils.ts
import {
  ActionType,
  AsyncActionCreatorBuilder,
  getType,
} from "typesafe-actions";

//E 의 타입을 지정하지 않는 경우 자동으로 any 로 지정
export type AsyncState<T, E = any> = {
  loading: boolean;
  data: T | null;
  error: E | null;
};

export const asyncState = {
  //화살표 함수에서 제너릭 사용 시 앞부분에
  //리턴 타입은 AsyncState<T,E>
  initial: <T, E>(initialData?: T): AsyncState<T, E> => ({
    loading: false,
    data: initialData || null,
    error: null,
  }),
  load: <T, E>(data?: T): AsyncState<T, E> => ({
    loading: true,
    data: data || null,
    error: null,
  }),
  success: <T, E>(data: T): AsyncState<T, E> => ({
    data,
    loading: false,
    error: null,
  }),
  error: <T, E>(error: E): AsyncState<T, E> => ({
    error,
    loading: false,
    data: null,
  }),
};

type AnyAsyncActionCreator = AsyncActionCreatorBuilder<any, any, any>;
//request, success, failure 추출함수
export function transformToArray<AC extends AnyAsyncActionCreator>(
  asyncActionCreator: AC
) {
  const { request, success, failure } = asyncActionCreator;
  return [request, success, failure];
}

export function createAsyncReducer<
  S,
  AC extends AnyAsyncActionCreator,
  K extends keyof S
>(asyncActionCreator: AC, key: K) {
  return (state: S, action: ActionType<AC>) => {
    //getType 해당 액션의 타입을 추출
    const [request, success, failure] = transformToArray(
      asyncActionCreator
    ).map(getType);
    switch (action.type) {
      case request:
        return {
          ...state,
          [key]: asyncState.load(),
        };
      case success:
        return {
          ...state,
          [key]: asyncState.success(action.payload),
        };
      case failure:
        return {
          ...state,
          [key]: asyncState.error(action.payload),
        };
      default:
        return state;
    }
  };
}

//modules/github/reducer.ts
import { createReducer } from "typesafe-actions";
import {
  asyncState,
  createAsyncReducer,
  transformToArray,
} from "../../lib/reducerUtils";
import { getUserProfileAsync } from "./actions";
import { GithubAction, GithubState } from "./types";

//기본상태
const initialState: GithubState = {
  userProfile: asyncState.initial(),
};

const github = createReducer<GithubState, GithubAction>(
  initialState
).handleAction(
  transformToArray(getUserProfileAsync),
  createAsyncReducer(getUserProfileAsync, "userProfile")
);

export default github;

```
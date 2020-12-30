---
layout: post
categories: React.js
tags: [React.js, Context, 비동기, 리팩토링]
---
이전에 작성한 UsersContext.js 의 반복되는 코드를 줄여보자.

먼저 새로운 파일을 만들어 api 를 반환하는 함수들을 모아준다.

```javascript
//api.js
import axios from "axios";

export async function getUsers() {
  const response = await axios.get(
    "https://jsonplaceholder.typicode.com/users"
  );
  return response.data;
}

export async function getUser(id) {
  const respones = await axios.get(
    `https://jsonplaceholder.typicode.com/users/${id}`
  );
  return respones.data;
}
```

이제 UserContext.js 에 있던 getUsers, getUser 을 편리하게 사용할 수 있는 함수를 만든다.

```javascript
//asyncActionUtils.js
export default function createAsyncDispatcher(type, promiseFn) {
  const SUCCESS = `${type}_SUCCESS`;
  const ERROR = `${type}_ERROR`;

  async function actionHandler(dispatch, ...rest) {
    //위에서 받은 type을 그대로 사용
    dispatch({ type });
    try {
      const data = await promiseFn(...rest);
      dispatch({
        type: SUCCESS,
        data,
      });
    } catch (e) {
      dispatch({ type: ERROR, error: e });
    }
  }

  return actionHandler;
}


//UsersContext.js
import React, { createContext, useReducer, useContext } from "react";
//api.js 안의 모든 함수들을 api 라는 객체로 들고온다.
import { createAsyncDispatcher } from "./asyncActionUtils";
import * as api from "./api";

const initialState = {
  users: {
    loading: false,
    data: null,
    error: null,
  },
  user: {
    loading: false,
    data: null,
    error: null,
  },
};

//로딩중
const loadingState = {
  loading: true,
  data: null,
  error: null,
};

//성공
const success = (data) => ({
  loading: false,
  data,
  error: null,
});

//에러
const error = (e) => ({
  loading: false,
  data: null,
  error: e,
});

function usersReducer(state, action) {
  switch (action.type) {
    case "GET_USERS":
      return {
        ...state,
        users: loadingState,
      };
    case "GET_USERS_SUCCESS":
      return {
        ...state,
        users: success(action.data),
      };
    case "GET_USERS_EROOR":
      return {
        ...state,
        users: error(action.error),
      };
    case "GET_USER":
      return {
        ...state,
        user: loadingState,
      };
    case "GET_USER_SUCCESS":
      return {
        ...state,
        user: success(action.data),
      };
    case "GET_USER_ERROR":
      return {
        ...state,
        user: error(action.error),
      };
    default:
      throw new Error(`Unhandled action type : ${error}`);
  }
}

const UsersStateContext = createContext(null);
const UsersDispatchContext = createContext(null);

export function UsersProvider({ children }) {
  const [state, dispatch] = useReducer(usersReducer, initialState);
  return (
    <UsersStateContext.Provider value={state}>
      <UsersDispatchContext.Provider value={dispatch}>
        {children}
      </UsersDispatchContext.Provider>
    </UsersStateContext.Provider>
  );
}

export function useUsersState() {
  const state = useContext(UsersStateContext);
  if (!state) {
    throw new Error("Cannot find UserProvider");
  }
  return state;
}

export function useUsersDispatch() {
  const dispatch = useContext(UsersDispatchContext);
  if (!dispatch) {
    throw new Error("Cannot find UsersProvider");
  }
  return dispatch;
}

export const getUsers = createAsyncDispatcher("GET_USERS", api.getUsers);
export const getUser = createAsyncDispatcher("GET_USER", api.getUser);
```

이제 reducer 에 해당하는 코드들도 전부 리팩토링 해주자.

```javascript
//asyncActionUtils.js
//액션 타입과 promise 를 만드는 함수를 받아옴.
export function createAsyncDispatcher(type, promiseFn) {
  const SUCCESS = `${type}_SUCCESS`;
  const ERROR = `${type}_ERROR`;

  async function actionHandler(dispatch, ...rest) {
    //위에서 받은 type을 그대로 사용
    dispatch({ type });
    try {
      const data = await promiseFn(...rest);
      dispatch({
        type: SUCCESS,
        data,
      });
    } catch (e) {
      dispatch({ type: ERROR, error: e });
    }
  }

  return actionHandler;
}

export const initialAsyncState = {
  loading: false,
  data: null,
  error: null,
};

//로딩중
const loadingState = {
  loading: true,
  data: null,
  error: null,
};

//성공
const success = (data) => ({
  loading: false,
  data,
  error: null,
});

//에러
const error = (e) => ({
  loading: false,
  data: null,
  error: e,
});

//reducer 작성시 조금 더 편하게 작성하게 도와줌
// type 은 액션 타입, key 는 user,users
export function createAsyncHandler(type, key) {
  const SUCCESS = `${type}_SUCCESS`;
  const ERROR = `${type}_ERROR`;

  function handler(state, action) {
    switch (action.type) {
      case type:
        return {
          ...state,
          [key]: loadingState,
        };
      case SUCCESS:
        return {
          ...state,
          [key]: success(action.data),
        };
      case ERROR:
        return {
          ...state,
          [key]: error(action.error),
        };
      default:
        return state;
    }
  }

  return handler;
}

//UserContext.js
import React, { createContext, useReducer, useContext } from "react";
//api.js 안의 모든 함수들을 api 라는 객체로 들고온다.
import {
  createAsyncDispatcher,
  createAsyncHandler,
  initialAsyncState,
} from "./asyncActionUtils";
import * as api from "./api";

const initialState = {
  users: initialAsyncState,
  user: initialAsyncState,
};

const usersHandler = createAsyncHandler("GET_USERS", "users");
const userHandler = createAsyncHandler("GET_USER", "user");

function usersReducer(state, action) {
  //switch 문에서는 return, break 를 하지 않으면 여러개의 casedp 동일한 코드 실행
  switch (action.type) {
    case "GET_USERS":
    case "GET_USERS_SUCCESS":
    case "GET_USERS_EROOR":
      return usersHandler(state, action);
    case "GET_USER":
    case "GET_USER_SUCCESS":
    case "GET_USER_ERROR":
      return userHandler(state, action);
    default:
      throw new Error(`Unhandled action type : ${action.type}`);
  }
}

const UsersStateContext = createContext(null);
const UsersDispatchContext = createContext(null);

export function UsersProvider({ children }) {
  const [state, dispatch] = useReducer(usersReducer, initialState);
  return (
    <UsersStateContext.Provider value={state}>
      <UsersDispatchContext.Provider value={dispatch}>
        {children}
      </UsersDispatchContext.Provider>
    </UsersStateContext.Provider>
  );
}

export function useUsersState() {
  const state = useContext(UsersStateContext);
  if (!state) {
    throw new Error("Cannot find UserProvider");
  }
  return state;
}

export function useUsersDispatch() {
  const dispatch = useContext(UsersDispatchContext);
  if (!dispatch) {
    throw new Error("Cannot find UsersProvider");
  }
  return dispatch;
}

export const getUsers = createAsyncDispatcher("GET_USERS", api.getUsers);
export const getUser = createAsyncDispatcher("GET_USER", api.getUser);

```

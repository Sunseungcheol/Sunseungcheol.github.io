---
layout: post
categories: React.js
tags: [React.js, Context, 비동기]
---
가끔씩 특정 데이터들을 여러 컴포넌트에서 사용하게 될 때, (예: 현재 로그인된 사용자의 정보, 설정 등) 그럴 때 Context 를 사용하면 개발이 편해진다.

먼저 사용할 기본정보를 작성해준다. 그 후 상태에 따라 (로딩, 에러, 성공) 객체를 바꿀 수 있도록 객체를 생성해 준다.

```javascript
//UserContext.js
import React, { createContext, useReducer, useContext } from "react";

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
```

이제 reducer 을 작성해 주는데 액션은 상태에 따라 총 여섯개를 작성해 주려고 한다. (users, user 각 3개)

```javascript
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
```

이제 이를 이용해 Context 를 만들어주는데, state 용과 dispatch 용 Context 를 따로 만들어 준다. 따로 만들어 줘야 후에 컴포넌트를 최적화 할 때 훨씬 편하다.

만든 Context 는 커스텀 훅을 통해 관리해준다.

```javascript
const UsersStateContext = createContext(null);
const UsersDispatchContext = createContext(null);

export function UsersProcider({ children }) {
  const [state, dispatch] = useReducer(usersReducer);
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
    throw new Error("Cannot find UserProvider");
  }
  return dispatch;
}
```

이제 API 를 요청하자

```javascript
export async function getUsers(dispatch) {
  const response = await axios.get(
    "https://jsonplaceholder.typicode.com/users"
  );
  try {
    dispatch({ type: "GET_USERS_SUCCESS", data: response.data });
  } catch (e) {
    dispatch({ type: "GET_USERS_ERROR", error: e });
  }
}

export async function getUser(dispatch, id) {
  const response = await axios.get(
    `https://jsonplaceholder.typicode.com/users/${id}`
  );
  try {
    dispatch({ type: "GET_USER_SUCCESS", data: response.data });
  } catch (e) {
    dispatch({ type: "GET_USER_ERROR", error: e });
  }
}
```

이제 App.js 에서 컨텍스트를 적용해주고 각 컴포넌트에서 사용해 주면된다.

```javascript
//App.js
import Users from "./components/Users";
import { UsersProvider } from "./components/UsersContext";

function App() {
  return (
    <UsersProvider>
      <Users></Users>
    </UsersProvider>
  );
}

export default App;

//Users.js
import React, { useState } from "react";
import User from "./User";
import { useUsersState, useUsersDispatch, getUsers } from "./UsersContext";

function Users() {
  const [userId, setUserId] = useState(null);
  const state = useUsersState();
  const dispatch = useUsersDispatch();

  const { loading, data: users, error } = state.users;
  const fetchData = () => {
    getUsers(dispatch);
  };
  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  //로딩은 끝났지만 users 의 값이 아직 없을경우
  if (!users) return <button onClick={fetchData}>불러오기</button>;

  return (
    <>
      <ul>
        {users.map((user) => (
          <li key={user.id} onClick={() => setUserId(user.id)}>
            {user.username} ({user.name})
          </li>
        ))}
      </ul>
      <button onClick={fetchData}>다시 불러오기</button>
      {/* userId 의 기본값은 null, */}
      {userId && <User id={userId}></User>}
    </>
  );
}

export default Users;

//User.js
import React, { useEffect } from "react";
import { useUsersState, useUsersDispatch, getUser } from "./UsersContext";

function User({ id }) {
  const state = useUsersState();
  const dispatch = useUsersDispatch();

  const { loading, data: user, error } = state.user;

  useEffect(() => {
    getUser(dispatch, id);
  }, [dispatch, id]);

  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  if (!user) return null;

  return (
    <div>
      <h2>{user.username}</h2>
      <p>
        <b>Email:</b> {user.email}
      </p>
    </div>
  );
}

export default User;

//UserContext.js
import React, { createContext, useReducer, useContext } from "react";
import axios from "axios";

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
    throw new Error("Cannot find UserProvider");
  }
  return dispatch;
}

export async function getUsers(dispatch) {
  const response = await axios.get(
    "https://jsonplaceholder.typicode.com/users"
  );
  try {
    dispatch({ type: "GET_USERS_SUCCESS", data: response.data });
  } catch (e) {
    dispatch({ type: "GET_USERS_ERROR", error: e });
  }
}

export async function getUser(dispatch, id) {
  const response = await axios.get(
    `https://jsonplaceholder.typicode.com/users/${id}`
  );
  try {
    dispatch({ type: "GET_USER_SUCCESS", data: response.data });
  } catch (e) {
    dispatch({ type: "GET_USER_ERROR", error: e });
  }
}
```


---
layout: post
categories: React.js
tags: [React.js, Context API]
---

```javascript
//App.js
import React, {
  useRef,
  useState,
  useMemo,
  useCallback,
  useReducer,
} from "react";
import CreateUser from "./CreateUser";
import useInputs from "./UseInputs";
import UserList from "./UserList";

function countActiveUsers(users) {
  console.log("활성 사용자 수를 세고 있습니다.");
  return users.filter((user) => user.active).length;
}

const initialState = {
  users: [
    {
      id: 1,
      username: "Sun",
      email: "123@naver.com",
      active: true,
    },
    {
      id: 2,
      username: "Jung",
      email: "456@naver.com",
      active: false,
    },
    {
      id: 3,
      username: "Kim",
      email: "789@naver.com",
      active: false,
    },
  ],
};

function reducer(state, action) {
  switch (action.type) {
    case "CHANGE_INPUT":
      return {
        ...state,
        inputs: {
          ...state.inputs,
          [action.name]: action.value,
        },
      };
    case "CREATE_USER":
      return {
        inputs: initialState.inputs,
        users: state.users.concat(action.user),
      };
    case "TOGGLE_USER":
      return {
        ...state,
        users: state.users.map((user) =>
          user.id === action.id ? { ...user, active: !user.active } : user
        ),
      };
    case "REMOVE_USER":
      return {
        ...state,
        users: state.users.filter((user) => user.id !== action.id),
      };
    default:
      throw new Error("Unhandled action");
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);

  ///이름은 맘대로 지정해도 상관이 없다.
  const [form, onChange, reset] = useInputs({
    //초기값 지정
    username: "",
    email: "",
  });
  const { username, email } = form;

  const nextId = useRef(4);
  const { users } = state;

  const onCreate = useCallback(() => {
    dispatch({
      type: "CREATE_USER",
      user: {
        id: nextId.current,
        username,
        email,
      },
    });
    nextId.current += 1;
    reset();
  }, [username, email, reset]);

  const onToggle = useCallback((id) => {
    dispatch({
      type: "TOGGLE_USER",
      id,
    });
  }, []);

  const onRemove = useCallback((id) => {
    dispatch({
      type: "REMOVE_USER",
      id,
    });
  }, []);

  const count = useMemo(() => countActiveUsers(users), [users]);
  return (
    <>
      <CreateUser
        username={username}
        email={email}
        onChange={onChange}
        onCreate={onCreate}
      ></CreateUser>
      <UserList
        users={users}
        onToggle={onToggle}
        onRemove={onRemove}
      ></UserList>
      <div>활성 사용자 수 : {count}</div>
    </>
  );
}

export default App;


//UserList.js
import React, { useEffect } from "react";

const User = React.memo(function User({ user, onRemove, onToggle }) {
  console.log("user렌더링" + user);
  const { username, email, id, active } = user;
  useEffect(() => {
    //console.log("값이 설정됨 :" + user);
    return () => {
      //console.log("값이 바뀌기전 :" + user);
    };
  }, [user]);
  return (
    <div>
      <b
        style={{
          color: active ? "green" : "red",
          cursor: "pointer",
        }}
        onClick={() => onToggle(id)}
      >
        {username} <span>({email})</span>
      </b>
      <button onClick={() => onRemove(id)}>삭제</button>
    </div>
  );
});
function UserList({ users, onRemove, onToggle }) {
  return (
    <div>
      {users.map((user) => (
        <User
          key={user.id}
          user={user}
          onRemove={onRemove}
          onToggle={onToggle}
        ></User>
      ))}
    </div>
  );
}

export default React.memo(UserList);
```

App.js 의 onRemove, onToggle 함수는 User 컴포넌트에서만 쓰이지만 User 컴포넌트에서 사용하기 위해 UserList 에서 먼저 받아주고 있다. 지금으로선 큰 문제가 없지만 컴포넌트 구조가 복잡해지는 경우 문제가 생긴다.

예를 들어 아래의 구조를 보면 text 는 Child 컴포넌트에서만 사용되는데 이를 위해 최상단부터 계속해서 text 를 받아 내려가고 있다.

```javascript
//ContextSample.js
function Child({ text }) {
  return <div>안녕하세요?{text}</div>;
}

function Parent({ text }) {
  return <Child text={text} />;
}

function GrandParent({ text }) {
  return <Parent text={text} />;
}

function ContextSample() {
  return <GrandParent text="Good" />;
}

export default ContextSample;
```

index.js 에서 위 파일을 렌더링 하면 `안녕하세요?Good` 가 출력되는 것을 볼 수 있다.

이때 Context API 를 사용하면 text 를 전역으로 설정하여 사용할 수 있다.

```javascript
import React, { createContext, useContext } from "react";

const MyContext = createContext("default");

function Child() {
  const text = useContext(MyContext);
  return <div>안녕하세요?{text}</div>;
}
```

먼저 상단에서 `createContext, useContext` 를 불러와 주고 `MyContext` 와 같이 변수를 작성해준다.

이제 Child 컴포넌트에서 사용하기 위해 useContext 를 사용하는데 파라미터로 위에서 작성한 변수를 넣어주면 된다.

이후 확인해보면 

`안녕하세요?default` 가 출력되는 것을 볼 수 있다.

만약 `MyContext` 의 값을 지정해 주고 싶다면 최상단 컴포넌트에서 Provider 라는 컴포넌트를 사용해 주면된다. Provider 가 사용되지 않으면 기본값으로 설정해준 값이 사용되게 된다. 

```javascript
function ContextSample() {
  return (
    <MyContext.Provider value="Good">
      <GrandParent />
    </MyContext.Provider>
  );
}
```

이제 `안녕하세요?Good` 가 출력되는 것을 볼 수 있다.

Context 값은 유동적으로 변경 될 수 도 있는데

```javascript
function ContextSample() {
  const [value, setValue] = useState(true);
  return (
    <MyContext.Provider value={value ? "Good" : "Bad"}>
      <GrandParent />
      <button
        onClick={() => {
          setValue(!value);
        }}
      >
        Click
      </button>
    </MyContext.Provider>
  );
}
```
버튼을 누름에 따라 `Good` 과 `Bad` 가 번갈아 출력되는 것을 볼 수 있다.

Context 는 파일 내부뿐만이 아니라 다른 파일에서도 작성해서 사용이 가능하다.

이번에는 App.js 의 `onToggle, onRemove` 함수에서 Context 를 활요해 보려한다.

먼저 App 컴포넌트 위에서 컨텍스트를 하나 생성해 주고

```javascript 
//App.js
export const UserDispatch = createContext(null);
```

App 컴포넌트에 Provider 로 보내주는데 값은 위에서 사용했던 dispatch 를 보내준다.

이제 필요없는 onToggle, onRemove 함수와 props 로 보냈던 것들을 다 지워준다. 

```javascript
return (
    <UserDispatch.Provider value={dispatch}>
      <CreateUser
        username={username}
        email={email}
        onChange={onChange}
        onCreate={onCreate}
      ></CreateUser>
      <UserList users={users}></UserList>
      <div>활성 사용자 수 : {count}</div>
    </UserDispatch.Provider>
  );
```

이제 UserList.js 로 돌아와 useContext 를 사용해주면 된다. 

```javascript
import React, { useContext, useEffect } from "react";
import { UserDispatch } from "./App";

const User = React.memo(function User({ user }) {
  const { username, email, id, active } = user;
  const dispatch = useContext(UserDispatch);

  useEffect(() => {
    //console.log("값이 설정됨 :" + user);
    return () => {
      //console.log("값이 바뀌기전 :" + user);
    };
  }, [user]);
  return (
    <div>
      <b
        style={{
          color: active ? "green" : "red",
          cursor: "pointer",
        }}
        onClick={() =>
          dispatch({
            type: "TOGGLE_USER",
            id,
          })
        }
      >
        {username} <span>({email})</span>
      </b>
      <button
        onClick={() =>
          dispatch({
            type: "REMOVE_USER",
            id,
          })
        }
      >
        삭제
      </button>
    </div>
  );
});
function UserList({ users }) {
  return (
    <div>
      {users.map((user) => (
        <User key={user.id} user={user}></User>
      ))}
    </div>
  );
}

export default React.memo(UserList);
```

만약 특정 함수를 여러 컴포넌트에 거쳐서 전달해야 할 일이 있다면, dispatch 를 관리 Context 를 만들어 필요한 곳에서 바로 불러와 사용한다면 구조도 깔끔해지고 훨씬 편해진다.

마지막으로 기존의 App.js 를 Context 를 사용해 깔끔하게 바꿔보자

```javascript
//App.js
import React, { useMemo, useReducer, createContext } from "react";
import produce from "immer";
import CreateUser from "./CreateUser";
import UserList from "./UserList";

window.produce = produce;
function countActiveUsers(users) {
  console.log("활성 사용자 수를 세고 있습니다.");
  return users.filter((user) => user.active).length;
}

const initialState = {
  users: [
    {
      id: 1,
      username: "Sun",
      email: "123@naver.com",
      active: true,
    },
    {
      id: 2,
      username: "Jung",
      email: "456@naver.com",
      active: false,
    },
    {
      id: 3,
      username: "Kim",
      email: "789@naver.com",
      active: false,
    },
  ],
};

function reducer(state, action) {
  switch (action.type) {
    case "CREATE_USER":
      return {
        inputs: initialState.inputs,
        users: state.users.concat(action.user),
      };
    case "TOGGLE_USER":
      return {
        ...state,
        users: state.users.map((user) =>
          user.id === action.id ? { ...user, active: !user.active } : user
        ),
      };
    case "REMOVE_USER":
      return {
        ...state,
        users: state.users.filter((user) => user.id !== action.id),
      };
    default:
      throw new Error("Unhandled action");
  }
}

export const UserDispatch = createContext(null);

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const { users } = state;

  const count = useMemo(() => countActiveUsers(users), [users]);
  return (
    <UserDispatch.Provider value={dispatch}>
      <CreateUser></CreateUser>
      <UserList users={users}></UserList>
      <div>활성 사용자 수 : {count}</div>
    </UserDispatch.Provider>
  );
}

export default App;

//CreateUser.js
import React, { useContext, useRef } from "react";
import useInputs from "./UseInputs";
import { UserDispatch } from "./App";

function CreateUser() {
  //console.log(CreateUser);
  const [{ username, email }, onChange, reset] = useInputs({
    username: "",
    email: "",
  });

  const nextId = useRef(4);
  const dispatch = useContext(UserDispatch);
  const onCreate = () => {
    dispatch({
      type: "CREATE_USER",
      user: {
        id: nextId.curren,
        username,
        email,
      },
    });
    reset();
    nextId.current += 1;
  };

  return (
    <div>
      <input
        name="username"
        placeholder="계정명"
        onChange={onChange}
        value={username}
      />
      <input
        name="email"
        placeholder="이메일"
        onChange={onChange}
        value={email}
      />
      <button onClick={onCreate}>등록</button>
    </div>
  );
}

export default React.memo(CreateUser);

//UseInput.js
import { useState, useCallback } from "react";

function useInputs(initialForm) {
  const [form, setForm] = useState(initialForm);
  const onChange = useCallback((e) => {
    const { name, value } = e.target;
    setForm((form) => ({ ...form, [name]: value }));
  }, []);
  const reset = useCallback(() => setForm(initialForm), [initialForm]);

  return [form, onChange, reset];
}

export default useInputs;

```
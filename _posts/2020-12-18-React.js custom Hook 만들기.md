---
layout: post
categories: React.js
tags: [React.js, Hooks]
---

리액트를 사용하다 보면 반복적인 로직들이 발생하는 경우들이 있다.
예를 들어 input 을 관리하는 이런 코드는 상당히 자주 쓰이게 되는 경우가 많다. 아래와 같이 보통 input 안의 name 값과 value 값을 e.target 으로 받아 사용해야 하기 때문이다.

```javascript 
const onChange = (e) => {
  const { name, value } = e.target;
  setInputs({
    ...inputs,
    [name]: value,
  });
};
```

이런 경우 Custom Hook 을 만들어 사용할 수 있다. 쉽게 생각해 javsscript 사용 시 자주 쓰이던 함수를 공통 파일에 빼고 계속해서 쓰던 느낌이다.

어쨋든 일단 만들어보자. 

```javascript
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

위에 부터 하나씩 풀어보자면 파라미터로 받은 `initialForm` 값은 해당 인풋 폼에서 관리할 초기값이다.

```javascript
function useInputs(initialForm)
```

이후 `form` 이라는 새로운 상태를 선언하는데 이 초기값은 파라미터로 받은 `initialForm` 이다.

```javascript
const [form, setForm] = useState(initialForm);
```

onChange 는 이전에 작성한 것과 크게 다르지 않다. `name, value` 를 `e.target` 으로 추출하고 `setForm()` 에서 변경해준다. `name` 값을 변경해준다.

```javascript
const onChange = useCallback((e) => {
    const { name, value } = e.target;
    setForm((form) => ({ ...form, [name]: value }));
  }, []);
```

추가적으로 `form` 을 초기화 시키는 `reset` 을 만들어 준다.

```javascript
const reset = useCallback(() => setForm(initialForm), [initialForm]);
```

마지막으로 return 문을 사용해 내보내줘야 하는데 이때 배열로 보내도 되고 객체로 보내도 된다.

```javascript
return [form, onChange, reset];
```

이제 다시 기존의 App.js 로 돌아가서 만든 커스텀 훅을 사용해보자.

```javascript
///App.js
import React, {
  useRef,
  useState,
  useMemo,
  useCallback,
  useReducer,
} from "react";
import CreateUser from "./CreateUser";
import UserList from "./UserList";

function countActiveUsers(users) {
  console.log("활성 사용자 수를 세고 있습니다.");
  return users.filter((user) => user.active).length;
}

const initialState = {
  inputs: {
    username: "",
    email: "",
  },
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
  const nextId = useRef(4);
  const { users } = state;
  const { username, email } = state.inputs;

  const onChange = useCallback((e) => {
    const { name, value } = e.target;
    dispatch({
      type: "CHANGE_INPUT",
      name,
      value,
    });
  }, []);

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
  }, [username, email]);

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

```

먼저 상단에서 useInput 을 불러와준 후

```javascript
import useInputs from "./UseInputs";
```

`initialState` 의 inputs 객체와 app 컴포넌트 안의 `const { username, email } = state.inputs;`, 그리고 onChange 함수는 이제 필요가 없으니 지워준다.

그 다음에 아까 만들었던 인풋을 설정해야 하는데

```javascript
///이름은 맘대로 지정해도 상관이 없다.
const [form, onChange, reset] = useInputs({
  //초기값 지정
  username: "",
  email: "",
});

const {username, email} = form;
```

기존에 state.inputs 에서 추출했던 username 과 email 은 이제 form 에서 추출해준다. 

reset() 은 onCreate 함수에 넣어주면 된다.

```javascript
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
```
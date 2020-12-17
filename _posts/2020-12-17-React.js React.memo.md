---
layout: post
categories: React.js
tags: [React.js, React.memo]
---
React.memo 는 컴포넌트에서 리렌더링이 불필요할 때 이전의 컴포넌트 값을 그대로 사용하는 함수이다. React.memo 를 사용하면 컴포넌트의 props 가 바뀌지 않는다면 리렌더링을 하지 않는다. 이 함수 사용으로 컴포넌트에 리렌더링 성능을 최적화 시킬 수 있다.

사용법은 매우 간단한 편인데 

```javascript
//UserList.js
import React, { useEffect } from "react";

const User = React.memo(function User({ user, onRemove, onToggle }) {
  console.log("user렌더링");
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
이런 식으로 React.memo 로 감싸주기만 하면된다.

여기서 문제가 하나 있는데 여러개의 User 컴포넌트 중 하나만 변경되어도 모든 User 컴포넌트가 리렌더링 되는 것이다. 

이유를 확인하기 위해 아래 App.js 를 살펴보면 

```javascript
import React, { useRef, useState, useMemo, useCallback } from "react";
import CreateUser from "./CreateUser";
import UserList from "./UserList";

function countActiveUsers(users) {
  console.log("활성 사용자 수를 세고 있습니다.");
  return users.filter((user) => user.active).length;
}

function App() {
  const [inputs, setInputs] = useState({
    username: "",
    email: "",
  });
  const { username, email } = inputs;
  const onChange = useCallback(
    (e) => {
      const { name, value } = e.target;
      setInputs({
        ...inputs,
        [name]: value,
      });
    },
    [inputs]
  );
  const [users, setUsers] = useState([
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
  ]);

  const nextId = useRef(4);

  const onCreate = useCallback(() => {
    const user = {
      id: nextId.current,
      username,
      email,
    };
    setUsers(users.concat(user));
    setInputs({
      username: "",
      email: "",
    });
    console.log(nextId.current); //4
    nextId.current += 1;
  }, [username, email, users]);

  const onRemove = useCallback(
    (id) => {
      setUsers(users.filter((users) => users.id !== id));
    },
    [users]
  );

  const onToggle = useCallback(
    (id) => {
      setUsers(
        users.map((user) =>
          user.id === id ? { ...user, active: !user.active } : user
        )
      );
    },
    [users]
  );

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
        onRemove={onRemove}
        onToggle={onToggle}
      ></UserList>
      <div>활성 사용자 수 : {count}</div>
    </>
  );
}

export default App;
```

UserList 에 props 로 보내는 onRemove 와 onToggle 도 deps 배열에 users 가 들어있다. 결과적으로 users 가 변경되면 onRemove 와 onToggle 도 바뀌게되 리렌더링 되는 결과가 나타나게 되는 것이다.

해결하기 위해서는 props 로 보내는 함수들이 users 를 참조하지 않도록 해야 하는데 함수형 업데이트를 해주면 users 를 참조하지 않아도 된다.

```javascript
const onCreate = useCallback(() => {
  const user = {
    id: nextId.current,
    username,
    email,
  };
  setUsers((users) => users.concat(user));

  setInputs({
    username: "",
    email: "",
  });
  console.log(nextId.current); //4
  nextId.current += 1;
}, [username, email]);

const onRemove = useCallback((id) => {
  setUsers((users) => users.filter((users) => users.id !== id));
}, []);
```

`setUsers((users) => users.concat(user));` 이렇게 해주면 setUsers 에 등록한 콜백함수의 파라미터(users) 에서 최신 users 를 조회하기 때문에 deps 에 users 를 넣어줄 필요가 없게된다. 결과적으로 onCreate 함수는 username 과 email 이 변경될때만, onRemove 함수는 처음 한번만 렌더링이 되게된다.

React.memo 또한 매번 사용하는 것이 아닌 컴포넌트에 최적화가 필요할지 아닐지에 대한 판단을 정확히 하고 사용해야한다.
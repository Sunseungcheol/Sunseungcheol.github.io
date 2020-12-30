---
layout: post
categories: React.js
tags: [React.js, API, 비동기]
---
먼저 API 연동을 위해 axios 를 설치해준다.

```
yarn add axios
```

`axios` 는 `REST API` 를 요청할 때 이를 `Promise` 기반으로 처리할 수 있도록 도와주는 라이브러리다. 


#### useState 와 useEffect 로 요청 상태 관리하기

컴포넌트에서 `API` 를 요청하는 가장 기본적인 방법은 `useState` 와 `useEffect` 로 데이터를 로딩하는 것이다.

`API` 를 요청할때는 3가지의 상태를 관리하는데

1. 요청결과
2. 로딩상태
3. 에러

이다.

먼저 `useState` 를 사용해 위의 3가지 상태를 관리해준다.

```javascript
//User.js
import React, { useState, useEffect } from "react";
import axios from "axios";

function Users() {
  const [users, setUsers] = useState(null);
  //API가 요청중인지에 대한 상태
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  return <div></div>;
}
```

그 후 컴포넌트가 처음 렌더링 될 때 `API` 를 받아와야 하는데 이를 위해 `useEffect` 를 이용해준다. 

```javascript
  useEffect(() => {
    const fetchUsers = async () => {
      try {
        setUsers(null);
        setError(null);
        //로딩이 시작됨
        setLoading(true);

        const response = await axios.get(
          "https://jsonplaceholder.typicode.com/users/"
        );
        //요청 성공시 받아온 데이터값을 users 로 설정
        setUsers(response.data);
      } catch (e) {
        setError(e);
      }
      //로딩이 끝남
      setLoading(false);
    };
    fetchUsers();
  }, []);
```

이제 각 상태에 따라 다른 결과물을 렌더링 해주면 된다.

```javascript
//User.js
  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  //로딩은 끝났지만 users 의 값이 아직 없을경우
  if (!users) return null;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>
          {user.username} ({user.name})
        </li>
      ))}
    </ul>
  );
```

<ul>
  <li>Bret (Leanne Graham)</li>
  <li>Antonette (Ervin Howell)</li>
  <li>Samantha (Clementine Bauch)</li>
  <li>Karianne (Patricia Lebsack)</li>
  <li>Kamren (Chelsey Dietrich)</li>
  <li>Leopoldo_Corkery (Mrs. Dennis Schulist)</li>
  <li>Elwyn.Skiles (Kurtis Weissnat)</li>
  <li>Maxime_Nienow (Nicholas Runolfsdottir V)</li>
  <li>Delphine (Glenna Reichert)</li>
  <li>Moriah.Stanton (Clementina DuBuque)</li>
</ul>;

***

버튼을 만들어 버튼을 만들때 다시 요청을 하는 방법은 간단하다. 기존의 `fetchUsers` 함수를 바깥으로 빼준 후 버튼을 만들어 버튼의 클릭 이벤트로 넣어주기만 하면된다.

```javascript
import React, { useState, useEffect } from "react";
import axios from "axios";

function Users() {
  const [users, setUsers] = useState(null);
  //API가 요청중인지에 대한 상태
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchUsers = async () => {
    try {
      setUsers(null);
      setError(null);
      //로딩이 시작됨
      setLoading(true);

      const response = await axios.get(
        "https://jsonplaceholder.typicode.com/users/"
      );
      //요청 성공시 받아온 데이터값을 users 로 설정
      setUsers(response.data);
    } catch (e) {
      setError(e);
    }
    //로딩이 끝남
    setLoading(false);
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  //로딩은 끝났지만 users 의 값이 아직 없을경우
  if (!users) return null;

  return (
    <>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            {user.username} ({user.name})
          </li>
        ))}
      </ul>
      <button onClick={fetchUsers}>다시 불러오기</button>
    </>
  );
}

export default Users;

```


#### useReducer 로 요청 상태 관리하기

`useReducer` 을 사용하면 요청 관리에 대한 로직을 따로 분리하여 나중에 재사용할 수 있다.

`reducer` 함수에서는 

1. LOADING
2. SUCCESS
3. ERROR

3가지 액션을 관리해준다.

```javascript
function reducer(state, action) {
  switch (action.type) {
    case "LOADING":
      return {
        loading: true,
        data: null,
        error: null,
      };
    case "SUCCESS":
      return {
        loading: false,
        data: action.data,
        error: null,
      };
    case "ERROR":
      return {
        loading: false,
        data: null,
        error: action.error,
      };
    default:
      throw new Error(`Unhandled action type : ${action.error}`);
  }
}
```

기존의 상태별로 렌더링을 해주던 if 문은 비구조화 할당을 통해 해결한다.

```javascript
const { loading, data: users, error } = state;
  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  //로딩은 끝났지만 users 의 값이 아직 없을경우
  if (!users) return null;
```

이제 상황에 맞게 `action` 을 `dispatch` 해주면 된다.

```javascript
import React, { useReducer, useEffect } from "react";
import axios from "axios";

function reducer(state, action) {
  switch (action.type) {
    case "LOADING":
      return {
        loading: true,
        data: null,
        error: null,
      };
    case "SUCCESS":
      return {
        loading: false,
        data: action.data,
        error: null,
      };
    case "ERROR":
      return {
        loading: false,
        data: null,
        error: action.error,
      };
    default:
      throw new Error(`Unhandled action type : ${action.error}`);
  }
}

function Users() {
  const [state, dispatch] = useReducer(reducer, {
    loading: false,
    data: null,
    error: null,
  });

  const fetchUsers = async () => {
    dispatch({ type: "LOADING" });
    try {
      const response = await axios.get(
        "https://jsonplaceholder.typicode.com/users/"
      );
      //요청 성공시 받아온 데이터값을 users 로 설정
      dispatch({ type: "SUCCESS", data: response.data });
    } catch (e) {
      dispatch({ type: "ERROR", error: e });
    }
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  const { loading, data: users, error } = state;
  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  //로딩은 끝났지만 users 의 값이 아직 없을경우
  if (!users) return null;

  return (
    <>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            {user.username} ({user.name})
          </li>
        ))}
      </ul>
      <button onClick={fetchUsers}>다시 불러오기</button>
    </>
  );
}

export default Users;
```


#### useAsync 커스텀 HOOK 만들어 사용하기

위에서 만든 `reducer` 을 활용해 커스텀 훅을 만들어 사용해보려한다.

새로운 파일을 만든 후 기존의 `reducer` 함수를 복사한다. 그 후 `useAsync` 함수를 만드는데 `callback` 함수와 `deps` 를 파라미터로 받는다. 

`callback` 에는 `API` 를 받아오는 함수를, `deps` 에는 후에 `useEffect` 를 사용해 컴포넌트가 로딩됐을 때나 어떤 값이 변경되었을 때 `API` 를 재요청 할건데 `useEffect` 의 두번째 파라미터를 그대로 받아와 사용한다.

```javascript
//UseAsync.js
import { useReducer, useEffect, useCallback } from "react";

function reducer(state, action) {
  switch (action.type) {
    case "LOADING":
      return {
        loading: true,
        data: null,
        error: null,
      };
    case "SUCCESS":
      return {
        loading: false,
        data: action.data,
        error: null,
      };
    case "ERROR":
      return {
        loading: false,
        data: null,
        error: action.error,
      };
    default:
      throw new Error(`Unhandled action type : ${action.error}`);
  }
}

function useAsync(callback, deps = []) {
  const [state, dispatch] = useReducer(reducer, {
    loading: false,
    data: null,
    error: null,
  });

  const fecthData = useCallback(async () => {
    dispatch({ type: "LOADING" });
    try {
      dispatch({ type: "SUCCESS", data });
    } catch (e) {
      dispatch({ type: "ERROR", error: e });
    }
  }, [callback]);

  useEffect(() => {
    fecthData();
  }, deps);

  //첫번째 값 = 현재상태, 두번째 값 = 특정 요청을 다시 시작하는 함수
  return [state, fecthData];
}

export default useAsync;
```

이제 `Users` 컴포넌트에서 `useAsync` 에 사용할 콜백함수를 만들어 준다.

```javascript
//Users.js
async function getUsers() {
  const response = await axios.get(
    "https://jsonplaceholder.typicode.com/users"
  );
  return response.data;
}

function Users() {
  const [state, refetch] = useAsync(getUsers, []);

  const { loading, data: users, error } = state;
  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  //로딩은 끝났지만 users 의 값이 아직 없을경우
  if (!users) return null;

  return (
    <>
      <ul>
        {users.map((user) => (
          <li key={user.id}>
            {user.username} ({user.name})
          </li>
        ))}
      </ul>
      <button onClick={refetch}>다시 불러오기</button>
    </>
  );
}

export default Users;
```

만약 처음에는 배열을 보여주지 않고 특정한 이벤트가 있을 때만 보여주게 하기 위해서는 `useAsync` 에서 하나의 파라미터를 더 주고 그 값에 따라 `return` 을 시키면된다.

```javascript
//UseAsync.js
function useAsync(callback, deps = [], skip = false) {
  const [state, dispatch] = useReducer(reducer, {
    loading: false,
    data: null,
    error: false,
  });

  const fetchData = async () => {
    dispatch({ type: "LOADING" });
    try {
      const data = await callback();
      dispatch({ type: "SUCCESS", data });
    } catch (e) {
      dispatch({ type: "ERROR", error: e });
    }
  };
  useEffect(() => {
    if (skip) return;
    fetchData();
    // eslint 설정을 다음 줄에서만 비활성화
    // eslint-disable-next-line
  }, deps);
  //첫번째 값 = 현재상태, 두번째 값 = 특정 요청을 다시 시작하는 함수
  return [state, fetchData];
}

//User.js
function Users() {
  const [state, refetch] = useAsync(getUsers, [], true);

  const { loading, data: users, error } = state;
  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  //로딩은 끝났지만 users 의 값이 아직 없을경우
  if (!users) return <button onClick={refetch}>불러오기</button>;
```


#### API 에 파라미터가 필요한 경우

`API` 에 파라미터가 필요할 때가 있다. 새로운 컴포넌트를 만들어 `props` 로 아이디 값을 받아 `API` 를 요청한다.

```javascript
import React from 'react';
import axios from 'axios';
import useAsync from './useAsync';

async function getUser(id) {
  const response = await axios.get(
    `https://jsonplaceholder.typicode.com/users/${id}`
  );
  return response.data;
}

function User({ id }) {
  const [state] = useAsync(() => getUser(id), [id]);
  const { loading, data: user, error } = state;

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
```

`ussAsync` 에 파라미터를 넣어 함수를 호출하는 함수를 만들고 `deps` 에 `id` 를 넣어 `id` 가 변경될 때 마다 다시 호출되도록 만들었다.

그후 `Users.js` 에서 `id` 값을 관리하는 `useState` 를 만들어 리스트를 클릭하면 클릭한 리스트의 `id` 값을 `userId` 로 설정되게 만들었다.

```javascript
function Users() {
  const [state, refetch] = useAsync(getUsers, [], true);
  const [userId, setUserId] = useState(null);

  const { loading, data: users, error } = state;
  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  //로딩은 끝났지만 users 의 값이 아직 없을경우
  if (!users) return <button onClick={refetch}>불러오기</button>;

  return (
    <>
      <ul>
        {users.map((user) => (
          <li key={user.id} onClick={() => setUserId(user.id)}>
            {user.username} ({user.name})
          </li>
        ))}
      </ul>
      <button onClick={refetch}>다시 불러오기</button>
      {/* userId 의 기본값은 null, */}
      {userId && <User id={userId}></User>}
    </>
  );
}

export default Users;
```

  <div id="root">
    <ul>
      <li>Bret (Leanne Graham)</li>
      <li>Antonette (Ervin Howell)</li>
      <li>Samantha (Clementine Bauch)</li>
      <li>Karianne (Patricia Lebsack)</li>
      <li>Kamren (Chelsey Dietrich)</li>
      <li>Leopoldo_Corkery (Mrs. Dennis Schulist)</li>
      <li>Elwyn.Skiles (Kurtis Weissnat)</li>
      <li>Maxime_Nienow (Nicholas Runolfsdottir V)</li>
      <li>Delphine (Glenna Reichert)</li>
      <li>Moriah.Stanton (Clementina DuBuque)</li>
    </ul>
    <button>다시 불러오기</button>
    <div>
      <h2>Leopoldo_Corkery</h2>
      <p>
        <b>Email:</b> Karley_Dach@jasper.info
      </p>
    </div>
  </div>

  ***

  `useAsync` 를 만들어 사용함으로써 `User` 컴포넌트를 더욱 쉽게 만들 수 있었다.


  #### react-async 사용하기

  위에서 만들어 사용한 `useAsync` 와 비슷한 기능을 하는 라이브러리가 있다. 사용하기 위해 설치해주자

  ```
    yarn add react-async
  ```

  `react-async` 의 가장 기본적인 기능 중 하나는  `useAsync` 이다. 위에서 작성한 커스텀 훅과 이름이 같다. 커스텀 훅으로 만든 `useAsync` 와 비슷하면서도 다르다.

  ```javascript
  const {data, error, isLoading} = useAsync({ promise: loadCustomer, customer: 1});
  ```

  `react-async` 는 배열형태가 아닌 객체형태로 받아온다. 

  ```javascript
  //User.js
  import React from "react";
import axios from "axios";
import { useAsync } from "react-async";

async function getUser({ id }) {
  const response = await axios.get(
    `https://jsonplaceholder.typicode.com/users/${id}`
  );
  return response.data;
}

function User({ id }) {
  const { data: user, error, isLoading } = useAsync({
    promiseFn: getUser,
    id,
    //watch 는 deps 와 비슷한 역할을 한다.(id 값이 변하면 다시 호출)
    watch: id,
  });

  if (isLoading) return <div>로딩중..</div>;
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

// Users.js
import React, { useState } from "react";
import axios from "axios";
import { useAsync } from "react-async";
import User from "./User";

async function getUsers() {
  const response = await axios.get(
    "https://jsonplaceholder.typicode.com/users"
  );
  return response.data;
}

function Users() {
  const [userId, setUserId] = useState(null);
  const { data: users, error, isLoading, reroad, run } = useAsync({
    //이전과 같이 처음에는 불러오지 않으려면 deferFn 을 사용 후 run 을 넣어줄것
    //promiseFn: getUsers,
    deferFn: getUsers,
  });

  if (isLoading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  //로딩은 끝났지만 users 의 값이 아직 없을경우
  if (!users) return <button onClick={run}>불러오기</button>;

  return (
    <>
      <ul>
        {users.map((user) => (
          <li key={user.id} onClick={() => setUserId(user.id)}>
            {user.username} ({user.name})
          </li>
        ))}
      </ul>
      <button onClick={reroad}>다시 불러오기</button>
      {/* userId 의 기본값은 null, */}
      {userId && <User id={userId}></User>}
    </>
  );
}

export default Users;
```
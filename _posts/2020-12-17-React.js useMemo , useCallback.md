---
layout: post
categories: React.js
tags: [React.js, useMemo, useCallback, Hooks]
---

### useMemo

useMemo 의 memo 는 `memoization` 을 뜻하는데  `memoization` 이란 기존에 수행한 연산의 결과값을 재사용하는 프로그래밍 기법을 말한다. 즉 

```javascript
//App.js
import React, { useRef, useState } from "react";
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
  const onChange = (e) => {
    const { name, value } = e.target;
    setInputs({
      ...inputs,
      [name]: value,
    });
  };
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

  const onCreate = () => {
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
  };

  const onRemove = (id) => {
    setUsers(users.filter((users) => users.id !== id));
  };

  const onToggle = (id) => {
    setUsers(
      users.map((user) =>
        user.id === id ? { ...user, active: !user.active } : user
      )
    );
  };

  const count = countActiveUsers(users);

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
<div id="root">
  <div>
    <input name="username" placeholder="계정명" value="sss" /><input
      name="email"
      placeholder="이메일"
      value=""
    /><button>등록</button>
  </div>
  <div>
    <div>
      <b style="color: green; cursor: pointer"
        >Sun <span>(123@naver.com)</span></b
      ><button>삭제</button>
    </div>
    <div>
      <b style="color: green; cursor: pointer"
        >Jung <span>(456@naver.com)</span></b
      ><button>삭제</button>
    </div>
    <div>
      <b style="color: red; cursor: pointer">Kim <span>(789@naver.com)</span></b
      ><button>삭제</button>
    </div>
  </div>
  <div>활성 사용자 수 : 2</div>
</div>

***
위는 UserList 와 creatUser 에서 보여주는 값이다.

상단의 함수 countActiveUsers 를 보면 콘솔을 찍어준 후 return 값으로 배열의 active 값이 true 인 요소들의 개수를 세고 있는데 문제는 배열 users 의 값이 변경될 때 뿐만이 아니라 input 의 값이 바뀌어 리렌더링 될 때 마다 불러와 지는 것이다. 

```javascript
//console.log

//활성 사용자 수를 세고 있습니다.
//활성 사용자 수를 세고 있습니다.
//3 활성 사용자 수를 세고 있습니다.
```

굳이 필요가 없는 부분에서도 함수를 계속해서 불러오는 것이다.

이를 해결하는 방법 중 하나로 hook 의 useMemo 함수가 있다. 위에서 설명 했듯이 useMemo 는 기존에 수행한 값이 바뀌었을 때만 함수를 실행하고 값이 변하지 않는다면 리렌더링시 이전의 값을 가져오게 해주는 함이다.

```javascript
import React, { useRef, useState, useMemo } from "react";

const count = useMemo(() => countActiveUsers(users), [users]);
```
useMemo 의 첫번째 파라미터는 함수형식으로 사용해야 하고 두번째 파라미터에는 deps 를 넣어주는데, 두번째 파라미터 deps 배열 안에 있는 값이 바뀌어야만 첫번째 파라미터의 함수를 실행하게 된다. 

<div id="root">
  <div>
    <input name="username" placeholder="계정명" value="aaaaa" /><input
      name="email"
      placeholder="이메일"
      value=""
    /><button>등록</button>
  </div>
  <div>
    <div>
      <b style="color: green; cursor: pointer"
        >Sun <span>(123@naver.com)</span></b
      ><button>삭제</button>
    </div>
    <div>
      <b style="color: red; cursor: pointer"
        >Jung <span>(456@naver.com)</span></b
      ><button>삭제</button>
    </div>
    <div>
      <b style="color: red; cursor: pointer">Kim <span>(789@naver.com)</span></b
      ><button>삭제</button>
    </div>
  </div>
  <div>활성 사용자 수 : 1</div>
</div>

```javascript
//console.log

//활성 사용자 수를 세고 있습니다.
```

이제 인풋의 값이 변경되도 리렌더링만 될 뿐 함수는 실행되지 않는 것을 볼 수 있다.


### useCallback

쉽게 생각해 useMemo 가 값을 재사용한다면 useCallback 은 함수를 재사용하게 해준다. 위의 App.js 의 onToggle, onRemove 등의 함수들은 컴포넌트가 리렌더링 될 때 마다 새로 만들고 있다. 굳이 필요없는 작업들을 하고 있는 것이다.

사용방법은 크게 useMemo 와 크게 다르지 않다. 첫번째 파라미터로 함수를 넣어주고 두번째 파라미터 deps 배열에 참조하는 값을 넣어주면 된다. 
App.js의 onChange 함수에 useCallback 을 사용해보자

```javascript
import React, { useRef, useState, useMemo, useCallback } from "react";

const onChange = useCallback((e) => {
    const { name, value } = e.target;
    setInputs({
      ...inputs,
      [name]: value,
    });
  },[inputs]);
```

이와 같이 작성하면 inputs 가 변경될 때만 함수를 다시 불러오고 변경되지 않은 경우 이전의 함수를 그대로 사용하게 된다.

이때 주의해야할 점이 있는데 만약 useCallback 내부에서 참조하게 되는 상태, 혹은 props 로 받아온 어떠한 값이 있다면 그것또한 deps 에 넣어줘야 한다.

만약 넣어주지 않는다면 함수 내부에서 해당 상태들을 참조할 때 최신값이 아닌 이전의 값들을 참조하게 되는 경우가 나올 수 있다.

```javascript
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
```

useMemo 와 useCallback 남발하지 말고 컴포넌트 최적화에 대한 판단이 정확히 섯을 때 사용해야한다.
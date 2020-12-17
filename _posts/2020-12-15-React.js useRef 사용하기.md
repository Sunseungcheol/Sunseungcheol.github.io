---
layout: post
categories: React.js
tags: [React.js, useRef]
---

### useRef 로 돔 선택하기

javascript 를 사용할 때 특정 돔에 접근하기 위해서 querySelector, getElementById 를 사용했는데 React 함수형 컴포넌트에서는 useRef 를 사용한다. 사용법은 간단하다. 

먼저 import 구문에서 useRef 를 불러와주고 변수를 작성하여 useRef() 를 담는다.

```javascript
import React, { useState, useRef } from "react";

const nameInputs = useRef();
```

그 후 접근하고자 하는 DOM 에 ref 값으로 위에서 작성한 변수를 넣어주면 된다.

```html
 <input
    ref={nameInputs}
  />
```

이로써 input 에 접근이 가능해졌다. 
이전에 공부했던 예제를 활용해 초기화 버튼을 누를시 input 에 focus 가도록 만들어봤다.

```javascript
import React, { useState, useRef } from "react";

function InputSample() {
  const [inputs, setInputs] = useState({
    name: "",
    nickName: "",
  });
  const { name, nickName } = inputs;
  //useRef
  const nameInputs = useRef();
  const onChange = (e) => {
    const { name, value } = e.target;
    setInputs({ ...inputs, [name]: value });
  };
  const onReset = () => {
    setInputs({
      name: "",
      nickName: "",
    });
    //밑에서 설정해준 Ref 로 돔에 접근 후 명령이 가능하다.
    nameInputs.current.focus();
  };
  return (
    <div>
      <input
        name="name"
        placeholder="이름"
        onChange={onChange}
        value={name}
        //ref 값 넣기
        ref={nameInputs}
      />
      <input
        name="nickName"
        placeholder="닉네임"
        onChange={onChange}
        value={nickName}
      />
      <button onClick={onReset}>초기화</button>
      <div>
        <b>값 : </b>
        {name}({nickName})
      </div>
    </div>
  );
}

export default InputSample;
```

결과를 확인하면 초기화 버튼을 눌렀을 때 input:name 으로 focus 가 가는 것을 확인할 수 있다.


### useRef 로 컴포넌트 안 변수 만들기

컴포넌트 내부에서 let 을 사용하여 변수를 만드는 경우 리렌더링 되는 순간 초기화가 된다. 초기화를 막으려면 useState 를 사용해야되는데 useState 같은 경우 값이 바뀌면 리렌더링 된다. 
이때 useRef 를 사용하면 리렌더링 되도 초기화가 되지 않고 값이 바뀌어도 리렌더링 되지 않는다.

먼저 맨위에서 useRef 를 불러온 후 App.js 에 변수를 만들어 UserList 의 props 로 보낸다.

```javascript
//App.js
import React, { useRef } from "react";
import UserList from "./UserList";

function App() {
  const users = [
    {
      id: 1,
      username: "Sun",
      email: "123@naver.com",
    },
    {
      id: 2,
      username: "Jung",
      email: "456@naver.com",
    },
    {
      id: 3,
      username: "Kim",
      email: "789@naver.com",
    },
  ];
  return <UserList users={users}></UserList>;
}

export default App;

//UserList.js
import React from "react";

function User({ user }) {
  return (
    <div>
      <b>
        {user.username} <span>({user.email})</span>
      </b>
    </div>
  );
}
function UserList(users) {
  return (
    <div>
      {users.map((user) => (
        <User key={user.id} user={user}></User>
      ))}
    </div>
  );
}

export default UserList;

```

그 후 App.js 에 useRef 로 관리할 변수 하나를 추가해주고 새로운 항목을 추가하기 위한 함수를 만든다.

```javascript
  const nextId = useRef(4);
  
  const onCreate = () => {
    console.log(nextId.current); //4
    nextId.current += 1;
  };
```
이렇게 하면 onCreate 가 불릴 때 마다 nextId 의 값이 1씩 추가된다. 이때 위에서 말했듯이 nextId 의 값은 계속 가지고 있으며 onCreate() 를 이용해 값을 변경해도 리렌더링 되지 않는다.
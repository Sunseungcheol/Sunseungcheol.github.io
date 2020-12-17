---
layout: post
categories: React.js
tags: [React.js, useEffect, Hooks]
---
Hooks 는 리액트 v16.8 에 새로 도입된 기능인데, 앞서 공부했던 함수형 컴포넌트에서 상태 관리를 할 수 있는 useState, 렌더링 직후 작업을 설정하는 useEffect 등의 기능등을 제공한다.

useEffect 는 처음 컴포넌트가 화면에 나타날 때, 화면에서 사라지게 될 때, 컴포넌트가 업데이트되거나 업데이트 전, 또 렌더링 될 때마다 어떠한 작업을 할 수 있도록 해준다.

먼저 useEffect 를 사용하기 위해서 상단에서 불러와야 한다.

```javascript
import React, { useEffect } from "react";
```

useEffct 를 사용할 때는 첫번째 인자로는 실행하고자 하는 작업들을 입력해주고 두번째 인자(deps) 로는 의존되는 배열을 넣어주는데 만약 이 두번째 값이 비어있다면 컴포넌트가 처음 실행될때만 나타나게 된다.

```javascript
useEffect(() => {
    console.log("컴포넌트가 화면에 나타남");
  }, []);
```

컴포넌트가 사라지거나 변경될 때 특정 작업을 하려면 cleanup(뒷정리) 함수를 리턴해주기만 하면 된다.

```javascript
useEffect(() => {
  console.log("컴포넌트가 화면에 나타남");
  return () => {
    console.log("컴포넌트가 화면에서 사라짐");
  };
}, []);
```

이렇게 하면 컴포넌트가 나타날때 콘솔창에 `컴포넌트가 화면에 나타남` 라는 문구가 사라질 때 `컴포넌트가 화면에 사라짐` 이라는 문구가 나타나는 것을 볼 수 있다.

보통 컴포넌트가 마운트 될 때 추가하는 작업들로는
  
  1. props 값을 state 로 설정하는 경우
  2. 컴포넌트가 나타날 때 API 를 요청하는 경우
  3. 라이브러리를 사용하는 경우
  4. setInterval, setTimeout 등을 사용하는 경우

등이 있다.

언마운트 될 때의 작업들은

  1. clearInterval, clearTimeout 등
  2. 라이브러리 인스턴스 제거

등이 있다.

이번에는 useEffect 의 deps 에 배열을 넣어 확인해 보려한다.

```javascript
  useEffect(() => {
    console.log("값이 설정됨 :" + user);
    return () => {
      console.log("값이 바뀌기전 :" + user);
    };
  }, [user]);
```

이렇게 하면 처음 렌더링 될때 콘솔창에 `"값이 설정됨 :" + user 정보` 가 출력되고 만약 배열 user 의 정보가 바뀌게 된다면 cleanUp 함수의 `"값이 바뀌기전 :" + user 정보` 이 먼저 출력된 뒤 `"값이 설정됨 :" + user 정보` 가 출력된다.

정리하자면 useEffect 에 deps 에 어떤 값을 넣게 된다면 해당 값이 바뀔 때마다 설정한 함수가 호출되고 바뀌기 전 cleanUp 함수가 호출된다.

만약 useEffect 에서 등록한 함수에서 props 로 받은 값을 참조하거나 useState 로 관리 중인 값을 참조하는 경우에는 deps 배열에 꼭 넣어줘야 된다. 
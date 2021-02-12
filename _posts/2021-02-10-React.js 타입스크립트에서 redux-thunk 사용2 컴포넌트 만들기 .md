---
layout: post
categories: React.js
tags: [React.js, TypeScript, redux-thunk]
---
이전 포스트에 이어서 컴포넌트를 만든다.

#### 프리젠테이셔널 컴포넌트

두개의 프리젠테이셔널컴포넌트를 만들건데 하나는 사용자의 계정 입력 후 조회 시 api 를 호출하는 컴포넌트와 받아온 정보를 그려주는 컴포넌트를 만들 것이다.

```tsx
//components/GithubUsernameForm.tsx
import React, { ChangeEvent, FormEvent, useState } from "react";
import "./GithubUsername.css";

type GithubUsernameFormProps = {
  onSubmitUsername: (username: string) => void;
};

function GithubUsernameForm({ onSubmitUsername }: GithubUsernameFormProps) {
  const [input, setInput] = useState("");

  const onSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    onSubmitUsername(input);
  };
  const onChange = (e: ChangeEvent<HTMLInputElement>) => {
    setInput(e.target.value);
  };

  return (
    <form onSubmit={onSubmit} className="GithubUsernameForm">
      <input type="text" onChange={onChange} value={input} />
      <button>조회</button>
    </form>
  );
}

export default GithubUsernameForm;


//components/GithubProfileInfo.tsx
import React from "react";
import "./GithubProfileInfo.css";

//받아올 props 타입지정
type GithubProfileInfoProps = {
  name: null;
  thumbnail: string;
  bio: null;
  blog: string;
};

function GithubProfileInfo({
  name,
  thumbnail,
  bio,
  blog,
}: GithubProfileInfoProps) {
  return (
    <div className="GithubProfileInfo">
      <div className="profile-head">
        <img src={thumbnail} alt="user thumbnail" />
        <div className="name">{name}</div>
      </div>
      <p>{bio}</p>
      {/* 블로그가 공백이 아닌경우 블로그 보여주기 */}
      <div>{blog !== "" && <a href={blog}>블로그</a>}</div>
    </div>
  );
}

export default GithubProfileInfo;
```

두 개의 프리젠테이셔널 컴포넌트를 만들었는데 props 로 받아오는 값들의 타입지정과 onChange, onSubmit 의 event 객체 타입 지정을 제외하고 크게 다른 점은 없다.


#### 컨테이너 컴포넌트

```tsx
//containers/GithubProfileLoader.tsx
import React from "react";
import { useDispatch, useSelector } from "react-redux";
import { RootState } from "../modules";
import GithubUsernameForm from "../components/GithubUsernameForm";
import GithubProfileInfo from "../components/GithubProfileInfo";
import { getUserProfileThunk } from "../modules/github";

function GithubProfileLoader() {
  const { data, loading, error } = useSelector(
    (state: RootState) => state.github.userProfile
  );
  const dispatch = useDispatch();

  const onSubmitUsername = (username: string) => {
    dispatch(getUserProfileThunk(username));
  };

  return (
    <>
      <GithubUsernameForm onSubmitUsername={onSubmitUsername} />
      {loading && <p style={{ textAlign: "center" }}>로딩중..</p>}
      {error && <p style={{ textAlign: "center" }}>에러 발생!</p>}
      {data && (
        <GithubProfileInfo
          bio={data.bio}
          blog={data.blog}
          name={data.name}
          thumbnail={data.avatar_url}
        />
      )}
    </>
  );
}

export default GithubProfileLoader;

```

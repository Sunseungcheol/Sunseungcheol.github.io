---
layout: post
categories: React.js
tags: [React.js, TypeScript]
---
리액트에서 타입스크립트를 사용하기 위해서는 처음 리액트 앱을 만들때 하나의 명령어를 추가해줘야한다.

```
$ npx create-react-app ts-react-tutorial --template typescript
```

이렇게 생성할 경우 기존과는 다르게 파일들이 .tsx 파일로 생성된다.

#### 컴포넌트 생성

먼저 하나의 컴포넌트를 만들고 App.tsx 에서 불러와보자.

```typescript
//Greetings.tsx
import React from "react";

//props 타입 지정하기
type GreetingsProps = {
  name: string;
};

//제너릭스로 위에서 만든 props 타입 넣어주기
const Greetings: React.FC<GreetingsProps> = ({ name }) => <div>{name}</div>;

export default Greetings;

//App.tsx
import React from "react";
import logo from "./logo.svg";
import "./App.css";
import Greetings from "./Greetings";

const App: React.FC = () => {
  return <Greetings name={"리액트"}></Greetings>;
};

export default App;
```

지금 보면 App 컴포넌트에서 `React.FC` 를 사용하는 것을 볼 수 있다.

`React.FC` 를 사용할 때는 두가지 이점이 있는데 하나는 props 에 기본적으로 children 이 들어가 있어 따로 받아줄 필요가 없다는 것이다. 두번째는 컴포넌트의 defaultProps, propTypes, contextTypes 를 설정 할 때 자동완성(vscode 자동완성)이 가능하다는 것이다.

하지만 아직까지 큰 단점이 하나 있는데 defaultProps 가 제대로 작동하지 않는 다는 것이다.

만약 defaultProps 를 사용하고 싶은 경우 아래와 같이 사용해야 한다.

```typescript
//props 타입 지정하기
type GreetingsProps = {
  name: string;
  mark?: string;
};

//제너릭스로 위에서 만든 props 타입 넣어주기
const Greetings: React.FC<GreetingsProps> = ({ name, mark = "!" }) => (
  <div>
    {name}
    {mark}
  </div>
);

export default Greetings;
```

또는 function 키워드로 작성해주면 사용이 가능하다.

```typescript
type GreetingsProps = {
  name: string;
  mark: string;
};

//제너릭스로 위에서 만든 props 타입 넣어주기
function Greetings({ name, mark }: GreetingsProps) {
  return (
    <div>
      {name}
      {mark}
    </div>
  );
}

Greetings.defaultProps = {
  mark: "!",
};
```

이번에는 함수타입을 받아보자. 

```typescript
type GreetingsProps = {
  name: string;
  mark: string;
  //파라미터도 받아오지 않고 아무 값도 가져오지 않는다.
  onClick: () => void;
};
```

함수에 파라미터를 받아오고 싶다면 아래와 같이 하면된다. 버튼에 onClick 을 넣어 확인해보자.

```typescript
//Greetings.tsx
import React from "react";

//props 타입 지정하기
type GreetingsProps = {
  name: string;
  mark: string;
  //파라미터도 받아오지 않고 아무 값도 가져오지 않는다.
  onClick: (name: string) => void;
};

//제너릭스로 위에서 만든 props 타입 넣어주기
function Greetings({ name, mark, onClick }: GreetingsProps) {
  const handleClick = () => onClick(name);
  return (
    <div>
      {name}
      {mark}
      <div>
        <button onClick={handleClick}>클릭</button>
      </div>
    </div>
  );
}

Greetings.defaultProps = {
  mark: "!",
};

export default Greetings;

//App.tsx
import React from "react";
import Greetings from "./Greetings";

const App: React.FC = () => {
  const onClick = (name: string) => {
    console.log(name);
  };
  return <Greetings name={"리액트"} onClick={onClick}></Greetings>;
};

export default App;

```
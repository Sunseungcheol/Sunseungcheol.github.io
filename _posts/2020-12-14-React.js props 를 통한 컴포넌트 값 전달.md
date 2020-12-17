---
layout: post
categories: React.js
tags: [React.js, props]
---

props 는 properties 의 줄임말로 리액트 컴포넌트를 속성을 설정할 때 사용하는 요소이다. props 의 값은 해당 컴포넌트를 불러와 사용하는 부모 컴포넌트에서 설정할 수 있다.
`아래 예제에서 부모 = App, 자식은 Hello 가 된다.`

```javascript
//App.js
import React from "react";
import Hello from "./Hello";

function App() {
  return <Hello name="sun" color="red"></Hello>;
}
export default App;

//Hello.js
import React from "react";

function Hello(props) {
  return <div style={{ color: props.color }}>안녕하세요 {props.name}</div>;
}
//name = sun, color = red
export default Hello;
```

위와 같이 Hello.js 에서 파라미터로 props 를 받아 props.xx 의 형태로 사용 가능한데 비구조화 할당 문법을 통해 조금 더 편하게 작성할 수 있다.


### 비구조화 할당

```javascript
function Hello({color, name}){
  return <div style={{ color }}>안녕하세요 {name}</div>;
}
```
Hello() 의 인자로 props 의 값에 해당하는 color, name 를 먼저 받아 바로 사용하는 것인데 위 <div> 의 style 또한 저렇게 생략 가능하다.


### dafaultProps 설정

만약 App.js 에서 name 값(props 값)을 보내지 않으면 Hello.js 에서 {name} 부분이 빈 값으로 출력되게 되는데 이때 값을 받지 않았을 때 사용할 기본값을 설정해 줄 수 있다.

Hello.js 를 아래와 같이 변경하고

```javascript
function Hello({ color, name }) {
  return <div style={{ color }}>안녕하세요 {name}</div>;
}

Hello.defaultProps = {
  name: "이름 없음",
};
```
App.js 를 이렇게 바꾸면 

```javascript
function App() {
  return (
    <>
      <Hello name="sun" color="red"></Hello>
      <Hello color="blue"></Hello>
    </>
  );
}
```

아래와 같은 결과가 출력된다.

##### 결과

<div style='color:red'>안녕하세요 sun</div>
<div style='color:blue'>안녕하세요 이름 없음</div>
***


### children

다음으로 컴포넌트 태그 사이의 값을 보여주는 props 인 children 이 있는데 이는 말 태그 사이의 내용을 보여주게 된다.

예를 들어 App.js 에서 Hello 컴포넌트 안에 `칠드런` 이라는 문자열을 넣어준 후 

```javascript
function App() {
  return (
    <>
      <Hello name="sun" color="red">칠드런</Hello>
    </>
  );
}
```
Hello.js 에서 props 값으로 children 을 받아 사용하게 되면

```javascript
function Hello({ color, name, children }) {
  return (
    <div style={{ color }}>
      안녕하세요 {name}, 저는 {children}
    </div>
  );
}
```

아래와 같이 나온다.

##### 결과

<div style="color: red;">안녕하세요 sun, 저는 칠드런</div>
***

이는 태그 안에 있는 컴포넌트 또한 마찬가지 인데 새로운 컴포넌트를 만들고 그 안에 기존의 Hello 컴포넌트를 넣는다면 Hello 컴포넌트는 보이지 않는다.

```javascript
//App.js
function App() {
  return (
    <Wrapper>
      <Hello name="sun" color="red">칠드런</Hello>
      <Hello color="blue"></Hello>
    </Wrapper>
  );
}

//Wrapper.js
function Wrapper() {
  const style = {
    border: "2px solid #000",
    padding: 16,
  };

  return <div style={style}></div>;
}
```

##### 결과

<div style="border: 2px solid rgb(0, 0, 0); padding: 16px;"></div>
***

Hello 컴포넌트를 출력하기 위해 마찬가지로 children 으로 받으면 된다.

```javascript
//Wrapper.js
  return <div style={style}>{children}</div>;
}
```

##### 결과

<div style="border: 2px solid rgb(0, 0, 0); padding: 16px;">
  <div style="color: red;">안녕하세요 sun, 저는 칠드런</div>
  <div style="color: blue;">안녕하세요 , 저는 </div>
</div>


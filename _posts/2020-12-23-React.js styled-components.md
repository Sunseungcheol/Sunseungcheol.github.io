---
layout: post
categories: React.js
tags: [React.js, CSS, styled-components]
---
JS 안에서 CSS 를 작성할 수 있는 기술이 있는데 이를 CSS in JS 라고 한다. 관련 라이브러리는 여러개가 있는데 그중 가장 인기있는 styled-components 를 사용해 보려고 한다.

먼저 사용을 위해 프로젝트에 라이브러리를 설치해야 한다.

`$ yarn add styled-components`

설치 후 사용을 위해 상단에서 불러온 후 

```javascript
//App.js
import styled, { css } from "styled-components";
```

아래와 같이 컴포넌트를 생성하는데 이때 styled.태그 를 사용해 원하는 태그를 작성하면 된다.

```javascript
const Circle = styled.div`
  width: 5rem;
  height: 5rem;
  background-color: blue;
  border-radius: 50%;
`;

function App() {
  return <Circle></Circle>;
}

export default App;
```

<div class="sc-jSgupP jMjhqp"></div>
***

이때 props 를 통해 값을 전달해줄 수 있는데 사용법은 아래와 같다.

```javascript
function App() {
  return <Circle color="black"></Circle>;
}
```

props 를 똑같이 넘겨주고 아래와 같이 Template Literal 방식을 통해 사용하면 된다.

```javascript
const Circle = styled.div`
  width: 5rem;
  height: 5rem;
  background-color: ${(props) => props.color};
  border-radius: 50%;
`;
```

<div class="sc-fubCfw gJPUuq" color="black"></div>
***

마찬가지로 props 의 값이 true 인지 false 인지에 따라 다른 값을 줄 수 도있다.

```javascript
const Circle = styled.div`
  width: 5rem;
  height: 5rem;
  background-color: ${(props) => props.color};
  border-radius: 50%;
  ${(props) =>
    props.huge &&
    css`
      width: 10rem;
      height: 10rem;
    `}
`;

function App() {
  return <Circle color="blue" huge></Circle>;
}
```




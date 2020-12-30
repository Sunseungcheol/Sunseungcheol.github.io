---
layout: post
categories: React.js
tags: [React.js, CSS, styled-components]
---
재사용성 높은 버튼 컴포넌트를 styled-components 로 구현할 수있다. 예전에 퍼블리싱을 할때는 공용 CSS 파일을 만들고, 각 사이즈별로 .btnL, .btnM 등의 클래스를 만들어 사용했었다.

먼저 기본적인 버튼을 만들고 App.js 에서 불러와준다.

```javascript
//App.js
import React from 'react';
import styled from 'styled-components';
import Button from './components/Button';

const AppBlock = styled.div`
  width: 512px;
  margin: 0 auto;
  margin-top: 4rem;
  border: 1px solid black;
  padding: 1rem;
`;

function App() {
  return (
    <AppBlock>
      <Button>BUTTON</Button>
    </AppBlock>
  );
}

export default App;

//Button.js
import React from 'react';
import styled from 'styled-components';

const StyledButton = styled.button`
  /* 공통 스타일 */
  display: inline-block;
  outline: none;
  border: none;
  border-radius: 4px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  padding-left: 1rem;
  padding-right: 1rem;

  /* 크기 */
  height: 2.25rem;
  font-size: 1rem;

  /* 색상 */
  background: #228be6;
  &:hover {
    background: #339af0;
  }
  &:active {
    background: #1c7ed6;
  }

  /* 기타 */
  & + & {
    margin-left: 1rem;
  }
`;

function Button({ children, ...rest }) {
  return <StyledButton {...rest}>{children}</StyledButton>;
}

export default Button;
```


#### polished

CSS in JS 에서 스타일 관련 유틸 함수(lighten(), darken() 등) 를 사용하고 싶다면 <a href="https://polished.js.org/docs/" target="_blank">polished</a> 라는 라이브러리를 사용하면 된다.

```
$ yarn add polished
```

설치 후 기존의 색상 관련된 부분을 아래와 같이 수정해 줄 수 있다.

```javascript
import { darken, lighten } from 'polished';

background: #228be6;
&:hover {
  background: ${lighten(0.1, '#228be6')};
}
&:active {
  background: ${darken(0.1, '#228be6')};
}
```

이제 다른 색 버튼들을 만들기어보자. 색상 코드를 지닌 변수를 Button.js 에서 선언 하는 대신 <a href="https://www.styled-components.com/docs/api#themeprovider" target="_blank">ThemeProvider</a> 라는 기능을 사용하여 styled-components 로 만드는 모든 컴포넌트에서 조회하여 사용 할 수 있도록 전역으로 만들 수 있다.

```javascript
//App.js
import styled, { ThemeProvider } from 'styled-components';

function App() {
return (
    <ThemeProvider
      theme={{
        palette: {
          blue: '#228be6',
          gray: '#495057',
          pink: '#f06595'
        }}}
    >
      <AppBlock>
        <Button>BUTTON</Button>
      </AppBlock>
    </ThemeProvider>
  );
}
```

`theme` 를 설정하면 `ThemeProvider` 내부에 렌더링 된 styled-components 로 만든 컴포넌트에서 그 값들을 조회할 수가 있다.

```javascript
//Button.js

/* 색상 */
  ${props => {
    const selected = props.theme.palette[props.color];
    return css`
      background: ${selected};
      &:hover {
        background: ${lighten(0.1, selected)};
      }
      &:active {
        background: ${darken(0.1, selected)};
      }
    `;
  }}

///

Button.defaultProps = {
  color: 'blue'
};
```

ThemeProvider 로 설정한 값은 styled-components 에서 props.theme 로 조회 할 수 있다. 이때 비구조화 할당을 통해 조금 더 간결하게 표현할 수 있다.

```javascript
 /* 색상 */
  ${({ theme, color }) => {
    const selected = theme.palette[color];
    return css`
      background: ${selected};
      &:hover {
        background: ${lighten(0.1, selected)};
      }
      &:active {
        background: ${darken(0.1, selected)};
      }
    `;
  }}
```

이제 이 로직을 따로 빼내에 조금 더 깔끔하게 사용할 수 있는데 방법은 간단하다.
변수를 만들어 위의 내용을 담아주고, styled-components 안에서 변수를 불러주기만 하면된다.

```javascript
const colorStyles = css`
  ${({ theme, color }) => {
    const selected = theme.palette[color];
    return css`
      background: ${selected};
      &:hover {
        background: ${lighten(0.1, selected)};
      }
      &:active {
        background: ${darken(0.1, selected)};
      }
    `;
  }}
`;

const StyledButton = styled.button`
///
  /* 색상 */
  ${colorStyles}
`;
```

사이즈도 같은 props 를 받아 사용할 수 있는데 props 의 값에 따라 설정이 변하도록 만들기 위해 하나의 변수에 담았다.

```javascript
const sizeStyles = css`
  ${props =>
    props.size === 'large' &&
    css`
      height: 3rem;
      font-size: 1.25rem;
    `}

  ${props =>
    props.size === 'medium' &&
    css`
      height: 2.25rem;
      font-size: 1rem;
    `}

    ${props =>
      props.size === 'small' &&
      css`
        height: 1.75rem;
        font-size: 0.875rem;
      `}
`;
```

위의 코드를 보면 같은 코드가 계속 반복되는 것을 볼 수 있는데 
sizes 라는 객체를 만들어 height 와 font-size 에 대한 정보를 담아주자.

```javascript
const sizes = {
  large: {
    height: '3rem',
    fontSize: '1.25rem'
  },
  medium: {
    height: '2.25rem',
    fontSize: '1rem'
  },
  small: {
    height: '1.75rem',
    fontSize: '0.875rem'
  }
};
```

그 후 비구조화 할당을 통해 props 의 사이즈 값만 추출해주고 height 와 font-size 에 각 값을 넣어주면 된다.

```javascript
//Button.js
const sizeStyles = css`
  ${({ size }) => css`
    height: ${sizes[size].height};
    font-size: ${sizes[size].fontSize};
  `}
`;

//App.js
import React from "react";
import styled, { ThemeProvider } from "styled-components";
import Button from "./components/Button";

const AppBlock = styled.div`
  width: 512px;
  margin: 0 auto;
  margin-top: 4rem;
  border: 1px solid black;
  padding: 1rem;
`;

const ButtonGroup = styled.div`
  & + & {
    margin-top: 1rem;
  }
`;

function App() {
    <ThemeProvider
      theme={{
        palette: {
          blue: '#228be6',
          gray: '#495057',
          pink: '#f06595'
        }}}
    >
      <AppBlock>
        <ButtonGroup>
          <Button size="large">BUTTON</Button>
          <Button color="pink">BUTTON</Button>
          <Button color="gray" size="small">
            BUTTON
          </Button>
        </ButtonGroup>
      </AppBlock>
    </ThemeProvider>
  );
}

export default App;

```

#### outline 과 fullWidth 주기

이번에는 props 를 보내 outline 과 fullWidth 옵션을 줘보자.

먼저 Button 컴포넌트에서 props 로 outline 을 보낸다. 이때 기본값을 설정해주지 않아도 되는데 boolean 값을 props 로 보내는 경우 받지 않으면 undefinde 가 되며 false 와 똑같이 인식하게 된다.

```javascript
//Button.js
function Button({ children, color, size, outline, ...rest }) {
  return (
    <StyledButton color={color} size={size} outline={outline} {...rest}>
      {children}
    </StyledButton>
  );
}
```

이제 기존의 색상을 관리하던 부분에서 outline 의 값에 따라서 CSS 를 변경해주면 된다.

```javascript
const colorStyles = css`
  ${({ theme, color }) => {
    const selected = theme.palette[color];
    return css`
      background: ${selected};
      &:hover {
        background: ${lighten(0.1, selected)};
      }
      &:active {
        background: ${darken(0.1, selected)};
      }
      ${(props) =>
        props.outline &&
        css`
          color: ${selected};
          background: none;
          border: 1px solid ${selected};
          &:hover {
            background: ${selected};
            color: white;
          }
        `}
    `;
  }}
`;
```

fullWidth 도 같은 방식으로 해주면 된다.

```javascript
//Button.js
import React from "react";
import styled, { css } from "styled-components";
import { darken, lighten } from "polished";

const colorStyles = css`
  ${({ theme, color }) => {
    const selected = theme.palette[color];
    return css`
      background: ${selected};
      &:hover {
        background: ${lighten(0.1, selected)};
      }
      &:active {
        background: ${darken(0.1, selected)};
      }
      ${(props) =>
        props.outline &&
        css`
          color: ${selected};
          background: none;
          border: 1px solid ${selected};
          &:hover {
            background: ${selected};
            color: white;
          }
        `}
    `;
  }}
`;

const sizes = {
  large: {
    height: "3rem",
    fontSize: "1.25rem",
  },
  medium: {
    height: "2.25rem",
    fontSize: "1rem",
  },
  small: {
    height: "1.75rem",
    fontSize: "0.875rem",
  },
};

const sizeStyles = css`
  ${({ size }) => css`
    height: ${sizes[size].height};
    font-size: ${sizes[size].fontSize};
  `}
`;

const fullWidthStyle = css`
  ${(props) =>
    props.fullWidth &&
    css`
      width: 100%;
      justify-content: center;
      &:not(:first-child) {
        margin-left: 0 !important;
        margin-top: 1rem;
      }
    `}
`;

const StyledButton = styled.button`
  /* 공통 스타일 */
  display: inline-block;
  outline: none;
  border: none;
  border-radius: 4px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  padding-left: 1rem;
  padding-right: 1rem;

  /* 기타 */
  & + & {
    margin-left: 1rem;
  }
  /* 크기 */
  ${sizeStyles}

  /* 색상 */
  ${colorStyles}

  ${fullWidthStyle}
`;

function Button({ children, color, size, outline, fullWidth, ...rest }) {
  return (
    <StyledButton
      color={color}
      size={size}
      outline={outline}
      fullWidth={fullWidth}
      {...rest}
    >
      {children}
    </StyledButton>
  );
}

Button.defaultProps = {
  color: "blue",
  size: "medium",
};

export default Button;

//App.js
import React from "react";
import styled, { ThemeProvider } from "styled-components";
import Button from "./components/Button";

const AppBlock = styled.div`
  width: 512px;
  margin: 0 auto;
  margin-top: 4rem;
  border: 1px solid black;
  padding: 1rem;
`;

const ButtonGroup = styled.div`
  & + & {
    margin-top: 1rem;
  }
`;

function App() {
  return (
    <ThemeProvider
      theme={{
        palette: {
          blue: '#228be6',
          gray: '#495057',
          pink: '#f06595'
        }}}
    >
      <AppBlock>
        <ButtonGroup>
          <Button size="large">BUTTON</Button>
          <Button color="pink">BUTTON</Button>
          <Button color="gray" size="small">
            BUTTON
          </Button>
        </ButtonGroup>
        <ButtonGroup>
          <Button size="large" outline>
            BUTTON
          </Button>
          <Button color="pink" outline>
            BUTTON
          </Button>
          <Button color="gray" size="small" outline>
            BUTTON
          </Button>
        </ButtonGroup>
        <ButtonGroup>
          <Button size="large" fullWidth>
            BUTTON
          </Button>
          <Button color="pink" outline fullWidth>
            BUTTON
          </Button>
          <Button color="gray" size="small" fullWidth>
            BUTTON
          </Button>
        </ButtonGroup>
      </AppBlock>
    </ThemeProvider>
  );
}

export default App;

```
---
layout: post
categories: React.js
tags: [React.js, Context API,Consumer]
---
프로젝트를 해보면서 Context, redux 등이 아직 모자라 복습의 필요성을 느꼇다.
먼저 정적인 상태에서 사용법을 다시 한번 익혀보자.

#### Context 기본 사용법

```js
//contexts/color.js
import { createContext } from "react";

//기본 상태 지정
const ConlorContext = createContext({ color: "black" });

export default ConlorContext;

//components/ColorBox.js
import React from "react";
import ConlorContext from "../contexts/color";

function ColorBox() {
  return (
    <ConlorContext.Consumer>
      {(value) => (
        <div
          style={{
            width: `64px`,
            height: "64px",
            background: value.color,
          }}
        />
      )}
    </ConlorContext.Consumer>
  );
}

export default ColorBox;
```

위와 같이 `Consumer` 컴포넌트를 통해 색상의 조회가 가능하다.

`Consumer` 는 context 변화를 구독하는 컴포넌트 인데 함수 컴포넌트 안에서 context 를 읽기 위해 쓸 수 있다.
또 `Consumer` 컴포넌트의 children 은 JSX 나 문자열이 아닌 함수여야 하는데, 저 때 value 값은 해당 context 의 Provider 중 가장 가까운 Provider 의 value 를 참조한다. 만약 상위에 Provider 가 없다면 값은 createContext() 에서 보낸 기본값과 동일하다.

참고로 `Consumer` 와 같이 내부에 함수를 넣어주는 패턴을 Function as a child, 또는 Render Props 라고 한다. 


#### Provider

만들어 놓은 Context 로 App.js 에서 렌더링되는 컴포넌트를 감싸주고 이때 Provider 를 사용해주면 Context 의 value 값을 변경할 수 있다.

```js
//App.js
import ColorBox from "./components/ColorBox";
import ColorContext from "./contexts/color";

function App() {
  return (
    <ColorContext.Provider value={{ color: "red" }}>
      <ColorBox></ColorBox>
    </ColorContext.Provider>
  );
}

export default App;
```

만약 Provider 를 사용할 때는 value 값을 반드시 명시해줘야 한다는 것을 잊지말자.
기존에 Context 에서 지정해놓은 기본값은 Provider 가 사용되지 않을 때만 사용된다.


#### 동적 Context 사용

Context 의 value 로 상태값만이 아닌 함수도 전달해 줄 수 있다.

```js
//contexts/color.js
import { createContext, useState } from "react";

const ColorContext = createContext({
  state: { color: "black", subColor: "red" },
  action: {
    setColor: () => {},
    setSubColor: () => {},
  },
});

const ColorProvider = ({ children }) => {
  const [color, setColor] = useState("black");
  const [subColor, setSubColor] = useState("red");

  //따로 분리해주는 것이 다른 컴포넌트에서 사용하기 편함
  const value = {
    state: { color, subColor },
    action: { setColor, setSubColor },
  };
  return (
    <ColorContext.Provider value={value}>{children}</ColorContext.Provider>
  );
};

//const ColorConsumner = ColorContext.Counsumer 과 같음
const { Consumer: ColorConsumner } = ColorContext;

//만든것들 내보내기
export { ColorProvider, ColorConsumner };

export default ColorContext;


//App.js
import ColorBox from "./components/ColorBox";
import ColorContext, { ColorProvider } from "./contexts/color";

function App() {
  return (
    <ColorProvider>
      <ColorBox></ColorBox>
    </ColorProvider>
  );
}

export default App;


//components/ColorBox.js
import React from "react";
import ColorContext, { ColorConsumner } from "../contexts/color";

function ColorBox() {
  return (
    <ColorConsumner>
      {(value) => (
        <>
          <div
            style={{
              width: "64px",
              height: "64px",
              background: value.state.color,
            }}
          ></div>

          <div
            style={{
              width: "64px",
              height: "64px",
              background: value.state.subColor,
            }}
          ></div>
        </>
      )}
    </ColorConsumner>
  );
}

export default ColorBox;

```

검은색 박스와 빨간색 박스가 정상적으로 나온다. 
이때 createContext 의 기본값은 실제 Provider 의 value 값과 일치하는 것이 좋다. 그래야 Context 코드를 볼 때 내부값이 어떻게 구성되어 있는지 파악하기가 쉽고 실수로 Provider 을 사용하지 않은 경우 에러가 발생하지 않는다.


#### 색상 선택 컴포넌트 만들기

위의 Context 에서 action 에 넣은 함수를 호출하는 컴포넌트를 만들어보자.

```js
//components/SelectColor.js
import React from "react";
import { ColorConsumner } from "../contexts/color";

const colors = ["red", "blue", "yellow", "pink", "green", "indigo"];

function SelectColor() {
  return (
    <div style={{ marginBottom: "15px" }}>
      <h2>색상을 선택하세요</h2>
      <ColorConsumner>
        {({ action }) => (
          <div style={{ display: "flex" }}>
            {colors.map((color) => (
              <div
                key={color}
                style={{
                  width: "30px",
                  height: "30px",
                  marginRight: "5px",
                  background: color,
                  cursor: "pointer",
                }}
                onClick={() => action.setColor(color)}
                onContextMenu={(e) => {
                  e.preventDefault(); //우클릭시 메뉴 뜨는 것 무시
                  action.setSubColor(color);
                }}
              ></div>
            ))}
          </div>
        )}
      </ColorConsumner>
    </div>
  );
}

export default SelectColor;

//App.js
import ColorBox from "./components/ColorBox";
import SelectColor from "./components/SelectColor";
import { ColorProvider } from "./contexts/color";

function App() {
  return (
    <ColorProvider>
      <SelectColor></SelectColor>
      <ColorBox></ColorBox>
    </ColorProvider>
  );
}

export default App;

```

색상을 선택하는 div 들에서 action 객체를 사용하기 위해 ColorConsumer 로 감싸주고 왼쪽 클릭이냐 오른쪽클릭이냐에 따라 setColor, setSubColor 를 줌으로써 좌클릭시 위의 박스, 우클릭시 아래의 박스 색상이 변하도록 해줬다.


#### useContext

이전에 공부했던 것과 같이 useContext 를 사용하면 Consumer 보다 더욱 편하게 값을 사용할 수 있다.
다만 함수형 컴포넌트에서만 사용이 가능하다.

```js
//components/ColorBox.js
import React, { useContext } from "react";
import ColorContext, { ColorConsumner } from "../contexts/color";

function ColorBox() {
  const { state } = useContext(ColorContext);
  return (
    <>
      <div
        style={{
          width: "64px",
          height: "64px",
          background: state.color,
        }}
      ></div>

      <div
        style={{
          width: "64px",
          height: "64px",
          background: state.subColor,
        }}
      ></div>
    </>
  );
}

export default ColorBox;

```


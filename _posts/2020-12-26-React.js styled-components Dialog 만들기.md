---
layout: post
categories: React.js
tags: [React.js, CSS, styled-components]
---
#### Dialog 만들기

기존에 작성한 Button.js 를 활용해 Dialog 를 만들어 보려고 한다.
먼저 기본적인 틀을 만들어주자.

```javascript
import React from "react";
import styled from "styled-components";
import Button from "./Button";

const DarkBkg = styled.div`
  position: fixed;
  display: flex;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.75);
`;

const DialogBlock = styled.div`
  width: 320px;
  padding: 1.5rem;
  background: white;
  border-radius: 2px;

  h3 {
    margin: 0;
    font-size: 1.5rem;
  }

  p {
    font-size: 1.125rem;
  }
`;

const ButtonGroup = styled.div`
  margin-top: 3px;
  display: flex;
  justify-content: flex-end;
`;

function Dialog({ title, children, confirmText, cancelText }) {
  return (
    <DarkBkg>
      <DialogBlock>
        <h3>{title}</h3>
        <p>{children}</p>
        <ButtonGroup>
          <Button color="gray">{cancelText}</Button>
          <Button color="pink">{confirmText}</Button>
        </ButtonGroup>
      </DialogBlock>
    </DarkBkg>
  );
}

Dialog.defaultProps = {
  cancelText: "취소",
  confirmText: "확인",
};
export default Dialog;

//App.js
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
        <Dialog title="삭제하시겠습니까?" confirmText="삭제" cancelText="취소">
          정말로 삭제하시겠습니까?
        </Dialog>
      </AppBlock>
    </ThemeProvider>
  );
}
```

여기서 App.js 의 Dialog 컴포넌트를 AppBlock 바깥으로 빼면 오류가 나게 된다. ThemaProvider 내부에는 하나의 엘리먼트만 존재해야 되기 때문이다. ThemaProvider 안에서 <></> 프레그먼트를 사용하면 간단히 해결된다.

이제 App.js 에서 필요한 클릭에 따라 변경되도록 함수를 만들어주고 하나의 boolean 값을 설정해 그 값에 따라 Dialog 컴포넌트가 보일지 말지 결정한다.

```javascript
//App.js
const [dialog, setDialog] = useState(false);
const onClick = () => {
  setDialog(true);
};
const onConfirm = () => {
  console.log("확인");
  setDialog(false);
};
const onCancle = () => {
  console.log("취소");
  setDialog(false);
};

///
<Dialog
        title="삭제하시겠습니까?"
        confirmText="삭제"
        cancelText="취소"
        visible={dialog}
        onConfirm={onConfirm}
        onCancle={onCancle}
      >
        정말로 삭제하시겠습니까?
</Dialog>

//Dialog.js
function Dialog({
  title,
  children,
  confirmText,
  cancelText,

  visible,
  onConfirm,
  onCancle,
}) {
  if (!visible) return null;
  return (
    <DarkBkg>
      <DialogBlock>
        <h3>{title}</h3>
        <p>{children}</p>
        <ButtonGroup>
          <ShortMarginBtn color="gray" onClick={onCancle}>
            {cancelText}
          </ShortMarginBtn>
          <ShortMarginBtn color="pink" onClick={onConfirm}>
            {confirmText}
          </ShortMarginBtn>
        </ButtonGroup>
      </DialogBlock>
    </DarkBkg>
  );
}
```

Dialog.js 에서는 받아온 visible 값을 사용해 visible 이 flase 면 null 을 반환하도록 설정해줬다. 


#### 트랜지션 구현하기

Dialog 가 나타날 때 트랜지션 효과를 주기 위해서는 기존 CSS 의 키프레임을 이용하면 된다.

```javascript
import styled, { keyframes } from "styled-components";

const fadeIn = keyframes`
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
`;

const slideUp = keyframes`
  from {
    transform: translateY(200px);
  }
  to{
    transform: translateY(0)
  }
`;
```

먼저 사용을 위해 상단에서 불러온 후 변수에 담아 사용하면 된다. 그 후 styled-components 에서 animation-name 을 작성한 변수명으로 사용해주면 된다. `ex) animation-name : ${ fadeIn }`

나타나게 하는 것에 비해 다시 사라지게 하는 것은 조금 번거로운데 이를 위해 useState 와 useEffect 가 필요하다.
Dialog 에서 현재 애니메이션을 보여주고 있다는 정보를 의미하는 animate 와 현재 visible 상태에 대한 정보를 감지할 수 있는 것 두가지가 필요하다.

```javascript
  const [animate, setAnimate] = useState(false);
  const [localVisible, setLocalVisible] = useState(visible); //props 로 받아온 visible 값
```

이후 useEffect 를 이용해 visible 이 true 에서 flase 로 바뀔 때 animate의 값을 설정해준다.

```javascript
useEffect(() => {
  //visible true -> false
  if (localVisible && !visible) {
    setAnimate(true);
    setTimeout(() => setAnimate(false), 250);
  }
  //visible 값이 바뀔 때마다 localVisible 동기화
  setLocalVisible(visible);
}, [localVisible, visible]);
```

이제 기존의 if 문도 수정해줘야하는데 애니메이트 중이 아니면서 로컬비지블이 아닐 때로 변경해줘야 한다.

```javascript
  if (!localVisible && !animate) return null;
```

이제 남은 건 효과만 구현해주면 된다. 키프레임을 이용해 slideDown 과 fadeOut 을 만들어주고 조건에 따라 animation-name 을 변경해주면 된다.

```javascript
//Dialog.js
import React, { useState, useEffect } from "react";
import styled, { keyframes, css } from "styled-components";
import Button from "./Button";

const fadeIn = keyframes`
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
`;

const fadeOut = keyframes`
  from {
    opacity: 1;
  }
  to {
    opacity: 0;
  }
`;

const slideUp = keyframes`
  from {
    transform: translateY(200px);
  }
  to{
    transform: translateY(0)
  }
`;

const slideDown = keyframes`
  from {
    transform: translateY(0);
  }
  to{
    transform: translateY(200px)
  }
`;

const DarkBkg = styled.div`
  position: fixed;
  display: flex;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.75);

  animation-duration: 0.25s;
  animation-name: ${fadeIn};
  animation-fill-mode: forwards;

  ${(props) =>
    props.disapper &&
    css`
      animation-name: ${fadeOut};
    `}
`;

const DialogBlock = styled.div`
  width: 320px;
  padding: 1.5rem;
  background: white;
  border-radius: 2px;

  animation-duration: 0.25s;
  animation-name: ${slideUp};
  animation-fill-mode: forwards;

  ${(props) =>
    props.disapper &&
    css`
      animation-name: ${slideDown};
    `}

  h3 {
    margin: 0;
    font-size: 1.5rem;
  }

  p {
    font-size: 1.125rem;
  }
`;

const ButtonGroup = styled.div`
  margin-top: 3px;
  display: flex;
  justify-content: flex-end;
`;

const ShortMarginBtn = styled(Button)`
  & + & {
    margin-left: 0.5rem;
  }
`;

function Dialog({
  title,
  children,
  confirmText,
  cancelText,
  visible,
  onConfirm,
  onCancle,
}) {
  const [animate, setAnimate] = useState(false);
  const [localVisible, setLocalVisible] = useState(visible); //props 로 받아온 visible 값

  useEffect(() => {
    //visible true -> false
    if (localVisible && !visible) {
      setAnimate(true);
      setTimeout(() => setAnimate(false), 250);
    }
    //visible 값이 바뀔 때마다 localVisible 동기화
    setLocalVisible(visible);
  }, [localVisible, visible]);

  if (!localVisible && !animate) return null;
  return (
    <DarkBkg disapper={!visible}>
      <DialogBlock disapper={!visible}>
        <h3>{title}</h3>
        <p>{children}</p>
        <ButtonGroup>
          <ShortMarginBtn color="gray" onClick={onCancle}>
            {cancelText}
          </ShortMarginBtn>
          <ShortMarginBtn color="pink" onClick={onConfirm}>
            {confirmText}
          </ShortMarginBtn>
        </ButtonGroup>
      </DialogBlock>
    </DarkBkg>
  );
}

Dialog.defaultProps = {
  cancelText: "취소",
  confirmText: "확인",
};
export default Dialog;

```
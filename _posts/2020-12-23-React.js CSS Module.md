---
layout: post
categories: React.js
tags: [React.js, CSS, CSS Module]
---

React 프로젝트에서 컴포넌트를 스타일링 할 때 CSS Module 을 사용하면 CSS 클래스가 중첩되는 것을 방지할 수 있다.
사용법은 간단한데 CSS 파일 확장자를 `.module.css` 로 하면 된다.

`Box.module.css` 라는 파일을 만든 후 .Box 에 대한 스타일을 만들어보자

```CSS
/*Box.module.css*/
.Box {
  background: black;
  color: white;
  padding: 2rem;
}
```

CSS 파일내에서 사용방법은 같지만 컴포넌트 파일에서 불러올 때 조금 다르게 불러온다. 
아래와 같이 `import styles from "./Box.module.css";` styles 를 붙여 불러온 후 className 을 설정할 때 `styles.Box` 와 같이 import 로 불러온 styles 를 참조하면 된다. 이렇게 하면 className 에 파일경로, 이름, 해쉬값, 클래스이름등이 사용 되어 만들어진다.

```javascript
import React from "react";
import styles from "./Box.module.css";

function Box() {
  return <div className={styles.Box}>{styles.Box}</div>;
}

export default Box;
```

```html
<div class="_src_Box_module_Box">_src_Box_module_Box</div>    
```

조금 더 자세히 알아보기 위해 새로운 프로젝트를 작성하고 체크값에 따라 변화되는 인풋을 만들어보자

```javascript
//App.js
import React, { useState } from "react";
import CheckBox from "./components/CheckBox";

function App() {
  const [check, setCheck] = useState(false);
  const onChange = (e) => {
    setCheck(e.target.checked);
  };
  return (
    <div className="App">
      <CheckBox onChange={onChange} checked={check}>
        다음 약관에 모두 동의
      </CheckBox>
    </div>
  );
}

export default App;

//CheckBox.js
import React from "react";

function CheckBox({ checked, children, ...rest }) {
  return (
    <div>
      <label>
        <input type="checkbox" {...rest} checked={checked} />
        <div>{checked ? "체크" : "체크안됨"}</div>
      </label>
      <span>{children}</span>
    </div>
  );
}

export default CheckBox;
```

checked 와 children 을 제외한 나머지 props 를 간단히 받아오기 위해 ...rest 를 사용했다.

이제 인풋을 가리고 체크에 대한 내용을 아이콘으로 변경해줄건데 이를 위해 `ract-icons` 라는 라이브러리를 사용하려고 한다.

```
yarn add react-icons
```

로 설치한 후 사용하면 되는데 자세한 사용법과 아이콘의 모양은 https://react-icons.github.io/react-icons/#/ 에서 확인이 가능하다.

```javascript
import { MdCheckBox, MdCheckBoxOutlineBlank } from "react-icons/md";

function CheckBox({ checked, children, ...rest }) {
  return (
    <div>
      <label>
        <input type="checkbox" {...rest} checked={checked} />
        <div>{checked ? <MdCheckBox /> : <MdCheckBoxOutlineBlank />}</div>
      </label>
      <span>{children}</span>
    </div>
  );
}
```

이제 CSS.mocule 파일을 만들어 주는데 여기서는 클래스 이름이 중복될 걱정을 하지 않아도 된다. 아래와 같이 작성한 후

```CSS
/*CheckBox.module.css*/
.checkBox {
  display: flex;
  align-items: center;
}

.checkBox label {
  cursor: pointer;
}

.checkBox input {
  position: absolute;
  width: 0;
  height: 0;
  opacity: 0;
}

.checkBox span {
  font-size: 1.125rem;
  font-weight: bold;
}

.icon {
  display: flex;
  align-items: center;
  font-size: 2rem;
  margin-right: 0.25rem;
  color: #2e2e2e;
}

.checked {
  color: #1376be;
}
```

CheckBox.js 에서 콘솔을 찍어 확인해 보면 아래와 같이 고유한 클래스 네임이 생성된 것을 볼 수 있다.

```javascript
import styles from "./CheckBox.module.css";

console.log(styles);
//checkBox: "CheckBox_checkBox__1uvSa"
//checked: "CheckBox_checked__2D8y_"
//icon: "CheckBox_icon__2zG6x"
```

```javascript
function CheckBox({ checked, children, ...rest }) {
  return (
    <div className={styles.checkBox}>
      <label>
        <input type="checkbox" {...rest} checked={checked} />
        <div className={styles.icon}>
          {checked ? (
            <MdCheckBox className={styles.checked} />
          ) : (
            <MdCheckBoxOutlineBlank />
          )}
        </div>
      </label>
      <span>{children}</span>
    </div>
  );
}
```

위와 같이 styles 를 불러온 후 styles 객체안의 값을 조회하는 형태로 사용하면 된다.

만약 하나에 두가지 이상의 클래스 네임이 들어갈 경우 작성하기가 번거로워 지는데 이때는 classnames 라이브러리르 통해 조금 더 편하게 사용할 수 있다.

```
yarn add classnames
```

설치 후 불러올 때 /bind 를 불러오는데 바인드는 CSS 모듈을 사용할 때 조금 더 쉽게 사용할 수 있도록 도와주는 유틸이다.

```javascript
import classNames from "classnames/bind";

const cx = classNames.bind(styles);
```

먼저 변수에 `classNames.bind` 를 통해 styles 를 담아준 후

```javascript
function CheckBox({ checked, children, ...rest }) {
  return (
    <div className={cx("checkBox")}>
    {/* <div className={cx("checkBox"), "다른 클래스", props}> */}
      <label>
        <input type="checkbox" {...rest} checked={checked} />
        <div className={cx("checkbox")}>
          {checked ? (
            <MdCheckBox className={cx("checkbox")} />
          ) : (
            <MdCheckBoxOutlineBlank />
          )}
        </div>
      </label>
      <span>{children}</span>
    </div>
  );
}
```

위와 같은 방식으로 사용해 주면된다.

혹시 module.css 내부에서 글로벌로 사용하고 싶은 이름이 있다면 글로벌 키워드를 넣어주면 된다.

```SCSS
/* ex.module.css*/

:global .mt-global-name{

}

/* ex.module.scss*/

:global {
  .mt-global-name{

  }
}
```

반대로 모듈을 사용하지 않은 CSS 파일에서 local 키워드를 넣어주면 CSS module 형태로 사용할 수 있다.

```SCSS
/* ex.module.css*/

:local .mt-global-name{

}

/* ex.module.scss*/

:local {
  .mt-global-name{

  }
}
```
---
layout: post
categories: React.js
tags: [React.js, input상태관리, useState]
---

### 하나의 인풋관리

하나의 input 관리는 크게 어렵지 않다. input 의 상태관리를 할 때도 버튼과 마찬가지로 useState 를 사용해 작업한다. 
먼저 useState 로 text 값을 관리할 수 있게 만든 후 기본값으로 공백을 주고 input onChange 이벤트가 있을때 마다 `onChange()` 함수를 줘 `setText()` 를 통해 input 의 value 를 text로 넣어주면 된다.

```javascript
import React, { useState } from "react";

function InputSample() {
  const [text, setText] = useState("");
  const onChange = (e) => {
    setText(e.target.value);
  };
  const onReset = () => {
    setText("");
  };
  return (
    <div>
      <input onChange={onChange} value={text} />
      <button onClick={onReset}>초기화</button>
      <div>
        <b>값 : </b>
        {text}
      </div>
    </div>
  );
}

export default InputSample;
```
만약 input 의 value 에 `{text}` 를 주지 않는다면 초기화 버튼 등, 다른 곳에서 text 의 값을 수정했을 때 input 은 변화가 없으니 주의해야한다.

#### 결과

<div>
  <input value="sunsunsun">
  <button>초기화</button>
  <div>
      <b>값 : </b>sunsunsun
  </div>
</div>
***


### input 이 여러 개일 때

input 이 여럿인 경우 먼저 input 에 name 값을 줘 그것을 참조하게 하고 useState 에서는 객체를 사용해 관리하는 방법을 사용하는게 좋다.
물론 useState 를 여러개 작성해서 사용해도 되기는 하지만 그건 효율성 측면이든 가독성 측면에서든 좋지 않은 방법이다.
먼저 input 을 아래와 같이 만드는데 input 을 컨트롤 하기 위해 name 값을 설정해준다.

```html
<div>
  <input name="name" placeholder="이름" onChange={onChange} />
  <input name="nickName" placeholder="닉네임" onChange={onChange} />
  <button onClick={onReset}>초기화</button>
  <div>
    <b>값 : </b>
  </div>
</div>
```
useState 에는 객체를 넣고, name 과 nickName 을 비구조화 할당을 이용해 추출해준다.

```javascript
  const [inputs, setInputs] = useState({
    name: "",
    nickName: "",
  });
  const { name, nickName } = inputs;
```

그 후 onChange 를 수정해보자.

```javascript
 const onChange = (e) => {
    const { name, value } = e.target;
    const nextInputs = {
      ...inputs,
      [name]: value,
    };

    setInputs(nextInputs);
  };
```

객체 상태를 업데이트 해줄 때는 반드시 기존의 상태를 복사한 후 거기에 새로운 값을 덮어씌우고 새로운 상태로 설정해줘야 한다. 이것을 `불변성`을 지킨다고 하는데 `불변성`을 지켜줘야만 리액트 컴포넌트에서 상태가 변경된 것을 감지할 수 있고 이에 따라 필요한 렌더링이 발생하게 된다.

따라서 스프레드 문법을 이용해 inputs 를 복사해 새로운 객체를 만들고 `[name]` 에 value 를 넣어 setInputs 에 새로 만든 객체를 넣어준다. 
이때 `name : value` 이런 식으로 대괄호 없이 사용할 경우 `name` 이라는 문자열이 들어가니 주의해야 한다. 대괄호로 감싸주면 name 이 무엇을 가리키냐에 따라 키값이 변경된다.
위의 문장을 조금 더 간단하게 만들면 굳이 nextInputs 를 만들지 않고 setInput 안에서 바로 로직을 실행해주면 된다.

```javascript
 const onChange = (e) => {
    const { name, value } = e.target;
    setInputs({ ...inputs, [name]: value });
  };
```

이제 onReset 도 같은 방법으로 inputs 의 name 과 nickName 을 빈값으로 바꾸어주고 input 에 각각의 value 값을 넣어주면 된다.

```javascript
import React, { useState } from "react";

function InputSample() {
  const [inputs, setInputs] = useState({
    name: "",
    nickName: "",
  });
  const { name, nickName } = inputs;
  const onChange = (e) => {
    const { name, value } = e.target;
    setInputs({ ...inputs, [name]: value });
  };
  const onReset = () => {
    setInputs({
      name: "",
      nickName: "",
    });
  };
  return (
    <div>
      <input 
        name="name" 
        placeholder="이름" 
        onChange={onChange} 
        value={name} />
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
<div>
  <input name="name" placeholder="이름" value="선">
  <input name="nickName" placeholder="닉네임" value="Sun">
  <button>초기화</button>
  <div><b>값 : </b>선(Sun)</div>
</div>
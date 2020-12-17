---
layout: post
categories: React.js
tags: [React.js, props]
---

propType를 이용해 컴포넌트의 필수 props 를 지정하거나 props 의 타입을 지정할 수 있다. 방법은 deafaultProp 와 비슷한데 사용전 import 구문을 통해 불러와야 한다.

```javascript
//MyComponents.js
import React, { Component } from "react";
import PropTypes from "prop-types";//propTypes 불러오기

function MyComponent({name, favoriteNumber}){ 
    return (
      <div>
        안녕하세요 제이름은 {name}입니다. <br />
        좋아하는 숫자는 {favoriteNumber} 입니다.
      </div>
    );
  }
}

MyComponent.propTypes = {
  name: PropTypes.string,
  favoriteNumber: PropTypes.number,
};

export default MyComponent;

//App.js
function App() {
  return <MyComponents name={3} favoriteNumber={"one"}></MyComponents>;
}
```
위와 같이 name 은 string, favoriteNumber 는 number 값으로 지정한 후 App.js 에서 name 값은 숫자, 좋아하는 숫자는 문자열로 설정했다.
결과는 보낸 값대로 출력이 되기는 하지만 콘솔창을 보면 경고메시지가 나온다.


```javascript
안녕하세요 제이름은 3입니다. 
좋아하는 숫자는 one 입니다.
//warning: Failed prop type: Invalid prop `name` of type `number` supplied to `MyComponents`, expected `string`.
//Warning: Failed prop type: Invalid prop `favoriteNumber` of type `string` supplied to `MyComponents`, expected `number`
```

다시 설정해 놓은 타입에 맞게 고치면 경고메시지가 사라지는 걸 알 수 있다.

```javascript
//App.js
function App() {
  return <MyComponents name={"sun"} favoriteNumber={1}></MyComponents>;
}
```

propType 을 지정할 떄 isRequired 를 붙여주면 필수 propTypes 로 설정되어 해당하는 props 가 없을 경우 콘솔창에 경고메시지를 띄우게 된다.

```javascript
//App.js
function App() {
  return <MyComponents name={"sun"} favoriteNumber={1}></MyComponents>;
}

//MyComponents.js
MyComponents.propTypes = {
  name: PropTypes.string,
  favoriteNumber: PropTypes.number,
};
//Warning: Failed prop type: The prop `favoriteNumber` is marked as required in `MyComponents`, but its value is `undefined`
```


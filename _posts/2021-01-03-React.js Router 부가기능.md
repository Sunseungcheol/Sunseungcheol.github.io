---
layout: post
categories: React.js
tags: [React.js, Router]
---

#### history 객체

history 객체는 라우터 컴포넌트에 전달되는 props 중 하나이다. history 객체를 통하여, 컴포넌트 내의 메소드에서 라우터에 직접 접근이 가능해진다. (뒤로가기, 특정경로로 이동, 이탈방지 등)

History 컴포넌트를 아래와 같이 만들고 이를 App.js 에서 불러보자

```javascript
//HistorySample.js
import React, { useEffect } from "react";

function HistorySample({ history }) {
  const goBack = () => {
    //goBack 뒤로 가기
    history.goBack();
  };
  const goHome = () => {
    //push 특정경로로 이동(기록이 남음)
    history.push("/");
  };
  const replaceToHome = () => {
    //replace 특정 경로로 이동(기록이 남지 않음)
    history.replace("/");
  };

  useEffect(() => {
    console.log(history);
    //페이지 이탈 방지
    const unblock = history.block("정말 떠나실건가요?");
    return () => {
      //컴포넌트가 언마운트 될 때는 unBlock 비활성화
      unblock();
    };
  }, [history]);
  return (
    <div>
      <button onClick={goBack}>뒤로가기</button>
      <button onClick={goHome}>홈으로 가기</button>
      <button onClick={replaceToHome}>홈으로 가기(리플레이스)</button>
    </div>
  );
}

export default HistorySample;

//App.js
function App() {
  return (
    <div>
      <ul>
        <li>
          <Link to="/">홈</Link>
        </li>
        <li>
          <Link to="/about">소개</Link>
        </li>
        <li>
          <Link to="/profiles">프로필 목록</Link>
        </li>
        <li>
          <Link to="/history">히스토리</Link>
        </li>
      </ul>
      <hr />
      {/* path 는 경로를 의미 */}
      <Route path="/" component={Home} exact></Route>
      <Route path="/about" component={About}></Route>
      <Route path="/profiles" component={Profiles}></Route>
      <Route path="/history" component={HistorySample}></Route>
    </div>
  );
}
```

위와 같이 여러 기능들이 있는데 위에 사용한 예제 이외에도 `goFord()` 등 여러 가지가 있다. 보통 가장 많이 쓰이는 두가지가 `push()` 와 `block()` 이다. 

위의 주석을 보면 페이지 이탈 방지라고 적혀 있는데 말 그대로 페이지를 나가려고 하면 alert 창이 뜨게 된다. 


#### withRouter

withRouter 은 라우터 컴포넌트가 아닌 곳에서 match, location, history 를 사용 할 수 있게 도와준다.

사용법은 간단한데 상단에서 withRouter 을 불러오고 하단에서 감싸주면 된다.

```javascript
import React from "react";
import { withRouter } from "react-router-dom";

function WithRouterSample() {
  return <div></div>;
}

export default withRouter(WithRouterSample);
```

이렇게 해주면 외부에서 사용할 때도 라우트 컴포넌트로 제작하지 않아도 라우터의 props 를 사용할 수 있다.

```javascript
//WithRouterSample.js
import React from "react";
import { withRouter } from "react-router-dom";

function WithRouterSample({ match, location, history }) {
  return (
    <div>
      <h4>location</h4>
      <textarea value={JSON.stringify(location, null, 2)} readOnly></textarea>
      <h4>match</h4>
      <textarea value={JSON.stringify(match, null, 2)} readOnly></textarea>
      <button onClick={() => history.push("/")}>홈으로</button>
    </div>
  );
}

export default withRouter(WithRouterSample);

//Profiles.js
function Profiles() {
  return (
    <div>
     //...
      <WithRouterSample></WithRouterSample>
    </div>
  );
}
```

`JSON.stringify` 는 JSON 으로 이루어진 객체를 문자열로 변화해 주고 null 과 2 를 넣어주면 문자열로 변화하는 과정 중 들여쓰기가 된다.

location 과 match 가 보여주는 경로기준은 다른데 
location 의 경우 어디서 불러오던지 같은 정보를 가져오는데 match 는 렌더링 된 위치를 기준으로 match 를 읽어온다.


#### Switch

`Switch` 를 사용하면 여러 가지 라우트 중 가장 먼저 매칭된 라우트 하나만을 보여준다.

먼저 기존의 App.js 의 상단에서 `Switch` 를 불러와주고 라우트 컴포넌트들을 감싸주면 된다.

```javascript
//App.js
import { Route, Link, Switch } from "react-router-dom";

//...
function App(){
    return (
        //...
    <Switch>
        <Route path="/" component={Home} exact></Route>
        <Route path="/about" component={About}></Route>
        <Route path="/profiles" component={Profiles}></Route>
        <Route path="/history" component={HistorySample}></Route>
    </Switch>
    )
}
```

`Switch` 는 보통 페이지를 못찾았을 때 유용하다. 예를 들어 아래와 같이 만든 후 아무 주소나 적게되면 아래와 같이 나오게 된다.

```javascript
      <Switch>
        <Route path="/" component={Home} exact></Route>
        <Route path="/about" component={About}></Route>
        <Route path="/profiles" component={Profiles}></Route>
        <Route path="/history" component={HistorySample}></Route>
        {/*path 가 없으면 모든 상황에 렌더링 된다.*/}
        <Route
          render={({ location }) => (
            <div>
              <h2>이 페이지는 존재하지 않습니다.</h2>
              <p>{location.pathname}</p>
            </div>
          )}
        ></Route>
      </Switch>
```

<div><h2>이 페이지는 존재하지 않습니다.</h2><p>/djifjdi</p></div>

***

이는 `Switch` 가 매칭되는 가장 첫번째 컴포넌트를 찾아 내려오다가 아무것도 찾지 못해 맨 아래까지 내려와 마지막의 "이 페이지는 존재하지 않습니다" 가 출력되게 된것이다. 


#### NavLink

`NavLink` 는 현재 경로와 `Link` 에서 사용하는 경로가 일치하는 경우 특정 스타일, 클래스 등을 적용할 수 있는 컴포넌트 이다. 기존의 Profiles.js 에서 Link 를 NavLink 로 변경하고 activeStyle 이란 걸 줬다.

```javascript
//Profiles.js
function Profiles() {
  return (
    <div>
      <h3>사용자 목록</h3>
      <ul>
        <li>
          <NavLink
            to="/profiles/sun"
            activeStyle={{ background: "black", color: "white" }}
          >
            sun
          </NavLink>
        </li>
        <li>
          <NavLink
            to="/profiles/kim"
            activeStyle={{ background: "black", color: "yellow" }}
          >
            kim
          </NavLink>
        </li>
      </ul>

      {/* render 로 함수를 바로 리턴할 수 있다. */}
      <Route
        path="/profiles"
        exact
        render={() => <div>사용자를 선택해주세요.</div>}
      ></Route>
      <Route path="/profiles/:username" component={Profile}></Route>
      <WithRouterSample></WithRouterSample>
    </div>
  );
}
```

경로가 바뀜에 따라 설정해 놓은 activeStyle 값으로 잘 바뀌는 것을 볼 수 있다.

만약 특정 class 를 주고 싶다면 아래와 같이 하면 된다.

```javascript
<NavLink to="/profiles/sun" activeStyle={{ background: "black", color: "white" }} activeClassName="active">
```

이외에 Prompt, Redirect, RouteConfig, Memory Router 등이 있는데 

Redirect: 페이지를 리디렉트 하는 컴포넌트
Prompt: history.block 의 컴포넌트 버전
Route Config: JSX 형태로 라우트를 선언하는 것이 아닌 배열/객체를 사용하여 라우트 정의(앵귤러, 뷰 같이)
Memory Router 실제로 주소가 존재하지는 않는 라우터. 리액트 네이티브나, 임베디드 웹앱에서 사용하면 유용

<a href="https://reacttraining.com/react-router/web/guides/philosophy" target="_blank">공식 문서</a>


#### useReactRouter Hook

`useReactRouter Hook` 을 통해 location, match, history 객체를 사용할 수 있다. 해당 기능은 아직 Router 에 정식 탑재된 기능은 아니다. withRouter 을 사용하는 것보다 편하다.

사용법은 간단하다.

```
yarn add use-react-router
```

설치후 상단에서 불러오기만 하면 된다.

```javascript
import useReactRouter from 'use-react-router';
```
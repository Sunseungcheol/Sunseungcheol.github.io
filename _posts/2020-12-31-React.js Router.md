---
layout: post
categories: React.js
tags: [React.js, Router]
---
react-router 을 통해 리액트 SPA를 만들 수 있다. 여기서 SPA 란 Single Page Application 의 약자로 페이지가 1개인 어플리케이션을 뜻한다. 나는 이전에 .NET 을 사용했기 때문에 react 를 공부하면서 처음 접해본 개념이었다. 

기존의 웹페이지는 페이지를 불러올 때 마다 서버로부터 리소스를 전달 받은 후 그려줬는데 SPA 구조는 전체를 가져온 후 필요한 부분을 보여주는 것이다.

내가 이해한게 맞다면 WOW 와 몬헌의 맵 리딩 차이와 같다. 맵을 크게 한번에 불러오는 것과 작은 여러개의 맵을 위치가 변동될 때 마다 불러오는 것의 것의 차이이다.


#### 기본 사용법

먼저 사용을 위해서 프로젝트에 패키지를 설치해준다.

```
yarn add react-router-dom
```

이제 프로젝트의 index.js 에 가서 `BrowserRouter` 을 불러오고 App 컴포넌트를 감싸주면 사용하기 위한 준비가 완료된다.

```javascript
//index.js
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import { BrowserRouter } from "react-router-dom";

ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById("root")
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

이제 간단한 실습을 위해 Home 과 About 두개의 컴포넌트를 만든다.

```javascript
//Home.js
import React from "react";

function Home() {
  return (
    <div>
      <h1>홈</h1>
      <p>이곳은 홈으로, 가장 먼저 보여지는 페이지입니다.</p>
    </div>
  );
}

export default Home;

//About.js
import React from "react";

function About() {
  return (
    <div>
      <h1>어바웃</h1>
      <p>이 프로젝트에서는 라우터 기초를 실습해보겠습니다.</p>
    </div>
  );
}

export default About;
```

이제 App.js 에 `Route` 컴포넌트를 추가해주는데, 특정 주소에 특정 컴포넌트를 보여주는 역할을 한다.

```javascript
import { Route } from "react-router-dom";
import About from "./About";
import Home from "./Home";

function App() {
  return (
    <div>
      {/* path 는 경로를 의미 */}
      <Route path="/" component={Home}></Route>
      <Route path="/about" component={About}></Route>
    </div>
  );
}

export default App;
```

프로젝트를 실행한 후 주소 뒤에 /about 를 쳐서 들어갔는데 예상과는 다르게 아래와 같이 나왔다.

<div><div><h1>홈</h1><p>이곳은 홈으로, 가장 먼저 보여지는 페이지입니다.</p></div><div><h1>어바웃</h1><p>이 프로젝트에서는 라우터 기초를 실습해보겠습니다.</p></div></div>

***

이런 문제가 발생하게 된 원인은 /about 또한 / 안에 속해있기 때문이다. 이를 해결하기 위해서는 컴포넌트에 `exact` 값을 true 로 보내주면 된다. `exact` 값을 true 로 넣어주면 경로와 완전히 일치하는 경우에만 보여주게 된다.

```javascript
//App.js
<Route path="/" component={Home} exact></Route>
```

이제 /about 을 들어가보면 about 만 나오는 것을 볼 수 있다.

Router 에서 경로를 이동하기 위해서는 Link 라는 컴포넌트를 사용하는데

```javascript
import { Route,Link } from "react-router-dom";
```

기존의 `a` 태그를 사용해서 이동하면 안된다. `a` 태그를 사용하게 되면 페이지가 새로 로딩된다. `Link` 태그는 to 를 사용해 경로를 변경하는데 작성방법은 아래와 같다.

```javascript
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
      </ul>
      <hr />
      {/* path 는 경로를 의미 */}
      <Route path="/" component={Home} exact></Route>
      <Route path="/about" component={About}></Route>
    </div>
  );
}
```

이제 확인해보면 새로운 요청 없이 경로가 바뀌는 것을 볼 수 있다.


#### 파라미터와 쿼리 

이제 Router 에서 파라미터와 쿼리를 날리고 받아보자.

먼저 파라미터를 날려보자

```javascript
//Profile.js
import React from "react";

const profileData = {
  sun: {
    name: `선선`,
    about: "선선은 선선이다.",
  },
  kim: {
    name: "킴킴",
    about: "킴킴은 킴킴이다.",
  },
};

//match : Route 컴포넌트에서 자동으로 넣어주는 props, 설정해줄 필요 없음
function Profile({ match }) {
  const { username } = match.params;
  const profile = profileData[username];

  if (!profile) {
    return <div>해당 사용자가 존재하지 않습니다.</div>;
  }

  return (
    <div>
      <h3>
        {username} ({profile.name})
      </h3>
      <p>{profile.about}</p>
    </div>
  );
}

export default Profile;
```

이제 App.js 에서 값을 날려주면 되는데 

```javascript
//App.js
<Route path="/profiles/:username" component={Profile}></Route>
//Profile.js
const { username, id } = match.params;
```

이제 주소창에 /profiles/sun 과 같이 검색하고자 하는 이름을 넣어주면 정상적으로 출력되는 것을 볼 수 있다. 만약 하나가 아닌 여러가지를 보내고 싶은 경우 아래와 같이 하면 된다.

```javascript
//App.js
<Route path="/profiles/:username/:id" component={Profile}></Route>
```

이번에는 쿼리를 사용해 보자.
쿼리를 조회할 때는 props 를 통해 location 을 받아와 조회할 수 있다.

```javascript
//About.js
function About({ location }) {
  console.log(location);
  return (
    <div>
      <h1>어바웃</h1>
      <p>이 프로젝트에서는 라우터 기초를 실습해보겠습니다.</p>
    </div>
  );
}
```

콘솔을 확인해 보기 위해 주소창에 /about?a=3 이라고 치면 아래와 같이 search 에 값이 보이는 것을 알 수 있다.

```
{pathname: "/about", search: "?a=3&b=2", hash: "", state: undefined}
hash: ""
pathname: "/about"
search: "?a=3"
state: undefined
__proto__: Object
```

사용을 위해서 search 의 값을 추출해야 되는데 이 작업은 qs 라는 라이브러리를 사용해 할 수 있다.

```
yran add qs
```

```javascript
//About.js
import React from "react";
import qs from "qs";

function About({ location }) {
  console.log(location);
  //파싱
  const query = qs.parse(location.search, {
    //설정 안해줄 시 물음표까지 들고옴
    ignoreQueryPrefix: true,
  });

  return (
    <div>
      <h1>어바웃</h1>
      <p>이 프로젝트에서는 라우터 기초를 실습해보겠습니다.</p>
    </div>
  );
}

export default About;
```

이제 어떠한 값에 따라 다르게 보여주도록 구현해 보자.

```javascript
function About({ location }) {
  //파싱
  const query = qs.parse(location.search, {
    //설정 안해줄 시 물음표까지 들고옴
    ignoreQueryPrefix: true,
  });

  //무조건 문자열로 들고온다.
  const detail = query.detatil === "true";

  return (
    <div>
      <h1>어바웃</h1>
      <p>이 프로젝트에서는 라우터 기초를 실습해보겠습니다.</p>
      {detail && <p>detail 값이 true 입니다. </p>}
    </div>
  );
}
```

detail 의 값이 true 면 `detail 값이 true 입니다.` 라는 문장이 나오도록 구현했다. 여기서 주의해야 할 점은 쿼리는 값을 무조건 문자열로 가져온다는 것이다.


#### 서브 라우트 만들기

서브 라우트란 라우트 안에 들어있는 또 다른 라우트다. 라우트로 설정한 컴포넌트 내부에서 라우트를 한번 더 쓰면 된다.

```javascript
//Profiles.js
import React from "react";
import Profile from "./Profile";
import { Link, Route } from "react-router-dom";

function Profiles() {
  return (
    <div>
      <h3>사용자 목록</h3>
      <ul>
        <li>
          <Link to="/profiles/sun">sun</Link>
        </li>
        <li>
          <Link to="/profiles/kim">kim</Link>
        </li>
      </ul>

      {/* render 로 함수를 바로 리턴할 수 있다. */}
      <Route
        path="/profiles"
        exact
        render={() => <div>사용자를 선택해주세요.</div>}
      ></Route>
      <Route path="/profiles/:username" component={Profile}></Route>
    </div>
  );
}

export default Profiles;

//App.js
import { Route, Link } from "react-router-dom";
import About from "./About";
import Home from "./Home";
import Profiles from "./Profiles";

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
      </ul>
      <hr />
      {/* path 는 경로를 의미 */}
      <Route path="/" component={Home} exact></Route>
      <Route path="/about" component={About}></Route>
      <Route path="/profiles" component={Profiles}></Route>
    </div>
  );
}

export default App;
```

보통 특정 경로에 텝이 있는 경우, 태그가 있는 경우 사용하면 매우 편하다.

***
// /profiles/sun

<div><ul><li><a href="/">홈</a></li><li><a href="/about">소개</a></li><li><a href="/profiles">프로필 목록</a></li></ul><hr><div><h3>사용자 목록</h3><ul><li><a href="/profiles/sun">sun</a></li><li><a href="/profiles/kim">kim</a></li></ul><div><h3>sun (선선)</h3><p>선선은 선선이다.</p></div></div></div>

***
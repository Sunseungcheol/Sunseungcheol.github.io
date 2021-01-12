---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, redux-saga, Routher, 라우터, History]
---
리덕스 사가에서 라우터의 히스토리를 사용해보자

```javascript
//index.js
const customHistory = createBrowserHistory();
const sagaMiddleware = createSagaMiddleware({
  context: {
    history: customHistory,
  },
});
```
`createSagaMiddleware` 안에서 context 를 등록하면 되는데 저렇게 조회해놓으면 사가에서 조회가 가능하다.

조회하는 방법은 간단하다. 이전의 /modules/posts.js 안의 goToHome 을 보자.

먼저 이것도 마찬가지로 thunk 함수와는 다르게 일반 액션객체를 반환한다.

```javascript
//modules/posts.js
//saga 홈으로 가기
const GO_TO_HOME = "posts/GO_TO_HOME";

//...

//홈으로 가는 사가 (history 이용)
export const goToHome = () => ({
  type: GO_TO_HOME,
});
```

이제 saga 함수를 작성해주면 되는데 그 안에서 getContext 를 사용해준다. getContext 도 effects 중 하나이다.

```javascript
//modules/posts.js
//홈으로 가기
function* goToHomeSaga() {
  //index.js 에서 설정한 context 이름
  const history = yield getContext("history");
  history.push("/");
}

//리덕스 모듈을 위해 사가를 모니터링하는 함수
export function* postsSaga() {
  yield takeEvery(GET_POSTS, getPostsSaga);
  yield takeEvery(GET_POST, getPostSaga);
  yield takeEvery(GO_TO_HOME, goToHomeSaga);
}
```

정리하자면 saga 내부에서 history 를 사용할 일이 있다면 saga 미들웨어를 만들때 context 에 history 를 등록해 사용하면 된다.
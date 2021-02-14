---
layout: post
categories: React.js
tags: [React.js, TypeScript, redux-saga]
---
이전에 작성했던 saga 를 조금 더 쉽게 사용하기 위해 유틸함수를 만들어보자.

```ts
//lib/createAsyncSaga.ts
//P = promiseCreator 호출 할 때 넣는 파라미터, T = 해당 promiseCreator 만든 promise 에서 내보내는 결과값

import { call, put } from "redux-saga/effects";
import { AsyncActionCreatorBuilder, PayloadAction } from "typesafe-actions";

//파라미터가 없는 경우도 작동할 수 있도록 or 로 처리
type PromiseCreatorFunction<P, T> =
  | ((payload: P) => Promise<T>)
  | (() => Promise<T>);

//타입가드 문법
//액션.payload 가 undefinde 가 아니라면  파라미터로 받아온 액션은 payload 액션타입이다.
function isPayloadAction(action: any): action is PayloadAction<string, any> {
  return action.payload !== undefined;
}

export default function createAsyncSaga<T1, P1, T2, P2, T3, P3>(
  asyncActionCreator: AsyncActionCreatorBuilder<
    // [액션타입 , 페이로드타입] or [액션타입 [페이로드타입,메타타입]]
    [T1, [P1, undefined]],
    [T2, [P2, undefined]],
    [T3, [P3, undefined]]
  >,
  promiseCreator: PromiseCreatorFunction<P1, P2>
) {
  return function* saga(action: ReturnType<typeof asyncActionCreator.request>) {
    try {
      //isPayloadAction 이 true 인 경우와 아닌 경우
      const result: P2 = isPayloadAction(action)
        ? yield call(promiseCreator, action.payload)
        : yield call(promiseCreator);
      yield put(asyncActionCreator.success(result));
    } catch (e) {
      yield put(asyncActionCreator.failure(e));
    }
  };
}

//modules/github/saga.ts
import { getUserProfileAsync, GET_USER_PROFILE } from "./actions";
//call = 특정 함수 호출, put = 액션 디스패치, takeEvery = 특정액션 모니터링 중 원하는 액션 들어올 시 정해놓은 사가함수 호출
import { takeEvery } from "redux-saga/effects";
import { getUserPeofile } from "../../api/guthub";
import createAsyncSaga from "../../lib/createAsyncSaga";

// function* getUserProfileSaga(
//   action: ReturnType<typeof getUserProfileAsync.request>
// ) {
//   try {
//     const userProfile: GithubProfile = yield call(
//       getUserPeofile,
//       action.payload
//     );
//     //성공시
//     yield put(getUserProfileAsync.success(userProfile));
//   } catch (e) {
//     //실패시
//     yield put(getUserProfileAsync.failure(e));
//   }
// }

const getUserProfileSaga = createAsyncSaga(getUserProfileAsync, getUserPeofile);

export function* githubSage() {
  //GET_USER_PROFILE 액션 감지시 사가함수 호출
  yield takeEvery(GET_USER_PROFILE, getUserProfileSaga);
}
```
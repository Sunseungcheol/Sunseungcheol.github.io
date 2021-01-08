---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, useSeletor]
---
기본적으로, `useSelector` 를 사용해서 리덕스 스토어의 상태를 조회 할 땐 상태가 바뀌지 않았다면 리렌더링 하지 않는다.

하지만 아래와 같이 새로운 객체를 만드는 상황등에서 상태가 바뀌었는지 바뀌었는지 확인을 할 수 없어 다른 컴포넌트가 변경되어도 계속해서 렌더링을 하는 상황이 벌어지게 된다. 
 
기존에 작성했던 `useSelector` 를 확인해보자

```javascript
  const { number, diff } = useSelector((state) => ({
    number: state.counter.number,
    diff: state.counter.diff,
  }));
```

문제를 방지하기 위해 두가지 방법이 있는데 첫번째는 아래와 같이 각 값별로 `useSelector` 를 따로 사용해 주는 것이다.

```javascript
  const number = useSelector((state) => state.counter.number);
  const diff = useSelector((state) => state.counter.diff);
```

두번째는 `useSelector` 의 두번째 파라미터로 `equalityFn` 을 보내주는 것인데 이전 상태와 현재 상태를 비교하는 함수이다. true 값이 나오면 리랜더링을 하지 않고 false 값이 나온 경우에만 리렌더링을 한다.

```javascript
  const { number, diff } = useSelector(
    (state) => ({
      number: state.counter.number,
      diff: state.counter.diff,
    }),
    //객체의 경우 값들을 다 따로 비교해줘야 한다.
    (left, right) => {
      return left.diff === right.diff && left.number === right.number;
    }
  );
```

위의 `equalityFn` 을 조금 더 편하게 사용할 수 있는데 리덕스의 `shallowEqual` 함수를 이용해주면 된다.

```javascript
import { useSelector, useDispatch, shallowEqual } from "react-redux";

///

  const { number, diff } = useSelector(
    (state) => ({
      number: state.counter.number,
      diff: state.counter.diff,
    }),
    shallowEqual
  );
```

두가지 방법 중 편한 걸 사용하면 된다.
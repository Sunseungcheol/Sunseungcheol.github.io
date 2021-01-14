---
layout: post
categories: TypeScript
tags: [TypeScript, Generics]
---
Genrtics 는 타입스크립트에서 함수, 클래스, TypeAlias, interface 를 사용할 때 여러 종류의 타입에 호환을 맞춰야 할 때 사용하는 문법이다. 

`<T>` 보통 이런 식으로 사용한다.

먼저 함수에서의 사용법을 알아보자. 

```typescript
function merge(a: any, b: any) {
  return {
    ...a,
    ...b,
  };
}

const merged = merge({ foo: 1 }, { bar: 2 });
```

위와 같은 상황에서 제너릭스를 쓰지 않으면 a 와 b 에 정확히 뭘 넣어야 할 지 모른다. 이때 제네릭스를 사용해주면 된다.

```typescript
function merge<T1, T2>(a: T1, b: T2) {
  return {
    ...a,
    ...b,
  };
}

const merged = merge({ foo: 1 }, { bar: 2 });
```

이제 merged. 해주면 vscode 에서 merged 안의 객체들을 보여주면서 뭘 넣어야 할지 유추할 수 있다.

쉽게 생각하면 any 를 사용한 경우 리턴값도 any 가 되어 유추하기가 힘들지만 제너릭스 사용한다면 그 타입이 지켜지면서 타입을 유추해낼수 있다.

interface 와 TypeAlias 에서도 비슷하게 사용한다.

```typescript
interface Items<T> {
  list: T[];
}

//넣어준 타입대로 지정해줘야 한다.
const items: Items<string> = {
  list: ["a", "b"],
};

type Items2<T> = {
  list: T[];
};

const items2: Items2<number> = {
  list: [1, 2, 3],
};

//제너릭스는 여러개도 사용 가능하다.
interface Items<T, V> {
  list: T[];
  value: V;
}

//넣어준 타입대로 지정해줘야 한다.
const items: Items<string, number> = {
  list: ["a", "b"],
  value: 123,
};
```

클래스에서 사용하는 방법은 아래와 같이 사용하면 된다.

```typescript
class Queue<T> {
  //T 로 이루어진 비율, 기본값 빈배열
  list: T[] = [];

  get length() {
    return this.list.length;
  }

  //Queue 에 새로운 것 등록
  enqueue(item: T) {
    //list 타입은 T[], item 은 T
    this.list.push(item);
  }

  //Queue 에서 첫번째 항목 추출
  dequeue() {
    return this.list.shift();
  }
}

const queue = new Queue();
queue.enqueue(0);
queue.enqueue(1);
queue.enqueue(2);
queue.enqueue(3);
queue.enqueue(4);

while (queue.length > 0) {
  console.log(queue.dequeue());
  //0
  //1
  //2
  //3
  //4
}
```
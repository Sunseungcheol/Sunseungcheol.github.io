---
layout: post
categories: javaScript
tags: [javaScript]
---

JavaScript는 es6부터 화살표 함수를 지원한다.
당연하게도 익스플로러는 지원되지 않는다.

```javascript
const hello1 = () => {
    console.log('hello');
};

//매개변수가 하나인 경우, 괄호 생략 가능
const hello2 = name => {
    console.log('hello2', name);  
};

const hello3 = (name,age) => {
    console.log('hello3', name, age);
};

//리턴
const hello4 = name => {
    return `hello4 ${name}`;
};

//화살표 함수는 한줄로 코딩시 {} return 생략 가능
const hello5 = name => `hello5 ${name}`;
```

---
layout: post
categories: React.js
tags: [React.js, Error]
---

scss 를 설치하고 사용하는데 아래와 같은 오류가 발생했다.

`Node Sass version 5.0.0 is incompatible with ^4.0.0.`

찾아보니 sass-loader 에서 발생하는 오류로 node-sass @latest 와 sass-loader 의 버전이 달라서 생기는 오류라고 한다.
해결법은 간단하다.

##### node sass 제거

```
npm uninstall node-sass
```

##### 최신 버전 설치

```
npm install node-sass@4.14.1
```

확인해보니 제대로 동작한다.

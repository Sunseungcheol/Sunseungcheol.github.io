---
layout: post
categories: React.js
tags: [React.js, Error]
---

npm start 를 하니 아래와 같은 오류가 떳다. 현재 3000 포트를 사용하는 중이라는 소리다.

`Something is already running on port 3000.`

죽이기 위해서는 3000번 포트의 현재 PID 가 무엇인지 확인해야 한다.

터미널에서 
```
netstat -ano 
```
를 입력하면 아래와 같이 현재 로컬에서 사용중인 포트들이 나온다.

```
활성 연결

  프로토콜  로컬 주소              외부 주소              상태            PID
  TCP    0.0.0.0:3000           0.0.0.0:0              LISTENING       13500

```

tskill PID 명령어를 통해 죽인 후 다시 npm start 를 입력하니 제대로 동작한다.
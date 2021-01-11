---
layout: post
categories: React.js
tags: [React.js, 리덕스, Redux, json-server, CORS, Proxy]
---
#### json-server

개발공부를 할 때 연습용으로 API를 받아오기 위해 json-server 를 이용해 연습용 restAPI 서버를 가져올 수 있다.

먼저 기존에 가짜로 만들었던 데이터(posts)를 크롬 콘솔창에 입력한 후 `JSON.stringify(posts, null, 2)` 를 입력하면 json 형태로 변경해서 돌려준다.

이제 프로젝트 디렉토리에 data.json 파일을 만들고 위에서 json 형태로 변형된 값들을 넣어주자.

```json
//data.json
{
  "posts":[
      {
        "id": 1,
        "title": "리덕스 미들웨어",
        "body": "리덕스 어렵다"
      },
      {
        "id": 2,
        "title": "redux-thunk",
        "body": "thunkthunk"
      },
      {
        "id": 3,
        "title": "리덕스 사가도 공부해야지",
        "body": "사가사가"
      }
    ]
}
```

이제 이를 기반으로 서버를 열어볼건데 터미널에 

```
npx json-server ./data.json(데이터경로) --port 4000(port)
```

를 입력해주면된다.

restAPI 를 쉽게 테스트 하기 위해서 <a href='www.getpostman.com'>getpostman</a> 를 이용하면 편한데 getpostman 안에서 여러 메소드를 사용할 수 있다.

이제 기존에 리덕스를 공부할 때 만들었던 posts 를 만들어 놓은 서버에서 직접 받아와 보자.

```javascript
//api/posts.js
import axios from "axios";

//getPosts 호출 시 Promise 객체가 만들어지고 500ms 후에 posts 리턴
export const getPosts = async () => {
  const response = await axios.get("http://localhost:4000/posts/");
  return response.data;
};

export const getPostById = async (id) => {
  const response = await axios.get(`http://localhost:4000/posts/${id}`);
  return response.data;
};
```

Network 에서 확인해보니 제대로 불러오고 있다. 실수로 `await` 를 빠드리고 했었는데 `await` 빼먹으니 서버에서 받아는 오는데 reponse.data 가 바로 리턴되서 아무것도 보이지 않았다.


#### CORS 와 proxy

기본적으로 API 를 요청할 때 브라우저의 현재 도메인과 호출하는 API 의 도메인이 다르다면 브라우저에서 해당 API 에 대한 결과물을 조회할 수 없다.

위에서 json-server 를 공부할 때 원래대로라면 API 도메인과 요청하는 브라우저의 도메인이 다르기 때문에 조회할 수 없어야 된다.

```
http://localhost:3000/
http://localhost:4000/posts
```

하지만 가능했던 건 json 에서 어디서 오는 요청이든 다 허용하는 것으로 이미 설정이 되있어서 가능했던 것이다.


만약 실제 서비스를 개발하게 된다면 원칙적으로는 백엔드 서버 쪽에 해당 도메인을 허용하도록 해야 하지만 웹팩 개발서버에서 제공하는 Proxy 를 사용하면 그럴 필요가 없어진다.

프록시를 사용하게 되면, API 를 요청할 때 백엔드 서버에 요청하는 것이 아닌 개발서버의 주소로 요청을 하게되고 그러면 웹팩 개발서버에서 요청을 받아 백엔드로 전달 후 백엔드에서 응답한 내용을 다시 브라우저로 전환한다.

```
브라우저 요청 -> 개발 서버 -> 프록시(백엔드에 요청 내용 전달) -> 백엔드 응답 -> 프록시(응답 내용 받음) -> 브라우저로 전달
```

이제야 예전에 하던 프록시 우회가 어떤 개념이었는지 이해가 얼추 된다.

원래 프록시 설정은 웹팩 설정을 통해 하지만, 리액트 프로젝트에서는 packge.json 에서 쉽게 설정할 수 있다.

```javascript
//package.json 에 추가
  "proxy":"http://localhost:4000"
```

이후 기존에 API 요청을 하던 주소를 `http://localhost:4000/posts` 에서 `/posts` 로 변경해준다.

```javascript
//api/posts.js
export const getPosts = async () => {
  const response = await axios.get("/posts");
  return response.data;
};

export const getPostById = async (id) => {
  const response = await axios.get(`/posts/${id}`);
  return response.data;
};
```

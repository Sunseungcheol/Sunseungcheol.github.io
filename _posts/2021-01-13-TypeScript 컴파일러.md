---
layout: post
categories: TypeScript
tags: [TypeScript, 컴파일러]
---
먼저 타입스크립트를 사용하기 위해 전역으로 설치를 해줘야 한다.

```
$ npm install -g typescript
```

그 후 javascript 파일로 다시 컴파일을 시켜줘야 하는데 방법은 간단하다. 예를 들어 hello.ts 라는 파일이 있다 가정했을 때 

```
$ tsc hello.ts
```

라고 터미널에 입력해주면 컴파일된 hello.js 파일이 생성되는 것을 볼 수 있다. 

타입스크립트는 기본적으로 자바스크립트 옛 버전(ex ES5) 도 모두 지원하기 때문에 타입스크립트 파일에서 let, const 등을 사용한 변수도 모두 var 로 변형된다.

이를 해결하기 위해 --target 이라는 명령어를 사용할 수 있다.

```
$ tsc hello.ts --target es6
```

타입스크립트 컴파일러 또한 설정을 줌으로써 매 컴파일 마다 굳이 옵션을 줄 필요가 없도록 할 수 있다. 
먼저 루트디렉토리에 tsconfig.json 파일을 생성해준다.

```json
{
    "include": [
        //src 하위 모든 파일 안의 모든 ts파일
        "src/**/*.ts"
    ],
    "exclude": [
        //노드 모듈 제외
        "node_modules"
    ],
    //컴파일 옵션
    "compilerOptions": {
        //옛 커먼 js 형식, node 에서는 import, export 기능이 지원되지 않음
        "module": "commonjs",
        //루트디렉토리
        "rootDir": "src",
        //타입스크립트 컴파일된 파일들이 만들어지는 폴더
        "outDir":"dist",
        //타켓 버전
        "target":"ES5"
    }
}
```

이제 src 폴더안에 두가지 파일을 만들어 확인해보자

```typescript
//src/util.ts
export default function add(a,b){
    return a+b
}
export function minus(a,b){
    return a-b
}
//src/hello.ts
import add from "./util";

const value = add(1, 3);
```

위와 같이 util.ts 에 add 라는 함수를 만들어 heelo.ts 에서 import 해서 사용했다. 현재 버전은 es5로 설정해 놓은 상태인데 컴파일해보자.

이제 별도의 명령 없이

```
$ tsc
```

만 입력해도 된다.

입력 후 확인해 보면 dist 폴더가 자동생성되고 그 안에 js  파일들이 있는 것을 확인 할 수 있다.

```javascript
//dist/util.js
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.minus = void 0;
function add(a, b) {
    return a + b;
}
exports.default = add;
function minus(a, b) {
    return a - b;
}
exports.minus = minus;

//dist/hello.js
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
var util_1 = require("./util");
var value = util_1.default(1, 3);
```

위와 같이 자동으로 변형되 컴파일 된 것을 볼 수 있다.
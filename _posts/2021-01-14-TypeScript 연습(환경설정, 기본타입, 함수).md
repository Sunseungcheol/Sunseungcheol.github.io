---
layout: post
categories: TypeScript
tags: [TypeScript]
---
타입스크립트를 연습해보자. 

````
mkdir ts-practice
cd ts-practice
````

디렉토리를 만든 후 

```
yarn init -y
```

을 하면 packge.json 파일이 생성된다.

이제 몇가지 패키지를 설치해줘야 하는데

```
yarn add typescript ts-node
```

typescript 는 프로젝트에서 타입스크립트를 사용하기 위해  필수적으로 설치해줘야 하는 모듈이고 ts-node 는 타입스크립트를 콘솔에서 바로 실행할 수 있도록 도와주는 모듈이다.

타입스크립트 파일을 컴파일하지않고 터미널에서 바로 읽고 싶을 때는 아래와 같이 입력해주면 된다.

```
yarn run ts-node ./파일명
```

설치가 완료됐으면

```
yarn run tsc --init
```

을 하면 프로젝트에 타입스크립트 설정 파일이 만들어진다. (tsconfig.json)

이제 타입스크립트 파일을 만들어 하나씩 연습해보자.


#### 기본 타입

타입스크립트의 기본 타입은 자바스크립트와 같은데 타입스크립트는 다른 대부분의 언어들과 마찬가지로 타입을 지정해줘야 한다는 점이 다르다.

타입을 지정할 때는 `:` 을 이용해 지정하는데 아래와 같다.

```typescript
let count :number = 0;
count = 2;
count = "dd";//오류 타입 불일치
```

vscode 상에서 보면 3번째 줄은 오류가 발생한다. 꼭 지정하지 않아도 처음에 0을 넣었기 때문에 카운트 값은 자동으로 number 타입으로 지정되 3번째 구문이 오류가 발생하는 것은 동일하다. 

```typescript
let count = 0;
count = 2;
count = "dd";//오류 타입 불일치
```

그래도 항상 넣어주는 게 알아보기가 쉽다.

```typescript
//src/practice.ts
let count: number = 0;
count = 2;
count = "dd";

const message: string = "hello world";
const done: boolean = false;

//배열 선언
const numbers: number[] = [1, 2, 3];
const messages: string[] = ["hello", "world"];

messages.push(2); //오류 타입 불일치

//undefind 일수도 있고 string 일 수도 있을때
let mightBeUndefinde: string | undefined = undefined;
//null 일 수도 있고 number 일 수도 있을 때
let nullableNumber: number | null = null;

//정해진 값 중에서만 들어와야 될 때
//ex) red, orange, yellow 중 하나만 들어와야 할때
let color: "red" | "orange" | "yellow" = "red";
color = `yellow`;
color = "balck"; //오류 타입 불일치
```

위와 같이 | 연산자를 통해 여러 개의 타입을 줄 수 도 있고, color 와 같이 받을 수 있는 특정한 값만을 지정해줄 수 도 있다.

이제 한번 컴파일 해보자.

```
yarn run tsc
```

```t
src/practice.ts:3:1 - error TS2322: Type 'string' is not assignable to type 'number'.

3 count = "dd";
  ~~~~~

src/practice.ts:12:15 - error TS2345: Argument of type 'number' is not assignable to parameter of type 'string'.

12 messages.push(2); //오류 타입 불일치
                 ~

src/practice.ts:23:1 - error TS2322: Type '"balck"' is not assignable to type '"red" | "orange" | "yellow"'.

23 color = "balck"; //오류 타입 불일치
   ~~~~~


Found 3 errors.

error Command failed with exit code 2.
```

위와 같이 아까 타입을 제대로 안넣은 값들이 오류로 발생된다. 이때 주의할 점은 터미널상에 오류는 발생했지만 컴파일은 된다는 것이다.


#### 함수

타입스크립트에서는 함수 선언시에도 타입을 지정해주지 않으면 오류가 발생한다.

```typescript
//x 는 number, y = number, function sum 의 return 값은 number
function sum(x: number, y: number): number {
  return x + y;
}

function sumArray(numbers: number[]): number {
  return numbers.reduce((acc, current) => acc + current, 0);
}

const total = sumArray([1, 2, 3, 4, 5, 6]);
console.log(total);//21
```

만약 함수에서 아무것도 반환하지 않는다면 자동으로 타입이 void로 설정된다.
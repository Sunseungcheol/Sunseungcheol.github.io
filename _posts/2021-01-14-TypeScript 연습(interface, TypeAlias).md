---
layout: post
categories: TypeScript
tags: [TypeScript, interface, TypeAlias]
---
#### interface

interface 는 간단히 말하면 class 또는 객체에 대한 타입을 지정할 때 사용된다.

```typescript
interface Shape {
  getArea(): number;
}

class Circle implements Shape {
  //Shape interface 안에 있는 것들을 충족시키지 않으면 오류가 나옴
  getArea() {
    return 1;
  }
}
```

원넓이와 삼각형의 넓이를 구하는 클래스를 만들어보자.

```typescript
interface Shape {
  getArea(): number;
}

//Shape interface 를 implements(클래스가 interface 에 있는 조건을 충족해야 한다.) 함
class Circle implements Shape {
  radius: number;
  //생성자
  constructor(radius: number) {
    this.radius = radius;
  }
  //Shape interface 안에 있는 것들을 충족시키지 않으면 오류가 나옴
  getArea() {
    return this.radius * this.radius * Math.PI;
  }
}

class Ractangle implements Shape {
  width: number;
  height: number;

  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}
```

이제 만든 두 클래스를 사용해보자. 여기서 new 연산자를 사용해 새로운 인스턴스를 제작하는데 이때 타입은 클래스 타입으로 된다. 

```typescript
const circle = new Circle(5); //type = Circle 타입
const ractangle = new Ractangle(10, 5); //type = Ractangle타입
```

여기서 `Circle` 와 `Ractangle` 은 둘다 `Shape` 를 implements 하고 있기 때문에 아래와 같은 작업도 가능하다.

```typescript
const shapes: Shape[] = [circle, ractangle];
//shapes 안에 있는 모든 도형들 넓이 추출하기
shapes.forEach((shape) => {
  console.log(shape.getArea());
});
```


위의 클래스들의 생성자를 보면 this.witdh = width 이런식으로 설정이 되어있는데 이는 private, public 등을 선언해주면 훨씬 짧게 사용할 수 있다.

```typescript
class Circle implements Shape {
  //생성자
  constructor(public radius: number) {}
  //Shape interface 안에 있는 것들을 충족시키지 않으면 오류가 나옴
  getArea() {
    return this.radius * this.radius * Math.PI;
  }
}

class Ractangle implements Shape {
  constructor(private width: number, private height: number) {}

  getArea() {
    return this.width * this.height;
  }
}
```

둘의 차이는 외부에서 접근을 할 수 있느야 없느냐의 차이인데 컴파일하게되면 js 파일에서는 public 이든 private 이든 상관이 없다.


#### interface 로 객체타입 지정하기

interface 로 객체타입을 지정해 줄 수도 있다.

```typescript
interface Person {
  name: string;
  //? 가 들어가게 되면 있을수도있고, 없을 수도 있음을 의미
  age?: number;
}

interface Developer {
  name: string;
  age?: number;
  skills: string[];
}

const person: Person = {
  name: "사람",
  age: 20,
};

const front: Developer = {
  name: "개발자",
  age: 29,
  skills: ["js", "React", "ts"],
};
```

만약 객체를 생성할 때 interface 에 설정해주지 않은 값을 넣거나, 설정한 값을 넣지 않을 경우 오류가 발생하게 된다.

위의 Person 과 Developer 을 보면 name 과 age 가 중복되는 것을 볼 수 있다. 이 문제는 상속을 통해 해결할 수 있다.

```typescript
interface Person {
  name: string;
  //? 가 들어가게 되면 있을수도있고, 없을 수도 있음을 의미
  age?: number;
}
//Person 상속
interface Developer extends Person {
  skills: string[];
}
```


#### TypeAlias

TypeAlias 를 사용해 interface 와 비슷한 작업을 해줄 수 있다.

```typescript
//interface 사용
interface Person {
  name: string;
  //? 가 들어가게 되면 있을수도있고, 없을 수도 있음을 의미
  age?: number;
}
//Person 상속
interface Developer extends Person {
  skills: string[];
}

//TypeAlias 사용
type Person = {
  name: string;
  //? 가 들어가게 되면 있을수도있고, 없을 수도 있음을 의미
  age?: number;
};
//Person 상속
type Developer = Person & {
  skills: string[];
};
```

TypeAlias 사용하면 interface 로 못하는 작업들을 몇 가지 할 수 있다. 하나 예롤 들자면 아래와 같은 작업을 할 수 있다.

```typescript
type People = Person[];
const people: People = [person, front];

type Color = "red" | "blue" | "green";
const color: Color = "blue";
```


#### 정리

객체를 사용할 때 왠만한 작업은 TypeAlias 를 사용해도 문제가 없다. interface 와 TypeAlias 중 더 편한 걸 사용하면 된다. 

다만 몇가지 특이 케이스가 있는데

+ 만약 라이브러리를 위한 타입 설정 시 interface 사용 권장
+ 어떤 것을 사용하든지 일관성 있게 사용


타입스크립트를 공부하다가 느낀건데 .NET 을 사용할 때 공부했던 c# 이랑 매우 비슷한 것 같다. c# 과 자바스크립트가 합쳐진 느낌이다.
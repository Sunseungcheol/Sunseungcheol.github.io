---
layout: post
categories: React.js
tags: [JavaScript, this]
---

자바스크립트 this 바인딩을 다시 보려고 한다.

이전에도 공부했지만 자바스크립트에서 함수를 호출할 때는 매게변수로 받는 인자값과 arguments 객체, this 인자가 함수 내부에 암묵적으로 전달된다.

this 인자는 함수 호출 방식에 따라 다른 객체를 참조(this 바인딩) 한다.


### 객체 메서드 호출 시 this 바인딩

객체의 프로퍼티가 함수인 경우, 이 함수를 메서드라 부르는데 메서드 내부에서 this 를 사용 시 `해당 메서드를 호출한 객체` 로 바인딩된다.
아래 예제를 살펴보자.

```js
let myObj = {
  name: "foo",
  sayName: function () {
    console.log(this.name);
  },
};

let newObj = { name: "Sun" };

newObj.sayName = myObj.sayName;

myObj.sayName();//foo
newObj.sayName();//Sun
```

위에서 말했듯이 this 는 자신을 호출한 객체에 바인딩된다. 
즉, myObj.sayName 의 this 는 myObj 를, newObj.sayName 의 this 는 newObj 를 가리켜 foo 와 Sun 이 출력된 것이다.


### 함수 호출 시 this 바인딩

자바스크립트 함수 호출 시 전역 객체, 브라우저의 경우 window 객체에 this 가 바인딩 된다.
자바스크립트의 모든 전역 변수는 전역 객체의 프로퍼티들인데 예를 들어 전역 변수로 foo 를 설정 후 window.foo 를 호출하면 같은 결과가 나온다.

```js
let foo = "foo";

console.log(foo); //foo
console.log(window.foo); //foo

let sayFoo = function () {
  console.log(this.foo);//함수 호출 시 this 는 전역 객체에 바인딩된다.
};

sayFoo(); //foo
```

위의 특성은 내부함수를 이용했을 때도 동일하게 동작한다.

```js
let value = 100;

let myObj = {
  value: 100,
  func1: function () {
    this.value += 1;//this.value = myObj 의 value

    func2 = function () {
      this.value += 1;//this.value = 전역으로 설정된 value 변수

      func3 = function () {
        this.value += 1;//this.value = 전역으로 설정된 value 변수
      };

      func3();
    };
    
    func2();
  },
};

myObj.func1();
```

위의 예제를 보면 func1 은 메소드이기 때문에 this 가 myObj 를 가리킨다. 하지만 그 안의 내부함수 func2, func3 의 this 는 window 객체를 가리킨다.
내부 함수도 결국 함수로 인식해 호출 시 함수호출로 취급되기 때문이다.

만약 위의 예제에서 내부함수에서도 myObj 의 value 값을 사용하고 싶다면 다른 변수에 값을 담는 방법이 있다. 보통 this 값을 저장하는 변수는 that 이라 짓는다.

```js
let myObj = {
  value: 100,
  func1: function () {
    let that = this;
    this.value += 1; //this.value = myObj 의 value

    func2 = function () {
      that.value += 1;

      func3 = function () {
        that.value += 1;
      };

      func3();
    };

    func2();
  },
};

myObj.func1();
```

이외에 call, apply 등을 통해 this 바인딩을 변경할 수 있다.


### 생성자 함수 호출 시 this 바인딩

위의 객체리터럴 방식 외에 생성자 함수를 이용해 객체를 생성할 경우 위의 경우와는 조금 다른 방식으로 동작한다.

```js
let Person = function (name) {
  this.name = name;
};

let foo = new Person("foo");
console.log(foo.name); //foo
```

위를 보면 foo 의 name 은 매게변수로 받은 name 값으로 설정된 것을 볼 수 있다.

이 이유는 생성자 함수가 동작하는 방식을 보면 이해가 빠르다.

1. 빈객체 생성 후 this 바인딩

생성자 함수 코드가 실행되기 전 빈 객체가 생성되는데 이 객체는 생성자 함수가 새로 생성하는 객체이다. 이때 이 객체로 this 가 바인딩 되는 것이다.
생성자 함수가 생성한 객체는 자신을 생성한 생성자 함수의 prototype 프로퍼티가 가리키는 객체를 자신의 프로토타입 객체로 설정한다.

2. this 를 통한 프로퍼티 생성

함수 내부에서 this 를 이용해 동적으로 프로퍼티, 메서드를 생성할 수 있다.

3. 생성된 객체리턴

리턴문이 따로 없을 경우, this 로 바인딩 된 새로 생성한 객체가 리턴된다. 하지만 리턴값이 따로 있을 경우, 생성자 함수를 이용해도 this 가 아닌 return 값이 리턴된다.
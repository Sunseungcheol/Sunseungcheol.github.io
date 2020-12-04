---
layout: post
categories: javaScript
tags: [javaScript]
---

es6이전까지는 prototype을 이용해 다른 언어의 상속과 비슷하게 사용했었는데 es6에서 Class 개념이 들어오며 상속이 가능하게 되었다.

{% highlight ruby %}
class Parent {
  name = 'Sun';
  hello() {
      console.log('hello', this.name);
  }
}

class Child extends Parent {}

const p = new Parent();
const c = new Child();
console.log(p,c);//

c.hello();//hello Sun
c.name = 'Jung';
p.hello();//hello Sun
c.hello();//hello Jung
{% endhighlight %}

이와 같이 extends 를 사용해 상속을 받으며 부모의 멤버변수 및 함수에도 접근 가능하다.

오버라이드 -override 또한 가능한데 오버라이드는 상속 받은 자식클래스가 부모클래스에 있는 변수 및 함수를 덮어씌우거나 추가해 사용하는 것을 말한다. c#을 공부할 때 봤던 오버라이드와 같다.

{% highlight ruby %}
class Parent {
  name = 'Sun';
  hello() {
      console.log('hello', this.name);
  }
}
class Child extends Parent {
  age = 28;
  hello() {
      console.log('hello', this.name, this.age);
  }
}

const p = new Parent();
const c = new Child();

console.log(p,c);//Parent { name: 'Sun' } Child { name: 'Sun', age: 28 }
p.hello();//hello Sun
c.hello();//hello Sun 28
{% endhighlight %}

이와 같이 상속받은 함수 및 변수를 바꾸는 것이 가능하다. 물론 자식 Class가 무언가를 변경해도 부모에게 영향은 가지 않는다.

자식 Class가 생성자를 변경할 때에는 super를 반드시 사용해줘야 하는데 이는 부모 Class의 생성자함수를 먼저 실행한 후 자신의 생성자를 실행하게 해준다.

{% highlight ruby %}
class Parent {
  name;

  constructor(name) {
      this.name = name;
  }
  hello() {
      console.log('hello', this.name);
  }
}

class Chlid extends Parent {
    age;

    constructor(name, age){
        super(name);// = Parent의 생성자
        this.age = age;
    }

    hello() {
        console.log('hello', this.name, this.age);
    }
}

const p = new Parent('Sun');
const c = new Chlid('Sun', 28);

console.log(p, c);//Parent { name: 'Sun' } Chlid { name: 'Sun', age: 28 }

{% endhighlight %}
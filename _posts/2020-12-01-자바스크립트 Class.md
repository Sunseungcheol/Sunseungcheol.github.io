---
layout: post
categories: javaScript
tags: [javaScript]
---

JavaScript가 es6에 들어오면서 class가 추가되었다. 이전에 c#을 공부할 때 맛보기로 봤었는데 다시 한번 개념을 정리해보려고 한다.
es6이전에는 prototype을 이용해 타언어의 class와 비슷한 방식으로 사용했었다.

class를 만드는 방식에는 2가지가 있는데 선언적방식과 class 표현식을 변수에 할당하는 방법이다.
{% highlight ruby %}
//선언적 방식
class A {};

//class 표현식을 변수에 할당
const B = class {};
{% endhighlight %}

class는 함수와는 다르게 호이스팅이 일어나지 않는다.

{% highlight ruby %}
new C();//C is not defined
class C{};
{% endhighlight %}

JavaScript의 Class도 다른 언어와 마찬가지로 생성자 -constructor 가 있다.
생성자는 클래스의 객체를 인서턴스화 시킬 때 자동으로 호출되므로 적절한 기본값 혹은 필요한 설정등이 필요한 경우 사용한다.

{% highlight ruby %}
class C {
    constructor(name, age) {
        console.log('constructor', name, age);
    }
}

console.log(new C('Sun', 28));//constructor Sun 28

console.log(new C());////constructor undefined undefined
{% endhighlight %}

JavaScript에서 멤버 변수를 받기 위해서는 위에서와 같이 생성자를 이용하는데 이때 기본값을 설정해줄 수가 있다.

{% highlight ruby %}
class C {
    name = 'no';
    age = 0;
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
}
{% endhighlight %}

이번에는 멤버함수를 받는 방법을 알아보자.

{% highlight ruby %}
class A {
    hello1(){
        console.log('hello1', this);
    }
    hello2 = () => {
        console.log('hello2', this);
    }
}
{% endhighlight %}

JavaScript에 Class가 추가되면서 get,set 개념 또한 같이 따라왔는데 이것 또한 사용방법이 다른 언어와 크게 다르지 않다.

{% highlight ruby %}
class A {
    _name = 'no name';

    get name() {
        return this._name + "@@";
    }

    set name(value) {
        this._name = value + '!!';
    }
}

const a = new A();
console.log(a);//A {_name: 'no name'}
a.name = 'Sun';
console.log(a);//A {_name: 'Sun'}
console.log(a.name);//Sun!!@@
console.log(a._name);//Sun!!
{% endhighlight %}

또한 get만 사용해 아래와 같이 ReadOnly 속성으로 만들 수 있다.

{% highlight ruby %}
class B {
    _name = 'no name';

    get name() {
        return this._name + "@@";
    }
}
{% endhighlight %}

정적메소드 -static도 추가되었는데 static은 클래스의 인스턴스 생성없이 호출이 가능하지만 인스턴스화 되면 호출이 불가능하다.

{% highlight ruby %}
class A {
    static age = 28;

    static hello() {
        console.log(A.age);
    }
}
A.hello();//28

//인스턴스화

const a = new A();
a.hello();//a.hello is not a function
{% endhighlight %}





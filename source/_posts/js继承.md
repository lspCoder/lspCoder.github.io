---
title: js继承
author: 苏木堇
tags:
- javascript知识点
- js继承
github: 'https://github.com/lspCoder/js-extend.git'
categories: []
date: 2018-01-14 15:33:00
---

说到继承，想必都不陌生。学过java的童鞋最清楚不过了，面向对象的一大特性就是继承。竟然js有也可以面向对象，那么js继承怎么实现呢？

#### 构造函数继承
继承从字面量上讲首先要有父子关系，这样才能叫继承，孩子继承父亲，然后孩子就有了父亲的特性和方法，<!--more-->我们很容易就能想到js的关键字call、apply，通过改变上下文的指向来调用不是自己的方法的方法。我们来看代码具体怎么实现：
	
```javascript
function Animal(kind) {
    this.kind = kind;
    this.eat = function (food) {
        console.log(this.name + 'eat ' + food);
    }
}

function Dog(name, color, kind) {
    Animal.apply(this, arguments);      //这一句必须写在构造函数最前面,因为写在后面会让
    this.name = name;
    this.color = color;
}

var dog1 = new Dog('大黄', '黑色', '狗');

console.log(dog1);
console.log(dog1.eat('骨头'));
console.log(dog1.constructor);           //Dog
console.log(dog1 instanceof Animal);        //false
console.log(dog1 instanceof Dog);         //true
console.log(dog1.eat === dog2.eat);    //false
```

当然用apply也可以，不过参数要对应Animal.call(this, name, color, kind);但是这一句必须在子类构造函数的开头写，不然父类的参数赋值会覆盖掉子类的值；另外这种方法的继承子类并不是父类的实例，也不完全符合继承的要求；并且最重要的一点就是性能问题，没实例化一次所有属性都实例一次，没有达到继承共享的特性，造成浪费。

#### 原型链继承
谈到js继承当然少不了原型链，原型链继承是js继承的主要手段。js里面类就是对象，每一个对象都有一个prototype属性，对象调用方法和属性首先会在自身上查找有没有对应的方法和属性，没有就会去查找prototype上有没有，在没有继续往上找，知道Object上都没有就会报错---undefined。那么我们继续看代码如何实现:

```javascript
function Animal() {
this.kind = '动物';
}

Animal.prototype = {
    eat: function (food) {
        console.log(this.name + 'eat ' + food);
    }
}

function Cat(name, color) {
    this.name = name;
    this.color = color;
}

Cat.prototype = new Animal();     //这样会把父类对象上的属性也实例一次，造成不必要的内存开销
Cat.prototype.constructor = Cat;           //这一句是为矫正constructor属性，不然会造成继承混乱

var cat1 = new Cat('大黄', '黄色');
var cat2 = new Cat('大花', '花色');

console.log(cat1.constructor);              //Cat
console.log(Animal.prototype.constructor == Cat.prototype.constructor);          //true
console.log(cat1.eat === cat2.eat);              //true
console.log(cat1 instanceof Cat);      //true
console.log(cat2 instanceof Cat);              //true
console.log(cat1 instanceof Animal);             //true
```

这样看起来似乎很棒，基本没什么毛病，子类即是父类的实例也是子类的实例，并且父类原型上的方法和子类实例的方法公用内存地址，节省开销，也可以复用，但是Cat.prototype = new Animal()这一句;会实例化父亲一次，这样必然就会把父亲的所有属性也实例一次，虽然子类不必要但是也会有父亲的的属性，显然也不是很好，并且还需要手动矫正constructor的指向；
那么我们试试不实例化父类实例直接复制不就可以解决了吗?我们继续看代码:

```javascript
Cat.prototype = Animal.prototype; 
```

只需要加一句就可以改变不需要实例化父类，看样子很不错，然而细细想，如果直接把父类的prototype赋值给子类，那么子类的prototype有修改那么父类必然也会修改，很显然这不是个好办法。

#### 拷贝继承
由上面直接赋值给子类，我们可以这样改进：
	
```javascript
function extend(Child, Parent) {　　　　
    var p = Parent.prototype;　　　　
    var c = Child.prototype;　　　　
    for (var i in p) {　　　　　　
        c[i] = p[i];　　　　　　
    }
    c.uber = p;　　
}
```
 这样既不需要手动矫正constructor的指向了，但是如果是复杂数据结构，这样的浅拷贝很显然会导致父子的数据会产生共享，子改变数据会影响父，反之也是；继续改进，通过递归实现深拷贝

```javascript
 function deepCopy(p, c) {　　　　
        var c = c || {};　　　　
        for (var i in p) {　　　　　　
            if (typeof p[i] === 'object') {　　　　　　　　
                c[i] = (p[i].constructor === Array) ? [] : {};　　　　　　　　
                deepCopy(p[i], c[i]);　　　　　　
            } else {　　　　　　　　　
                c[i] = p[i];　　　　　　
            }　　　　
        }　　　　
        return c;　　
    }
```


​    
像这样，显然没什么问题，但是循环递归，属性多了，效率也不会太高。我们继续研究。

#### 原型继承升级版
竟然原型继承会实例父亲并且会给子类附上父类的属性，那么我们可不可以用一个空函数来作为中介，先把父类的prototype存起来，在赋给子类；

```javascript
function extend (Child, Parent) {
    var F = function(){};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.prototype.constructor = Child;
    Child.superClass = Parent,prototype;
}
```

#### 项目地址

github: https://github.com/lspCoder/js-extend.git

#### 参考文献
- http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html
- http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance_continued.html
- https://www.cnblogs.com/humin/p/4556820.html
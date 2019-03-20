---
title: 'javascript中call,apply,bind总结'
catalog: true
date: 2017-12-16 17:16:29
subtitle:
header-img:
tags:
- javascript知识点
comments: true
---

#### javascript中的this
要想了解call,apply和bind函数，首先就必须要了解js中this的指向，因为这三个函数就是为了动态的改变this而生的。
在ECMAScript中，this是执行上下文中的一个属性，那么什么事上下文呢?通俗的说就是运行js的环境。<!-- more-->
#### 全局代码中的this
在浏览器中运行js那么this的指向就是window，如果在node命令行中运行js，那么this的指向就是Global.
#### 函数代码中的this
函数相当于一个把共同功能放在一起的一个小的代码块，那么在函数中执行的上下文就是当前函数，所以this也就是函数本身。

```javascript
var dog = {
    name: "doudou",
    say: function() {
        console.log(this.name);
    },
    isSelf: function() {
        console.log(this === dog);
	}
}
dog.isSelf();      //输出true
var dog2 = dog.isSelf;

dog2();      //输出false
```
我们来看一个例子，最后输出的结果第一个是true，第二个是false；为什么会是这种情况呢？第一个是true是因为isSelf是由dog调用的，那么this就是dog本身，但是dog.isSelf赋值给了dog2然后调用dog2，dog2()就相当于window.dog2()，那么第二个this就是window，简单的说就是谁调用这个函数this就是谁；那么如果我们要想让第二个输出是true那么我们就需要用到下面几个函数了。


#### call函数和apply函数
还是上面的例子，竟然dog2的this指向不是dog，那么我们可以用call和apply改变this的指向

```javascript
dog2.call(dog);            //输出为true
dog2.apply(dog);           //输出为true
```
那么这两个函数有什么区别呢？区别就在于第二个函数的参数，如果函数有参数，那么用call的话就需要按顺序传递进来，就像这样

```javascript
function foo(arg1, arg2...){

}

foo.call(scope, arg1, arg2...)
```
apply函数第二个参数就是arguments数组

	foo.apply(scope, [arg1, arg2...])
还是来一个例子，给dog加一个eat函数

```javascript
var dog = {
    name: "doudou",
    say: function() {
        console.log(this.name);
    },
    isSelf: function() {
        console.log(this === dog);

        // console.log(this);
    },
    eat: function (food1, food2) {
        console.log(this.name + ' eat: ' + food1 + ' and ' + food2);
    }
}
dog.eat('meat', 'water');      //输出doudou eat: meat and water

var cat = {
    name: 'mimi',
}

dog.eat.call(cat, 'fish', 'fish');          //输出mimi eat: fish and fish
dog.eat.apply(cat, ['fish', 'fish']);          //输出mimi eat: fish and fish
```

本来cat没有eat方法的，但是dog有，因此可以调用dog的eat方法改变this的指向为cat,就可以让cat也具有eat方法，因此这个两个方法可以用到继承里面，这个下次有时间在总结。另外通过这两个方法可以简单的实现数组的最大值的查找

    var  numbers = [5, 458 , 120 , -215 ]; 
    var maxInNumbers = Math.max.apply(Math, numbers),   //458
    maxInNumbers = Math.max.call(Math,5, 458 , 120 , -215); //458

#### bind函数
那么bind函数是干嘛用的呢，顾名思义绑定this用的；当我们如果要写一个点击事件的时候，点击事件里面的this就是dom本身，但是我们如果想改变this就可以用到bind了；

	var foo = {
	    bar : 1,
	    eventBind: function(){
	        $('.someClass').on('click',function(event) {
	            /* Act on the event */
	            console.log(this.bar);      //1
	        }.bind(this));
	    }
	}
当然我们也可以用一个变量把this存起来，var self= this;但是有的时候我可能并不能直接调用一个函数，通过其他的方式调用了一个函数，然后这个函数又用了this,因为这个函数的上面问改变了，那么在函数中用this可能就会报错，因此这就是bind引用的场景。

#### 总结
最后在总结一下：
- apply 、 call 、bind 三者都是用来改变函数的this对象的指向的；
- apply 、 call 、bind 三者第一个参数都是this要指向的对象，也就是想指定的上下文；
- apply 、 call 、bind 三者都可以利用后续参数传参；
- bind 是返回对应函数，便于稍后调用；apply 、call 则是立即调用 。
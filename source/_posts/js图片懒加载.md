---
title: js图片懒加载
catalog: true
tags:
- js插件封装
- 懒加载
comments: true
github: 'https://github.com/lspCoder/lazyload.git'
categories: []
author: lsp
date: 2018-01-11 21:34:00
subtitle:
header-img:
---

### 什么是懒加载

懒加载其实就是延迟加载，是一种对网页性能优化的方式，比如当访问一个页面的时候，优先显示可视区域的图片而不一次性加载所有图片，当需要显示的时候再发送图片请求，避免打开网页时加载过多资源。<!-- more-->

### 懒加载原理

img标签有一个属性是src，用来表示图像的URL，当这个属性的值不为空时，浏览器就会根据这个值发送请求。如果没有src属性，就不会发送请求。

那么我们可以利用这一特性先不给img设置src，把图片真正的URL放在另一个自定义属性data-src中，在需要的时候也就是图片进入可视区域的之前，将URL取出放到src中。

### dom可视区域和判断

#### 方法一

是很常见的一种做法，也是最好理解的一种
![image](https://i.loli.net/2018/09/16/5b9dff997d93a.png)
从图上可以很直观的看出来，2为客户端可以看到的大小，也就是可视区的大小，6为目标元素也就是img,要想知道图片是否在可视区，那么我们从图上可以看到只要获取到img距离文档顶部的高度在减去滚动条滚动的距离和可视区的高度作为比较就可以得出。

- 可视区的大小：window.innerHeight || document.documentElement.clientHeight

- img距离顶部文档的距离：img.offsetTop

- 滚动条滚动的距离：document.documentElement.scrollTop || document.body.scrollTop 

然后根据图片分析img在可视区的公式应该满足：

```javascript
img.offsetTop - document.body.scrollTop < window.innerHeight
```

#### 方法二
通过getBoundingClientRect()方法来获取元素的大小以及位置，MDN上是这样描述的：

    The Element.getBoundingClientRect() method returns the size of an element and its position relative to the viewport.

这个方法返回一个名为ClientRect的DOMRect对象，包含了top、right、botton、left、width、height这些值。
![image](https://i.loli.net/2018/09/16/5b9dffb94c34e.png)

那么top其实就是img到顶部文档的距离，所以我们的公式应该为：

```javascript
var bound = el.getBoundingClientRect();
var clientHeight = window.innerHeight;
bound.top < clientHeight
```

一般我们会在后面加一个预先加载下一张图片的距离，所以最后的公式为：

```javascript
bound.top < clientHeight + 100

/**
 * @description 检测元素是否在可视窗口内
 * @param {*} element 
 * @param {*} scope 
 */
var isInSight = function (element, scope) {
    var bound = element.getBoundingClientRect();               //The Element.getBoundingClientRect() method returns the size of an element and its position relative to the viewport.
    var clientHeight = window.innerHeight;      //文档可视高度

    // var offsetTop = element.offsetTop;
    // var scrollTop = document.body.scrollTop || document.documentElement.scrollTop;
    // return offsetTop - scrollTop < clientHeight;

    return bound.top <= clientHeight + scope.options.preload;              //加一百位预加载
}
```

### 图片加载

然后我们已经知道哪些图片在可视区，那么我们就获取在可视区的图片img,拿到我们之前在dom里面设置的data-src属性的图片真实地址;设置到img的src中;

```javascript
/**
     * @description 验证图片
     */
    checkImgs: function () {
        var imgs = document.querySelectorAll('img');
        var self = this;
        [].forEach.call(imgs, function (el) {
            if(el.isloaded) return;
            el.src = self.get('placeholder');
            setTimeout(function () {
                if (isInSight(el, self)) {
                    self.loadingImg(el);
                }
            }, self.get('delay'));

        })
    },

    /**
     * @description 加载图片
     */
    loadingImg: function (el) {
        var source;
        if (el.dataset) {      //兼容性判断
            source = el.dataset.src;
        } else {
            source = el.getAttribute(this.options.attr);
        }
        var self = this;

        var img = new Image();
        img.src = source;

        img.onload = function () {
            el.src = this.src;
            if (img.complete == true) {
                el.isloaded = true;
                self.options.callback && self.options.callback();  //加载完成后的回调函数
            }
        }

        img.onerror = function () {
            el.src = self.get('placeholder');
        }
    }
```

placeholder为图片加载前的loading图片，等图片onload完成，在设置原始图片到img;

### 事件优化之函数节流

函数节流字面上的意思就是节约流量，也就是节省内存开销，像滚动条事件和窗口缩放事件都会频繁的触发事件，那么久很可能造成页面崩溃和不必要的性能开销，那么常见的做法就是隔一定的事件触发一次指定函数，从而减少函数一直触发的弊病。那么让我们来看看代码吧

```javascript
 /**
 * @description 函数节流
 * @description 也可以使用时间戳
 * @param {*回调函数} fn 
 * @param {*延迟时间} delay 
 */
var throttle = function (fn, delay) {
    var timer = null;
    return function () {
        var context = this;
        clearTimeout(timer);
        timer = setTimeout(function () {
            fn.apply(context, arguments);
        }, delay);
    }
}
```

这是一个很简单也很常见的节流函数，运用setTimeout也延迟调用函数。竟然是隔一段时间触发一次，那么我们也可以使用Date时间间隔来实现函数节流

```javascript
/**
 * 函数节流方法
 * @param Function fn 延时调用函数
 * @param Number delay 延迟多长时间
 * @param Number atleast 至少多长时间触发一次
 * @return Function 延迟执行的方法
 */
var throttle = function (fn, delay, atleast) {
    var timer = null;
    var previous = null;

    return function () {
        var now = +new Date();

        if ( !previous ) previous = now;

        if ( now - previous > atleast ) {
            fn();
            // 重置上一次开始时间为本次结束时间
            previous = now;
        } else {
            clearTimeout(timer);
            timer = setTimeout(function() {
                fn();
            }, delay);
        }
    }
};
```

### 完整代码

请看这里github:https://github.com/lspCoder/lazyload.git
欢迎star和issure

### 参考资料

- http://blog.csdn.net/itzhongzi/article/details/77466779
- https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect
- http://www.alloyteam.com/2012/11/javascript-throttle/


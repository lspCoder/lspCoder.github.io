---
title: 记一次高分屏下canvas模糊问题
date: 2019-01-07 19:48:33
tags:
- canvas
- devicepixelratio
comments: true
---

#### 	前言

​	最近在做项目的时候发现，在公司电脑上没问题，在自己电脑上确有问题。做的是canvas的项目，在自己电脑上运行的时候，发现，会出现点击选不中的问题还有，canvas刷新会有残影问题。首先可以确定，这两个问题都是canvas元素边界有问题，但是从代码上来看是没问题的，因此我就猜测是否和屏幕有关，毕竟canvas某些问题确实和屏幕有关甚至和硬件显卡有关。

#### devicePixelRatio属性

果然找出一个属性不同： **devicePixelRatio**；<!-- more -->

然后仔细研究了下这个属性的含义，mdn上解释如下：

> 此属性返回当前显示设备的物理像素分辨率与CSS像素分辨率的比值。该值也可以被解释为像素大小的比例：即一个CSS像素的大小相对于一个物理像素的大小的比值。

公司的台式机的devicePixelRatio为1，而我自己的电脑为1.25；因为我的电脑是高清屏的，那么什么是高清屏呢?

高清显示屏原理如下：

1. 一种具备超高像素密度的液晶屏
2. 同样大小的屏幕上显示的像素点由1个变为多个

那么我们就能知道，高清屏上一个像素点变成devicePixelRatio个像素点，因此canvas画布也会收到影响，同样的图片，在高清屏上会变大，但是canvas实际尺寸没有变大，因为图片会缩放，导致模糊。

#### canvas宽高与css宽高

那么如何解决canvas高分屏问题呢？既然高分屏下canvas的像素点会变多，导致画布缩放，那么我们能不能通过某种方法把canvas缩放回去呢？答案是可以的。

我们首先认识一下canvas的像素，我们先绘制一段文字

```javascript
<canvas id="canvas1" width="300" height="150"></canvas>
....


ctx1.beginPath();
ctx1.font = '20px arial';
ctx1.fillText('Html5 canvas', 50, 50);
```

这样我们就得到如下：

![](https://i.loli.net/2019/01/07/5c335e311e318.png)

我们在创建一个画布，这次我们通过css设置画布的宽高:

```javascript
<canvas id="canvas2" style="width: 200px; height: 200px"></canvas>
.....



ctx2.beginPath();
ctx2.font = '20px arial';
ctx2.fillText('Html5 canvas', 50, 50);
```

这次我们得到如下效果：

![](https://i.loli.net/2019/01/07/5c335f6cae420.png)

我们可以很明显看到画布上的文字有明显的缩放，这是为什么呢?我们可以这么理解： **canvas是绘制图片的，我们使用canvas绘制完图片后，首先生成一张根据canvas的宽高的图片，然后通过dom树由css渲染出来，因此canvas的宽高是图片的实际宽高，css的宽高是实际渲染出来的尺寸**。**那么我们回过头来理解devicePixelRatio，这个属性返回的是设备的物理像素分辨率与CSS像素分辨率的比值，我们的canvas绘制出来后图片因为高清屏设备的影响，导致图片变大，然而我们在浏览器的渲染窗口并没有变大，因此图片会挤压缩放使得canvas画布会变得模糊**，尽然高分屏的像素点变多了，导致图片变大，那么我们可以通过设置canvas的宽高设置devicePixelRatio倍的画布大小，然后设置canvas缩放比例为devicePixelRatio倍，保持和canvas等比例放大，然后这样我们相当于在绘制图片的时候就缩放图片变大，然后通过css渲染又会缩放回去，这样canvas的大小就不会失真了。代码如下：

```javascript
function makeHighRes(canvas) {
    var ctx = canvas.getContext('2d');
    // Get the device pixel ratio, falling back to 1.               
    var dpr = window.devicePixelRatio || window.webkitDevicePixelRatio || window.mozDevicePixelRatio || 1;

    // Get the size of the canvas in CSS pixels.
    var oldWidth = canvas.width;
    var oldHeight = canvas.height;
    // Give the canvas pixel dimensions of their CSS
    // size * the device pixel ratio.
    canvas.width = Math.round(oldWidth * dpr);
    canvas.height = Math.round(oldHeight * dpr);
    canvas.style.width = oldWidth + 'px';
    canvas.style.height = oldHeight + 'px';
    // Scale all drawing operations by the dpr, so you
    // don't have to worry about the difference.
    ctx.scale(dpr, dpr);
    return ctx;
}
```

另外：网上有一些解决办法比较古老，使用了backingStoreRatio 这个属性，这个属性已经废弃！

此外我们也可以使用这个库 [hidpi-canvas-polyfill](https://github.com/jondavidjohn/hidpi-canvas-polyfill)，他也是使用的我们这个方法来改变canvas绘制视图的，不过库中还把所有canvas绘制api都缩放了相应的devicePixelRatio倍，考虑很完善。

#### 参考链接

- https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/HTML-canvas-guide/SettingUptheCanvas/SettingUptheCanvas.html
- https://developer.mozilla.org/zh-CN/docs/Web/API/Window/devicePixelRatio
- https://www.jianshu.com/p/2cd5143cf9aa
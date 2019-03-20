---
title: 由canvas实现btn效果有感
tags:
  - canvas
  - H5
  - js图像
  - javascript
categories: []
toc: false
date: 2018-11-13 20:26:05
---

#### 前言

​	最近项目上有个工业自动化的需求，做一个按钮，有按下弹起效果，你肯定会说这不是so easy吗?是的，没错，用css，分分钟写一个，不过我们是做的拓扑图，用的canvas，因此我就想css画图也是gpu绘制，css能做的效果，我大canvas理应也能实现，于是我试着用css的实现方式做了一个canvas版的按钮效果。

#### css3版的立体按钮效果

想想我们用css怎么做一个按钮，首先我们就能想到按钮需要有阴影，我们可以使用box-shadow，他的语法如下：

> /* x偏移量 | y偏移量 | 阴影颜色 */
> box-shadow: 60px -16px teal;
>
> /* x偏移量 | y偏移量 | 阴影模糊半径 | 阴影颜色 */
> box-shadow: 10px 5px 5px black;
>
> /* x偏移量 | y偏移量 | 阴影模糊半径 | 阴影扩散半径 | 阴影颜色 */
> box-shadow: 2px 2px 2px 1px rgba(0, 0, 0, 0.2);
>
> /* 插页(阴影向内) | x偏移量 | y偏移量 | 阴影颜色 */
> box-shadow: inset 5em 1em gold;
>
> /* 任意数量的阴影，以逗号分隔 */
> box-shadow: 3px 3px red, -1em 0 0.4em olive;
>
> /* 全局关键字 */
> box-shadow: inherit;
> box-shadow: initial;
> box-shadow: unset;

<!--more-->

首先我们定义一个btn类，先画出一个按钮的基础样式

```css
.btn {
    border-radius: 5px;
    padding: 15px 25px;
    font-size: 22px;
    text-decoration: none;
    margin: 20px;
    color: #fff;
    position: relative;
    display: inline-block;
}
```

然后我们定义一下按钮的背景色

```css
.blue {
    background-color: #55acee;
}
```

这是我们的按钮基础样式就完成了，如下所示：



[![F3oaF0.png](https://s1.ax1x.com/2018/12/08/F3oaF0.png)](https://imgchr.com/i/F3oaF0)

接下来我们在来为按钮添加阴影

```css
.btn:active {
    transform: translate(0px, 5px);
    -webkit-transform: translate(0px, 5px);
    box-shadow: 0px 1px 0px 0px;
}

.blue {
    background-color: #55acee;
    box-shadow: 0px 5px 0px 0px #3C93D5;
}
```

给y方向5px的偏移，就会有阴影的效果，有点立体感觉，然后按下按钮之后我们还要去掉阴影，然后按钮的位置要向下偏移响应的5px，这样效果如下所示:

[![F3oWY6.gif](https://s1.ax1x.com/2018/12/08/F3oWY6.gif)](https://imgchr.com/i/F3oWY6)

#### canvas版的按钮效果

css的按钮我们已经知道了其实也就是利用box-shadow绘制阴影来实现按钮的样式，那么canvas我们怎么绘制阴影呢？翻一翻万能的[mdn](https://developer.mozilla.org/zh-CN/)

> [`shadowOffsetX = float`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowOffsetX)
>
> `shadowOffsetX` 和 `shadowOffsetY `用来设定阴影在 X 和 Y 轴的延伸距离，它们是不受变换矩阵所影响的。负值表示阴影会往上或左延伸，正值则表示会往下或右延伸，它们默认都为 `0`。
>
> [`shadowOffsetY = float`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowOffsetY)
>
> shadowOffsetX 和 `shadowOffsetY `用来设定阴影在 X 和 Y 轴的延伸距离，它们是不受变换矩阵所影响的。负值表示阴影会往上或左延伸，正值则表示会往下或右延伸，它们默认都为 `0`。
>
> [`shadowBlur = float`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowBlur)
>
> shadowBlur 用于设定阴影的模糊程度，其数值并不跟像素数量挂钩，也不受变换矩阵的影响，默认为 `0`。
>
> [`shadowColor = color`](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/shadowColor)
>
> shadowColor 是标准的 CSS 颜色值，用于设定阴影颜色效果，默认是全透明的黑色。

第一步我们先绘制矩形，绘制矩形我们使用moveTo和lineTo，因为按钮一般是带有圆角的，所以我们封装一个绘制圆角矩形的

```javascript
createPath: function (x, y, width, height, radius) {
            this.ctx.moveTo(x, y + radius);
            this.ctx.lineTo(x, y + height - radius);
            this.ctx.quadraticCurveTo(x, y + height, x + radius, y + height);
            this.ctx.lineTo(x + width - radius, y + height);
            this.ctx.quadraticCurveTo(x + width, y + height, x + width, y + height - radius);
            this.ctx.lineTo(x + width, y + radius);
            this.ctx.quadraticCurveTo(x + width, y, x + width - radius, y);
            this.ctx.lineTo(x + radius, y);
            this.ctx.quadraticCurveTo(x, y, x, y + radius);
        },
```

圆角矩形绘制完成后我们在接着绘制阴影，

```javascript
setShadow: function (xoffset, yoffset) {
    var style = this.style;
    this.ctx.shadowOffsetX = xoffset || 0;
    this.ctx.shadowOffsetY = yoffset || 5;
    this.ctx.shadowBlur = 0;
    this.ctx.shadowColor = style.shadowColor;
},
```

阴影绘制完后我们还要绘制文本，毕竟是canvas，绘制文本不像css那么方便，我们需要计算文字宽度来定位，代码如下：

```javascript
 drawText: function () {
     var xoffset = this.ctx.measureText(this.text).width;
     var x = this.x,
         y = this.y;
     if (this.state === 'active') {
         y = y + 5;
     }
     this.ctx.save();
     this.ctx.beginPath();
     this.ctx.font = "30px Micosoft yahei";
     this.ctx.fillStyle = this.fontColor;
     this.ctx.textBaseline = 'middle';
     this.ctx.textAlign = 'center';
     this.ctx.fillText(this.text, x + (this.width - xoffset) / 2 + 10, y + (this.height - 22) / 2 + 5, this.width);
     this.ctx.closePath();
     this.ctx.restore();
 },

```

另附textAlign和textBaseLine的说明：

textAlign：

![](https://i.loli.net/2018/12/08/5c0b8fe927047.png)

textBaseLine：

![textBaseLine](https://i.loli.net/2018/12/08/5c0b90132f21e.png)

这样效果就基本大功告成了

![](https://i.loli.net/2018/12/08/5c0b925c935ee.gif)

canvas事件可以具体看我的代码和我以前的[博客](https://lspcoder.github.io/)

#### canvas发光效果

上面介绍了如何绘制canvas阴影立体按钮，接下来我们来实现试试更进一步的效果，多层阴影叠加效果，在css中我们的box-shadow可以叠加多层阴影效果，并且通过逗号分隔。那么我们canvas是不是也可以同样实现呢？我们知道canvas是基于状态的，我们如果要绘制多层阴影叠加，那么就需要保存每次绘制的阴影，层叠在一起，那么我们改下代码，每画一层阴影就需要保存一次canvas状态:

```javascript
drawText: function (shadowx, shadowy, blur, shadowColor) {
            var xoffset = this.ctx.measureText(this.text).width;
            var x = this.x,
                y = this.y;
            this.ctx.save();
            this.ctx.beginPath();
            this.setShadow(shadowx, shadowy, blur, shadowColor);
            this.ctx.font = "300px Micosoft yahei";
            this.ctx.fillStyle = this.fontColor;
            this.ctx.textBaseline = 'middle';
            this.ctx.textAlign = 'center';
            this.ctx.fillText(this.text, x + (this.width - xoffset) / 2 + 10, y + (this.height - 22) / 2 + 5, this.width);
            this.ctx.closePath();
            this.ctx.restore();
        },
        
        setShadow: function (shadowx, shadowy, blur, shadowColor) {
            this.ctx.shadowOffsetX = shadowx || 0;
            this.ctx.shadowOffsetY = shadowy || 0;
            this.ctx.shadowBlur = blur || 10;
            this.ctx.shadowColor = shadowColor;
        },
```

效果如下：

[![F8GQ4U.gif](https://s1.ax1x.com/2018/12/08/F8GQ4U.gif)](https://imgchr.com/i/F8GQ4U)

不太完美，有时间在继续优化，动画的发光还是不太柔和，与css比效果还是有点欠缺，以后再优化，先mark一下，不喜勿喷。

#### 总结

知识都是由互通性的，多学习多思考，不能学了这个忘那个，要能融会贯通（互相抄袭借鉴）![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1544886129&di=e1a7332aebb2c1b1b2de9f03f1790bb8&imgtype=jpg&er=1&src=http%3A%2F%2Fwww.17qq.com%2Fimg_biaoqing%2F66848895.jpeg)

#### 参考链接

https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-shadow

https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Applying_styles_and_colors#%E9%98%B4%E5%BD%B1_Shadows
---
title: vue结合HBuilder开发机票航班app
date: 2018-08-13 22:22:47
tags:
- vue
- HBuilder
- App
comments: true
github: 'https://github.com/lspCoder/yida.git'
---

现如今移动端已是越发蓬勃，相较于前几年web盛行的时代，手机用户才是王道，对于我们web前端的开发er们，我们也来撸一个app，什么不会安卓？不会swift？对不起，只要会javascript，html，css就行。

#### 前期准备工作

##### vue环境搭建

毕竟vue和react这么火，本项目以vue框架为主（本菜鸟以前是搞angular，所以对于vue很友好）。好了，废话不多说，首先安装node运行环境，下载地址：https://nodejs.org/en/。<!--more-->安装好node后，默认自带npm，然后打开命令行工具，问项目基于windows开发，mac用户请自行百度。打开cmd输入以下命令：

```powershell
npm i vue-cli -g
```

或

```powershell
npm install vue-cli -g
```



本项目基于vue自带脚手架搭建的web项目；然后命令行输入：

```powershell
vue init webpack-simple yida(项目名)
```

然后一路回车，默认配置就行，这时你的目录就会生成很多文件，这时vue默认脚手架为我们搭建的框架，最后在命令行输入：

```powershell
npm install    (###安装依赖)

###依赖装完后启动项目
npm run dev
```

然后你就可以在浏览器访问项目了，说明vue项目搭建成功。

#### 框架选型

JS框架已经选择好，那么就开始整合UI框架，本项目使用Muse-UI框架——一个基于vue2.0的Material Design风格的UI组件库。[详细介绍可以参考官网](https://muse-ui.org/#/zh-CN)。

这里只简单介绍引入框架，在main.js中添加如下代码：

```javascript
import MuseUI from 'muse-ui'     //引入museUI框架

import minireset from 'minireset.css'
import materialIcons from './assets/iconfont/material-icons.css'
// import myIconfont from './assets/myIconfont/iconfont.css'
import 'muse-ui/dist/muse-ui.css'
import 'muse-ui/dist/theme-light.css' // 使用 light 主题

Vue.use(MuseUI);
```

另外Muse-UI 推荐使用 Google 的 Material 字体图标库，有关所有可用图标的列表，请访问官方的[Material Icons](https://material.io/icons/)页面。这里我是直接下载了这个字体文件，放在了assets目录，当然你也可以像官网一样使用cdn引入字体图标。

当时做这个项目的时候还没出3.0，项目中使用的是2.10版本的MuseUI有问题的地方和有坈余的地方可以直接参考官网。

竟然是开发移动端App自然少不了手势插件，引入vue-touch插件：

```javascript
import VueTouch from 'vue-touch'      //手机touch事件插件

Vue.use(VueTouch, { name: 'v-touch' });
```



#### 组件开发

##### 路由组件

首先我们分析下app的页面结构，如下图：

![](https://i.loli.net/2018/09/03/5b8d2ce683dc7.png)

底部一个导航菜单，四个页面，很简单，复制粘贴四个路由就搞定，如下：

```javascript
		{
			path: 'home',
			name: 'Home',
			meta: { index: 5 },
			component: resolve => require(['@/pages/Home'], resolve),
		}, {
			path: 'favoriteFlight',
			name: 'FavoriteFlight',
			meta: { index: 6 },
			component: resolve => require(['@/pages/FavoriteFlight'], resolve),
		}, {
			path: 'order',
			name: 'OrderPage',
			meta: { index: 7 },
			component: resolve => require(['@/pages/OrderPage'], resolve),
		}, {
			path: 'person',
			name: 'Person',
			meta: { index: 8 },
			component: resolve => require(['@/pages/Person'], resolve),
		}
```

是不是很简单，万能的复制粘贴（手动滑稽）。但是城市选择页面如下所示：

![](https://i.loli.net/2018/09/03/5b8d2ce71e934.png)

不包含底部导航栏，占据整个页面，很显然只有一个路由渲染钩子是不行的，我们需要嵌套路由。首先我们定义一个app.vue，

```html
/*
 * @Author: lsp 
 * @Date: 2017-05-19 13:22:02 
 * @Last Modified by:   lsp 
 * @Last Modified time: 2017-05-22 13:22:02 
 * @describe 没有头部标题和尾部导航的页面
 */
<template>
  <div id="app">
    <transition :name="transitionName">
			<router-view></router-view>
		</transition>
  </div>
</template>
```

不包含顶部和头部的占据整个页面的一个路由，然后再在里面嵌套一个layout.vue，

```html
/* * @Author: lsp 
* @Date: 2017-05-19 13:22:02 
* @Last Modified by: lsp 
* @Last Modified time: 2017-05-22 13:22:02 
*/
<template>
	<div>
		<app-header v-show="showHeader"></app-header>
		<transition :name="transitionName">
			<router-view></router-view>
		</transition>
		<bottom-navigation class="bottom_nav"></bottom-navigation>
	</div>
</template>

<script>
	import appHeader from './components/Header'
	import bottomNavigation from './components/BottomNavigation'
```

然后我们重新定义我们的路由，如下：

```javascript
	{
		path: '/',
		name: 'Layout',
		component: resolve => require(['@/Layout'], resolve),
		meta: { index: 4 },//meta对象的index用来定义当前路由的层级,由小到大,由低到高
		redirect: '/guide1',
		children: [{
			path: 'home',
			name: 'Home',
			meta: { index: 5 },
			component: resolve => require(['@/pages/Home'], resolve),
		}, {
			path: 'favoriteFlight',
			name: 'FavoriteFlight',
			meta: { index: 6 },
			component: resolve => require(['@/pages/FavoriteFlight'], resolve),
		}, {
			path: 'order',
			name: 'OrderPage',
			meta: { index: 7 },
			component: resolve => require(['@/pages/OrderPage'], resolve),
		}, {
			path: 'person',
			name: 'Person',
			meta: { index: 8 },
			component: resolve => require(['@/pages/Person'], resolve),
		}]
	}, {
		path: '/cityChoose',
		name: 'CityChoose',
		meta: { index: 9 },
		component: resolve => require(['@/pages/CityChoose'], resolve),
	}, {
		path: '/flightList',
		name: 'FlightList',
		meta: { index: 10 },
		component: resolve => require(['@/pages/FlightList'], resolve),
	}, {
		path: '/flightDetail',
		name: 'FlightDetail',
		meta: { index: 11 },
		component: resolve => require(['@/pages/FlightDetail'], resolve),
	}, {
		path: '/login',
		name: 'Login',
		meta: { index: 12 },
		component: resolve => require(['@/pages/Login'], resolve),
	}, {
		path: '/register',
		name: 'Register',
		meta: { index: 13 },
		component: resolve => require(['@/pages/Register'], resolve),
	}, {
		path: '/personMessage',
		name: 'PersonMessage',
		meta: { index: 14 },
		component: resolve => require(['@/pages/PersonMessage'], resolve),
	}, {
		path: '/setting',
		name: 'Setting',
		meta: { index: 15 },
		component: resolve => require(['@/pages/Setting'], resolve),
	}
```

这样我们就定义好我们的路由了，主页和关注航班这种作为子路由，想城市选择这种作为外层的路由，这样就可以完美解决刚刚的问题啦。细心的同学可能看到我定义的路由里面有一个meta属性，那么它们是干什么的呢？各位看官继续往下看。

##### 页面切换动画

我们的app在页面切换的时候都是有一个慢慢转场的动画的，这样的用户体验才是最好的，不是直白的直接切换页面，点击右边的导航菜单会向左滑动切换页面，点左边的菜单会向右滑动切换页面，具体效果如下：

![](https://i.loli.net/2018/09/08/5b9365d09eb98.gif)

那么如何实现呢，还记得上一个路由组件里面我们定义了一个神奇的属性meta，官网的描述是这样的：

![](https://i.loli.net/2018/09/08/5b93679b03e4e.png)

因此我们用这个属性记录页面的层级和和标识，并且约定，meta字段如果递增说明页面是前进的状态，动画效果向左滑，反之向右：

```javascript
<transition :name="transitionName">
		<router-view></router-view>
</transition>

watch: {//使用watch 监听$router的变化
    $route(to, from) {
        //如果to索引大于from索引,判断为前进状态,反之则为后退状态
        if(to.meta.index > from.meta.index){
            //设置动画名称
            this.transitionName = 'slide-left';
        }else{
            this.transitionName = 'slide-right';
        }
    }
}

/* 页面切换效果 */
.slide-right-enter-active,
.slide-right-leave-active,
.slide-left-enter-active,
.slide-left-leave-active {
  will-change: transform;
  transition: all 500ms;
  position: absolute;
}
.slide-right-enter {
  opacity: 0;
  transform: translate3d(-100%, 0, 0);
}
.slide-right-leave-active {
  opacity: 0;
  transform: translate3d(100%, 0, 0);
}
.slide-left-enter {
  opacity: 0;
  transform: translate3d(100%, 0, 0);
}
.slide-left-leave-active {
  opacity: 0;
  transform: translate3d(-100%, 0, 0);
}
```

因此我们需要在路由中定义好每个页面的meta的大小，来赋予它左滑还是右滑，具体请看代码。

##### loading组件

数据加载还需要有loading加载动画，我们继续封装loding组件，这部分我是使用css动画来实现的，具体代码请看github。

##### 启动页组件

一般app启动都会有一个启动页，占据全屏，左上角有个跳过，本项目我也做了几个启动页，利用路由，首先导航至guide页面，然后添加手机滑动事件，滑动到下一个引导页，实现思路如下：

```javascript
<!-- vue-touch提供的滑动事件指令，可以参考vue-touch文档 -->
        <v-touch @swipeleft="onSwipeLeft" @swiperight="onSwipeRight">
            <transition :name="transitionName">
                <router-view ></router-view>
            </transition>
        </v-touch>
        
        
        onSwipeLeft() {
        	// router转场后会自动触发vueg的转场特效
            switch (this.$route.name) {
                case 'default':
                case 'guide1':
                    this.$router.push('guide2');
                    break
                case 'guide2':
                    this.$router.push('guide3');
                    break
            }
        },
        onSwipeRight() {
            this.$router.back();
        },
```

![](https://i.loli.net/2018/09/08/5b936aa326b9f.gif)



gif可能会有卡顿，大家可以直接clone代码，本地运行看看。



#### 服务端

我们这个demo项目，航班数据是抓取某东的机票数据，接口请求过于频繁可能会受限制，需要验证码，服务端我是使用的[bmob](https://www.bmob.cn/login),个人感觉不好用，不如使用[leancloud](https://leancloud.cn/)，因为bmob没有模块化的包，引入到vue很麻烦，当时也是踩了很多坑。短信服务也是使用的bmob的sdk的短信api，各位读者可以自行测试玩玩。本项目作为一个入门级的vue项目，各位大佬不喜勿喷，有问题大家一起学习交流，感谢支持。

#### 项目打包

首先运行vue-cli的打包命令，npm run build；然后用hbuilder打开build生成的dist文件夹，如下操作：

![](https://i.loli.net/2018/09/08/5b936e49320fe.png)

![](https://i.loli.net/2018/09/08/5b936e682b8cb.png)

按照提示一路打包，等待一会dist目录会多一个unpackage的文件夹，app文件就打包在release目录，可以直接在手机上安装体验。

以上就是本项目一点点介绍，有问题，欢迎大家一起来探讨。

另附github地址：https://github.com/lspCoder/yida.git


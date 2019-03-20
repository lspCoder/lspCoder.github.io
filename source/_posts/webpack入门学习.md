---
title: webpack入门学习
catalog: true
date: 2017-12-18 22:06:52
subtitle:
header-img:
tags:
- 前端打包工具
github: https://github.com/lspCoder/webpack-seed.git
comments: true 
---

#### 使用webpack前的准备
现在es2016越来越火了，前端模块化的构建思想已经深入人心。因此诞生webpack模块化打包工具，该工具不同于gulp，可以很好的以模块化的方式打包前端项目，Webpack的工作方式是：把你的项目当做一个整体，通过一个给定的主文件（如：index.js），Webpack将从这个文件开始找到你的项目的所有依赖文件，使用loaders处理它们，最后打包为一个（或多个）浏览器可识别的JavaScript文件。<!-- more-->

废话不多说，我们先开始webpack的准备工作，首先你需要安装node环境，然后切换到你的项目目录，执行以下命令

	//全局安装
	npm install -g webpack
	//安装到你的项目目录
	npm install --save-dev webpack
两种方式都可以。
网上很多博客都是webpack1或者2的，我最新也在学习webpack，所以我想分享下最新版的webpack的配置和打包。
项目目录结构如下

  webpack-seed
    │  index.template.html
    │  package-lock.json
    │  package.json
    │  webpack.config.js
    │
    ├─app
    │  │  index.js
    │  │
    │  ├─font
    │  ├─img
    │  ├─js
    │  │      module1.js
    │  │      module2.js
    │  │
    │  ├─less
    │  │      style.less
    │  │
    │  └─libs
    │          plugin.js
    │
    └─dist

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```
这是最基础的配置，一个入口一个出口。

#### webpack基础配置

```javascript
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');
module.exports = {
    devtool: 'inline-source-map', //开发调试用
    // entry: './src/index.js',
    entry: {
        app: './app/index.js',
        vendor: ['jquery', './app/libs/plugin']
    },
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    devServer: {
        contentBase: './dist',           //server指向地址
        port: 8080          //server端口默认为8080
    },
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader']
        }, {
            test: /\.(png|svg|jpg|gif)$/,
            use: ['file-loader']
        }, {
            test: /\.(woff|woff2|eot|ttf|otf)$/,
            use: ['file-loader']
        }]
    },
    plugins: [
        new CleanWebpackPlugin('dist/*.*', {          //清空dist目录
            root: __dirname,
            verbose: true, //Write logs to console.
            dry: false // If true, remove files on recompile. // Default: false
        }),
        new webpack.BannerPlugin('vonnie 版权所有，翻版必究'),
        new HtmlWebpackPlugin({
            title: 'Getting start',
            // template: './index.template.html'       //创建模板html,支持ejs语法
        }),
        // new webpack.optimize.UglifyJsPlugin(), //压缩js代码
        new webpack.optimize.CommonsChunkPlugin({
            name: 'vendor' // 指定公共 bundle 的名称。
        })
	]
}
```
这是我自己搭建的webpack脚手架配置，配置大多数和网上差不多。

#### webpack中的loaders配置
webpack3中的loaders的写法和1,2略有不同，常见的loader有css-loaders,style-loaders，还有预处理的less-loaders和sass-loaders。以及处理图片和字体资源的file-loaders。这些属于插件，使用前需要执行安装命令

	npm install css-loader style-loader file-loader --save-dev

#### webpack-dev-server以及热更新
使用webpack打包后需要在server中运行web项目，所以我们需要启动一个server，webpack-dev-server插件。
执行webpack-der-server --open就能打开一个server，server的基本配置如下：

	devServer: {
	    contentBase: './dist',           //server指向地址
	    port: 8080          //server端口默认为8080
	},

#### webpack常用插件
webpack有很多的插件可以使用，这里我只介绍我所使用的插件，更多的插件请参考[官方文档](https://doc.webpack-china.org)
html-webpack-plugin插件用来生成入口html页面，可以配置template指定模板路径，模板支持ejs的语法传递参数。
clean-webpack-plugin插件很简单就是清空木匾目录下的所有文件，每次打包使用新的打包文件。
另外着重介绍一下exports-loader这个插件，使用的时候我们这样引入

	require("exports-loader?plugin!./libs/plugin.js");       //这一句代表往非模块化的js文件最后加入一句export["plugin"]

为什么会用到这个？当然是为了兼容一些老的库和插件，有的插件没有用到标准的模块化规范，那么在引入的时候我们直接import或者require就会报错，找不到模块，那么这么写就相当于在引入的文件的末尾加上一句export["file"] = file;这样我们就能用require语句来引用这个模块了。或者你也可以在webpack.config.js中的loaders里面去引用。

#### webpack中碰到的错误和疑惑
常见的错误就是找不到模块和路径的问题，webpack查找路径是相对于入口文件来引入的，如果是npm的包，那么可以直接引入文件名例如jquery。
目前还有一个问题就是引入babel转译es6会报错，
ERROR in ./src/index.js
Module build failed: Error: Couldn't find preset "@babel/preset-env" relative to directory 
还需要多琢磨琢磨，后续会持续更新博客，本webpack的项目可以在clone到本地自己研究，github地址是：https://github.com/lspCoder/webpack-seed.git
欢迎star;

#### 具体参考

- [webpack脚手架搭建](https://www.cnblogs.com/Ruth92/p/5909773.html)
- [webpack傻瓜式指南](https://zhuanlan.zhihu.com/p/20367175)
- [webpack3官网](https://doc.webpack-china.org)
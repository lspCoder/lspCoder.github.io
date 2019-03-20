---
title: es6环境搭建入门
date: 2018-12-19 22:30:49
tags:
- javascript
- es6
- rollup
- babel
comments: true
---

### 前言

​	记得去年写过一次webpack-seed项目，今年我们换点别的口味。认识认识rollup.js。为什么要使用rollup.js？一个webpack不就够了吗？随着es2015模块的日益成熟，越来越多的库和框架使用了es201模块来编写js，而此时rollup诞生了，基于es2015的模块设计，完美的可以搭建我们的es6环境开发。那么我们开始配置我们的rollup吧。

### 安装rollup和命令行打包

全局安装rollup当然，你也可以局部安装，只不过局部安装运行命令需要带上路径。

```javascript
npm install rollup --global # or `npm i rollup -g` for short
```

主要的打包命令如下：

```
rollup index.js -f cjs
```

`-f` 选项（`--output.format` 的缩写）指定了所创建 bundle 的类型为commonjs（还有其他类型如：iife，amd，es）;<!-- more -->

### 使用配置文件打包

在项目目录新建一个rollup.config.js文件，如下：

```javascript
// rollup.config.js
export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};
```

最简单的配置如上所示，和webpack不一样的是入口配置叫input不是entry，老版本的rollup是叫entry,并且出书文件叫file不叫dest，0.48版本之后有些变量名改了。

> The `dest` and `format` options are now grouped together as a single `output: { file, format, ... }` object. `output` can also be an array of `{ file, format, ... }` objects, in which case it behaves similarly to the current `targets`. Other output options — `exports`, `paths` and so on — can be added to the `output` object (though they will fall back to their top-level namesakes, if unspecified).

如果继续使用老版打包，控制台会报如下

![](https://i.loli.net/2018/12/19/5c1a5fa05fce0.png)

如果想打包成多种模块类型，可以配置output为如下所示：

```javascript
output: [{
            file: 'dist/bundle.cjs.js',
            format: 'cjs'
        },
        {
            file: 'dist/bundle.umd.js',
            moduleName: 'res',
            format: 'umd'
        },
        {
            file: 'dist/bundle.es.js',
            format: 'es'
        },
        {
            file: 'dist/bundle.iife.js',
            format: 'iife'
        }
    ]
```

### 监听文件变化自动打包

安装`rollup-wacth`模块，再在rollup命令后面加上`-w`参数，就能让rollup监听文件变化

```javascript
npm i rollup-watch --save-dev
// or
yarn add rollup-watch --dev
```

然后添加打包命令：

```javascript
"dev": "rollup -c -w"
```

并且可配置排除监听的文件：

```javascript
watch: {
     exclude: ['./src/lib.js']
}
```

### 使用插件

#### 编译es6模块

虽然rollup支持es6模块，但是打包后便不会转换成es5的代码，因此我们还需要引用babel，这里有个坑，官网之说安装rollup-plugin-babel这个插件就可以了，但实际操作确还缺少一个依赖@babel/core，因此需要安装两个依赖：

```javascript
npm i rollup-plugin-babel babel-preset-latest --save-dev
// or
yarn add rollup-plugin-babel babel-preset-latest --dev

npm i rollup-plugin-babel @babel/core -D
```

在Babel实际编译代码之前，需要进行配置。 创建一个新文件`.babelrc`：

```json
{
  "presets": [
    ["latest", {
      "es2015": {
        "modules": false
      }
    }]
  ],
  "plugins": ["external-helpers"]
}
```

"modules": false是为了避免babel把我们的模块转化为CommonJS，而导致rollup的一些处理失败。

配置完这些后，但是又报其他问题：

![](https://i.loli.net/2018/12/19/5c1a647f352c9.png)

看提示是说babel新版本不支持helpersNamespaces，需要更新@babel/plugin-external-helpers这个模块，但是我更新了这个模块到最新的，好像也没用，可能是我没理解这个报错，求大佬指点，然后我就把babel降级到6.几的版本，这个rollup-plugin-babel也使用的3.几的版本。

```javascript
npm install --save-dev rollup-plugin-babel@3
```

然后执行打包命令，成功，没有报错，看看打包文件，也已经转换成es5语句了，目前先这样解决，还不清楚是什么原因，可能版本不对应。

安装babel转换的时候才发现，现在babel也已经更新了好多版本，插件也是眼花缭乱，乍一看这几个插件都差不多的名字：babel-preset-latest，babel-preset-es2015，babel-preset-env。到底怎样才是正确的打开方式呢？

当你安装这个版本的编译器时，babel会温馨的提示你如下信息：

![](https://i.loli.net/2018/12/20/5c1b86086e346.png)

他会告诉你推荐使用babel-preset-env，为什么呢？因为es2015这个插件从名字上就是帮你降级把es6代码编译为es5，es2016可以把代码编译为es6，然而有一些es6的属性，很多浏览器已经支持，例如generator，然后esXX插件还是会将这类es6的代码编译为es5，很没有必要，所以就有babel-preset-env这个插件，通过名字我们可以看出这个插件可以根据环境来配置，什么意思呢？就是说我们可以根据代码运行的环境来配置，哪些代码需要编译为es5哪些代码不需要编译。

另外，这两个插件的babel配置稍有不同，.babelrc配置如下：

```javascript
//babel-presets-es2015
{
  "presets": [
    ["es2015", {
      "modules": false
    }]
  ],
  "plugins": [
    "external-helpers"
  ]
}

//babel-preset-env
{
  "presets": [
    ["env", {
      "targets": {
        "chrome": 52,
        "browsers": ["last 2 versions", "safari 7"]
      },
      "modules": false,
      "debug": true
    }]
  ],
  "plugins": [
    "external-helpers"
  ]
}
```

如果只配置chrome浏览器，可以看到打包后的代码如下:

![](https://i.loli.net/2018/12/20/5c1b8d0aa6bc7.png)

class和extends等关键字并没有编译成es5因为chrome支持这些关键字。

那么babel-preset-latest呢？rollup.js官网讲解使用babel就是用的这个编译插件，它支持现有所有ECMAScript版本的新特性，包括处于stage 4里的特性（已经确定的规范，将被添加到下个年度的）。babel-preset-env 功能类似 babel-preset-latest，优点是它会根据目标环境选择不支持的新特性来转译。不过实验性的属性（babel-preset-latest不支持的）需要手动安装配置相应的plugins或者presets。配置文件如下:

```javascript
{
  "presets": [
    ["latest", {
      "es2015": {
        "modules": false
      }
    }]
  ],
  "plugins": ["external-helpers"]
}
```

#### 引入第三方模块

因为很多模块依赖是CommonJS的，因此我们还需要一个插件把CommonJS转化为es6模块，这样才能让rollup解析:

```javascript
npm i rollup-plugin-commonjs --save-dev
// or
yarn add rollup-plugin-commonjs --dev
```

并且需要注意插件引用的顺序也是对编译有关联的:

```javascript
plugins: [
        resolve({
            jsnext: true, //jsnext表示将原来的node模块转化成ES6模块，main和browser则决定了要将第三方模块内的哪些代码打包到最终文件中
            main: true,
            browser: true
        }),
        commonjs(),
        babel({
            exclude: 'node_modules/**'
        })
    ]
```

#### 为rollup配置外部模块和全局变量

有些第三方模块，我们可能便不想一起打包到我们的代码中，这是我们就需要配置external来指定外部模块一如jquery;

```javascript
external: ['jquery'],
```

全局变量：`Object` 形式的 `id: name` 键值对，用于`umd`/`iife`包。配置这个后，打包的代码会传入如下代码：

```javascript
/*
var MyBundle = (function ($) {
  // 代码到这里
}(window.jQuery));
*/


//配置如下
globals: {
    jquery: '$'
 }
```

#### 配合CDN来使用

不多说，直接上配置：

```javascript
paths: {
     d3: 'https://d3js.org/d3.v4.min'
}
```

#### css打包配置

css我们一般使用less和sass等简洁的写法，我们先安装一下插件:

```javascript
npm i -D rollup-plugin-postcss postcss-simple-vars postcss-nested postcss-cssnext cssnano
```

- `postcss-simple-vars` — 可以使用Sass风格的变量(e.g. `$myColor: #fff;`，`color: $myColor;`)而不是冗长的CSS语法(e.g. `:root {--myColor: #fff}`，`color: var(--myColor)`)。这样更简洁；我更喜欢较短的语法。

- `postcss-nested` — 允许使用嵌套规则。实际上我不用它写嵌套规则；我用它简化[BEM友好的选择器](http://getbem.com/naming/)的写法并且划分我的区块，元素和修饰到单个CSS块。

- `postcss-cssnext` — 这个插件集使得大多数现代CSS语法([通过最新的CSS标准](https://www.w3.org/Style/CSS/current-work))可用，编译后甚至可以在不支持新特性的旧浏览器中工作。

- `cssnano` — 压缩，减小输出CSS文件大小。相当于JavaScript中对应的UglifyJS。

  此处我们使用的sass，因此还需要依赖node-sass模块，

  ```javascript
  npm i -D node-sass
  ```

整个sass配置如下:

```javascript
postcss({
	plugins: [
        simplevars(),
        nested(),
        cssnext({
        	warnForDuplicates: false,
        }),
        cssnano(),
    ],
		extensions: ['.css'],
	}),
```

打包之后我们可以看到js中的代码如下:

![](https://i.loli.net/2018/12/20/5c1bacef7e61f.png)

#### 其他的配置

我们熟悉的sourcemap也是具备的，还有less和sass插件的支持，并且生产环境还需要压缩代码和replace一些变量，我们可以使用以下插件:

```javascript
npm i -D rollup-plugin-uglify rollup-plugin-replace rollup-plugin-postcss
```

但是实际操作中又碰到些奇奇怪怪的问题，uglify插件又报错误:

![](https://i.loli.net/2018/12/20/5c1ba8dd32869.png)

不知道什么原因，说uglify不是个函数，估计是版本问题，降级为3.0.0版本即可。

### 总结

总体来说，rollup适合打包js类库之类的，而webpack使用打包应用，万物接模块。后面我会使用rollup来写一个系列的js绘图类库，先搭建好环境做做准备。

有兴趣的同学记得关注一波我的公众号，感谢支持:

![](https://i.loli.net/2018/12/20/5c1bb10658d25.jpg)

### 参考链接：

- rollup官网地址:https://www.rollupjs.com/guide/zh
- https://segmentfault.com/a/1190000007543178
- https://babeljs.io/docs/en/babel-preset-latest/
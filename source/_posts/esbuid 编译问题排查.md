---
title: esbuid 编译问题排查
tags:
  - 问题排查
  - vite
date: 2022-04-19 13:37:00
---

## 1.背景

在使用 Esbuild 替代 Babel transform  代码时，代码可以成功编译，但是会出现一些运行时错误。出现的错误比较诡异，排查过程也遇到了一些困难，因此记录下这个过程。

首先简单介绍一下 esbuild  的几个显著特点，支持将

- 代码编译为 es6+;
- 使用 go 编写，很快;
- transform + minify;

## 2.问题定位

因为这是在代码打包之后才会遇到的错误（其实 dev 环境也可以遇到，但 dev 环境使用了 cache-loader ，三方因素较多不便于排查问题），所以我们为了方便排查问题，我们需要关闭 webpack 的代码压缩和混淆功能。

关闭方法如下：

```js
// 在 webpack.config.js  中关闭代码压缩，同时显示 moduleId
optimization: {
 	minimize: false,
    moduleIds: 'named',
    chunkIds: 'named',
}

// 关闭 webpack.prod.js HashedModuleIdsPlugin， 直接注释该 plugins
// new webpack.ids.HashedModuleIdsPlugin()
```



重新编译代码启动，可以看到下面的错误：

![esbuild-debug-issue1.png](https://s2.loli.net/2023/03/03/u7GQC2UeqN6fr5M.png)

我们知道，webpack 在处理模块化时，是将每个模块分解为一个个的闭包函数处理的。这里的 __spreadValues 是 esbuild 在每个闭包函数中注入的运行时代码，主要是一些es新特性的 polyfill，也可以类比为 babel-runtime 的一些 helper 方法。

![esbuild-debug-issue2.png](https://s2.loli.net/2023/03/03/E1tUMiVkPKOljfb.png)

经过查看，闭包里确实是有这个函数注入的，但是为什么访问时确实 undefined 呢？

通过查看错误的堆栈（下图），发现了一些端倪。 

![esbuild-debug-issue3.png](https://s2.loli.net/2023/03/03/RjiAFYIXHJtoGfw.png)

在入口文件 main.js 中，当前文件的所有引入，webpack 都会加载对应的包，__webpack__require__ 会查看当前依赖的模块是否在缓存中出现，同时执行模块的内部方法。

查看下图中的文件依赖关系，很明显的发现，这些文件中存在着 循环引用，并且层级很深。因为在 commonjs  规范中，如果模块间存在循环引用，就只输出已经执行的部分，还未执行的部分不会输出. 

怎么理解呢， 比如 A 依赖 B ，B 又依赖 A， 那么真实的执行过程其实是，在 A 中检测到 B 的依赖，会先去执行 B ，然后这个时候 B 文件如果依赖了 A，会直接从 requiire.cache 中拿 A 已经执行的部分，这里 A 中暴露出去的未执行的模块变量就很可能是 undefined.  这里感兴趣的可以看下：https://www.ruanyifeng.com/blog/2015/11/circular-dependency.html


**于是猜测：在处理模块依赖时，因为存在循环引用，所以某些模块并没有真正的被加载，所以导致运行时错误**


那么，怎么验证这个问题呢？通过对比 esbuild 和 babel 的 helper 方法的位置发现， esbuild 的 helper 方法是放在依赖文件的后面的，那么我只需要把 helper 文件放到依赖文件之前是不是就可以了。

![esbuild-debug-issue4.png](https://s2.loli.net/2023/03/03/S8xNObWavj9qtRX.png)

![esbuild-debug-issue5.png](https://s2.loli.net/2023/03/03/fVrGhEw56Uevpdu.png)

果不其然，错误消失了，但同时也出现了下面的报错，出现错误的原因同样是其他文件的循环依赖。

我尝试在一个最简单的循环依赖 demo 中用 esbuild 去编译，在 node 的运行时会报 undefined，在浏览器的运行时同样有此报错。所以，问题基本可以定位，确实是因为系统内部过深的循环依赖。

![esbuild-debug-issue6.png](https://s2.loli.net/2023/03/03/M1ayLqi8ZSf9TsK.png)

## 3.如何解决

那么问题的原因知道了，该怎么解决呢？

### 3.1 解决完项目内部所有的循环依赖

想法很美好，实现很困难。通过使用 circular-dependency-plugin 可以检测出目前项目中的循环依赖，目前项目中有近 400 处的循环依赖。

![esbuild-debug-issue7.png](https://s2.loli.net/2023/03/03/ypSMa38PYdcsKD7.png)

所以改动是非常困难的。

### 3.2 为什么使用 babel-loader 为什么不会有问题？

fms 使用的 babel 预设是 [babel-preset-vue-app](https://www.npmjs.com/package/babel-preset-vue-app)， 通过其[源码](https://github.com/vuejs/babel-preset-vue-app/blob/master/src/index.js)看出目前的编译预设是 ie9， 也就是说目前fms 编译后的代码是 es5 规范的。但是 esbuild  不支持将资源编译到 es6 以下。

举个简单的例子，对于下面的代码，下图分别是源代码在 esbuild  与 babel 编译后的代码对比

```js
export const foo = () => { console.log('foo') }
```

通过 esbuild 编译
```js
// ...some runtime code
module.exports = __toCommonJs(stdin_exports);
const foo = () => {
  console.log("foo");
}
```

通过 babel 编译
```js
var foo = function() {
  console.log("foo");
}
```
      

因此问题显而易见，在 es5 的版本中，存在变量提升，所以即使出现循环依赖，也不会出现 undefined 类似的运行时错误。

当然 es5 → es6 还有非常多新的特性（http://es6-features.org/#Constants），因此也可能会有更多新的问题。




### 3.3 esbuild 可否支持到 es5 

不太幸运的是， 官方的态度是不会去支持 es5。 这里有个讨论可以看下： https://github.com/evanw/esbuild/issues/297

社区有些方案基本还是用上了 babel 去做转化， 但是既然使用 esbuild 再用一次 babel 就有些多此一举了。

### 3.4 尝试使用 swc-loader

对于上面的问题，一个比较好的方案是，使用 swc-loader  替代 babel 将代码编译为 es5 版本，使用 esbuildMinifyPlugin  替代 Terser 压缩代码。

但是在使用 swc-loader 时，发现其对 vue 中的 jsx  语法支持性不好。




## 4.反思
在一系列排查后，关于使用 esbuild-loader 替换 babel-loader 还没有找到一个完美的解决方案。目前仅使用到了 esbuild 的 `esbuildMinifyPlugin` （代码压缩）。
新项目的历史负担较小，可以尝试使用 esbuild 。如果是内部项目，也可以尝试使用 esm ，就不必处理 `tree-shaking / split-chunk` 相关的内容了。
我们习惯了将代码编译更具兼容性的 es5, 所以我们常常忽视循环依赖的副作用。但是依然推荐打开 eslint 的 循环依赖检测：`"import/no-cycle": [2, { maxDepth: 2 }]` ， 我们应该避免写出循环依赖过深的代码。
esbuild  包括 swc  对 css 的编译支持度都不太高，因为依然需要 `vue-loader/less-loader`
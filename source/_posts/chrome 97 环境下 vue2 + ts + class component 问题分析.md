---
title: chrome 97 环境下 vue2 + ts + class component 问题分析
tags:
  - chrome
  - 问题排查
date: 2022-01-19 16:20:56
---

## 一、背景

2022 年的 1月 4 日，Chrome 浏览器发布了新年的第一个正式版本 97.0.4692.71， 为我们带来了 WebTransport / Dev Tool Recorder 等[有趣的特性](https://chromestatus.com/features)(ps:这些特性跟今天要讲的问题没有半毛钱关系)，但同时也为我的开发工作带来了一点小小的困扰。

## 二、现象

某日在升级最新的 chrome 97 版本后，打开熟悉的项目，本地启动会报大量的类似下面的 props 警告，从而导致下面行为异常：

> "Avoid mutating a prop directly since the value will be overwritten whenever the parent component re-renders. Instead, use a data or computed property based on the prop's value. Prop being mutated:...."




具体的异常行为表现为：接收 props 的组件不能正常渲染，页面白屏，需手动触发才能拿到 props。

在排除了一些缓存、重启类似的的问题后，最后定位到:

在使用了 chrome 97 + 开发模式 + vue2 + vite + ts + class component 的情况下，可稳定复现此问题。其他非 chromium 内核的浏览器或者低版本的 chrome 无此问题。

附上一些使用的库版本：
- vue2 : vue@2.6.14
- vite: vite@2.7.10
- vuex: vuex@3.6.2
- typescript: typescript@4.5.4
- vue-property-decorator: vue-property-decorator@9.1.2
- chrome: chrome@97.0.4692.71

## 三、过程

不用读系列：以下是在排查问题时的回顾，也可为自己后续排查问题提供一些反思

首先，怀疑是 node_modules 或者 vite 版本问题，删除了相关 cache 、lock 文件 问题依然存在 反思：遇到问题时的常规思路就是重启大法，只解决眼前问题，缺乏排错思维

确定出错行为，初步排查是 chrome 97 版本 + 开发模式下出错，并在 demo 工程中稳定复现 反思：排除一切变量，确定问题真正存在。我坦白，进行到这一步时，我已经有了一种预感：我发现了 Chrome 的 bug!!!

回归错误本身，分析出错内容，解决问题 反思：出错的原因很简单，就是 props 的初始化没有生效。在尝试了几种思路，很容易测试出了 四中的解法

分析深层次的原因，求证 反思：在这一步，怀疑是 chrome 97 在处理 未声明的变量时 做了一些特殊处理导致 vue 在处理时当作了一个 undefined 的 data 属性去处理。

这里感谢每天翻规范的伟良大佬，提供了一些论据。具体的问题分析在下文。

## 四、解法
### 解法零： 更新最新的 Chrome 版本

更新：将 Chrome 更新至最新已无此问题。

最新版本，对于上面出现的问题，chrome 会提示一个警告：Cannot redefine property: msg。 但是不会阻塞Js 任务

### 解法一：tsconfig 不使用 esnext

如果使用了 typescript, 使用 vue-cli 生成的 tsconfig.json 其中的 target 和 module 默认是 esnext, 这种方式在 chrome 97 下存在问题。将 target 和 module 的选项修改为 非 esnext 的选项均可。

```
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    // ……
  }
}
```




### 解法二：修改 class component props 的使用方式

使用 vue2 + class 组件时，我们常常有以下两种写法

**方式1： 使用 Vue.extend继承**

```js
import { Component, Prop, Vue } from "vue-property-decorator";

const HelloWorldProps = Vue.extend({
  props: {
    msg: String,
    data: Object,
    listData: Array,
  },
});

@Component
export default class HelloWorld extends HelloWorldProps {
  created(): void {
    console.log(this.msg);
    console.log(this.data);
    console.log(this.listData);
  }
}
```




方式2： 使用 vue-property-decorator提供的 @props 能力

```js
import { Component, Prop, Vue } from "vue-property-decorator";

@Component({
  props: {
    msg: String,
    data: Object,
    listData: Array,
  },
})
export default class HelloWorld extends Vue {
  // 我们常会声明几个未初始化的变量，是为了解决 ts this 的报错问题
  // 还有一种直接引入 @props() 装饰器的写法，和本方法内部行为一致，归为一种
  msg!: string;
  data!: any;
  listData!: any;

  created(): void {
    console.log(this.msg);
    console.log(this.data);
    console.log(this.listData);
  }
}
```




使用 方式1 不会存在问题，但在使用 方式2 时，需注释掉那些未声明的变量（这会带来 ts 错误警告，可忽略），如下：

```js
import { Component, Prop, Vue } from "vue-property-decorator";

@Component({
  props: {
    msg: String,
    data: Object,
    listData: Array,
  },
})
export default class HelloWorld extends Vue {
  created(): void {
    console.log(this.msg);
    console.log(this.data);
    console.log(this.listData);
  }
}
```

## 五、原因

首先我们来看一段测试代码：

```js
class Base {
  
  constructor () {
    this.init()
  }

}

class Test extends Base {
  msg;
  init () {
    Object.defineProperty(this, 'msg', {
        set (val) {
          console.log('msg set: ', val);
        },
        configurable: true
    })
  }
}

const test = new Test();
```




上面的这段代码，在非 chrome97 的浏览器，未初始化的属性不会触发 set 方法， 但是新版本的会触发。

所以，对应到上面的问题，在过去我们使用 props 时，通常会有 `@Prop({ type: String, default: "" }) msg!: string;` 类似的写法。这种方式在旧版本里，msg 并不会触发 set 方法，vue 也并不会将其看做是一个响应式的属性。但是在新版本中，msg 触发 set 方法，vue 将其看作是一个 undefined 的变量处理了，所以在页面初次渲染，拿到 props 将永远都是 undefined, 从而产生异常行为。



这里还有几个相关的 issue(参考链接 3，4):

tsc repo: https://github.com/tc39/proposal-class-fields#public-fields-created-with-objectdefineproperty




下面这个 bug 已经修复：

- bug： https://bugs.chromium.org/p/v8/issues/detail?id=12421&q=%5B%5BSet%5D%5D%20type%3DBug&can=2
- v8 changes: https://chromium.googlesource.com/v8/v8/+/e81ef8be9c59d2f37bb7b61183d3c2ee9d67158a

## 六、参考链接

- 重现仓库
- [tsc](https://github.com/tc39/proposal-class-fields#public-fields-created-with-objectdefineproperty)
- [v8 bug](https://bugs.chromium.org/p/v8/issues/detail?id=12421&q=%5B%5BSet%5D%5D%20type%3DBug&can=2)
---
title: '深入浅出nodejs学习笔记--第十、十一章 测试 产品化'
date: 2017-04-21 14:37:19
categories:
  - nodejs
tags:
  - nodejs
  - 学习笔记
---

# **单元测试**

我们知道后端都有单元测试，比如学习Java用到的Junit，很好用，那么前端有没有单元测试呢？答案当然是有的。这里就简单总结一下前端单元测试的内容和一些常用的测试框架。

## **单元测试编写原则：**

我们都知道做单元测试可以有很多好处，但是在了解单元测试之前，先来看一下单元测试的编写规则，在编写可测试的代码需要注意以下3个问题：

* 单一职责： 尽量细分代码的职责，不要给一段代码附加太多而逻辑从而使代码变得不可控。
* 接口抽象： 对于大的项目，业务逻辑比较复杂，要记得写接口，针对接口进行测试。
* 层次分离： MVC就是最好的例子。



## **断言**

单元测试的核心应用就是断言，用一个最简单的例子介绍一下什么是断言：

    var assert = require('assert')
    assert.equal(Math.max(1, 100), 100)

  equal()是断言模块的内置方法，用于判断实际值与期望值是否相等。类似的方法还有ok()、notEqual()等。上句代码的意思是判断 Math.max(1, 100) 的输出结果是不是100，如果不是，就抛出异常。

  

  ## **测试框架**

  单元测试很简单，下面介绍两个常用的单元测试库，**jasmine 和 mocha**

  **jasmine**

  Jasmine的开发团队来自PivotalLabs，他们一开始开发的JavaScript测试框架是JsUnit，来源于著名的JAVA测试框架JUnit。JsUnit是xUnit的JavaScript实现。但是JsUnit在2009年后就已经停止维护了，他们推出了一个新的BDD框架Jasmine。Jasmine不依赖于任何框架，所以适用于所有的Javascript代码。

  下面直接放一个官方的使用案例：

  <pre class="prettyprint">describe(<span class="hljs-string">"A spec"</span>, <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span> {</span>
    <span class="hljs-keyword">var</span> foo;
    beforeEach(<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span> {</span>
      foo = <span class="hljs-number">0</span>;
      foo += <span class="hljs-number">1</span>;
    });
    afterEach(<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span> {</span>
      foo = <span class="hljs-number">0</span>;
    });
    it(<span class="hljs-string">"is just a function, so it can contain any code"</span>, <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span> {</span>
      expect(foo).toEqual(<span class="hljs-number">1</span>);
    });
    it(<span class="hljs-string">"can have more than one expectation"</span>, <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span> {</span>
      expect(foo).toEqual(<span class="hljs-number">1</span>);
      expect(<span class="hljs-literal">true</span>).toEqual(<span class="hljs-literal">true</span>);
    });
    describe(<span class="hljs-string">"nested inside a second describe"</span>, <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span> {</span>
      <span class="hljs-keyword">var</span> bar;
      beforeEach(<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span> {</span>
        bar = <span class="hljs-number">1</span>;
      });
      it(<span class="hljs-string">"can reference both scopes as needed"</span>, <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span> {</span>
        expect(foo).toEqual(bar);
      });
    });
  });</pre>

  jasmine单元测试有二个核心的部分：describe 函数块和it函数块

  describe和it函数都有二个参数： 

  第一个参数：测试描述； 

  第二个参数：测试用的具体逻辑

  

  **mocha**

  mocha的使用和jasmine类似，这里也直接放一个例子：

  <pre class="prettyprint">var <span class="hljs-built_in">assert</span> = <span class="hljs-built_in">require</span>(<span class="hljs-string">'assert'</span>);
  describe(<span class="hljs-string">'Array'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span></span> {
    describe(<span class="hljs-string">'#indexOf()'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span></span> {
      it(<span class="hljs-string">'should return -1 when the value is not present'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span></span> {
        <span class="hljs-built_in">assert</span>.equal(-<span class="hljs-number">1</span>, [<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>].indexOf(<span class="hljs-number">4</span>));
      });
    });
  });</pre>

describe块称为”测试套件”（test suite），表示一组相关的测试。它是一个函数，第一个参数是测试套件的名称（随便起，能让别人看懂就好），第二个参数是一个实际执行的函数。 

it块称为”测试用例”（test case），表示一个单独的测试，是测试的最小单位。它也是一个函数，第一个参数是测试用例的名称（随便起，能让别人看懂就好），第二个参数是一个实际执行的函数。

这里只简单介绍，具体使用可以看阮一峰的mocha实例教程： [http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)



## **性能测试**

性能测试主要包括基准测试、压力测试、和负载测试：

*   基准测试： 基准测试的目的是统计在多少时间内执行了多少方法。
*   压力测试：压力测试就是模拟请求测试网络接口，查看吞吐率，响应时间和并发数。常用的工具是ab、siege、http_load,使用很简单，直接看文档就可以使用，这里不多赘述
*   负载测试： 负载测试和压力测试很像，主要是测试数据在超负荷环境中运行，程序是否能够承担。



# **产品化**

产品化的内容很简单，重点在于实践和工程化中的运用。主要包括以下：

## **项目工程化**

目前现有的一些项目工程管理，不如webpack，gulp等，用起来都很方便。



## **性能优化**

以下是几个常见的提升性能的方法：

*   动静分离： 动态请求和静态请求分离开来
*   启用缓存： 适当利用缓存，但是不可以过分利用缓存
*   多线程架构：具体可以参照第九章的笔记
*   读写分离：主要针对数据库，因为读写操作速度是不一样的，读的速度是高于写入的速度的，所以对于不同的业务，可以对数据库做读写分离处理，具体可以查看博客：[https://my.oschina.net/candiesyangyang/blog/203425](https://my.oschina.net/candiesyangyang/blog/203425)



## **日志**

对于程序来说，写日志是十分重要的，对于异常日志的捕捉都是对日后程序维护的利器，一般情况推荐把日志存为文件，不推荐存入数据库。



## **监控报警**

所谓监控主要包括对 日志监控、响应时间、进程监控、磁盘监控、内存监控、CPU占用监控、CPU load监控、I/O负载、网络监控、应用状态监控、DNS监控。报警一般都是通过短信或者邮件来执行。



## **稳定性**

为保证应用的稳定性，可以采用多机器、多机房、多备份的方式来保证。



## **异构共存**

主要是指与其他语言的兼容性，协同作为。

* * *
<br/>前端新手，弱鸡一枚，如有错误，请指正，谢谢！

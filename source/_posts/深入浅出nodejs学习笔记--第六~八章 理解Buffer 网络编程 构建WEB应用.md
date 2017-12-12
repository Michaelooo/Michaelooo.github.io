---
title: '深入浅出nodejs学习笔记--第六~八章 理解Buffer 网络编程 构建WEB应用'
date: 2017-04-19 14:34:26
categories:
  - nodejs
tags:
  - nodejs
  - 学习笔记
---


## **第六章 理解Buffer**

这一章要理解的不多，都是一些buffer的常见操作，看API就可以熟悉，如果做过后台的就不会陌生，这里需要注意的几个地方就是

*   **Buffer所占用的内存**不是通过V8分配的，属于堆外内存，所以意思就是其实在V8启动的时候就会有一个Buffer对象一直常驻内存，无需通过require引入。
*   **Buffer的内存分配**分为小内存分配和大内存分配，小内存分配一般指的是小于 8kb 的 Buffer 的对象，大内存当然就是大于 8kb的Buffer 对象。
*   **Buffer的转换**主要体现在字符串转Buffer和Buffer转字符串，字符串转Buffer直接通过构造函数来实现。
        new Buffer(str, [encording]) //encording值编码类型
Buffer转字符串通过 toString() 可以实现。
        buf.toString([encording], [start], [end])

*   **Buffer的性能**，通过预先转换静态内容为Buffer对象，可以有效减少CPU的重复使用，从而节省服务器资源。


## **第七章 网络编程**

这一章也比较简单，主要分为四个，构建TCP服务、构建UDP服务、构建HTTP服务、构建WebSocket服务，前两个不是重点，后两个比较常用。不多说，直接看代码。

**构建TCP服务**

<pre class="prettyprint"><span class="hljs-comment">//示例，创建一个TCP服务器端</span>
<span class="hljs-keyword">var</span> net = <span class="hljs-built_in">require</span>(<span class="hljs-string">'net'</span>) <span class="hljs-comment">//依赖node自带的net模块</span>

<span class="hljs-keyword">var</span> server = net.createServer(<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">(socket)</span> {</span>
    <span class="hljs-comment">//新的连接</span>
    socket.on(<span class="hljs-string">'data'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">(data)</span> {</span>
        console.log(<span class="hljs-string">'连上了'</span>)
    })
    <span class="hljs-comment">//断开连接</span>
    socket.on(<span class="hljs-string">'end'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">(data)</span> {</span>
        console.log(<span class="hljs-string">'连接断开'</span>)
    })
    socket.write(<span class="hljs-string">'创建一个TCP服务器端'</span>)
})

<span class="hljs-comment">//监听</span>
server.listen(<span class="hljs-number">1234</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">()</span> {</span>
    console.log(<span class="hljs-string">'已绑定1234端口号'</span>)
})</pre>

**构建一个UDP服务器端和一个UDP客户端**

<pre class="prettyprint"><span class="hljs-comment">//示例，创建一个UDP服务器端</span>
<span class="hljs-keyword">var</span> dgram = <span class="hljs-built_in">require</span>(<span class="hljs-string">'dgram'</span>)

<span class="hljs-keyword">var</span> server = dgram.createSocket(<span class="hljs-string">'udp4'</span>)

server.on(<span class="hljs-string">'message'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">(msg, rinfo)</span> {</span>
console.log(<span class="hljs-string">'服务器得到了'</span> + msg + <span class="hljs-string">'来自'</span> + rinfo.address + <span class="hljs-string">':'</span> + rinfo.port)
})

server.on(<span class="hljs-string">'listening'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">()</span> {</span>
<span class="hljs-keyword">var</span> address = server.address()
console.log(<span class="hljs-string">'已绑定'</span> + rinfo.address + <span class="hljs-string">':'</span> + rinfo.port)
})

<span class="hljs-comment">//示例，创建一个UDP客户端，与UDP服务器端对话</span>
<span class="hljs-keyword">var</span> dgram = <span class="hljs-built_in">require</span>(<span class="hljs-string">'dgram'</span>)

<span class="hljs-keyword">var</span> message = <span class="hljs-keyword">new</span> Buffer(<span class="hljs-string">'test.txt'</span>)
<span class="hljs-keyword">var</span> client = dgram.createSocket(<span class="hljs-string">'udp4'</span>)

<span class="hljs-comment">//通过客户端发送给网络，参数分别对应 要发送的Buffer  Buffer的偏移  Buffer的长度  目标端口 目标地址 完成后的回调</span>
client.send(message, <span class="hljs-number">0</span>, message.length, <span class="hljs-number">1234</span>, <span class="hljs-string">'localhost'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">(err, bytes)</span> {</span> 
client.close();
})</pre>

**构建一个HTTP服务器**

这个真的好熟悉，如下：

<pre class="prettyprint">//示例，创建一个HTTP服务器, <span class="hljs-keyword">http</span>模块继承于TCP中的net模块
var <span class="hljs-keyword">http</span> = <span class="hljs-built_in">require</span>(<span class="hljs-string">'http'</span>)

<span class="hljs-keyword">http</span>.createServer(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-title">req</span>, <span class="hljs-title">res</span>) {</span>
    res.writeHead(<span class="hljs-number">200</span>, {<span class="hljs-string">'Content-Type'</span>: <span class="hljs-string">'text/plain'</span>})
    res.<span class="hljs-keyword">end</span>(<span class="hljs-string">'雷猴\n'</span>)
}).listen(<span class="hljs-number">1234</span>, <span class="hljs-string">'localhost'</span>)

console.<span class="hljs-built_in">log</span>(<span class="hljs-string">'已绑定 http://localhost:1234'</span>)</pre>

**构建WebSocket服务**

WebSocket也是一种基于事件的编程模型，所以和Node结合也是相得益彰，同时WebSocket实现了客户端和服务器端之间的长连接，Node事件驱动的方式十分擅长于大量的客户端保持高并发的连接。

<pre class="prettyprint">//示例，WebSocket在客户端的应用
var <span class="hljs-built_in">socket</span> = <span class="hljs-built_in">new</span> WebSocket(<span class="hljs-string">'ws://localhost:1234/update'</span>)

<span class="hljs-built_in">socket</span>.onopen = <span class="hljs-function"><span class="hljs-keyword">function</span> () {</span>
    setInterval(<span class="hljs-function"><span class="hljs-keyword">function</span> () {</span>
        <span class="hljs-keyword">if</span> (<span class="hljs-built_in">socket</span>.bufferedAmount === <span class="hljs-number">0</span>) {
            <span class="hljs-built_in">socket</span>.<span class="hljs-built_in">send</span>(getUpdateData())
        }
    }, <span class="hljs-number">50</span>)
}

<span class="hljs-built_in">socket</span>.onmessage = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-title">event</span>) {</span>
   <span class="hljs-comment"> //todo: event.data 的处理</span>
}</pre>

**网络服务与安全**

还有一个关于网络编程的便是网络安全。node在网络安全上提供了3个模块，**crypto**、**tls**、**https**。其中crypto主要用于加密解密，tls模块提供了与net模块类似的功能，区别在于建立在TSL/SSL加密的TCP连接之上，https与http基本都是一致的，区别在于前者更安全。


## **第八章 构建Web应用**

关于构建Web应用这一章，其实现有其他的框架讲的而且运用的已经很详细，比如KOA，比如Express，试着运用这两个做一个web应用更能加深理解，这里就总结一下构建Web应用的组成

**基础功能**

*   **请求方法**：常见的请求方法有GET、POST、PUT、DELETE，存在于请求报文的第一行的第一个单词，通常为大写。
*   **路径解析**：浏览器将请求解析成报文，位于请求报文的第二行。
*   **查询字符串**：即请求传递的参数。
*   **cookie**：记录服务器和客户端之间的状态。服务器端生成向客户端发送 –&gt; 浏览器保存cookie –&gt;
    每次浏览器发送请求都会携带cookie，cookie会造成带宽浪费，所以可以减少cookie的大小，为静态的组件使用不同的域名，减少DNS的查询来避免。
*   **session**：与cookie作用类似，但是session只保留在服务器端，并且常驻内存（利用Redis或者Memcached可以统一转移到集中的数据存储中）。session也会有安全问题，但是相对较小，常见的漏洞就是XSS漏洞（跨站脚本攻击）。
*   **缓存**：利用浏览器来缓存静态资源，目的是提升加载速度从而提升体验。
*   **Basic认证**：请求报文头部的Authorization，这种方式有缺陷，因为在网络传输中这些验证接近于明文，所以不可取。

**数据上传**

*   **表单数据**：即常见的form表单提交。
*   **JSON/XML**: 提交的数据是JSON/XML格式的，现在大部分的交互都是用的JSON，XML的也有用，比如微信公众号平台的开发的交互就是用的XML，这个真的贼坑。
*   **附件上传**：利用form-data来实现附件上传。
*   **数据上传安全**：数据上传的安全性问题主要体现在内存限制和跨站伪造请求的问题上，所以一要对上传做限制，而是在开发中要加hash值做标识，就是加一个随机数。

**路由解析**

*   **文件路径型**：分为静态文件和动态文件，不需要解释。
*   **MVC**：前端MVC，之前一篇博客有讲，现在前端都是MV*。
*   **RESTful**: 表述性状态转移，强调所有的资源都是可以通过URL访问到，对URL做文章，与MVC不冲突。可以看下阮一峰的博客：[http://www.ruanyifeng.com/blog/2014/05/restful_api.html](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

**中间件**

*   **异常处理**：中间件的核心就是尾调用next()，所以对于一些异常，需要在next()添加一个参数,并且把捕获到的异常传递过来。
*   **性能**：中间件的出现时服务于具体业务的，所以要特殊特用，性能问题并不大。

**页面渲染**

*   **内容响应**：内容的响应主要依赖于报文中的Content-*字段，它决定了客户端会以什么样的方式来作出响应，下载、跳转等。
*   **视图渲染**：一般是通过模板加数据共同生成出来的。
*   **模板**：比视图渲染更进一步，模板的使用的对前端页面的一种复用，是对html体的复用。比如javaweb中的jsp，或者PHP，当然这些都相对成熟，对于初出茅庐的node来说，现在常用的模板渲染模块有jade,heredoc,ejs等。

* * *

<br/>前端新手，弱鸡一枚，如有错误，请指正，谢谢！

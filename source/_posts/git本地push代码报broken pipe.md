---
title: 'git本地push代码报broken pipe'
date: 2017-03-06 22:42:17
categories:
  - Github
tags:
  - github
  - 爬坑之路
---

以下基于windows开发环境


最近在本地写了一点代码，于是就想把它放到Github上去，前面的步骤什么的不多说，就是最后在push代码的时候总是跑出异常 

诺，就是下面的这个样子的

![这里写图片描述](http://img.blog.csdn.net/20170306223640001?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzcwNzI0OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

搜索了很多资料，先pull再push，删除远程readme文件，甚至想到用gitosis配置权限（都是些什么鬼，我特么又不是要自己搭服务器……），但是还是没有解决。最后考虑是github配置问题，查阅一些先人的遗迹，终于找到解决办法。perfect

为了防止以后遇到意想不到的错误，建议小白修改一下配置

**第一，设置长久连接**

在git安装目录下，找到/etc/ssh/ssh_config,然后随便用什么编辑器打开，添加

不知道怎么写看着配置文件上面的格式来写就行了

	Host *  
	ServerAliveInterval 120 
这个配置的作用就是保证ssh ServerAliveInterval始终连接，我这次出现错误的原因就是在无线局域网环境下，ssh连接到远程主机后如果一段时间内不操作，就会出现掉线的情况，所以最后加上此配置项

**第二， 修改post缓存大小** 

因为如果缓存不够用，有时候也会发生broken pipe的情况，所以最好的对post的缓存大小作设置 

这个很简单，直接在git bash 上输入git config http.postBuffer ，value指的就是设置的缓存大小，如下：

<pre class="prettyprint">git config <span class="hljs-keyword">http</span>.postBuffer <span class="hljs-number">52428800</span>

52428800的单位是字节，git默认的是1MB，我们把它设置成50MB

以上，就可以在同性网里开心的玩耍了。

* * *

<br/>前端新手，弱鸡一枚，如有错误，请指正，谢谢！
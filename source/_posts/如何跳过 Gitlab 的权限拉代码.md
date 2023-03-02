---
title: 如何跳过 Gitlab 的权限拉代码
date: 2022-01-11 16:00:56
tags:
  - git
---

!!! note 声明，这只能当作一个临时方案，建议还是申请权限进行代码获取。


起因：有个同事问我参考一个项目的工程结构，但是因为一些原因，他没有该工程的权限，同时也没有大的必要去申请`Developer`的权限。

一开始我是这么做的：
1. 备份我的工程文件到 file_bak
2. 去除掉 node_modules & dist 之类的包和构建文件
3. 打包 zip 然后选择文件发送

貌似解决了这个问题，但：**我的工程变化很大，他无法及时拉取我的变更**。有没有更好的方式呢？

于是想到，Git 不是宣称去中心化吗，然后我们还是在一个局域网环境，那理论上我们是可以互相 clone 的，参考下面的操作：

首先把我们想要分享的目录暴露出来， 我的工作目录是 `~/Documents/company_name`，那么执行下面的命令

```sh
cd ~/Documents/company_name
git daemon --verbose --export-all --base-path=.
```

然后其他人就可以直接 clone 了,找到你的本机 IP 
```sh
git clone git://your-host-ip/dev-help-portal-fe new-project-name
```
执行 `git remote show origin` 看下，以下为示例

```sh
* remote origin
  Fetch URL: git://10.12.188.229/dev-help-portal-fe
  Push  URL: git://10.12.188.229/dev-help-portal-fe
  HEAD branch: master
  Remote branches:
    feature/SPXFM-17075-init                        tracked
    feature/SPXFM-19666-login-page                  tracked
    feature/SPXFM-20841-add-bundle-analysis-support tracked
    master                                          tracked
  Local branch configured for 'git pull':
    master rebases onto remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

另外，你可以第一步的启动服务就作为一个守护进程开机启动（这里就不介绍了），那么别人就可以随时随地的拉取变更了，但是**建议还是用完就 `ctrl + C`中止掉服务**。


参考链接：[git 局域网 两个客户端之间同步? - 知乎](https://www.zhihu.com/question/54672976/answer/406797841)
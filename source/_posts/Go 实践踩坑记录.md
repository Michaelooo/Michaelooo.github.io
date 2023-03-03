---
title: Go 实践踩坑记录
tags:
  - golang
date: 2022-02-23 13:34:00
---

记录FE 开发 Go 中遇到的一些问题：

## 1. context.WithValue 的 key 不允许使用 built-in 类型（go-static-check）

解决办法：https://stackoverflow.com/questions/40891345/fix-should-not-use-basic-type-string-as-key-in-context-withvalue-golint

## 2. golang的数据类型与mysql的数据类型对应的关系

https://gobea.cn/blog/detail/j5ynaOod.html

## 3. 有关 go 的零值问题

https://youwu.today/skill/backend/golang-zero-value-and-reference-and-null-value-testing/

## 4. 查看引用 会打开超链接

在用户配置中禁用

```json
"[go]": {
  "editor.links": false
},
```

## 5.使用 wire 声明代码遇到 no provider found xxx 类似的错误

因为 wire 是一个依赖注入的框架，所以你需要提前声明，遇到这类问题，可自行检查

view & server 使用到的一些方法是否在 wire_set 文件作声明（类似 View.NewConfigService）
一些 facade 接口，接口的定义和实现是否一致

## 6. Go本地调用 python服务的抛出 400 错误

在用 go 调用 python 的服务时，本地联调过程（直接访问 BE 的 IP）发现服务会走不通, 报 400 和一堆乱码错误，但是部署了测试环境却可以正常访问。目前只能是 BE 部署了测试环境再进行联调，比较影响效率

解决办法： 使用 IP 访问时，应使用 HTTP 协议，非 HTTPS 协议 参考： https://stackoverflow.com/questions/49389535/problems-with-flask-and-bad-request

## 7. Goproxy 报错

修改 GOProxy 地址，重新进入项目即可

参考：https://townsyio.medium.com/how-to-configure-your-go-modules-proxy-in-vscode-fa41f29192fe

![go-proxy-error.png](https://s2.loli.net/2023/03/03/xTzlR5yFEDmgfNr.png)

## 8. grpc 局限

由于注册中心的限制，只有相同环境才能调用 grpc。由于内部系统只有两个环境 test 和 live，在两个环境下，能分别调用 test 和 live 环境下的grpc。由于没有 uat 环境，所以无法访问 uat 服务，要用 grpc 请考虑环境是否匹配。
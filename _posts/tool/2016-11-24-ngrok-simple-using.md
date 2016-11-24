---
layout: post
title: ngrok 的简单使用
category: 工具
tags: Ngrok
description: ngrok 的简单使用
---

### ngrok 介绍

> ngrok 是一个反向代理，通过在公共的端点和本地运行的 Web 服务器之间建立一个安全的通道。ngrok 可捕获和分析所有通道上的流量，便于后期分析和重放。 --[百度百科](http://baike.baidu.com/item/ngrok)

### ngrok 应用场景

1. 在本机开发好的网站想让客户测试或者让朋友帮忙调试，不再需要上传到服务器上面，使用 ngrok 就可以解决。
2. 微信开发也不需要再上传到服务器，ngrok 可以解决80端口问题，实现微信服务器直接访问到本机的 web 服务。
3. ngrok 不但提供了一个在外网能够安全的访问内网 Web 主机，还能捕获所有请求的 http 内容，方便调试，甚至还支持 tcp 层端口映射。

### ngrok安装配置

- ngrok 支持 Mac OS X，Linux，Windows 平台。由于博主使用的 Windows 系统，因此安装配置以  Windows 系统为例。

1. [官网下载](https://ngrok.com/download) windows 版压缩包。
2. 将下载的压缩包解压到合适的位置，例如： `E:\ngrok` 目录下。
3. 在官网注册登录(支持 GitHub 账号授权登录)，获取自己的 `authtoken` 密钥 。
4. cmd 工具下进入解压后的目录，输入：`ngrok authtoken 密钥 `,得到:  
    Authtoken saved to configuration file: C:\Users\xiaohai/.ngrok2/ngrok.yml
5. 输入 `ngrok http 80`  , 将会显示如下内容：

```
ngrok by @inconshreveable                                                                               

Session Status                online
Account                       ishaiua (Plan: Free)
Version                       2.1.18
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://2f9793a0.ngrok.io -> localhost:80
Forwarding                    https://2f9793a0.ngrok.io -> localhost:80

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```
至此安装配置结束，得到了一个随机的支持 http 和 https 的地址，该地址指向本地的 `localhost:80` ，其中端口可以修改，访问 `http://localhost:4040` 可以查看所有请求内容。

### 总结

优点：前面已经讲了。

缺点：给的域名是随机分配的，每次开启都不样，不易记住（付费除外）；不稳定，随时可能被墙。



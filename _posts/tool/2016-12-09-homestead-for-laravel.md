---
layout: post
title: Laravel 项目配置独立 Homestead
category: 工具
tags: Homestead Laravel
description: 为 Laravel 项目配置独立 Homestead 虚拟机
---

### why？

全局安装 Homestead 将会使每个项目共享同一个 Homestead 盒子，避免一个项目安装的工具影响到另一个项目的开发，有时候我们需要为某个单独的项目安装独立的 Homestead 虚拟机 ，单独的 Homestead 虚拟机可以一条命令行轻松销毁，不会影响已有项目。

### 前提

- [在 Windows10 系统已經成功安裝好 Homestead](http://laravelacademy.org/post/2749.html)

### 安装 Laravel 项目

```
composer create-project laravel/laravel=5.2.* laravel-app --prefer-dist
```

### 安装 Homestead

在项目根目录下安装 laravel/homestead 

```
composer require laravel/homestead --dev
```

### 生成配置文件


```
vendor/bin/homestead make
``` 
打开配置文件 Homestead.yaml :

![Homestead.yaml](http://og6e4y8ws.bkt.clouddn.com/homestead.yaml.png)

### 共享文件

Homestead.yaml 文件中的 folders 属性列出了所有你想在 Homestead 环境共享的文件夹列表。这些文件夹中的文件若有变动，他们将会同步在你的本机与 Homestead 环境里。你可以将你需要的共享文件夹都配置进去。

```
folders:
    - map: "E:/Laravel/Haiua-app"
      to: "/home/vagrant/haiua-app"
```

### Nginx 站点

Homestead.yaml 文件中的 sites 属性允许你简单的对应一个 域名 到一个 homestead 环境中的目录。一个例子的站点被配置在 Homestead.yaml 文件中。同样的，你可以加任何你需要的站点到你的 Homestead 环境中。Homestead 可以为你每个进行中的 Laravel 应用提供方便的虚拟化环境。


```
sites:
    - map: haiua.app
      to: "/home/vagrant/haiua-app/public"
```
### 数据库

```
databases:
    - homestead
```
如果使用虚拟机中的数据库 应该设置数据库端口为 33060 。

### 配置 hosts 文件

修改本机的 hosts 文件, 允许通过自定义域名访问, 这个域名是在上面的 Homestead.yaml 里面设置的.

```
192.168.10.10 haiua.app
```

### 启动

```
vagrant up
```


### vagrant ssh

```
$ vagrant ssh
Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-22-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
Last login: Sat Dec 10 13:57:26 2016 from 10.0.2.2
vagrant@haiua-app:~$ 

```
### 其他命令

```
vagrant ssh   //登陆虚拟机

vagrant provision //修改配置后需运行

vagrant halt  //关闭

vagrant destroy  //销毁
```



### 错误类型 1

```
Timed out while waiting for the machine to boot. This means that
Vagrant was unable to communicate with the guest machine within
the configured ("config.vm.boot_timeout" value) time period.

If you look above, you should be able to see the error(s) that
Vagrant had when attempting to connect to the machine. These errors
are usually good hints as to what may be wrong.

If you're using a custom box, make sure that networking is properly
working and you're able to connect to the machine. It is a common
problem that networking isn't setup properly in these boxes.
Verify that authentication configurations are also setup properly,
as well.

If the box appears to be booting properly, you may want to increase
the timeout ("config.vm.boot_timeout") value.

```

仅供参考方案：

![timeout](http://og6e4y8ws.bkt.clouddn.com/homestead.timeout.png)

### 总结

Homestead 这尼玛就是坑。 各种坑，坑死你不要命。


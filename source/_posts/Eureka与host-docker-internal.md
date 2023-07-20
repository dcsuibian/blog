---
title: Eureka与host.docker.internal
tags: Docker Eureka Java
description: Eureka与host.docker.internal冲突问题
abbrlink: 4064957586
date: 2021-05-23 21:38:19
---

# 问题描述

## 环境：

1. OS : Windows 10
2. Java : 11
3. Spring : 2.3.11 RELEASE
4. Spring Cloud : Hoxton.SR11

注意：我的主机的ip是**10.65.7.32**（局域网的，要是公网的也不可能放出来）

## 具体情况

将Eureka和服务提供方部署在同一台主机上时，Eureka显示的服务提供方为`host.docker.internal`：

![image-20210523214836537](https://wexcdn.com/img/20210523223340.png)

同时，网址打不开：

![image-20210523214915436](https://wexcdn.com/img/20210523223343.png)

# 解决方案

解决方案网上基本都能搜到：

## ① 删除`C:\Windows\System32\Drivers\etc\hosts`的部分条目。

来自[StackOverFlow](https://stackoverflow.com/questions/57319678/spring-boot-cloud-eurka-windows-10-eurkea-returns-host-docker-internal-for-clien)

![image-20210523215248826](https://wexcdn.com/img/20210523223346.png)

## ② 设置`eureka.instance.prefer-ip-address`为`true`

## ③ 设置`eureka.instance.hostname`

# 细节剖析

要是直接给解决方案，那倒没什么好写了，接下来来分析一下出现这个问题的原因。

<br/>

首先，我们知道，`host.docker.internal`是为了在docker虚拟机中提供宿主机ip的。

因为是虚拟机，所以`localhost`和`127.0.0.1`都会被认为是虚拟机的地址，但是我们发现，docker里的虚拟机是可以正常访问互联网的，所以就有了一个办法，把宿主机当做一个路由器，用跨网段访问的方式。

![image-20210523220214497](https://wexcdn.com/img/20210523223349.png)

实际上Docker为我们做的事就是更改本机的hosts文件，让`host.docker.internal`解析成宿主机的网卡的地址。

![image-20210523220352973](https://wexcdn.com/img/20210523223351.png)

实际上，ping是可以ping通的。但是我们会发现，在浏览器上就不行了。

![image-20210523221003172](https://wexcdn.com/img/20210523223353.png)

![image-20210523221018621](https://wexcdn.com/img/20210523223458.png)

但我们可以看到，直接输入ip地址是可以的。

关于这个问题，我没有特别去查相关的资料，不过既然这样，我认为`host.docker.internal`这个应该是解析不成正常的域名的，所以还是要换成ip.

可是，为什么我发送给Eureka会把服务的地址设为`host.docker.internal`呢？

让我们直接看看IDE的提示：

![image-20210523221556109](https://wexcdn.com/img/20210523223356.png)

原来，当我们不填写hostname的时候，hostname就会从操作系统推导，所以被推成了`host.docker.internal`。

那么，就有两种方式解决这个问题了：

```yaml
eureka:
  instance:
    hostname: 10.65.7.32 # ①
    # prefer-ip-address: true # ②
```

不过，具有迷惑性的是，无论你采用哪一种方式，在Dashboard上都仍然会有`host.docker.internal`。但实际上，它们的链接已经变成了真正的地址，可以正常使用了。

![image-20210523222651132](https://wexcdn.com/img/20210523223358.png)

## 再具体一点

再具体一点，看看Eureka的客户端到底发了啥。

![image-20210523223015802](https://wexcdn.com/img/20210523223405.png)

![image-20210523223029002](https://wexcdn.com/img/20210523223408.png)

现在可以看到了，原来` host.docker.internal:ingredient-service:0`是这个实例的id，而hostname就是我们刚刚改的部分。

如果你采用了改hostname的方式，那么这里的hostname就会变成你改成的值。

而如果你采用了改`eureka.instance.prefer-ip-address`，那么其实hostname的值会变成你的ip地址，也就是这里的10.65.7.32。

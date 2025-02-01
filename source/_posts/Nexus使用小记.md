---
title: Nexus使用小记
tags: Nexus Docker
description: Sonatype Nexus Repository的使用记录
abbrlink: 2826596411
date: 2025-02-02 06:05:45
---


# Nexus使用小记

Nexus（全称为Sonatype Nexus Repository）是一个软件存储库管理器。可以拿来做Docker私服、Maven私服等等。

这篇文章主要关注其作为Docker私服和Docker Hub镜像加速的功能。

Nexus分

* Sonatype Nexus Repository OSS
* Sonatype Nexus Repository Pro

后者要钱。所以我们用OSS。

另外，本篇文章我们会使用Docker的方式安装Nexus。因为更加方便简单，也是大多数教程的选择。

## 目标

1. 建立一个内网私有镜像仓库
2. 缓存从Docker Hub下载的镜像，加速内网其他机器的拉取

## 环境

* Nexus服务器：192.168.81.198（debian8-hyperv，Docker version 27.5.1, build 9f9e405）
* 测试客户端：192.168.81.190（debian-hyperv，Docker version 27.2.0, build 3ab425）

都是Debian 12，都安装并配置了sudo，都把当前用户（普通用户，非root）加到了`docker`用户组里。

另外，你还得有代理工具。

> **提示**
>
> 如果你需要安装Docker，那你可能需要配置APT代理。
>
> 如果你需要拉取Nexus镜像，那你需要配置[`docker pull`代理](https://www.lfhacks.com/tech/pull-docker-images-behind-proxy/)

## 过程记录

### 1、安装Nexus

参考：https://hub.docker.com/r/sonatype/nexus/

```shell
sudo mkdir -p /data/nexus # 创建数据文件夹
sudo chown -R 200 /data/nexus # 更改文件夹所有者

docker run \
  -d \
  --name nexus \
  --restart always \
  -p 8081:8081 \
  -p 8082:8082 \
  -p 8083:8083 \
  -v /data/nexus:/nexus-data \
  sonatype/nexus3:3.76.1 # 其中8081是Nexus的Web端口，其他两个端口后面会讲做什么用
```

登录`http://Nexus服务器地址:8081`，就可以看到如下页面了：

![image-20250202040147803](https://wexcdn.com/img/image-20250202040147803.png)

值得一提的是，这个“Loading...”时间会很长，而且Loading结束后也只会显示如下内容：

![image-20250202040239772](https://wexcdn.com/img/image-20250202040239772.png)

这是因为我们还没有给Nexus本身设置代理，等设置了代理以后就不太一样了。

### 2、Nexus基本设置

点击右上角的“Sign in”，会显示一个登录框，这里我们用账号`admin`登录，密码存储在容器内`/nexus-data/admin.password`文件里，如果你刚刚跟我一样设置了`/data/nexus`为数据文件夹，那么可以直接：

```shell
cat /data/nexus/admin.password
```

登陆后会显示设置向导：

![image-20250202040629891](https://wexcdn.com/img/image-20250202040629891.png)

然后要你重设密码。

重点是配置匿名访问这一步，需要“Enable anonymous access”（允许匿名访问）。

![image-20250202040725956](https://wexcdn.com/img/image-20250202040725956.png)

然后就是配置HTTP代理了，点击左上方齿轮图标，然后选左侧“System”的“HTTP”。勾选“Enable HTTP proxy”和“Enable HTTPS proxy”，并填好你代理服务器的IP和端口，然后保存。

![image-20250202041409960](https://wexcdn.com/img/image-20250202041409960.png)

设置完以后你可以点击齿轮左侧的盒子图标，这时候“Loading...”应该会快点，而且会显示一些其他东西：

![image-20250202041546931](https://wexcdn.com/img/image-20250202041546931.png)

### 3、激活Docker Bearer Token Realm

去“Realms”中将“Docker Bearer Token Realm”激活。

![image-20250202045751815](https://wexcdn.com/img/image-20250202045751815.png)

### 4、创建Blob Store

Blob Store其实就像一个文件夹，其实都存到一起也可以，但是还是分开来更清晰。

![image-20250202041801312](https://wexcdn.com/img/image-20250202041801312.png)

创建三个，一个叫`docker-hub-proxy`，一个叫`docker-hosted`，一个叫`docker-all`。

![image-20250202041945847](https://wexcdn.com/img/image-20250202041945847.png)

![image-20250202042008231](https://wexcdn.com/img/image-20250202042008231.png)

![image-20250202050806822](https://wexcdn.com/img/image-20250202050806822.png)

### 5、创建Repository（仓库）

#### 简要介绍

Nexus的仓库分三种：

* proxy，代理仓库
* hosted，本地仓库
* group，组仓库（其实就是把其他仓库进行一个聚合）

![image-20250202042052807](https://wexcdn.com/img/image-20250202042052807.png)

#### `docker (proxy)`类型的仓库

首先创建`docker (proxy)`类型的仓库，并如下设置：

![image-20250202044900637](https://wexcdn.com/img/image-20250202044900637.png)

![image-20250202042720992](https://wexcdn.com/img/image-20250202042720992.png)

其中允许匿名访问是比较重要的，“Enable Docker V1 API”这个你开不开都可以，“HTTP Authentication”里填的是你Docker Hub的用户名密码，因为匿名用户拉取镜像有限制。另外Remote Storage填的URL其实就是你`docker pull`失败时显示的URL（**但是要去掉`v2/`**）：

![image-20250202043030909](https://wexcdn.com/img/image-20250202043030909.png)

创建仓库后，你就可以测试一下了。我们来到测试机（192.168.81.190）然后将`/etc/docker/daemon.json`修改成这个样子：

![image-20250202043431884](https://wexcdn.com/img/image-20250202043431884.png)

其中`insecure-registries`主要是为了不让他用HTTPS，`registry-mirrors`是告诉Docker这个地址是个镜像仓库。另外，`registry-mirrors`的地址来自于这里：

![image-20250202045404627](https://wexcdn.com/img/image-20250202045404627.png)

然后：

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

可以看到成功了：

![image-20250202045102343](https://wexcdn.com/img/image-20250202045102343.png)

并且如果你去Nexus中查看，也会看到相关内容：

![image-20250202045155915](https://wexcdn.com/img/image-20250202045155915.png)

![image-20250202045211387](https://wexcdn.com/img/image-20250202045211387.png)

其实到这步为止，你就已经能够把Nexus当作一个镜像加速器了。

但是现在还不能push。而且之前的8082和8083也没有用到。另外`daemon.json`中的`repository/docker-hub-proxy/`后缀有点丑，能不能去掉？别急，后面会回答这些问题。

#### `docker (hosted)`类型的仓库

![image-20250202050549993](https://wexcdn.com/img/image-20250202050549993.png)

这个仓库的作用就是存储一些我们构建的Docker镜像，只存在Nexus中，不传到互联网上。

#### `docker (group)`类型的仓库

![image-20250202050929844](https://wexcdn.com/img/image-20250202050929844.png)

其中重点是把`docker-hosted`和`docker-hub-proxy`加入到组中。

现在你大概就能猜到`docker (group)`仓库的含义了。没错，他就是聚合其他的仓库，使之看上去像一个仓库。

也就是`docker-hosted`和`docker-hub-proxy`中的所有内容，同样也会出现在`docker-all`中。

**所以，以后，我们所有的`docker pull`操作都应该来自于`docker-all`而不是之前的`docker-hub-proxy`。**因为，我们要pull的镜像既有可能是我们自己构建的，也有可能是来自Docker Hub的。

#### 关于端口

好的，现在回过头来说明8082和8083端口的事。

第一，8083端口（`docker-all`用）主要是为了简洁，我们现在直接使用`http://192.168.81.198:8081/repository/docker-all/`也可以。但是我们想去掉后面的路径后缀，在`daemon.json`中使用`http://192.168.81.198:8083/`。

第二，8082端口（`docker-hosted`用）是为了push。你可能已经注意到了，我们需要从`docker-all`拉取，往`docker-hosted`推送，因为前者只是一个group，一个聚合视图，而不是一个实际的仓库。（其实往group推也是可以的，但那是Nexus Pro的功能）

好，现在我们知道了需要往`docker-hosted`推，那不能直接使用`http://192.168.81.198:8081/repository/docker-hosted/`这个URL吗？这样不是省一个端口吗？

答案是：**不行**。

让我们先做个小实验，`docker login`一个不存在的地址：

![image-20250202052512320](https://wexcdn.com/img/image-20250202052512320.png)

肯定是失败了的，但失败不重要，我们要注意上下内容的变化。可以看到我们给出的是一个完整的URL`http://baidu.com:8765/abc/def/`，协议方式是HTTP，路径参数是`/abc/def/`。但是下面在请求的时候使用了`https://baidu.com:8765/v2/`，协议方式是HTTPS，路径参数是`/v2/`。

也就是说，对于我们给的URL，它只看了`主机地址:端口`这部分，其他都不管了。

所以我们`docker login http://192.168.81.198:8081/repository/docker-hosted/`这个也会失败：

![image-20250202054322927](https://wexcdn.com/img/image-20250202054322927.png)

可以看到，原有的路径参数没了，所以失败了。至于这里为啥没变成HTTPS，那是因为我们刚刚设置过`daemon.json`的`insecure-registries`了。

所以我们现在新增了一个端口，那就是8082。这样就可以使用`http://192.168.81.198:8082/v2/`了。

### 6、配置测试机并测试

在理解了以上内容后，我们把测试机的`/etc/docker/daemon.json`改成这样：

![image-20250202054754984](https://wexcdn.com/img/image-20250202054754984.png)

可以看到，其中已经没有8081的事了。

同样，重启Docker：

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

```shell
docker login 你的Nexus服务器的地址:8082
```

![image-20250202055044224](https://wexcdn.com/img/image-20250202055044224.png)

测试一下拉取：

![image-20250202055128557](https://wexcdn.com/img/image-20250202055128557.png)

编写Dockerfile并推送：

```dockerfile
FROM alpine

CMD ["echo","Hello Nexus!!!"]
```

![image-20250202055512646](https://wexcdn.com/img/image-20250202055512646.png)

可以看到`docker-all`中已经有了对应文件：

![image-20250202055655997](https://wexcdn.com/img/image-20250202055655997.png)

然后，再删除本地镜像并拉取运行：

![image-20250202055853682](https://wexcdn.com/img/image-20250202055853682.png)

## 总结

总体来说，我认为Sonatype Nexus Repository确实挺不错，功能全面而且方便。

如果未来有机会，我也会尝试把它作为Maven仓库或者NPM仓库看看。

## 参考

* [nexus3代理仓库的使用](https://www.cnblogs.com/hukey/p/18532480)


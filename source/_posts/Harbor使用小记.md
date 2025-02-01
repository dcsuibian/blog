---
title: Harbor使用小记
tags: Harbor Docker
description: Harbor（一个开源的镜像仓库）的使用记录
abbrlink: 315853384
date: 2025-02-01 23:14:46
---


# Harbor使用小记

Harbor是一个开源的镜像仓库。详情可以去[Harbor官网](https://goharbor.io/)或[GitHub项目](https://github.com/goharbor/harbor)了解更多详情，这里就不作更多介绍。

这篇文章是对我使用Harbor的过程和其中发现的坑点做一个记录。

## 目标

1. 建立一个内网私有镜像仓库
2. 缓存从Docker Hub下载的镜像，加速内网其他机器的拉取

## 缺点

### `docker pull`必须加上前缀

先说一个很重要的点，Harbor可以作为镜像代理，但在`docker pull`时必须要加上前缀（`Harbor服务器/Harbor项目名`）。详情请看[GitHub上的这个issue](https://github.com/goharbor/harbor/issues/21339)。

什么意思？

如果你以前用过阿里云的镜像加速器的话，那你一定知道，进行如下配置后：

![image-20250201202929859](https://wexcdn.com/img/image-20250201202929859.png)

你可以直接使用`docker pull xxx`来拉取Docker Hub的`xxx`镜像，其中流量会走这个镜像加速器。

但对于Harbor来说，你必须使用`docker pull Harbor服务器/Harbor项目名/xxx`才可以。

### 系统重启后很容易启动不了

详见[这个issue](https://github.com/goharbor/harbor/issues/12334)。

这时候需要`cd`进`harbor`文件夹（里面有个`docker-compose.yml`），然后`sudo docker compose restart`。

## 环境

* 操作系统：Debian 12
* Docker：最新版
* Harbor版本：v2.12.2（最新）
* Harbor文档版本：2.12.0（最新）
* 辅助工具：VSCode（带Remote - SSH插件）
* 正常的网络连接（需要用到代理的地方我会说明的）

## 过程记录

### 0、Harbor官网

首先，我们去Harbor官网：

![image-20250201204309142](https://wexcdn.com/img/image-20250201204309142.png)

其中Getting started会带你跳转到[安装文档](https://goharbor.io/docs/2.12.0/install-config/)，Download now会跳转到[GitHub的Releases页面](https://github.com/goharbor/harbor/releases)。让我们先看看安装文档（忽略掉使用k8s安装Harbor）：

![image-20250201204634100](https://wexcdn.com/img/image-20250201204634100.png)

接下来都会按照这几步走。

### 1、Harbor先决条件

详情可以自己去[对应页面](https://goharbor.io/docs/2.12.0/install-config/installation-prereqs/)看。总结一下就是硬件最小需要2 CPU、4 GB内存、40 GB存储空间，软件需要比较新的Docker Engine、Docker Compose、OpenSSL（前两个安装Docker后就会有，第三个应该是Debian自带了），网络端口需要80、443、4443（不要有其它软件占用了这几个端口，另外其实端口可以改）。

所以基本要做的就是安装Docker，[Debian安装Docker](https://docs.docker.com/engine/install/debian/)的过程我就省略了。

**这里需要先设置代理，我的建议是修改`/etc/apt/apt.conf`，添加代理内容。**具体怎么改我就不说了。

另外要将当前用户添加到`docker`用户组里：

```shell
sudo usermod -aG docker $USER
```

退出并重新登录后运行`docker ps`，看到以下内容就成功了：

![image-20250201210623683](https://wexcdn.com/img/image-20250201210623683.png)

此时我们`docker pull`应该是会失败的，因为众所周知的网络原因（我们只给`apt`开了代理，`docker pull`没有设置代理）：

![image-20250201210754293](https://wexcdn.com/img/image-20250201210754293.png)

### 2、下载Harbor

[Harbor v2.12.2](https://github.com/goharbor/harbor/releases/tag/v2.12.2)。

一定要下载offline的这个。

![image-20250201210955021](https://wexcdn.com/img/image-20250201210955021.png)

你想的话还可以下载`md5sum`验证下，我就不弄了。

然后把他传到Debian上。（你可以用`scp`命令，我就直接使用VSCode拖进去了）

我是放到了`家目录/softwares/Harbor`目录中。

![image-20250201211726802](https://wexcdn.com/img/image-20250201211726802.png)

`cd`到对应目录，然后解压：

```shell
tar -zxvf harbor-offline-installer-v2.12.2.tgz
```

完成后会出现一个`harbor`文件夹：

![image-20250201211900787](https://wexcdn.com/img/image-20250201211900787.png)

```shell
cd harbor # 进入对应文件夹
cp harbor.yml.tmpl harbor.yml # 前者是模板文件，后者才是我们真正要用的
```

### 3、修改配置文件

因为我只在内网使用，所以有所简化。并没有完全安装官方文档来。

总的来说就是改`harbor.yml`：

![image-20250201212654257](https://wexcdn.com/img/image-20250201212654257.png)

![image-20250201212800072](https://wexcdn.com/img/image-20250201212800072.png)

1. `hostname`我改成了我这台Debian的IP地址`192.168.81.198`，你应该改成你的IP或域名
2. `http`相关的设置我保持了不变
3. `https`的我全部注释掉了，因为我在局域网，不需要HTTPS
4. `harbor_admin_password`是Harbor中`admin`账户的初始密码，我没改
5. `data_volume`从`/data`改成了`/data/harbor`
6. **`proxy`里的比较重要，你需要改成你的代理服务器，因为Harbor要从Docker Hub拉镜像的话也得要代理**
7. 其他的我看不懂就没改

### 4、运行安装脚本

`cd`到`harbor`目录下，然后：

```shell
sudo ./install.sh
```

看到以下内容说明正常了：

![image-20250201213352354](https://wexcdn.com/img/image-20250201213352354.png)

然后，通过IP地址访问Harbor。如果你没改过`admin`初始密码，那密码就是`Harbor12345`。

![image-20250201213455685](https://wexcdn.com/img/image-20250201213455685.png)

![image-20250201213616932](https://wexcdn.com/img/image-20250201213616932.png)

### 5、镜像Docker Hub

点击左侧“系统管理”下的“仓库管理”，点击“新建目标”，然后如下设置：

![image-20250201213939855](https://wexcdn.com/img/image-20250201213939855.png)

其中访问ID和访问密码输入你Docker Hub的用户名和密码。（不输也可以，但是Docker官方对匿名用户拉取镜像非常严格）

点击“测试连接”，如果没问题就点“确定”。

接下来要创建项目，“项目”这个名词可能比较难以理解，他是Harbor中的一个逻辑分组单位。按我的说法他就是URL中的一段而已。请先继续看下去。

点击左侧的“项目”，然后是“新建项目”，然后这么设置：

![image-20250201215008718](https://wexcdn.com/img/image-20250201215008718.png)

待会儿你可以看到，项目名称`docker-hub`会成为URL中的一段。另外两个`-1`代表没有限制。

接着，选择左侧“系统管理”的“机器人账户”，点击“添加机器人账户”：

![image-20250201215108994](https://wexcdn.com/img/image-20250201215108994.png)

![image-20250201215132308](https://wexcdn.com/img/image-20250201215132308.png)

![image-20250201215155904](https://wexcdn.com/img/image-20250201215155904.png)

![image-20250201215219144](https://wexcdn.com/img/image-20250201215219144.png)

![image-20250201215307711](https://wexcdn.com/img/image-20250201215307711.png)

这里我偷懒给了所有权限了，因为我还没详细了解过Harbor的权限设计。

建议使用“导出到文件”，防止你忘了。另外可以注意到我们输入的名字是`test`，但最终的名字是`robot$test`。

### 6、在另一台主机上测试

我这里搭建Harbor的主机是`192.168.81.198`，然后我选择在另一台Debian主机`192.168.81.190`上测试（这台我称为Harbor客户端，Harbor客户端上也得安装Docker）。

修改Harbor客户端的`/etc/docker/daemon.json`：

```shell
sudo mkdir -p /etc/docker
sudo vi /etc/docker/daemon.json
```

![image-20250201220012570](https://wexcdn.com/img/image-20250201220012570.png)

重启Docker：

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

登录Harbor：

![image-20250201220431475](https://wexcdn.com/img/image-20250201220431475.png)

现在，我们找个镜像`pull`一下试试看：

```shell
docker pull 192.168.81.198/docker-hub/nginx:latest
```

![image-20250201224457177](https://wexcdn.com/img/image-20250201224457177.png)

可以看到`docker pull`以后对应的项目里就有了。证明他缓存了，你还可以删除镜像后再试一遍。

**你可以发现，这里不能直接`docker pull nginx:latest`，这就是我说的缺点一。**

### 7、推送镜像

要推送镜像，我们首先得建一个项目：

![image-20250201225322759](https://wexcdn.com/img/image-20250201225322759.png)

然后，在Harbor客户端上构建并推送一个简单的镜像：

![image-20250201225726371](https://wexcdn.com/img/image-20250201225726371.png)

然后就可以在项目中看到了：

![image-20250201225858521](https://wexcdn.com/img/image-20250201225858521.png)

### 8、运行推送的镜像

![image-20250201230111334](https://wexcdn.com/img/image-20250201230111334.png)

可以看到，成功了！！！（为了演示，我提前清空了Harbor客户端服务器的镜像）

## 总结

这是对我使用Harbor的一个记录，但是在写完这篇文章后我才发现：

**Harbor不太好用，有不少问题。**

我已经在考虑换一个软件了，不过如果你确定还是要用Harbor，那希望这篇文章可以帮助到你。


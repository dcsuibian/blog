---
title: JetBrains IDE设置
tags: 工具 Java JetBrains IDEA
description: JetBrains IDE设置
abbrlink: 1158072922
date: 2023-11-14 23:33:35
---

## 开门见山

### 通用设置


0. Settings -> Appearance & Behavior -> New UI（设置 -> 外观与行为 -> 新UI）。取消勾选`Enable new UI`（取消勾选`启用新UI`）。先不要重启。
1. Settings -> Appearance & Behavior -> System Settings -> HTTP Proxy（设置 -> 外观与行为 -> 系统设置 -> HTTP代理）。改成`Auto-detect proxy settings`（改成`自动检测代理设置`）。Apply应用。
2. Settings -> Plugins。安装中文插件。重启。
3. 新项目的设置 -> 编辑器 -> 代码样式。`行分隔符`选择`Unix 和 macOS (\n)`。
4. 新项目的设置 -> 编辑器 -> 文件编码。`项目编码`改成`UTF-8`。
5. 新项目的设置 -> 工具 -> 终端。`Shell路径`改成你pwsh的（如果你有装的话）。
6. 设置 -> 外观与行为 -> 系统设置。取消勾选`启动时重新打开项目`。`在以下位置打开项目`选则`新窗口`。
7. 设置 -> 编辑器 -> 常规 -> 代码补全。取消勾选`区分大小写`。
8. 设置 -> 设置同步。点击`启用设置同步...`。`在以下范围内同步设置：`选`仅XXX实例`。

### IntelliJ IDEA

1. 新项目的设置 -> 构建、执行、部署 -> 构建工具 -> Maven。`Maven主路径`选择你安装的Maven位置。

### WebStorm

1. 新项目的设置 -> 语言和框架 -> Node.js。填上`Node解释器`和`软件包管理器 (M)`。
2. 新项目的设置 -> 语言和框架 -> JavaScript --> Prettier。选择`自动Prettier配置 (A)`。

### DataGrip

1. 设置 -> 外观与行为 -> 系统设置。填上`默认项目目录`。

### 备注

上文所说的`Settings`和`设置`对应下图的1，`新项目的设置`对应下图的2。

<img src="https://wexcdn.com/img/image-20231115003803449.png" alt="image-20231115003803449" style="zoom:67%;" />

## 补充说明

JetBrains家的IDE都很好用。但要调整设置却颇为恶心。首先，在启动页面，你并不能更改设置信息。

<img src="https://wexcdn.com/img/image-20231115003440440.png" alt="image-20231115003440440" style="zoom: 50%;" />

必须等到你打开一个项目以后，才能在项目左上角的`文件 -> 设置`和`文件 -> 新建项目的设置 -> 新项目的设置`里调整设置信息。因此，我建议在初期设置时创建一个`empty-dir`，等所有设置完成以后，再删掉它。

另外，设置分成两种，`跟项目有关的`和`跟项目无关的`，`跟项目有关的`还分成`当前项目的`和`未来项目的`（即项目的默认设置）。容易出问题的地方在于：如果你从`文件 -> 设置`点进去，那么你会看到的是`跟项目无关的`设置和`当前项目的`设置混合的结果。

以更改项目编码为例，从`文件 -> 设置`和`文件 -> 新建项目的设置 -> 新项目的设置`点进去都有文件编码设置。但如果你是只是设置了前者，比如改成了UTF-8，那下次打开其他项目可能还是默认的GBK（如果你是Windows的话）。所以一定要小心。

### 清空项目级的设置

IDEA会把项目相关的设置放在对应文件夹下的`.idea`文件夹里（隐藏的）。

![image-20220819225739106](https://wexcdn.com/img/image-20220819225739106.png)

如果你要清空一个项目的设置，请

1. 关闭IDEA的当前项目窗口
2. 删除`.idea`文件夹
3. 重新打开这个项目

**如果你的项目已经配置了很多信息，或者你使用的是DataGrip，不要轻易这么做。**

## 详细内容（带图）

### 通用设置

#### 关闭新UI

我的个人喜好，我不太喜欢新的UI。

![image-20231114234234536](https://wexcdn.com/img/image-20231114234234536.png)

![image-20231115002839961](https://wexcdn.com/img/image-20231115002839961.png)

#### 开启代理

JB家IDE默认是不使用代理的，所以一定要把这个打开。如果你本来就没有代理，这个对你没有影响。但如果你有网络优化工具的话，这个选项的作用就很大了。这个是网络设置，会影响到插件的安装。**一定要设置好这个并应用后再去装插件。**

![image-20231114234458322](https://wexcdn.com/img/image-20231114234458322.png)

![image-20231115002828825](https://wexcdn.com/img/image-20231115002828825.png)

#### 装中文插件

对英文不好的同学来说，重要性不用多说了吧。

![image-20231114234719515](https://wexcdn.com/img/image-20231114234719515.png)

#### 更改行分隔符为`\n`

建议配合以下命令使用：

```shell
git config --global core.autocrlf false
git config --global core.safecrlf true
```

如果你不懂什么意思的话，百度或谷歌一下吧。

![image-20231114235156094](https://wexcdn.com/img/image-20231114235156094.png)

#### 更改字符编码为UTF-8

![image-20231114235400646](https://wexcdn.com/img/image-20231114235400646.png)

#### 更改终端

![image-20231114235609678](https://wexcdn.com/img/image-20231114235609678.png)

- 我个人会自行安装最新版本的PowerShell（目前是v7），也就是这里的`pwsh.exe`，你没安装就没有
- 老版本的IDEA可能默认使用的是cmd，非常不推荐

#### 启动时行为与项目打开行为

![image-20231115000429652](https://wexcdn.com/img/image-20231115000429652.png)

#### 代码补全不区分大小写

![image-20231115000619610](https://wexcdn.com/img/image-20231115000619610.png)

#### 开启设置同步

![image-20231115000928662](https://wexcdn.com/img/image-20231115000928662.png)

### IntelliJ IDEA

#### Maven设置

![image-20231115032906249](https://wexcdn.com/img/image-20231115032906249.png)

### WebStorm

#### Node.js设置

![image-20231115001947439](https://wexcdn.com/img/image-20231115001947439.png)

#### 自动Prettier配置

![image-20231115033724163](https://wexcdn.com/img/image-20231115033724163.png)

### DataGrip

#### 默认项目目录

![image-20231115002306722](https://wexcdn.com/img/image-20231115002306722.png)

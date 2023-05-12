---
title: SSH Cheat Sheet
tags: SSH
description: SSH常用命令和配置
date: 2023-05-07 18:13:14
---


# Cheat Sheet

1、SSH公钥验证：

```
ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key.pub
```



# SSH公钥验证

当我们第一次连接到某个SSH服务器时，往往会出现以下提示：

![image-20230507175011398](https://wexcdn.com/img/image-20230507175011398.png)

这部分的意思是：你第一次连接到这个SSH服务器，这是服务器公钥的指纹（因为完整的公钥可能会非常长），请验证下是否正确。如果一致，说明这个SSH链接是安全的，可以防止中间人的攻击。

这里可以看到，给的是ED25519的公钥，指纹算法用的是SHA256。最关键的是后面这一串字符。

![image-20230507175326201](https://wexcdn.com/img/image-20230507175326201.png)

那么，怎么确认呢？首先你得找一个肯定已经安全的方式连上此台主机：

1. 物理接触，最佳最简单
2. 如果是租借的云主机，可以从网页上连接到主机上

连上以后，`cd`到存放SSH公钥文件的文件夹，然后执行以下命令：

```shell
ssh-keygen -l -f <公钥文件>
```

默认情况下，Linux服务器的公钥文件应该存储在`/etc/ssh`中，在其中，我们重点关注的是名字中带`ed25519`和`.pub`的公钥文件，使用上面的命令进行计算。

![image-20230507175923236](https://wexcdn.com/img/image-20230507175923236.png)

可以看到，公钥的指纹是一致的，此时我们便可以放心地连接到此SSH服务器了。

Windows服务器的则是在`%PROGRAMDATA%\ssh`，另外需要管理员权限才能访问：

![image-20230507180039895](https://wexcdn.com/img/image-20230507180039895.png)

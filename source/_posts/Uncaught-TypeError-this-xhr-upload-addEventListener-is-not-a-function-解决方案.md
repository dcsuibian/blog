---
title: '''Uncaught TypeError: this.xhr.upload.addEventListener is not a function''解决方案'
date: 2021-04-22 12:57:17
tags: JavaScript
description: Mockjs对XMLHttpRequest的影响
---

# 解决方案

https://github.com/nuysoft/Mock/issues/127

我的解决方案是使用了greper提供的mockjs-x

![image-20210422130849038](images/image-20210422130849038.png)

# 问题描述

问题发生在[Ant Design Vue Pro](https://github.com/vueComponent/ant-design-vue-pro)整合[vue-simple-uploader](https://github.com/simple-uploader/vue-uploader)时。

在选择上传文件后，控制台爆出了`Uncaught TypeError: this.xhr.upload.addEventListener is not a function`错误。

![image-20210422130526879](images/image-20210422130526879.png)

![image-20210422130604237](images/image-20210422130604237.png)

#  发生时的状态

当前日期是2021/04/22。

Ant Design Vue Pro的commit记录为：2f663728bfdba21a9b89a4a4a5698ae802087b61

![image-20210422130258391](images/image-20210422130258391.png)

vue-simple-uploader的版本为：0.7.6

![image-20210422130404783](images/image-20210422130404783.png)

# 解决过程

解决方案一搜就搜到了：

[Axios + mockjs: request.upload.addEventListener is not a function 的原因和解决办法](https://blog.csdn.net/caplike/article/details/104734602)

## 查错

不过还是没搞明白为什么出错，所以自己研究下。

![image-20210422131303875](images/image-20210422131303875.png)

![image-20210422131317749](images/image-20210422131317749.png)

压缩好的代码肯定看不懂。所以去`node_modules`里找到`vue-simple-uploader`，去看它的源代码。

![image-20210422131501522](images/image-20210422131501522.png)

源代码入口是`index.js`。

![image-20210422131706514](images/image-20210422131706514.png)

![image-20210422131651789](images/image-20210422131651789.png)

可以看到，`Uploader`组件是从`./components/uploader.vue`导入的。而`uploader.vue`又是导入了`simple-uploader.js`。

去搜一下的话，发现就是`vue-simple-uploader`作者的另一个项目。

![image-20210422131858401](images/image-20210422131858401.png)3

有兴趣可以去看看。不过我们先直接把项目里用到`vue-simple-uploader`的地方换成用源代码。

```javascript
// 原来的
// import uploader from 'vue-simple-uploader'
// const Uploader=uploader.Uploader

// 使用源代码
import Uploader from 'vue-simple-uploader/src/components/uploader'
```

再次上传文件，可以看到新的报错信息。

![image-20210422132242866](https://dcsuibian-public-resources.oss-cn-hangzhou.aliyuncs.com/img/20210422132251.png)

注意，变成了chunk.js，点进去看看：

![image-20210422132328732](images/image-20210422132328732.png)

原来是`simple-uploader`里的一个`chunk.js`出错了。

## 分析

> [MockJS Ajax拦截原理](https://juejin.cn/post/6904153889163968526)
>
> ![image-20210422132723086](images/image-20210422132723086.png)

其实已经很清楚了，MockJS会覆盖原生的XMLHttpRequest，所以`chunk.js`里new的已经不是真正的XMLHttpRequest了。

![image-20210422133127851](images/image-20210422133127851.png)

而github的issue里较前面的解决方案也很清楚，无论是哪种方式都是在原型链上动手脚。

## 解决

### 第一次尝试

发现这个issue已经是2016年的了，目前Mockjs的版本为1.1.0，所以怀疑官方是不是已经解决了。

![image-20210422134609264](images/image-20210422134609264.png)

不过要注意，不知道什么原因，ant design pro vue使用的不是mock而是mockjs2。

![image-20210422134838564](images/image-20210422134838564.png)

Mockjs官方是`nuysoft`的Mock，而mockjs2是`sendya`的Mock，但是`sendya`的是从`nuysoft`的fork出来的，所以我们先尝试下换成mockjs试试。

![image-20210422135101885](images/image-20210422135101885.png)



答案是：

![image-20210422135922976](images/image-20210422135922976.png)

没有

### 第二次尝试

注意，这里是在第一次的尝试的基础上改，也就是说仍然用的mockjs而不是mockjs2。

github上说是在`node_modules/mockjs/dist/mock.js`的8308行加上这句代码（跟之前博客里看到的一样）：

```javascript
MockXMLHttpRequest.prototype.upload = xhr.upload;
```

![图片](images/aHR0cHM6Ly91cGxvYWRlci5zaGltby5pbS9mLzdEdkpKVjNKaWp3VlNGSXgucG5nIXRodW1ibmFpbA)

mockjs2的代码位置有一些变化。



<span style="color:red">不过这么做了以后，我的网页又不知道出了什么错，就一直在转圈的页面，控制台也不报错。应该是代码里出现了死循环了</span>

既然不行，我就放弃了。而且其实就算能成功，我也**不建议这么改**。

因为node_modules是不会上传到git上的，这也就意味着别人如果下载了这个项目，`npm install`后他也要去node_modules改，不太合适。

### 第三次尝试

既然官方没解决，就只能用其它人的了。

![image-20210422144322868](images/image-20210422144322868.png)

如上，`npm i mockjs-x --save`后，替换所有mockjs成mockjs-x就好。

![image-20210422144513241](images/image-20210422144513241.png)

![image-20210422144934383](images/image-20210422144934383.png)

成功！！！

# 总结

问题的出现原因就是Mockjs覆盖了XMLHttpRequest导致其它使用XHR的代码出错。



个人很感谢greper，但说实话，使用mockjs-x这种方式也不太完美。单纯为了修复一个bug去使用另一个包。这样的话，mockjs官方有更新时就比较麻烦。

准备去寻找一下其它方式。。。
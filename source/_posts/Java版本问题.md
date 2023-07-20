---
title: Java版本问题
tags: Java
description: javac的-source、-target、--release相关问题
abbrlink: 1125112739
date: 2021-05-08 23:22:47
---

# 结论

开门见山，结论如下：

* Java的`.class`文件是向后兼容的。所以如果你在JDK7或JDK6的环境下编译出了一个`.class`文件，那么它在之后的Java8、Java11、Java16甚至更以后的Java上都能跑。
* 如果你的目标平台是Java N，那么你写代码时就不应该用Java M（假设M>N）的新功能。比如，如果你的代码用了Java 11的新功能，你就不能让它运行在Java 8上。
* 用`--release`吧。

# 正文

## ①、向后兼容性

就我个人的经验，如果我需要使用一个新的jar包，那么我会直接去Maven Repository搜，搜到后添加到依赖里就可以了，没有管过jar包的作者是在Java几上面编译的。那这些jar包的作者是怎么控制兼容性的呢？

答案很简单：用低一些的JDK版本编译就好。

比如，一个简单的HelloWorld程序：

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

在Java11上编译、运行：

![image-20210509002551122](https://wexcdn.com/img/20210509162639.png)

切换到Java8后运行：

![image-20210509002828041](https://wexcdn.com/img/20210509162642.png)

失败。

但让我们仔细看看这个错误信息：

![image-20210509003010440](https://wexcdn.com/img/20210509162644.png)

原来，`.class`文件有版本的。JDK升级的时候，`.class`文件也会升级。JDK11编译出的`.class`是55.0版本，而Java8最多只能支持到52.0版本。

但相反，如果我使用JDK8进行编译，使用Java 11进行运行呢？

![image-20210509151310425](https://wexcdn.com/img/20210509162645.png)

![image-20210509151340644](https://wexcdn.com/img/20210509162647.png)

可以看到，这是没问题的。

如果你去查资料可以发现，**Java支持`binary`级别的向后兼容**：

![image-20210509010220564](https://wexcdn.com/img/20210509162648.png)

简单来说如果你在JDK7或JDK6的环境下编译出了一个`.class`文件，那么它在之后的Java8、Java11、Java16甚至更以后的Java上都能跑。我这里想强调的是**这种向后兼容性是有官方保证的**。

<br/>

事实上，我之所以可以直接导jar包，是因为我一般在使用Java 11（毕竟长期支持），而现在用的最多的却还是Java 8。

以Spring为例。下载最新的5.3.6的jar包后，查看`META-INF/MAINFEST.MF`文件可以发现，它是在JDK 8上编译的。

![image-20210509151914535](https://wexcdn.com/img/20210509162650.png)

而因为向后兼容性的存在，所以使用Java 8以上版本的人都能使用这个`jar`包。

## ②、转译

上面的情况是你作为一个框架的使用者思考的问题，而下面的问题是你作为一个框架的开发者思考的问题。

<br/>

如果你用JavaScript写过比较工程化的项目，那么你也许会知道一些工具可以把一些比较新的语法转译掉。

比方说，[Babel](https://babeljs.io/)就可以把一些ES6的代码转成ES5的代码以在老的平台上运行。

![image-20210509000716422](https://wexcdn.com/img/20210509162651.png)

那么很自然地，你也许会想，那Java有没有类似的工具呢？如果我完全不用新特性，或只用了很简单的新特性（例如Java10的`var`关键字），那么是不是就可以运行在更低版本的Java上？

这还是挺有可能发生的，例如当你下载JDK的时候你选择了较新的JDK11。写了一些工具代码想发布出去供人使用，但听说大家都偏爱Java 8，所以会想，Java能不能把我的Java11的代码转成Java8的代码，或者更直接一点——直接帮我把Java11的代码编译成适合Java 8的`.class`文件。

<br/>

<br/>

剧透一下，**不行！！！**

不过如果你真的没有用到任何新特性，那么你也不用下载JDK8，使用这个命令就行：

```shell
javac -source 8 -target 8 Main.java
```

<br/>

实际上，如果查看过`javac`的命令行文档。那么你会发现`-source`和`-target`选项。

![image-20210509153418933](https://wexcdn.com/img/20210509162653.png)

默认地，如果当前你用的是JDK11，那么`-source`和`-target`的值都会是11，很合理。

当我第一次看到这个的时候，我以为用这两个参数就能实现交叉编译。

结果我错了：

![image-20210509005402751](https://wexcdn.com/img/20210509162656.png)

这两个参数有一个要求：`-source`的值必须小于等于`-target`的值。而当你不输入其中一个参数时，另一个就默认采用当前的Java版本（这个例子中是11）。

其实也就差不多判了交叉编译的死刑了。

[stackoverflow](https://stackoverflow.com/questions/54447541/how-to-produce-code-in-java-11-but-target-java-8-and-above)上有关于这个问题的回答：

![image-20210509153932072](https://wexcdn.com/img/20210509162659.png)

简单的说，Java不支持这种交叉编译的原因是，新版本的Java中提供的一些新功能往往是确确实实地对应着`.class`文件的变化的。所以支持这种功能比较困难。

不过，如果你完全不用到任何新特性，那么你也不用去特地下载一个JDK8，只要这样就可以了：

![image-20210509154318633](https://wexcdn.com/img/20210509162700.png)

可能你已经发现了，Maven中也有对应的选项：

![image-20210509154436985](https://wexcdn.com/img/20210509162702.png)

## ③、`--release`选项

看了上面的例子，你可能会想：

`-source`的值必须小于等于`-target`的值。

当`-source`等于`-target`版本的时候，那就相当于把JDK降了版本。

那当`-source`小于`-target`版本的时候，又有什么用呢？

<br/>

在网上搜索了一段时间后，我得出了结论：基本没什么用。

 因为你想啊：

如果你指定了`-source`是8，而`-target`是11，那么你依然得按Java8来编写代码。

而且编译出来的`.class`文件也不能在Java8上面运行。

这种用法可能有一定的作用，但至少我还没碰到过需要这么写的情况。

<br/>

大部分情况下，`-source`和`-target`都是相等的，而且你可以发现，这俩总是如影随行。

因为只写`-source 8`相当于`-source 8 -target 11`，干脆不写就好。

只写`-target 8`相当于`-source 11 -target 8`，这就报错了呀。

<br/>

Java官方也发现了这个问题，所以它们推出了：`--release`。（貌似是从Java9出现的）

![image-20210509160017409](https://wexcdn.com/img/20210509170927.png)

查看官方的文档可以发现，现在用`--release N`就可以替代`-source N -target N`，

但也不是完全相同，可以看到还有两个`-bootclasspath`和`--system`参数。

不过这就超过这篇文章的范围了（其实是我也不太清楚有什么区别，但也懒得查）。

<br/>

Maven中的对应配置：

![image-20210509160705247](https://wexcdn.com/img/20210509162704.png)

# 多余的话

我对Java的版本问题挺满意的。

不用担心以前的jar包不能用，可以让开发者比较舒服地过渡到新版本。

又避免了交叉编译的复杂性。

真的感觉恰到好处。

<br/>

一定要对比（黑）一下JavaScript。（这篇文章主要就是受它的影响）

相对于Java来说，JavaScript的历史包袱真的好重。

最主要的原因：浏览器是JavaScript的主战场。开发者不能确定这些程序会在什么样古老的浏览器上运行。因此即使ES6已经推出好几年了，仍然需要用Babel等工具转译成ES5的，增加了配置的复杂性（真的挺复杂的，而且转译也不太完美）。

Java注重于后端，相比来说，开发者就更舒服一点。用了新的特性，就得要新的运行时环境。要不你就别用。

我个人更喜欢Java这种，大多数开发人员应该也是——Node.js就是个例子。

# 参考资料

1. [JEP 247: Compile for Older Platform Versions](http://openjdk.java.net/jeps/247)
2. [Upgrading major Java versions](https://blogs.oracle.com/java-platform-group/upgrading-major-java-versions)
3. [How to produce code in Java 11, but target Java 8 and above?](https://stackoverflow.com/questions/54447541/how-to-produce-code-in-java-11-but-target-java-8-and-above)
4. [如何使用Javac的source参数](https://www.sunmoonblog.com/2018/08/27/javac-source/)
5. [javac中的source和target的区别](https://segmentfault.com/q/1010000002959346)

<br/>

<br/>


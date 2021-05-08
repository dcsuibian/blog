---
title: Java版本问题
date: 2021-05-08 23:22:47
tags: Java
description: javac的-source、-target、--release相关问题
---

# 结论

开门见山，结论如下：

* Java的`.class`文件是向后兼容的。所以如果你在JDK7或JDK6的环境下编译出了一个`.class`文件，那么它在之后的Java8、Java11、Java16甚至更以后的Java上都能跑。
* 如果你的目标平台是Java N，那么你写代码时就不应该用Java M（假设M>N）的新功能。比如，如果你的代码用了Java 11的新功能，你就不能让它运行在Java 8上。
* 用`--release`吧。

# 前言

其实对Java的版本问题出现好奇心是在用了其它语言之后。

因此接下来的介绍过程中，会引入一些其它语言的对比。

# 正文

## ①、转译？

如果你用JavaScript写过比较工程化的项目，那么你也许会知道一些工具可以把一些比较新的语法转译掉。

比方说，[Babel](https://babeljs.io/)就可以把一些ES6的代码转成ES5的代码以在老的平台上运行。

![image-20210509000716422](images/image-20210509000716422.png)

那么很自然地，你也许会想，那Java有没有类似的工具呢？如果我只用了新特性的一小部分，或者完全不用，那么是不是就可以运行在更低版本的Java上？

这还是挺有可能发生的，例如当你下载JDK的时候你选择了较新的JDK11。写了一些工具代码想发布出去供人使用，但听说大家都偏爱Java 8，看了看自己的代码，发现没用到什么新特性。准备试一试。

比如，一个简单的HelloWorld程序：

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

在Java11上编译、运行：

![image-20210509002551122](images/image-20210509002551122.png)

切换到Java8后：

![image-20210509002828041](images/image-20210509002828041.png)

失败。

但让我们仔细看看这个错误信息：

![image-20210509003010440](images/image-20210509003010440.png)

原来，`.class`文件有版本的。JDK升级的时候，`.class`文件也会升级。JDK11编译出的`.class`是55.0版本，而Java8最多只能支持到52.0版本。

## ②、`-source`、`-target`选项

其实上面的例子的解答很简单，使用如下命令就好：

```bash
javac -source 8 -target 8 Main.java
```

在Java11上编译、运行：

![image-20210509003847555](images/image-20210509003847555.png)

在Java8上运行：

![image-20210509003957220](images/image-20210509003957220.png)

Ok，没问题，现在我不用下载JDK8，只要增加两个参数就好（可以看到在Java11上运行也没可以，这里就涉及到一点向后兼容性了，我们稍后再谈）



但是，但是，当你用了一段时间后，你可能会发现，这个`-source`和`-target`总是如影随行、如胶似漆。

默认地，如果当前你用的是JDK11，那么`-source`和`-target`的值都会是11，很合理。

但你无法去掉其中的任何一个。

如果你去掉了`-target`，那么没有意义，因为你是要放到更低级别的目标上。（实际测试的话，即使你这么做了，编译器也只会把代码视作是Java8的，而仍然编译成Java11对应的`.class`文件）

那么如果你去掉了`-source`呢？

![image-20210509005402751](images/image-20210509005402751.png)

可以看到，没有生成`.class`文件，编译失败。编译器认为`-source`是11，而目标版本是8，不允许这么操作。

这也就判了交叉编译的死刑了，即使你没用到Java11的新特性，只要编译器认为代码是Java11的，就不允许这么操作。再说万一如果你真的用了一些呢。。。

## ③ 暂时总结

实际上，如果你去查资料可以发现，Java支持`binary`级别的向后兼容：

![image-20210509010220564](images/image-20210509010220564.png)

所以如果你在JDK7或JDK6的环境下编译出了一个`.class`文件，那么它在之后的Java8、Java11、Java16甚至更以后的Java上都能跑。

它这里还提到了`Compatibility Guide`，不过可以看到，`Binary Compatibility`里并没有什么内容。

![image-20210509010431866](images/image-20210509010431866.png)

你也可以在[stackoverflow](https://stackoverflow.com/questions/54447541/how-to-produce-code-in-java-11-but-target-java-8-and-above)上找到为什么这样是不行的：![image-20210509010734128](images/image-20210509010734128.png)

其中，Holger的回答就是在JDK升级的过程中，`.class`文件也升级了，JDK11的一些新功能确确实实会对应`.class`文件的变化，所以很难生成适用于Java8的`.class`文件。

而我认为Buurman的回答更具合理性，他说：如果你没有用到任何Java11的新功能，那么你的源代码基本上本来就可以算是Java8的代码了。

# 参考资料

1. [JEP 247: Compile for Older Platform Versions](http://openjdk.java.net/jeps/247)

2. [Upgrading major Java versions](https://blogs.oracle.com/java-platform-group/upgrading-major-java-versions)

3. [How to produce code in Java 11, but target Java 8 and above?](https://stackoverflow.com/questions/54447541/how-to-produce-code-in-java-11-but-target-java-8-and-above)


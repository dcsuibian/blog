---
title: 浅谈CSS像素、视口及移动端适配
tags: CSS
---

在学习CSS的初期，CSS像素、物理像素、布局视口、可见视口等诸多概念总是弄得我头晕。因此写下这篇文章，给新人们讲一讲我自己的理解。

一上来先讲概念肯定很让人困惑，因此我们先来讲一讲这些概念是怎么来的，才能让大家有个更清晰的认识。

# 历史

我相信，**像素（pixel）**、**分辨率（resolution）**、**像素密度(pixel density)**的概念大家应该都不陌生，这里就不过多解释了。

![设备分辨率](./assets/r7uWo.gif)



## 低ppi时代

在Web技术的早期，网页直接用“像素”为单位其实是合适的。那时候各种设备的分辨率普遍不高，或者分辨率大时尺寸也大，各种显示屏的ppi数值**在一个级别**。因此，早期的网页就使用“px”作为布局排版的单位，这时候也就没有“CSS像素”的概念，实际就都是物理像素。

<br/>

一个具体的情景：

一块1024 x 768（4:3）的显示屏，宽27cm，高20cm。那么它的像素密度就是96ppi，大概就是1cm可以放37个物理像素。

这时候，如果一个字的大小为“16px”，那么每个字就大约就0.43cm宽，这时候字的大小适中，人眼大小能够较好地分辨。

<br/>

## 高ppi时代

随着技术的发展，显示屏分辨率也变得越来越高，而屏幕尺寸并不会变得那么大，这就导致了设备的像素密度变大了很多，很多不同设备的ppi已经**不在一个级别**了。

例如，如果还是这个尺寸的屏幕，但分辨率两个轴上各翻了一倍，达到了2048 x 1536。这时候ppi就变成了192，而原来“16px”的字就只有0.21cm宽了。很明显，字太小了，不适合阅读了。

<br/>

然而，许许多多的网页已经使用了“px”作为单位了，我们也就不能随意地更换。因此，为了保持兼容性。就出现了**CSS像素（CSS pixel）**这个东西。简单地说，就是做一个转换，将“1px”对应变换成多个物理像素。

而这个变换的比率，则由设备状态、操作系统、浏览器等综合决定。

> 打开浏览器的控制台，输入
> ```javascript
> window.devicePixelRatio
> ```
>
> 你就可以看到这个比率：
>
> ![image-20210827110740199](./assets/image-20210827110740199.png)
>
> “2”即代表在我的浏览器上一个CSS像素会变换成2个物理像素。（一个维度的）

对于高ppi的设备来说，只要更改对应的比率就好，这样在不同ppi的屏幕上，显示的网页就不至于过分小。

这里要总结两点：

* **不用顾虑，一般我们说“像素”，那么就是指“CSS像素”**

不管是CSS、JavaScript的API还是网上看到的各种教程，在Web领域，不带前缀的“像素”就是指“CSS像素”。不要想太多，即使是`window.devicePixelRatio`也不是给你物理像素数，而是教你怎么换算。

* 当设备的ppi接近时，`window.devicePixelRatio`往往一样。



## 移动端时代

我们知道，现在的手机分辨率都是很高的，但尺寸又很小，这就导致了它们的`devicePixelRatio`很大。

点击桌面浏览器控制台的左上角，可以模拟移动设备：

![image-20220209223330904](./assets/image-20220209223330904.png)

以iPhone SE为例：

<img src="./assets/image-20220209224428506.png" alt="image-20220209224428506" style="zoom:33%;" />

<img src="./assets/image-20220209223533229.png" alt="image-20220209223533229" style="zoom:33%;" />

可以看到，iPhone SE有着326的高ppi，因此`devicePixelRatio`也大。只有。对于手机这么大的设备来说，给375 x 667的CSS像素尺寸是对的。

<br/>

但问题在于：**手机的尺寸太小了**。（用CSS像素衡量）

针对电脑设计的网页，往往都会假定设备的宽度有600px以上，很明显，对于这些网页，手机端的显示就会有问题。

于是，手机浏览器就采用了一个方法，假装自己有约1000px那么宽。因此，显示的效果就如下：

<img src="./assets/image-20220209225932169.png" alt="image-20220209225932169" style="zoom:33%;" />

看起来就像是用电脑访问一样，但完全看不清楚字，需要缩放。对于手机用户来说，虽然麻烦点，但还是可以用的。





























































# 归档

CSS中的像素是以96ppi确定的。(CSS2的时代是90，2.1以后就是96了)。96ppi是个什么概念？就是1英寸96个物理像素点。我们这里不使用英寸，换算一下大概就是1厘米37个物理像素点这样。这是一个例子：

![image-20210827104918790](./assets/image-20210827104918790.png)

上面相当于在说如果有一块屏幕，分辨率为1024x768，宽27厘米，高20厘米，那么它就是一块96ppi的屏幕，每个物理像素点占1/96英寸（或1/37厘米）这么大。在这样的屏幕上，CSS像素和物理像素是1对1的关系。在这么一个设备上，你操控CSS像素就相当于直接操控物理像素了，不用做什么计算。

<br/>

但是，如果换做更加高清的屏幕呢？想想看，有一个同样宽27厘米，高20厘米，但分辨率为2048x1536的屏幕。这时候如果CSS像素和物理像素仍然保持着1对1的关系，那么大部分内容都会变得很小，导致难以看清。比如说，如果你原来设置的字体大小是16像素，大概会显示成0.43厘米，而现在则只有0.21厘米，就会看得十分难受。因此：

![image-20210827105910313](./assets/image-20210827105910313.png)

<br/>

这个所谓的“用户代理”一般就是浏览器。也就是说，如果屏幕的物理像素密度与96ppi相差特别大，浏览器就会调整CSS像素与物理像素的比值。

不过要注意，是**差距特别大**时才会调整。比如，1块24寸的1080p屏幕，它的ppi是92。虽然和标准不太一样，但浏览器基本上仍然会使用1CSS像素对应1物理像素这样的关系。而1块27英寸的4k屏幕，那ppi就是163，这个差距就大了，浏览器就会进行一定的调整。

如果想查看你浏览器的调整设置，可以在控制台中输入：



查看。



例如，我的4k屏幕是2，2k屏幕是1.25，1080p的则是1。

回到一开始iphoneX的例子上：

![image-20210827110926263](./assets/image-20210827110926263.png)

可以看到，是3（1125是375的3倍，而2436是812的3倍），这就说明了CSS像素和物理像素的对应关系。

也就是说，当你在代码里写了`font-size:16px`后，在iphoneX上运行，那么一个字的大小差不多是48个物理像素，对于458ppi的iphoneX来说，就是0.27cm那么大。（算出来才发现好像有点小啊，但如果CSS像素和物理像素意义对应，那应该只有0.08cm那么大）

总之CSS像素就像是一个中间层，避免过高ppi对原有的网页产生过于巨大的影响。

## 阶段总结

### 1、 当我遇到一个“像素”的时候，我怎么知道它是CSS像素还是物理像素呢？

实际上，尽管JavaScript有`window.devicePixelRatio`这种与设备物理像素有关的接口。

但其大部分api，offsetX、offsetY之类的返回的都是CSS像素。

也就是说一般你遇到的和“像素”有关的JavaScript、CSS啥的都是指CSS像素。

### 2、如果进行了缩放，那不会糊吗？

如果缩放就会糊的话，那还要买高清显示器干什么，实际上，虽然我们操作的是CSS像素，但浏览器具体显示的时候还是用物理像素。

举个例子吧，如果要在网页上显示矢量图的一条线：

![image-20210827120442151](./assets/image-20210827120442151.png)

那么在1080p的屏幕里，大概就是这样的：

![image-20210827120529045](./assets/image-20210827120529045.png)

那么现在换成4k的呢？不应该是这样：

![image-20210827120831560](./assets/image-20210827120831560.png)

因为如果直接用4个物理像素显示1个CSS像素的话，用户就看不出区别了。

所以应该是这样

![image-20210827120947365](./assets/image-20210827120947365.png)

可以看到，更加精细，宏观上的话，就会显示更少的锯齿感。

其它的东西，图片、文字等也都会更好，因为它们还是看物理像素的。

# 图片

如果只是在网页里用像素这个单位的话，那其实没有什么要注意的。

我担心的就是甲方那些作图的人，他们用的作图的软件应该跟网页没有什么关系，可能就是PhotoShtop、Cad之类的。所以直接说“像素”可能不太好，还是跟他们说比例最好。

![image-20210827124318424](./assets/image-20210827124318424.png)

比如，现在如果要在网页上显示完整图片的话，那图片大小最大应该是2208px左右，而字体最小只有12px。

那么它们给的图片的中字体的大小应该就是图片宽度的12/2208=0.5%这么大。

（举个例子而已，我们要适配1080p的屏幕（可能还要小），那这个还得重新算一下，而且对字体大小的定义也有一定影响。）

<br/>

而如果要导出位图的话，就是越清晰越好，分辨率越大越好（如果不考虑文件大小的话）。

比如导出分辨率为16000x9000的图片，字体应该最小是1600x5%=80吧。


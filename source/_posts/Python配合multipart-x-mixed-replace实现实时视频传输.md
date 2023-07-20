---
title: Python配合multipart/x-mixed-replace实现实时视频传输
tags: Python
description: 用python程序来解析multipart/x-mixed-replace内容
abbrlink: 583835402
date: 2021-03-19 15:23:54
---
# 参考链接

* [使用 multipart/x-mixed-replace 实现 http 实时视频流](https://segmentfault.com/a/1190000018563132)，关于`multipart/x-mixed-replace`的介绍，实现用的是JavaScript。
* [Video Streaming with Flask](https://blog.miguelgrinberg.com/post/video-streaming-with-flask)，Python使用Flask来进行推流的相关介绍。

# 基本使用

这部分可以直接看[Video Streaming with Flask](https://blog.miguelgrinberg.com/post/video-streaming-with-flask)的内容，我只是进行了一定魔改。

安装flask与opencv-python

```bash
pip install opencv-python flask
```

建立如下目录结构：

![image-20210320123717627](https://wexcdn.com/img/20210320124309.png)

`app.py`内容：

```python
import cv2
from flask import Flask, render_template, Response

app = Flask(__name__,
            static_url_path='/')  # 设置静态文件路径（我不准备用模板，所以HTML也是放在static文件夹中）
camera = cv2.VideoCapture(0)  # 初始化摄像头


@app.route('/')
def index():
    return app.send_static_file('index.html')


def gen():
    while True:
        ret, frame = camera.read()  # 摄像头读取一帧
        frame = cv2.imencode('.jpg', frame)[
            1].tobytes()  # opencv存储的图片数据不能用，所以需要进行转码
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')


@app.route('/video_feed')
def video_feed():
    return Response(gen(),
                    mimetype='multipart/x-mixed-replace; boundary=frame')


if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)

```

index.html内容：

```html
<html>
  <head>
    <title>Video Streaming Demonstration</title>
  </head>
  <body>
    <h1>Video Streaming Demonstration</h1>
    <img src="/video_feed">
  </body>
</html>
```

启动应用程序，打开网页就能看到视频传输成功：

![image-20210320124124702](https://wexcdn.com/img/20210320124310.png)

我的更改主要是使用了静态HTML文件，以摄像头读取的内容作为视频源，其它没什么。

能看到视频就说明成功了。

# 简要分析

目前来说，我们已经实现了一个简简单单的视频传输流程。但如果只是这样，只要看参考链接就好。

接下来我们要实现不使用浏览器来解析请求。

<br/>

不过在此之前，让我们分析下这个东西的原理。

让我们看看Flask的日志：

![image-20210320124821978](https://wexcdn.com/img/20210320141100.png)

再看看MIME类型中的`multipart`。我们可以知道，整个视频传输流程都只是在一次HTTP请求中完成的。再看看MIME类型中的`replace`，我们大概可以推断出整个视频传输过程的原理。

与普通的HTTP请求类似，浏览器向`/video_feed`发出请求，浏览器返回内容。

不过特殊的地方是浏览器的响应报文，响应报文应该仍然是一个响应头（其中指明了`multipart/x-mixed-replace`的返回链接），但由`multipart`我们可以知道，响应报文的body部分是由多部分组成的，只要连接不断开，服务器就会往body里面不断增添新的内容。

而浏览器接收到这么多内容是怎么做的呢？注意看`x-mixed-replace`中的`replace`，其实它是把最新拿到的一部分数据替换掉以前的数据，在我们的例子里就是用新的一帧图像替换掉前一帧图像。

也就是说，`multipart/x-mixed-replace`只是告诉浏览器响应body中会不断追加数据，且请浏览器用新的一部分数据替换原来的。而浏览器又通过另一部分`Content-Type: image/jpeg`标记来知道每一部分的内容是图像的。

<br/>

总结：也就是说服务端得到的视频会被转换成一张张jpeg图片，传给浏览器。浏览器不断用新的图片替换以前的图片，就实现了视频的效果。这也就是为什么没法传输音频的原因了。

# HTTP内容解析

理解了以上内容，那就可以进行下一步的操作了。

目前来说，浏览器自动帮我们解析了HTTP请求，我们做的只是将`<img>`标签的链接指向它。

```html
<img src="/video_feed">
```

但我现在有一个需求，如果我想在其它应用程序中使用这种视频流方式该怎么办呢？

很明显，我们也必须实现对这种MIME类型的解析。

不过以我目前搜到的资料来看，好像没有什么现成的库实现了这个功能。

所以我就自己写了一个，不过需要先安装`requests`库。

```bash
pip install requests
```

客户端代码：

```python
import cv2
import numpy as np
import requests

url = 'http://127.0.0.1:5000/video_feed'
res = requests.get(url, stream=True)  # steam=True不能少
bytes = b'\r\n'  # 目前收到的二进制内容
cst = b'\r\n--frame\r\nContent-Type: image/jpeg\r\n\r\n'
now = 0
next = -1
for chunk in res.iter_content(chunk_size=1024):
    bytes += chunk
    next = bytes.find(cst, now + 1)
    if -1 != next:  # 说明有新的一帧到了
        bin_data = bytes[now + len(cst):next]
        image = cv2.imdecode(np.frombuffer(bin_data, np.uint8),
                             cv2.IMREAD_UNCHANGED)
        cv2.imshow('frame', image)  # 只是为了显示
        cv2.waitKey(1)
        bytes = bytes[next:]
        now = 0
        next = -1
res.close()
```

写的很随意，我们知道请求体里面的内容大概会是`--frame\r\nContent-Type: image/jpeg\r\n\r\n二进制图片内容\r\n--frame\r\nContent-Type: image/jpeg\r\n\r\n二进制图片内容\r\n--frame\r\nContent-Type: image/jpeg\r\n\r\n二进制图片内容\r\n--frame\r\nContent-Type: image/jpeg\r\n\r\n二进制图片内容`······

可以看到，每一部分的分隔符可以是`\r\n--frame\r\nContent-Type: image/jpeg\r\n\r\n`，所以我们可以通过检测这个来分析哪里是下一帧。（最开始的`--frame\r\nContent-Type: image/jpeg\r\n\r\n`前没有`\r\n`，但我们为了方便起见，就在`bytes`里初始化了一个`\r\n`）

<br/>

`bin_data`的内容就是图片的二进制内容了，`numpy`和`cv2`只是为了将它显示出来，如果你不需要，可以删掉。

<br/>

优点：这种解析方法大概在其它语言里也能用。

缺陷：这个程序是看到下一个`\r\n--frame\r\nContent-Type: image/jpeg\r\n\r\n`才知道前一帧的结束位置。所以最后一帧恐怕就没有处理。而且虽然这样写我测试成功了，但我仍然觉得这样写不太好（因为我对HTTP不是很了解，所以可能会有不知道的坑点存在）。

# 更进一步

目前来说，已经能实现视频的传输了，不过还有两个问题：

* 代码难以编写，其它代码很难和这部分结合
* 存在一个目前还没有提到的累计问题

<br/>

先说第二个问题，如果你把客户端代码中的`cv2.waitKey(1)`改成`cv2.waitKey(1000)`，就会看到这个问题，视频基本上是以一种极慢速的方式播放的。

这是因为服务端的代码根本就没有考虑客户端的接收速率，它只是单纯地从摄像头采集一帧，然后塞到响应body里。它的发送速率主要是取决于读取一帧所需要的时间和网络传输所需要的时间。

```python
def gen():
    while True:
        ret, frame = camera.read()  # 摄像头读取一帧
        frame = cv2.imencode('.jpg', frame)[
            1].tobytes()  # opencv存储的图片数据不能用，所以需要进行转码
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')
```

所以，最重要的就是你的应用程序必须能够及时处理，否则你得到的画面可能就不是实时的了。针对这一点，我目前采用了多线程的解决方案：

http_camera.py：

```python
import threading
import time

import cv2
import numpy as np
import requests

from device.camera import Camera


class HttpCamera(Camera):
    def __init__(self, url):
        self.__url = url
        self.__frame = None
        self.__last_frame = None
        self.__running_flag = False  # 代表还在不在运行。

    def isOpened(self):
        return self.__running_flag

    def open(self):
        if self.isOpened():  # 如果已经open过了，第二次open就直接忽略。
            return
        response = requests.get(self.__url,
                                stream=True)  # 向目标url请求

        def tmp():
            bytes = b'\r\n'
            cst = b'\r\n--frame\r\nContent-Type: image/jpeg\r\n\r\n'
            now_position = 0  # 这一帧的开始位置
            try:
                for chunk in response.iter_content(chunk_size=1024):
                    if not self.__running_flag:  # 如果在连接过程中用户close了，那就不用再执行了。
                        break
                    bytes += chunk
                    next_position = bytes.find(cst, now_position + 1)
                    if -1 != next_position:
                        bin_data = bytes[now_position + len(
                            cst):next_position]  # 截取出图片的二进制数据
                        self.__frame = cv2.imdecode(
                            np.frombuffer(bin_data, np.uint8),
                            cv2.IMREAD_UNCHANGED)
                        bytes = bytes[next_position:]
                        now_position = 0
                response.close()  # 释放连接。
            except Exception:
                self.__running_flag = False
                raise Exception('服务端结束了视频')

        self.__running_flag = True
        threading.Thread(target=tmp).start()
        return

    def read(self):
        if not self.isOpened():
            self.open()
        while True:
            if self.__frame is not None and id(self.__frame) != id(
                    self.__last_frame):
                self.__last_frame = self.__frame
                return True, self.__frame
            else:
                time.sleep(0.001)  # 让出CPU

    def release(self):
        self.__running_flag = False
        return


if '__main__' == __name__:
    camera = HttpCamera('http://127.0.0.1:5000/video_feed')
    while True:
        ret, image = camera.read()
        cv2.imshow('image', image)
        k = cv2.waitKey(1) & 0xFF
        if 27 == k:
            break
    cv2.destroyAllWindows()
    camera.release()
```

模仿了`cv2.VideoCapture(X)`的接口，提供了`open`，`isOpened`，`read`和`release`的接口，感觉上就像一个普通的摄像头一样。

每次调用`read`方法得到的都是最新的视频帧。

# 总结

在本机上进行了测试，使用这种视频流方法的延迟平均在154ms左右，而rtmp的延迟则在700ms以上，总体来说还是较好地满足了我的需求。

缺陷：

* 我自己对自己的代码不太满意，不知道有没有更好的写法。
* 不是很喜欢用python的多线程（个人看法）。
* 没有很好地处理http连接发生错误的情况。
* 解决方案本身缺陷，不是专业的视频推流协议，而且使用了`multipart/x-`的实验性特性，可能存在某些坑点。

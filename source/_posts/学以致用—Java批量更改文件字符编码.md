---
title: 学以致用——Java批量更改文件字符编码
date: 2021-05-01 19:45:32
tags: Java
description: 批量编码转换
---

# 代码

参考：疯狂Java讲义（第五版）

```java
import java.io.IOException;
import java.nio.charset.Charset;
import java.nio.file.Files;
import java.nio.file.Path;

public class Main {
    public static void main(String[] args) throws IOException {
        // 我的文件夹位置
        String codes = "C:/Users/dcsuibian/study/疯狂Java讲义/codes";
        Files.walk(Path.of(codes))
                // 过滤掉所有非 .java 文件
                .filter(path -> path.toString().endsWith(".java"))
                .forEach(path -> {
                    try {
                        String s = Files.readString(path, Charset.forName("GBK"));
                        Files.writeString(path, s, Charset.forName("UTF-8"));
                    } catch (IOException e) {
                        System.out.println("出错的文件：" + path);
                        e.printStackTrace();
                    }
                });
    }
}
```

# 效果

运行前：

![image-20210501195024201](https://wexcdn.com/img/20210501195301.png)

![image-20210501195059993](https://wexcdn.com/img/20210501195304.png)

<br/>



运行后：

![image-20210501195133387](https://wexcdn.com/img/20210501195306.png)

成功！！！
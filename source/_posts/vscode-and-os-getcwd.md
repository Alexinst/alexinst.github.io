---
title: VSCode 工作目录和 os.getcwd() 之间的冲突
date: 2018-05-27 23:08:28
tags:
    - VScode
    - Python
    - os.getcwd()
categories:
    - Python
---

**简述**： `VScode`用起来觉得不错，相较之下，`Sublime Text3`对中文字符串的支持就不怎么样了。不过`VSCode`也存在一个问题：`VSCode`工作目录和函数 `os.getcwd()` 有冲突。

<!-- more -->
<br />

# 问题
　　我发现在`VScode`中写`Python`代码，若是有调用 `os.getcwd()` ，调用结果将可能存在错误，因为这个函数和`VSCode`的工作目录存在冲突。

## 具体情况
```Python
import os
print(os.getcwd())
```

　　比如，`VSCode` 的工作目录为 `F:\Python` ，而代码如上的`.py`程序文件存放路径为 `F:\Python\Python_work` 时，执行程序的输出结果是：`F:\Python`。


## 解决办法

　　经过`Google`，我在 CSDN 博主[ljzhang](https://blog.csdn.net/leorx01)的一篇[博文](ihttps://blog.csdn.net/leorx01/article/details/71141643)中找到了解决办法。

　　将程序对应代码改成如下，就能够获取正确的当前目录路径。

```Python
import os
print(os.path.dirname(__file__))
或者
print(os.path.abspath(os.path.dirname(__file__)))
```

　　产生这种情况的具体原因是什么，我还不清楚。以后在 `VScode`写 `Python` , 只能避免使用函数 `os.getcwd()` 了。

　　以上！





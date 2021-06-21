---
title: 搭建简易 web 服务器过程中的问题
date: 2018-05-27 09:49:47
tags:
    - Python
    - Web Server
    - HTTP
    - 存疑
categories:
    - Python
---

**简述**： [**_500 lines or less_**](https://github.com/aosabook/500lines) 是一本在Github上开源的Python教程书籍，内容包含有18个Python Project。正如书名，都是低于500行代码的Project。

　　不过，项目是基于`Python2`的，而我自学的是`Python3`，因此编写过程中遇到了2个问题。

<br /><!-- more -->

# 1 库名变化
## 1.1 问题

　　原Python2程序中导入了库 `BaseHTTPServer` ，但在Python3中，该库已经改为 [`http.server`](https://docs.python.org/3/library/http.server.html) 。
## 1.2 解决办法

　　~~`import BaseHTTPServer`~~  ---> `import http.server`

　　就是这样，而已。

<br />
# 2 网页编码

## 2.1 问题

　　原Python2程序中有一段类似如下的html代码， 

```html
Page = """\
    <html>
    <body>
    <p>Hello, web!</p>
    </body>
    </html>
    """
```

　　但运行时出现 `TypeError: a bytes-like object is required, not 'str'`。

## 2.2 解决办法

　　解决办法有两种，一种是使用前缀 `b` 将str转化为bytes；另一种是使用 `str.encode('utf-8')`。
```html
Page = b"""\
    ...all the other code...
    """
或者
Page = """\
    ...all the other code...
    """.encode('utf-8')
```

　　我在StackOverFlow上找到[相关解释](https://stackoverflow.com/questions/42612002/python-sockets-error-typeerror-a-bytes-like-object-is-required-not-str-with?noredirect=1&lq=1)，大概意思是：在Python3中字符串是Unicode形式，但在网络中，数据应该是以字节的形式传输。

>The reason for this error is that in Python 3, strings are Unicode, but when transmitting on the network, the data needs to be bytes strings instead. So... a couple of suggestions:
>
>......
>
>Best solution:
```python
output = 'Thank you for connecting'
c.sendall(output.encode('utf-8'))
```

<br />以上！




















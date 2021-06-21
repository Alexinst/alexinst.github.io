---
title: 「PyQt5」1. QtWidgets 模块中 Class 的通用函数/方法

date: 2018-06-17 09:00:31

tags:
    - python
    - pyqt5
    - common methods
    - gui
categories:
    - Python
    - PyQt5
---

**简述**：`PyQt5` 是 `Python` 和 `Qt` 库的融合，用来开发 GUI 应用程序。本文记录了一些 `PyQt5.QtWidgets` 中部分 `Class` 的少数函数/方法。

<!-- more -->
<br />

# 1 学习文档

- [PyQt5中文教程](http://code.py40.com/pyqt5/14.html)：这是[PyQt5 Tutorials](https://pythonspot.com/en/pyqt5/)的翻译，可用于入门。但只是初级的教程，而且使用的图床大概是QQ相册，许多图片无法显示。
- [Qt for Python](https://doc.qt.io/qtforpython/index.html): 顾名思义，这是Python的PyQt5官方文档，但对于类的函数或方法的解释太简单，有些鸡肋。
- [Qt Documentation for C++](https://doc.qt.io/qt-5/qtmodules.html)：这是用于C++的PyQt5.11文档，文字解释很具体，但并不能直接应用于Python。

<br />

# 2 QtWidgets

　　`QtWidgets`模块提供了一套创造经典桌面风格的用户界面的`UI Class`，包含有：
- `QWidget`：
- `QPushButton`：按键
- `QComboBox`：多选框
- `QLabel`：标签
- `QLineEdit`、`QTextEdit`：单行文本框和多行文本框
- ......

## 2.1 控件Text

```Python
<CONTORL_NAME>.setText('...')  # 设置控件Text
text = <CONTORL_NAME>.Text()   # 提取控件Text
```

　　`<CONTROL_NAME>`指的是控件`Class`的`Instance`名。

## 2.2 控件可用性

```Python
<CONTROL_NAME>.setEnabled(True)
<CONTROL_NAME>.setEnabled(False)
```

## 2.3 Text在控件中的位置

```Python
from PyQt5.QtCore import Qt
...


class window(QWidget):
    def __init__(self):
        ...
    
    def initUI(self):
        self.lbl = QLabel()
        set.lbl.setAlignment(Qt.AlignCenter)
        # or set.lbl.setStyleSheet("qproperty-alignment: 'AlignCenter';")
...
```
　　以`QLabel`为例，方法`setAlignment`的参数可以有：
- `Qt.AlignLeft`
- `Qt.AlignCenter`
- `Qt.AlignRight`
- `Qt.AlignTop`：顶部
- `Qt.AlignBottom`：底部
- ......

<br />
**以上！**
<br />

  

  

  

  

  

  
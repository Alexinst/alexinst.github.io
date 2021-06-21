---
title: 「PyQt5」2. QPushButton 的 signals 和 slots

date: 2018-06-17 09:53:51

tags:
    - python
    - pyqt5
    - qpushbutton
    - signal
    - slot
    - event-driven

categories:
    - Python
    - PyQt5
---

**简述**：`Graphical Application`是事件驱动型的，和控制台或终端程序不同。所谓事件，就是诸如点击按钮、选择多选框中一条选项这样的用户行为。本文记录了`QPushButton`的`signals`和`slots`用法。

<!-- more -->
<br />

# 1 名词解释

- `signal`翻译为信号，指事件发生时`PyQt5 widget`发送的信号。
- `slot`：翻译为槽（奇奇怪怪），是任何可调用的函数或方法。

　　对于`QPushButton`来说，`signal`是点击按钮时`QPushButton`发送的`Clicked()`信号，`slot`可以是自己编写的可调用函数/方法，也可以是`PyQt5`自有的函数/方法。

# 2 QPushButton的signals

　　[`QPushButton`](https://doc.qt.io/qtforpython/PySide2/QtWidgets/QPushButton.html?highlight=qpushbutton)具有如下signal：
- [`clicked()`](https://doc.qt.io/qtforpython/PySide2/QtWidgets/QAbstractButton.html#PySide2.QtWidgets.PySide2.QtWidgets.QAbstractButton.clicked)：This signal is emitted when the button is activated (i.e., pressed down then released while the mouse cursor is inside the button), when the shortcut key is typed, or when click() or animateClick() is called. Notably, this signal is not emitted if you call setDown(), setChecked() or toggle().
- [`pressed()`](https://doc.qt.io/qtforpython/PySide2/QtWidgets/QAbstractButton.html#PySide2.QtWidgets.PySide2.QtWidgets.QAbstractButton.pressed)：This signal is emitted when the button is pressed down.
- [`released()`](https://doc.qt.io/qtforpython/PySide2/QtWidgets/QAbstractButton.html#PySide2.QtWidgets.PySide2.QtWidgets.QAbstractButton.released)：This signal is emitted when the button is released.
- [`toggled()`](https://doc.qt.io/qtforpython/PySide2/QtWidgets/QAbstractButton.html#PySide2.QtWidgets.PySide2.QtWidgets.QAbstractButton.toggled)：This signal is emitted whenever a checkable button changes its state. 
- [`destroyed()`](https://doc.qt.io/qtforpython/PySide2/QtCore/QObject.html#PySide2.QtCore.PySide2.QtCore.QObject.destroyed)：类被摧毁，信号不可阻断。
- [`objectNameChanged()`](https://doc.qt.io/qt-5/qobject.html#objectNameChanged)：类名变化。

　　前四者继承自`PyQt5.QtWidgets.QWidget.QAbstractButton`，后二者继承自`PyQt5.QtCore.QObject`。

# 3 实例

```Python
# coding=utf-8

import sys
from PyQt5.QtWidgets import (QWidget, QPushButton, QApplication,
                             QLabel, QGridLayout)
from PyQt5.QtCore import Qt


class Exmaple(QWidget):
    def __init__(self):
        super().__init__()
        
        self.setup_ui()
    
    def setup_ui(self):
        self.btn = QPushButton('Click me')
        self.btn.clicked.connect(self.prt)  # 链接signal与slot
        self.lbl = QLabel()
        
        self.grid = QGridLayout()
        self.grid.addWidget(self.btn, 1, 0)
        self.grid.addWidget(self.lbl, 2, 0)

        self.setLayout(self.grid)
        self.grid.setSpacing(20)  # 控件与控件、控件与窗口边界之间的留白
        self.setWindowTitle('Example')
        self.show()
    
    def prt(self):
        self.lbl.setText('Clicked!')
        self.lbl.setAlignment(Qt.AlignCenter)


if __name__ == "__main__":
    app = QApplication(sys.argv)
    ex = Exmaple()
    sys.exit(app.exec_())
```

　　`self.btn.clicked.connect(self.prt)`将`clicked()`与处理程序`prt()`链接起来，其中，`clicked()`是`signal`，`prt()`是`slot`。除此之外，还有另外一种链接办法：

```python
PyQt5.QtCore.QObject.connect(widget, QtCore.SIGNAL(‘signalname’), slot_function)
```

<br />
**以上！**
<br />









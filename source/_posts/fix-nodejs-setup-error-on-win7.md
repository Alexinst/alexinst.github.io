---
title: 「NodeJs」Win7 下安装 NodeJs

date: 2018-10-01 21:43:23

tag:
    - Node.js
    - win7
    
categories:
    - Manual
---

**简述**：`win7`系统下安装`NodeJs`莫名失败（Node.js Setup Wizard ended prematurely），我经过搜索和尝试，最后终于安装成功。为此耗费了足足五个小时的时间。。。

<!-- more -->
<br/>

# 报错信息

![roll back](https://res.cloudinary.com/hexo-pics/image/upload/v1538398125/hexo-2018/10/roll_back.png)
![wizard ended](https://res.cloudinary.com/hexo-pics/image/upload/v1538398125/hexo-2018/10/wizard_ended.png)

　　在快要安装成功时，出现回滚。我的天，真是夭寿！

　　这番报错，毫无有价值的参考信息。
<br/>

# 解决方法
## 下载Windows Binary版本
　　下载地址：[node-v8.12.0-win-x64](https://nodejs.org/dist/v8.12.0/node-v8.12.0-win-x64.zip)

　　8.12.0 版本是`LTS（Long-term Support）`稳定版本。

## 设置环境变量
1. 选择合适位置放置解压后的Nodejs文件夹
2. 右击`我的电脑`， 选择`属性`。
3. 点击位于左侧的`高级系统设置`。
4. 在跳出的对话框中，点击右下角的`环境变量`。
5. 设置两个环境变量。示例：D:\nodejs， D:\nodejs\node_modules。

## 测试
　　按住`win`+`R`，在`运行`对话框中输入`cmd`，回车。

　　在`命令行窗口`中分次输入
```shell
node -v

npm -v
```

　　响应分别为
```
8.12.0

6.4.1
```

　　至此，若响应如上，则宣告安装成功。

　　若不是，那就。。。

<br />



**以上**


---
title: Debian 使用 apt 安装软件时提示插入光盘

date: 2018-09-02 19:45:19

tags:
    - Linux
    - Debian
    - apt
    - insert_disc

categories:
    - Linux
 
---

**简述**：`apt`是`Debian`的安装包管理工具。

<!-- more -->
<br />

# 问题
在VMware中成功安装Debian 9.2，进入终端运行：
```
sudo apt-get update
```

结果出现如下错误：
```
Media change: please insert the disc labeled
 'Debian GNU/Linux......'
in the drive '/media/cdrom/' and press enter
```

# 解决办法
打开并修改文件`sources.list`：
```
vi /etc/apt/sources.list
```

找到：
```
deb cdrom:[Debian GNU/Linux ......] .......
```

注释：
```           
# deb cdrom:[Debian GNU/Linux ......] .......            

```

最后：
```
sudo apt-get update
```






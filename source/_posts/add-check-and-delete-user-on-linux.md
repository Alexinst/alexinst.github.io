---
title: 「Linux」增删用户以及赋予 sudo 权限
date: 2019-08-27 11:17:52
tags: 
	- Linux
	- user
	- sudo
categories:
	- Linux
---

**简述** ：`Linux`日常使用不建议用`root`登录，因为`root`具有全部权限。为此，需要添加一个普通用户，而为了正常使用，还需要赋予其`sudo`权限。

<!-- more -->

<br />

# 1. 添加普通用户并赋予 sudo 权限

## 1.1 创建普通用户帐号

```bash
$ sudo adduser USERNAME
```

**全文`USERNAME` 指代用户名，例如 Alex、Sam 等等。**

为运行创建新用户的命令，当前 user 必须是`root`或者具有`sudo`权限的普通 user。

创建过程中，需要输入用户密码，其他信息的输入可选择跳过。

## 1.2 将用户添加至 `sudo` group

```bash
$ sudo usermod -aG sudo USERNAME
```

`usermod` 手册：[Here](https://manpages.ubuntu.com/manpages/cosmic/en/man8/usermod.8.html)

<br />

# 2 测试用户权限

切换至新用户：

```bash
$ su - USERNAME
```

测试权限：

```bash
$ sudo whoami
```

输出应为：

```bash
root
```

<br />

# 3 修改密码

## 3.1 修改自己的密码

```bash
$ passwd
```

## 3.2 修改他人的密码

```bash
$ sudo passwd USERNAME
```

修改**其他用户**的密码，你必须是`root`用户或者具有`sudo`权限 。

<br />

# 4 移除用户的 sudo 权限

```bash
$ sudo deluser USERNAME sudo
```

验证用户是否已从`sudo`组删除：

```bash
$ sudo -lU USERNAME
```

`deluser`手册：[Here](https://manpages.ubuntu.com/manpages/bionic/en/man8/deluser.8.html)；`sudo`手册：[Here](https://manpages.ubuntu.com/manpages/bionic/en/man8/sudo.8.html)

<br />



# 5 删除用户（慎重）

```bash
$ sudo deluser --remove-home USERNAME
```

`--remove-home`：删除用户时移除其`home`目录。不加此选项，`home`目录不会被删除。

<br />



# 6 查看所有用户和组

## 6.1 查看所有用户

```bash
$ cat /etc/group
```

## 6.2 查看所有组

```bash
$ cat /etc/passwd
```

## 6.3 查看当前用户所在组

```bash
$ groups
```

<br />

# 7 参考

[深入理解 sudo 与 su 之间的区别](https://linux.cn/article-8404-1.html)

[How To Create a Sudo User on Ubuntu](https://linuxize.com/post/how-to-create-a-sudo-user-on-ubuntu/)

[一起学习在 Ubuntu 上授予和移除 sudo 权限](http://blog.itpub.net/31559985/viewspace-2638312/)

<br />

**以上！**
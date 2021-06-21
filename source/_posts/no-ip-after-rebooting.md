---
title: 「Linux」VPS 重启后未分配 IP
date: 2021-06-16 23:38:09
tags:
	- Linux
	- Ubuntu
	- Internet
	- DHCP
categories:
	- Linux
---

**简述**：VPS（Ubuntu OS）重启后出现未分配 IP 地址而无法连接的问题，具体现象为 `ifconfig` 命令未显示网络接口 eth0 的信息。解决办法为通过 VNC 登陆 VPS，调用 `ifconfig` 启用网络接口 eth0，再调用 `dhclient` 向 DHCP 服务器请求 IP。



<!-- more -->

<br />

# 1 问题与解决

## 1.1 问题描述

VPS（Ubuntu OS）重启后出现未分配 IP 地址而无法连接的问题。具体现象为

`cat /etc/network/interfaces` 的信息中包含网络接口 lo 与 eth0。

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address XXX.XXX.XXX.XXX
    netmask 255.255.255.0
    broadcast XXX.XXX.XXX.255
    network XXX.XXX.XXX.0
    gateway  XXX.XXX.XXX.1

dns-nameserver 8.8.8.8
```

但 `ifconfig` 命令只显示网络接口 lo 的信息，即 eth0 未激活。

```
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
    inet 127.0.0.1  netmask 255.0.0.0
    inet6 ::1  prefixlen 128  scopeid 0x10<host>
    loop  txqueuelen 1000  (Local Loopback)
    RX packets 43  bytes 5520 (5.5 KB)
    RX errors 0  dropped 0  overruns 0  frame 0
    TX packets 43  bytes 5520 (5.5 KB)
    TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

在本地 ping 远程 VPS，显示连接超时，也无法通过 SSH 登录 VPS。



## 1.2 解决方案

1. 激活网络接口 eth0。shell 命令：

   ```shell
   $ sudo ifconfig eth0 up
   ```

   网络接口的名字不一定是 eth0，可通过 `cat /etc/network/interfaces` 查看所有网络接口。

2. 通过 DHCP 协议配置网络接口的参数。shell 命令：

   ```shell
   $ sudo dhclient eth0
   ```

3. 查看网络接口 eth0 的信息。shell 命令：

   ```shell
   $ sudo ifconfig eth0
   ```

   

<br />

# 2 ifconfig

`ifconfig` 是 interface configuration 的缩写，能够显示网络接口信息，配置网络接口。不给定参数的情况下，`ifconfig` 会显示当前启用的网络接口信息。



## 2.1 语法

```shell
ifconfig [-v] [-a] [-s] [interface]
ifconfig [-v] interface [aftype] options | address ...
```



## 2.2 用法

| 写法                                    | 含义                                       |
| --------------------------------------- | ------------------------------------------ |
| `ifconfig`                              | 显示可用网络接口的信息。                   |
| `ifconfig -a`                           | 显示所有网络接口的信息。                   |
| `ifconfig eth0`                         | 显示网络接口 eth0 的信息。                 |
| `ifconfig eth0 up`                      | 启用网络接口 eth0。                        |
| `ifconfig eth0 down`                    | 停用网络接口 eth0。                        |
| `ifconfig eth0 172.16.25.125`           | 设置网络接口 eth0 的 IP 地址。             |
| `ifconfig eth0 netmask 255.255.255.224` | 设置网络接口 eth0 的子网掩码。             |
| `ifconfig eth0 broadcast 172.16.25.63`  | 设置网络接口 eth0 的广播地址。             |
| `ifconfig eth0 mtu 1000`                | 设置网络接口 eth0 的 MTU（最大传输单元）。 |



<br />

# 3 dhclient

dhclient 的全写是 Dynamic Host Configuration Protocol（DHCP） Client。

DHCP 协议用于让中央服务器给主机分配一个网络 IP 地址。

起初，主机设备的网络接口并未被分配一个 IP 地址，所以该主机无法连接。（可以通过 `ip address` 或 `ifconfig -a` 来验证这一点）。

调用 `dhclient`，能够向 DHCP 服务器请求一个 IP 地址。主机向本地网络中的广播地址发送一个 DHCP discovery message，（在路由器或网关）运行的 DHCP 服务器收到消息后向该主机发送一个 DHCP offer message，提供一个 IP 地址。

`dhclient` 的配置文件是 `dhclient.conf`。



## 3.1 语法

```
dhclient 
[ -4 | -6 ] [ -S ] [ -N [ -N...  ] ] [ -T [ -T...  ] ] 
[ -P [ -P...  ] ] -R ] [ -i ] [ -I ] [ -4o6 port ] 
[ -D LL|LLT ] [ -p port-number ] [ -d ] 
[ -df duid-lease-file ] [ -e VAR=value ] 
[ -q ] [ -1 ] [ -r | -x ] [ -lf lease-file ] 
[ -pf pid-file ] [ --no-pid ] [ -cf  config-file  ] 
[-sf  script-file ] [ -s server-addr ] [ -g relay ]
[ -n ] [ -nw ] [ -w ] [ --dad-wait-time seconds ]
[ --prefix-len-hint length ] [ --decline-wait-time seconds ] [ -v ] [ --version ]
[ if0 [ ...ifN ] ]
```



## 3.2 选项

| 选项 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| -p   | 指定 DHCP 客户端监听的端口号（默认端口号 86）                |
| -d   | 总是以前台方式运行程序                                       |
| -q   | 安静模式，不打印任何错误的提示信息                           |
| -r   | 释放 IP 地址                                                 |
| -n   | 不配置任何接口                                               |
| -x   | 停止正在运行的 DHCP 客户端，而不释放当前租约，杀死现有的 dhclient |
| -s   | 在获取 IP 地址之前指定 DHCP 服务器                           |
| -w   | 即使没有找到广播接口，也继续运行                             |



<br />

# 4 参考

- [ubuntu 下 eth0 网卡信息不见了 - CSDN](https://blog.csdn.net/cmh477660693/article/details/52760236)
- [如何在 Linux 命令行上查看 IP 地址 - Howtoing](https://www.howtoing.com/check-ip-address-on-linux)
- [15 Useful “ifconfig” Commands to Configure Network Interface in Linux - Tecmint](https://www.tecmint.com/ifconfig-command-examples/)
- [What does dhclient do? - Stack Overflow](https://stackoverflow.com/questions/63961142/what-does-dhclient-do)



<br />

**以上！**

<br />
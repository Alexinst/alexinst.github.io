---
title: 「TensorFlow」1. 安装 CUDA 和 cuDNN
date: 2019-10-22 21:01:37
tags:
    - CUDA
    - cuDNN
    - TensorFlow
categories:
    - TensorFlow
---

**简述：**为了编译 `TensorFlow C++`，再次踩进 `CUDA` 之坑。这是一条不归路，但还是必须做个整理记录，因为这也许不是最后一次。

<!-- more -->
<br />


系统环境：

- `OS`：`Ubuntu 14.04.6 LTS x64 (trusty)`
- `RAM`：`64 GB`
- `GPU`：`GTX TITAN x 4`

安装版本：

- `CUDA Toolkit`：10.0 
- `cuDNN`：7.4.2


<br />

# 1 更新驱动

1. 打开 ”附加驱动“ (`Ubuntu 14.04 LTS`)
   - `Ubuntu-desktop` + `Ubuntu` 主题：按 `Win` 键，搜索”附加驱动“
   - `Ubuntu-desktop` + `Gnome `主题 ：左上角”应用程序“ -> 系统工具 -> 首选项 -> 附加驱动
2. 选择驱动程序：
   - 选择 ”使用 NVIDIA binary driver - version 410.56 来自 nvidia-410 (开源) ”
   - 点击右下角“应用更改”
   - 等待完成驱动更新

![update nvidia driver](https://res.cloudinary.com/hexo-pics/image/upload/v1571749948/hexo-2019/10/update_nvidia_driver.png)

<br />



# 2 默认安装位置

| Component    | 默认安装位置                             |
| ------------ | -------------------------------------- |
| `CUDA Toolkit` | `/usr/local/cuda-10.0`                 |
| `CUDA Samples` | `$(HOME)/NVIDIA_CUDA-10.0_Samples`     |
| `cuDNN`      | `/usr/lib/x86_64-linux-gnu`  &  `/usr/include` |

<br />

# 3 安装 CUDA

1. 验证是否安装 `gcc`

   ```bash
   $ gcc --version
   ```

   如果输出如：

   ```bash
   Command 'gcc' not found, but can be installed with:
   sudo apt install gcc
   ```

   则意味着当前主机尚未安装 `gcc`，需运行以下指令进行安装：

   ```bash
   $ sudo apt update
   $ sudo apt install build-essential manpages-dev
   ```

2. 下载

   下载地址：[CUDA 10.0](https://developer.nvidia.com/cuda-10.0-download-archive)

   ![download cuda](https://res.cloudinary.com/hexo-pics/image/upload/v1571749947/hexo-2019/10/cuda_download.png)

3. 安装
   - 首先，停用 `gdm`（`Ubuntu` 桌面环境），在命令行界面（ `Ctrl + Alt + F1 ~ F6` ）或者远程连接（ `SSH` ）的命令行界面下进行指令输入
   ```bash
   $ sudo service gdm stop
   ```
   - 随后，安装 `CUDA` 以及补丁包，按照命令行提示进行操作
   ```bash
   $ sudo sh cuda_10.0.xxx_xxx.xx_linux.run
   $ sudo sh cuda_10.0.xxx.x_linux.run
   ```

   - 最后，恢复桌面环境
   ```bash
   $ sudo service gdm start
   ```

4. 验证是否安装成功，共有三种方法：
   ```bash
   $ nvcc -V
   ```
   输出应为：
   ```bash
   nvcc: NVIDIA (R) Cuda compiler driver
   Copyright (c) 2005-2018 NVIDIA Corporation
   Built on Sat_Aug_25_21:08:01_CDT_2018
   Cuda compilation tools, release 10.0, V10.0.130
   ```
   <br />
   ```bash
   $ cat /proc/driver/nvidia/version
   ```
   输出应为：
   ```bash
   NVRM version: NVIDIA UNIX x86_64 Kernel Module  418.56  Fri Mar 15 12:59:26 CDT 2019
   GCC version:  gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.4) 
   ```
   <br />

   ```bash
   $ cat /usr/local/cuda/version.txt
   ```
   输出应为：
   ```bash
   CUDA Version 10.0.130
   ```

5. 系统重启

   若出现异常，[这里](#Q&A)也许能有相似的情况。

6. 检查
   检查设备文件 `/dev/nvidia*` 是否存在，且文件权限是否正确 (666)。通常情况下，这些文件会被需要用到 NVIDIA 驱动的应用程序 (如 `CUDA` 应用或 `X server`) 经由调用 `nvidia-modprobe`自动生成。

   但你也可以通过下面的脚本程序，手动创建这些文件。
   ```bash
   #!/bin/bash
   
   /sbin/modprobe nvidia
   
   if [ "$?" -eq 0 ]; then
     # Count the number of NVIDIA controllers found.
     NVDEVS=`lspci | grep -i NVIDIA`
     N3D=`echo "$NVDEVS" | grep "3D controller" | wc -l`
     NVGA=`echo "$NVDEVS" | grep "VGA compatible controller" | wc -l`
   
     N=`expr $N3D + $NVGA - 1`
     for i in `seq 0 $N`; do
       mknod -m 666 /dev/nvidia$i c 195 $i
     done
   
     mknod -m 666 /dev/nvidiactl c 195 255
   
   else
     exit 1
   fi
   
   /sbin/modprobe nvidia-uvm
   
   if [ "$?" -eq 0 ]; then
     # Find out the major device number used by the nvidia-uvm driver
     D=`grep nvidia-uvm /proc/devices | awk '{print $1}'`
   
     mknod -m 666 /dev/nvidia-uvm c $D 0
   else
     exit 1
   fi
   ```

7. 配置环境变量

   ```bash
   $ export PATH=/usr/local/cuda-10.1/bin:/usr/local/cuda-10.0/NsightCompute-2019.1${PATH:+:${PATH}}
   $ export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64\
                            ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
   ```

   这样设置是临时性的，你可以将之复制到 `~/.bashrc` 中，以达到永久设置的目的。

   <br />



# 4 安装 cuDNN

1. 下载 

   下载地址：[cuDNN 所有版本](<https://developer.nvidia.com/rdp/cudnn-archive>)

   找到 `Download cuDNN v7.4.2 (Dec 14, 2018), fro CUDA 10.0`，下载三个 `DEB` 文件（`cuDNN Runtime Library`、`cuDNN Developer Library`、 `cuDNN Code Samples and User Guide`）

   ![download cudnn](https://res.cloudinary.com/hexo-pics/image/upload/v1571749949/hexo-2019/10/cudnn_download.png)

2. 安装

   ```bash
   $ sudo dpkg -i libcudnn7_7.4.2.xx-1+cuda10.0_amd64.deb  // 运行时库
   $ sudo dpkg -i libcudnn7-dev_7.4.2.xx-1+cuda10.0_amd64.deb  // 开发者库
   $ sudo dpkg -i libcudnn7-doc_7.4.2.24-1+cuda10.0_amd64.deb  // 代码案例和用户指南
   ```

3. 验证
   ```bash
   $ grep -i cudnn_major -A 2 /usr/include/cudnn.h
   ```
   输出应为：
   ```bash
   #define CUDNN_MAJOR 7
   #define CUDNN_MINOR 4
   #define CUDNN_PATCHLEVEL 2
   --
   #define CUDNN_VERSION (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)
   
   #include "driver_types.h"
   ```

<br />

# 5 问题与解决 <span id="Q&A"></span>

- 无限重复登陆；重启后黑屏，左上角有一下划线不断闪烁：

  解决方案：远程登录 (`SSH`) ，更新 `GPU` 驱动至更高版本

  ```shell
  ssh NAME@IP -p 22
  ......
  ......
  
  $ ubuntu-drivers devices
  == /sys/devices/pci0000:00/0000:00:03.0/0000:03:00.0/0000:04:10.0/0000:05:00.0 ==
  vendor   : NVIDIA Corporation
  modalias : pci:vxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  driver   : nvidia-418 - third-party free
  driver   : xserver-xorg-video-nouveau - distro free builtin
  driver   : nvidia-415 - third-party free
  driver   : nvidia-430 - third-party free recommended
  driver   : nvidia-384 - third-party non-free
  driver   : nvidia-410 - third-party free
  
  $ sudo apt install nvidia-xxx
  
  $ sudo reboot
  ```

- 启动并登陆系统后只显示壁纸，无顶栏与左侧快捷栏

  解决方案有两种：
  1. 新建一个用户账号，注销当前账号，换用新账号登陆。原理未知。
  2. 换用系统主题 (`Ubuntu` -> 其他)。在刚启动系统，进行账号登录时，在登录按钮左边有一个齿轮状的按钮，点击一下你就了解了。

<br />

# 6 参考

- [CUDA installation Guide Linux - NVIDIA](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
- [cuDNN install - NVIDIA](https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html)
- [install gcc - linuxize](https://linuxize.com/post/how-to-install-gcc-compiler-on-ubuntu-18-04/)
- [install the nvidia drivers - linuxconfig](https://linuxconfig.org/how-to-install-the-nvidia-drivers-on-ubuntu-18-04-bionic-beaver-linux)

<br />



# 7 系列

1. {% post_link installation-of-cuda-toolkit-and-cudnn 「TensorFlow」1. 安装 CUDA 和 cuDNN %}
2. {% post_link build-tensorflow-cpp 「TensorFlow」2. 编译 TensorFlow C++  %}



<br />

**以上！**

<br />
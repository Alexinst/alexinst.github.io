---
title: 「TensorFlow」2. 编译 TensorFlow C++
date: 2019-11-05 12:59:39
tags:	
	- TensorFlow
	- bazel
	- build
	- manual
categories:
	- TensorFlow
---

**简述**：编译 `TensorFlow C++` 很费劲，需要一颗向死之心。虽然成功编译已经是半个月前的事，但我拖到现在才整理成稿。

<!-- more -->

<br />



系统环境：

- `OS`：`Ubuntu 14.04.6 LTS x64 (trusty)`
- `RAM`：`64 GB`
- `GPU`：`NVIDIA GTX TITAN x 4`
- `CUDA Toolkit`：10.0
- `cuDNN`：7.4.2

安装版本：

- `bazel`：0.21.0
- `TensorFlow`：1.13.1

<br />



# 1 安装 bazel

幸好 `bazel 0.21.0`  支持 `Ubuntu 14.04` ，不然就只能砸电脑了，虽然没钱赔！

[官方步骤]( https://docs.bazel.build/versions/0.21.0/install-ubuntu.html )：

1. 安装依赖： `pkg-config`, `zip`, `g++`, `zlib1g-dev`, `unzip`, 和 `python`。

   ```bash
   $ sudo apt-get install pkg-config zip g++ zlib1g-dev unzip python
   ```

2. 下载安装脚本，这是[bazel 发布页面](https://github.com/bazelbuild/bazel/releases)。

   ```bash
   $ cd /tmp
   $ wget https://github.com/bazelbuild/bazel/releases/download/0.21.0/bazel-0.21.0-installer-linux-x86_64.sh
   ```

3. 运行脚本：

   ```bash
   $ chmod +x bazel-0.21.0-installer-linux-x86_64.sh
   $ ./bazel-0.21.0-installer-linux-x86_64.sh --user
   ```

   使用 `--user` 会将 `bazel` 安装在 `$HOME/bin` 路径下，并将配置文件 `.bazelrc` 设置在 `$HOME` 路径下。

4. 配置环境变量

   如果你有使用 `--user`，则：

   ```bash
   $ echo "# bazel path
   export PATH="$PATH:$HOME/bin"" >> ~/.bashrc
   $ source ~/.bashrc       
   ```

   官方教程只设置了临时环境变量，但我习惯一步到位。

5. 验证

   ```bash
   $ bazel version
   WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
   INFO: Invocation ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   Build label: 0.21.0
   ..................................................
   ```


<br />



# 2 环境初始化

1. 拉取 TensorFlow

   ```bash
$ mkdir ~/Programs
$ cd Programs/
$ git clone -b r1.13 https://github.com/tensorflow/tensorflow.git TensorFlow/
   ```

2. 运行环境初始化脚本

   ```bash
$ cd TensorFlow/tensorflow/contrib/makefile
$ chmod +x build_all_linux.sh
$ ./build_all_linux.sh
   ```

<br />



# 3 编译 TensorFlow

1. 配置编译要求

   ```bash
   $ cd ~/Programs/TensorFlow
   $ ./configure
   WARNING: Duplicate rc file: /home/ttt/Programs/tensorflow-r1.13-4/.tf_configure.bazelrc is read multiple times, most recently imported from /home/ttt/.bazelrc
   WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
   INFO: Invocation ID: a16de493-8f9e-46f8-96bc-0903f0e36c3b
   You have bazel 0.21.0 installed.
   Please specify the location of python. [Default is /usr/bin/python]: /home/ttt/miniconda3/bin/python
   
   Found possible Python library paths:
     /home/ttt/miniconda3/lib/python3.6/site-packages
   Please input the desired Python library path to use.  Default is [/home/ttt/miniconda3/lib/python3.6/site-packages]
   
   Do you wish to build TensorFlow with XLA JIT support? [Y/n]: y
   XLA JIT support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: n
   No OpenCL SYCL support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with ROCm support? [y/N]: n
   No ROCm support will be enabled for TensorFlow.
   
   Do you wish to build TensorFlow with CUDA support? [y/N]: y
   CUDA support will be enabled for TensorFlow.
   
   Please specify the CUDA SDK version you want to use. [Leave empty to default to CUDA 10.0]: 
   
   
   Please specify the location where CUDA 10.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: 
   
   
   Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 7]: 
   
   
   Please specify the location where cuDNN 7 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: 
   
   
   Do you wish to build TensorFlow with TensorRT support? [y/N]: n
   No TensorRT support will be enabled for TensorFlow.
   
   Please specify the locally installed NCCL version you want to use. [Default is to use https://github.com/nvidia/nccl]: 
   
   
   Please specify a list of comma-separated Cuda compute capabilities you want to build with.
   You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
   Please note that each additional compute capability significantly increases your build time and binary size. [Default is: 6.1,6.1,6.1,6.1]: 
   
   
   Do you want to use clang as CUDA compiler? [y/N]: n
   nvcc will be used as CUDA compiler.
   
   Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]: 
   
   
   Do you wish to build TensorFlow with MPI support? [y/N]: n
   No MPI support will be enabled for TensorFlow.
   
   Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native -Wno-sign-compare]: 
   
   
   Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: n
   Not configuring the WORKSPACE for Android builds.
   
   Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See .bazelrc for more details.
   	--config=mkl         	# Build with MKL support.
   	--config=monolithic  	# Config for mostly static monolithic build.
   	--config=gdr         	# Build with GDR support.
   	--config=verbs       	# Build with libverbs support.
   	--config=ngraph      	# Build with Intel nGraph support.
   	--config=dynamic_kernels	# (Experimental) Build kernels into separate shared objects.
   Preconfigured Bazel build configs to DISABLE default on features:
   	--config=noaws       	# Disable AWS S3 filesystem support.
   	--config=nogcp       	# Disable GCP support.
   	--config=nohdfs      	# Disable HDFS support.
   	--config=noignite    	# Disable Apacha Ignite support.
   	--config=nokafka     	# Disable Apache Kafka support.
   	--config=nonccl      	# Disable NVIDIA NCCL support.
   Configuration finished
   ```

2. 编译 `TensorFlow C++` 库
   - `CPU` 版本
   ```bash
   $ bazel build --config=opt //tensorflow:libtensorflow_cc.so
   ```

   - `GPU` 版本
   ```bash
   $ bazel build --config=opt --config=cuda //tensorflow:libtensorflow_cc.so
   ```

<br />



# 4 测试环境

1. 新建目录及文件

   ```bash
   $ mkdir -p ~/Projects/demo
   $ cd ~/Projects/demo
   $ mkdir build src
   $ touch CMakeLists.txt src/main.cpp
   ```

2. 编写测试代码 `main.cpp`

   ```c++
   #include <tensorflow/core/platform/env.h>
   #include <tensorflow/core/public/session.h>
   #include <iostream>
   
   using namespace std;
   using namespace tensorflow;
   
   int main()
   {
       Session* session;
       Status status = NewSession(SessionOptions(), &session);
       if (!status.ok()) {
           cout << status.ToString() << "\n";
           return 1;
       }
       cout << "Session successfully created.\n";
   }
   ```

3. 编写 `CMakeLists.txt`

   ```cmake
   cmake_minimum_required(VERSION 3.15)
   project(demo)
   
   set(CMAKE_CXX_STANDARD 11)
   set(PROGRAMS_DIR /home/ttt/Programs)
   set(TENSORFLOW_DIR ${PROGRAMS_DIR}/TensorFlow)
   
   include_directories(${TENSORFLOW_DIR})
   include_directories(${TENSORFLOW_DIR}/bazel-genfiles)
   include_directories(${TENSORFLOW_DIR}/tensorflow/contrib/makefile/downloads/absl)
   include_directories(${TENSORFLOW_DIR}/tensorflow/contrib/makefile/downloads/eigen)
   
   link_directories(${TENSORFLOW_DIR}/bazel-bin/tensorflow)
   
   add_executable(demo main.cpp)
   target_link_libraries(LaneNet tensorflow_cc tensorflow_framework)
   ```

4. 目录结构

   ```
   ├── build
   | 
   ├── CMakeLists.txt
   |
   ├── src
   | └── main.cpp
   ```

5. 编译并运行测试程序

   ```shell
   $ cd ~/Programs/demo/build
   $ cmake ..
   $ make
   $ ./demo
   ```

   输出：

   ```bash
   ..................................................
   Session successfully created.
   ```

<br />

#  5 疑难杂症

1. `bazel` 版本问题

   ```
   ERROR:/home/ttt/.cache/bazel/_bazel_ttt/12fb0bc5892f7f0d9058b186b377c0bf/external/local_config_cc/BUILD:57:1: in cc_toolchain rule @local_config_cc//:cc-compiler-k8: Error while selecting cc_toolchain: Toolchain identifier 'local' was not found, valid identifiers are [local_linux, local_darwin, local_windows]
   ```

   解决办法：`bazel` 0.19.1 -> 0.21.0

2. `eigen` 版本问题

   ```bash
   /usr/local/tensorflow/include/third_party/eigen3/unsupported/Eigen/CXX11/Tensor:1:42: 
   fatal error: unsupported/Eigen/CXX11/Tensor: 没有那个文件或目录
    #include "unsupported/Eigen/CXX11/Tensor"
   ```

   解决办法：更新 `eigen` 至 3.3，且添加到 `CMakeLists.txt` 的搜索目录

3. 找不到 `abseil-cpp` 

   ```bash
   /usr/local/tensorflow/include/tensorflow/core/lib/core/stringpiece.h:29:38: fatal error: absl/strings/string_view.h: 没有那个文件或目录
    #include "absl/strings/string_view.h"
   ```

    解决办法（[来源](https://www.cnblogs.com/buyizhiyou/p/10405634.html)）：下载 `abseil-cpp` 并添加到 `CMakeLists.txt` 的搜索目录

4. `CMakeLists.txt` 

   ```bash
   /home/ttt/Programs/tensorflow-r1.13-3/tensorflow/core/framework/tensor_shape.h:22:48: fatal error: tensorflow/core/framework/types.pb.h: 没有那个文件或目录
    #include "tensorflow/core/framework/types.pb.h"
   ```

   解决办法：`include_directories(${TENSORFLOW_DIR}/bazel-genfiles)`

<br />



# 6 参考

- [Installing Bazel on Ubuntu - bazel](https://docs.bazel.build/versions/1.1.0/install-ubuntu.html#install-on-ubuntu)
- [Ubuntu安装TensorFlow C++ - GitHub]( https://blog.csdn.net/MOU_IT/article/details/87976152#5%E3%80%81%E9%85%8D%E7%BD%AETensorFlow%E7%9A%84%E4%BE%9D%E8%B5%96%EF%BC%9Aprotobuf%E5%92%8Ceigen )
- [将Tensorflow源码编译成C++库文件 - hemajun815](https://github.com/hemajun815/tutorial/blob/master/tensorflow/compilling-tensorflow-source-code-into-C%2B%2B-library-file.md#%E6%96%B9%E5%BC%8F%E4%BA%8C)
- [Tensorflow C++ 从训练到部署(1)：环境搭建](http://www.liuxiao.org/2018/08/ubuntu-tensorflow-c-%E4%BB%8E%E8%AE%AD%E7%BB%83%E5%88%B0%E9%A2%84%E6%B5%8B1%EF%BC%9A%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/)
- [Standalone C++ Build — TF + OpenCV - Medium](https://medium.com/@tomdeore/standalone-c-build-tensorflow-opencv-6dc9d8a1412d)
- [Use TensorFlow C++ API with OpenCV3 - Medium](https://medium.com/@fanzongshaoxing/use-tensorflow-c-api-with-opencv3-bacb83ca5683)
- [Tensorflow c++ 实践及各种坑](https://cloud.tencent.com/developer/article/1006107)

<br />



# 7 系列

1. {% post_link installation-of-cuda-toolkit-and-cudnn 「TensorFlow」1. 安装 CUDA 和 cuDNN %}

2. {% post_link build-tensorflow-cpp 「TensorFlow」2. 编译 TensorFlow C++  %}



<br />

**以上！**

<br />
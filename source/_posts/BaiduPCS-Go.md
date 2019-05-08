---
title: BaiduPCS-Go：仿Linux命令行的百度网盘客户端
date: 2018-05-25 23:08:41
tags:
    - BaiduPCS-GO
    - manual
categories:
    - Manual
---

**简述**： BaiduPCS-Go是一个模仿Linux命令的百度网盘的第三方不限速客户端。使用这个客户端，很酷炫，但也不一定很方便。

<!-- more -->
<br />

# 特色
　　这是一个仿Linux命令行的百度网盘客户端。（是的，不限速）

## 注意
文件名和目录名可用通配符\*补全(使用\*代替n个字符，n>=0)。
```bash
# 示例：
d /a_directory/b_directory/c_file.txt =
d /a*/b*/c*
```
## 特点
- 跨平台（Windows，macOS，Android等）
- 支持多账户。
- 网盘内列出文件和目录, 支持通配符匹配路径;
- 下载网盘内文件, 支持网盘内目录 (文件夹) 下载, 支持多个文件或目录下载, 支持断点续传和高并发高速下载。
- 离线下载，支持http/https/ftp/电驴/磁力链协议。
- 好玩，不过没有一点Linux基础，就不怎么好玩了。

<br />
# 程序下载
　　项目地址: [iikira/BaiduPCS-Go][1]
　　下载地址: [iikira/BaiduPCS-Go releases][2]
## Windows
　　下载地址:
- 64位系统：[BaiduPCS-Go-v3.3.3-windows-x64.zip][3]
- 32位系统：[BaiduPCS-Go-v3.3.3-windows-x86.zip][4]

## Android/macOS
- Android：需要用到 Termux 或 NeoTerm 类软件。这里只提一下。
- macOS：没有macOS的设备，这里属于凑字数。。

<br />
# 程序运行
　　下载后解压缩，双击 BaiduPCS-Go.exe。界面如下：

```shell
----
BaiduPCS-Go - 百度网盘客户端 for windows/amd64
  .......
  .......
  .......
BaiduPCS-Go >
```


## 登录/退出/切换

```bash
login     # 登录
logout    # 退出当前账户
su/chuser # 切换账户
```

　　在命令行窗口中输入 `login` ，再根据提示输入账号和密码，即可登录百度账号。

　　还有其他登录方式，如 `login -bduss=<BDUSS>`。([获取bduss][5])

　　`logout` 和 `su` / `chuser` 的用法也比较简单。

## 查看当前账户及已登录账户
```bash
loglist
```
## 切换目录
```bash
cd -l /Path/To/File
# cd = change directory = 切换目录。
# /Path/To/File = 文件路径，绝对路径或相对路径。
# -l: 显示目标目录下的子文件及子目录。
```

　　例：`cd /a/b/c/d`
## 下载
```bash
d /Path/To/File
# d = download = 下载
```
　　例：`d /a/b/c/d/e.txt`。文件路径也可以是相对路径的。

## web功能

```bash
web #启用web功能
```

　　输入上述命令后，在浏览器栏输入[localhost:8080](http://localhost:8080)，就可以在本地查看你的网盘目录。

　　很不错的功能。

<br />
# 帮助菜单

```bash
h
```

　　输入 `h` ，可以看到帮助菜单。这里做成表格，以便观看

| 用法 	| [global options] command [command options] [arguments...]	|
| :---:	| :--------------------------------------------------------	|

|  COMMANDS  			| 含义                                    			|
| :-------------------- | :------------------------------------------------ |
| tool    				| 工具箱                                    			|
| help / h   			| Shows a list of commands or help for one command 	|
| **其他：**           	|                                                   |
| run     				| 执行系统命令 										|
| sumfile / sf 			| 获取文件的秒传信息 									|
| web     				| 启用 web 客户端 (测试中) 							|
| **百度账号：**   		|                                                  	|
| login   				| 登录百度账号          								|
| loglist 				| 获取当前帐号, 和所有已登录的百度帐号 					|
| logout  				| 退出当前登录的百度帐号  								|
| su / chuser 			| 切换已登录的百度帐号 									|
| **百度网盘：**   		|   												|
| cd      				| 切换工作目录     									|
| cp      				| 拷贝(复制) 文件/目录 								|
| download / d 			| 下载文件或目录 										|
| ls / l / ll 			| 列出当前工作目录内的文件和目录 或 指定目录内的文件和目录 	|
| meta    				| 获取单个文件/目录的元信息 (详细信息) 					|
| mkdir    				| 创建目录 											|
| mv      				| 移动/重命名 文件/目录 								|
| offlinedl/clouddl / od| 离线下载 											|
| pwd     				| 输出当前所在目录 (工作目录) 							|
| quota   				| 获取配额, 即获取网盘的总储存空间, 和已使用的储存空间 		|
| rapidupload / ru 		| 手动秒传文件 										|
| rm     				| 删除 单个/多个 文件/目录 								|
| upload / u  			| 上传文件或目录 										|
| **配置:**     			|  													|
| config   				| 显示和修改程序配置项 									|

| GLOBAL OPTIONS:	| 全局选项 							|
| :---------------- | :-------------------------------- |
| --verbose  		| 启用调试 	|
| --help / -h 		| 查看帮助 							|
| --version / -v 	| 查看版本号 						| 


[1]:https://github.com/iikira/BaiduPCS-Go
[2]:https://github.com/iikira/BaiduPCS-Go/releases
[3]:https://github.com/iikira/BaiduPCS-Go/releases/download/v3.3.3/BaiduPCS-Go-v3.3.3-windows-x64.zip
[4]:https://github.com/iikira/BaiduPCS-Go/releases/download/v3.3.3/BaiduPCS-Go-v3.3.3-windows-x86.zip
[5]:https://github.com/iikira/BaiduPCS-Go/wiki/%E5%85%B3%E4%BA%8E-%E8%8E%B7%E5%8F%96%E7%99%BE%E5%BA%A6-BDUSS
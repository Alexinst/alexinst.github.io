---
title: 「Hexo」配置多终端推送文章
date: 2019-06-24 22:44:20
tags:
	- Hexo
	- multiple-pc
categories:
	- Manual
---



**简述**：Hexo 博客系统很轻便，经常配合 Github Page 使用。多终端推送更新是个问题，不过网上的解决方法很多，也基本一致，窝也记录一下自己的操作过程。

<!-- more -->

<br />

# 1 更新Repo

```shell
$ npm audit
```

根据终端提示内容，对 Repo 进行更新。在其他终端拉取仓库并进行简要配置后，就可以推送文章了。

当然，麻烦总是多过预期。不过，`warning`存在的意义就是被忽视，解决`error`就好了（

<br />

# 2 第一终端配置

本地 Hexo 对应文件夹其实就有` .gitignore `文件，可知 Hexo 也是支持将源文件存放到 Github 的。

注意事项：

- 本地 `themes` 文件夹下有`git clone`下载的主题，需先将对应目录下的 `.git` 文件夹删除，并将 themes 剪切到别处。将 Hexo 文件夹 `add`->`commit`->`push`三连后，再将之剪切回来，另行 `add`->`commit`->`push` 操作三连。

在 Hexo 对应目录下执行以下指令：

```Shell
$ git init
$ git checkout -b hexo				// 新建并切换到分支 hexo
$ git add .							
$ git commit -m 'Hexo Source'
$ git remote add origin git@github.com:USERNAME/USERNAME.github.io.git
$ git push origin hexo				// 推送到 Github Page 仓库的 hexo 分支
```

<br />

# 3 其他终端配置

```shell
$ git clone -b hexo git@github.com:USERNAME/USERNAME.github.io.git DIRNAME/
$ cd DIRNAME
$ npm install						// 这里不需要 npm init
```

<br />

# 4 新增文章并推送（任意终端）

```Shell
$ git pull origin hexo				// 拉取 Github 端的最新仓库，防止 commit 时出现冲突
$ hexo new "POST_NAME"
$ git add source/
$ git commit -m "COMMIT_TITLE"
$ git push origin hexo
$ hexo g -d
```



参考自：

<https://resalee.github.io/2017/11/01/hexo-github-blog/>

<https://www.zhihu.com/question/21193762>



<br />

以上！ 	
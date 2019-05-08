---

title: Hexo部署到Github出错

date: 2018-05-25 10:57:36

tags:
    - Hexo
    - blog

categories:
    - Hexo
---


**简述**: 将本地Hexo博客部署到Github Pages时，Git Bash出现报错：无法读取Github用户。在repo的url链接中加入用户名和密码，即可解决问题。

<!-- more -->
<br />

# 问题
　　搭建Hexo博客，按照网上教程，在`hexo d`这一步出现如下错误：

```
fatal: HttpRequestException encountered.
   ��������ʱ�����
bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: fatal: HttpRequestException encountered.
   ��������ʱ������
bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': No error
    at ChildProcess.<anonymous>
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:191:7)
    at ChildProcess.cp.emit
    at maybeClose
    at Process.ChildProcess._handle.onexit (internal/child_process.js:226:5)
```


# 解决办法

　　编辑博客文件下的`_config.yml`。

　　找到`#Deployment`，修改如下：

```
deploy:
    type: git
    repo:  https://<用户名>:<密码>@github.com/ares-x/ares-x.github.io.git 
    branch: master

```


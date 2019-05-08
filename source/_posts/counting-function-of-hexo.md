----
title: Hexo 开启阅读计数功能

date: 2018-09-09 19:25:05

tag:
    - Hexo
        - counting_function
        - 
categories:
    - Manual
----



**简述**：本文简练描述`Hexo`博客如何开启自带的阅读计数功能。

<!-- more -->
<br />

# leancloud配置
## 第一步
先注册。

## 第二步
创建应用，名字随意，计价方案为**开发版**。
![create an app](https://res.cloudinary.com/hexo-pics/image/upload/v1536494010/hexo-2018/09/create_app.png)

## 第三步
创建Class，名字**必须**为`Counter`，默认ACL权限设置为**无限制**。
![create a class](https://res.cloudinary.com/hexo-pics/image/upload/v1536494010/hexo-2018/09/create_class.png)

## 第四步
在**设置**中获取`appid`和`appkey`，之后跳到**[NexT配置](#conf)**。
![appid and appkey](https://res.cloudinary.com/hexo-pics/image/upload/v1536494010/hexo-2018/09/app_id_and_key.png)

<br />

# NexT配置
打开`NexT`主题目录下的`_config.yml`，找到`leancloud_visitors`位置，并如下修改之。
```yml
# Show number of visitors to each article.
# You can visit https://leancloud.cn get AppID and AppKey.
leancloud_visitors:
  enable: true
  app_id: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx #<app_id>
  app_key: xxxxxxxxxxxxxxxxxxxxxxxxxxxx #<app_key>
  # Dependencies: https://github.com/theme-next/hexo-leancloud-counter-security
  # If you don't care about security in lc counter and just want to use it directly
  # (without hexo-leancloud-counter-security plugin), set the `security` to `false`.
  security: false  # 如果设置了web安全域名，此处须改为 true。
  betterPerformance: true
```

<br />

# 设置web安全域名（非必须）
在`设置->安全中心`中，`web安全域名`处填写博客域名。
![web secure domain](https://res.cloudinary.com/hexo-pics/image/upload/v1536494010/hexo-2018/09/web_secure_domain.png)

如果打开浏览器调试模式（`F12`），出现`403`错误，则意味着域名填写出错。
![brower 403](https://res.cloudinary.com/hexo-pics/image/upload/v1536495903/hexo-2018/09/broswer_403.png)

那就只有两个办法：
- 死磕
- 放弃设置web安全域名

我选择第二个办法，毕竟第一个办法失败了，毕竟只是个小破站。所以，在`NexT`主题的`_config.yml`中将`leancloud_visitors`的`security`设置为`false`。

**以上！**
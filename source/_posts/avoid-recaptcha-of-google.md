---
title: 避免访问 Google 时出现的人机验证 reCaptcha

date: 2018-06-18 10:48:15

tags:
    - google
    - recaptcha
    - proxy
    - vps
    - digitalocean

categories:
    - Experience
---

**简述**：`DigitalOcean`也许资格太老，福利太好，被人滥用，导致现在开出`VPS`的`IP`可能存在问题。具体表现为，每次清空`cookie`，再次使用`Google`或`Youtube`就会出现人机验证`reCaptcha`。

<!-- more -->
<br />

# 1 问题
　　我也用过好几家云服务商的`VPS`，诸如`DigitalOcean`、`VirMach`、`Free-www`、`CenterHop`等。

　　对于我，至今出现`Google`的人机验证`reCaptcha`这问题，都是在使用`DO`家`VPS`搭建的扶墙服务的情况下。很显然，`DO`家的`IP`因为`VPS`被滥用，问题挺大的。

![reCpatcha](https://res.cloudinary.com/hexo-pics/image/upload/v1529297410/hexo-2018/06/recaptcha.gif)

　　这个人机验证是真的烦。

<br />

# 2 解决办法

　　在通过如`V2ray`使用`Google`时，如果出现验证码，那么页面下方会告知此时访问`Google`的`IP`地址，你就能看到具体是`IPv4`被封还是`IPv6`被封啦。

## 禁用IPv6
　　如果是`IPv6`地址存在问题，那就禁用`IPv6`，只使用`IPv4`访问网络。

　　编辑`/etc/sysctl.conf`，在文件末尾加入：
```
# disable ipv6
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
```

## 强制IPv4
　　如果是`IPv4`地址存在问题，那就只使用`IPv6`访问网络。当然，鉴于国内目前除了教育网，`IPv6`协议尚未普及，所以这种情况可能就比较尴尬了。

　　方法是，在`VPS`的`hosts`文件中，指定`Google`等的`IPv6`地址。编辑`/etc/hosts`，加入：
```
2607:f8b0:4005:801::200e google.com
2607:f8b0:4005:801::200e www.google.com
2607:f8b0:4007:805::100f scholar.google.cn
2607:f8b0:4007:805::100f scholar.google.com
2607:f8b0:4007:805::100f scholar.google.com.hk
2607:f8b0:4007:805::100f scholar.l.google.com
```

如上`IPv6`地址可能已失效，请参考[lennylxx/ipv6-hosts](https://raw.githubusercontent.com/lennylxx/ipv6-hosts/master/hosts)中提供的最新IPv6地址。



# 3 参考

- [通过VPS使用VPN或ShadowSocks访问Google或Google Schoolar出现验证码等的解决方法](https://www.polarxiong.com/archives/%E9%80%9A%E8%BF%87VPS%E4%BD%BF%E7%94%A8VPN%E6%88%96ShadowSocks%E8%AE%BF%E9%97%AEGoogle%E6%88%96Google-Schoolar%E5%87%BA%E7%8E%B0%E9%AA%8C%E8%AF%81%E7%A0%81%E7%AD%89%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95.html)

<br />
**以上！**
<br />
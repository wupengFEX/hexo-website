---
title: Github Pages 部署 https
date: 2017-07-30 13:15:08
tags: [Github Pages, HTTPS, CloudFlare]
---

> 不用多说大家也知道，https 是向网络安全进发的必经之路，同时也是推进 http 2.0 经历的一个阶段，目前很多大站都已经部署了 https，比如百度，淘宝。

对于有强迫症的开发者来说，一定忍受不了下面这种情况，在 **chrome://flags/** 中开启了 Mark non-secure origins as non-secure 的情况下，浏览器地址拦会出现一道不安全的红色显眼字眼。

<img width=500 src="/img/articles/github-pages-https/chrome-https-1.jpeg">

<img width=500 src="/img/articles/github-pages-https/chrome-https-2.jpeg">

这篇文章分享是：以 hexo 搭建个人网站，并将域名解析到个人申请的域名后，如何搭建 https 服务。很多人眼中 https 服务的搭建一定是要有自己的服务器、一定是动态网站、一定得有证书才能进行，那么诸如 hexo 搭建的这样一些纯静态的网站想要部署 https 可谓望成莫及，其实不然，一些免费的服务商已经为我们提供好了一整套服务，比如 CloudFlare、又拍云、七牛等 CDN 提供厂商。网站会基于服务商提供的免费功能动态获取证书，并为我们生成 https 服务。本文基于 [CloudFlare](https://www.cloudflare.com/) 进行讲解。

#### 一、域名解析
我们在申请域名后通常会将其解析到 @ 和 www 下，不论是以 cname 的方式还是以 IPV4 等其他方式。如果你希望通过 CloudFlare 来部署 https，那么需要修改一下之前的配置：

** 1. 顶级域名解析 **

将顶级域名解析到 CloudFlare 提供的以下两个 ip 上，可参考 [官方说明](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider)，这里域名厂商以万网域名为例

![](/img/articles/github-pages-https/DNS-1.jpeg)

** 2. cname 解析 **

默认网站已经添加了 cname 文件，作用是将域名 example.com 解析为 www.example.com

![](/img/articles/github-pages-https/DNS-2.jpeg)

** 3. 检测解析结果 **

可以通过以下命令进行检测，如果结果出现下图所示 ip，则说明解析已生效，否则就需要等待一段时间

```
dig +noall +answer
```

![](/img/articles/github-pages-https/DNS-3.jpeg)

#### 二、CloudFlare 注册站点信息

** 1. 扫描站点 **

目的是获取到 dns 解析信息，如果没有扫描出任何信息就需要手动添加

![](/img/articles/github-pages-https/CloudFlare-1.jpeg)

** 2. 添加解析信息 **

按照图中进行添加即可

![](/img/articles/github-pages-https/CloudFlare-2.jpeg)

** 3. 选择免费选项 **

![](/img/articles/github-pages-https/CloudFlare-3.jpeg)

** 4. 配置域名厂商 ns 信息 **

在进行了上一步之后会来到这个界面，里面提供了 name server 信息，用于域名解析服务地址

![](/img/articles/github-pages-https/CloudFlare-4.jpeg)

到域名服务厂商管理页面进行修改，替换为上图产出的 ns 地址，仍以万网为例

![](/img/articles/github-pages-https/CloudFlare-5.jpeg)

** 5. 配置 Crypto **

在 CloudFlare 上设置完上一步之后需要配置下 ssl 模式，选择 full 模式，目的是全站都走 https 协议

![](/img/articles/github-pages-https/CloudFlare-6.jpeg)

** 6. 配置 Page Rules **

一般来说只需要配置一个规则，匹配路径，强制走 https 即可，如果还有其他需求可以自己设置

![](/img/articles/github-pages-https/CloudFlare-7.jpeg)

** 7. check 结果 **

以上所有的工作都已就绪，可以访问网站试试效果，如果暂时还没有走 https，可能是需要一段生效时间，耐心等待吧，同时也可以在 overview 里查看设置信息生效情况

![](/img/articles/github-pages-https/CloudFlare-8.jpeg)

至此，你就可以看到一个完整的 [https 案例](https://itoss.me/)了
![](/img/articles/github-pages-https/website-https.jpeg)

---
title: Google AMP Access 调研
date: 2016-12-10 09:57:28
tags: [AMP, Google, Access]
---

> 本文是对Google AMP ACCESS的探索和调研，究其具体运作流程，原理，实现和各种策略！

#### 前言
AMP-ACCESS能够允许发布者对页面内容进行访问权限的控制，通过内容标记和用户访问情况进行综合评价，从而实现对付费墙和订阅功能的支持。

#### 名词解释
- Reader: 访客，浏览AMP文档的用户
- Publisher: 发布商
- AMP Runtime: 执行AMP文档的javascript运行环境
- Access Content Markup: 模块中以属性形式定义的，规定访问权限的标示
- Authorization endpoint: 授权端点
- Pingback endpoint: 计量系统调用（类似统计）的端点

#### ACCESS机制职责
- 创建和维护用户
- 控制计量（允许一定数量的自由视图）
- 负责登录
- 负责验证用户
- 负责访问权限和授权控制
- 基于访问参数进行灵活配置

#### 组成模块
- AMP Reader ID
    - 由AMP机制生成；
    - 唯一标示Reader-Publisher对；
    - 发布者用其标示每个访问者，并与身份库进行映射；
    - ID以设备进行构造，有限期为1年，或者止于清除cookie；
    - ID不能在多设备之间共享；
-  Access Content Markup
    - 由发布者定义，控制页面模块的访问权限；
    - 以html标签属性形式存在，如`<div amp-access=“access”>`；
    - 与Authorization endpoint共同决定网页模块是否展示；
- Authorization endpoint
    - 由发布者定义，描述文档的哪一部分可以使用；
    - 被AMP Cache/ AMP Runtime调用；
    - 返回访问参数，如access, subscriber等；
    - 在header的script中进行配置；
    - 协助解析markup；
- Pingback endpoint
    - 由发布者定义，描述访问状态；
    - 被AMP Cache/ AMP Runtime调用；
    - 在用户浏览页面或成功登陆之后调用；
    - 记录和更新计量信息；
    - 可以设置noPingback: true来禁止；
- Login Link and Login Page
    - 验证用户；
    - 将用户身份与AMP ID连接起来；

#### 流程

![整体功能流程图](/img/articles/google-amp/google-access-timing-diagram.png)
![整体功能时序图](/img/articles/google-amp/google-access-flow-chart.png)

- 加载页面时，如果有缓存，从AMP Cache上获取数据，否则从Server取数据。其中获取的数据中包含markeup信息；
- 请求成功之后，AMP Runtime调用Authorization endpoint机制来拿到授权参数，再跟返回数据中的markup进行比对(执行markup表达式)；表达式执行的结果（true/false）则决定是否展示该模块；
- 如果请求失败，则从返回文档的配置信息中（head->script中定义)读取配置，如果该配置信息存在，则通过配置中的authorizationFallbackResponse获取授权参数；若authorizationFallbackResponse没有被定义，则不执行markup表达式，此时模块是否展示取决于初始化文档时是否定义了amp-access-hide属性，如果定义了则隐藏，否则展示；
- AMP Runtime决定哪些模块需要展示之后将完整页面展示给用户；
在用户浏览或者登陆完成之后，AMP Runtime会调用Pingback endpoint机制来进行计量计算，该操作主要记录访问者的访问状态，如某个模块对于每个访客最多允许访问多少次，目前已经访问了多少次，这样可以保证再次访问时AMP Runtime可以做出正确的展示策略； 

#### 文档配置信息
在页面head的script中进行配置，包括以下一些信息；
- authorization：授权参数返回接口；
- pingback：计量接口；
- noPingback：是否使用pingback;
- login：登陆地址；
- authorizationFallbackResponse：如果请求失败，通过该对象自定义授权参数；
- authorizationTimeout：超时时间；
- type：平台参数，规定适用于的平台，如client，server； 

#### 安全策略
- Authorization endpoint和Pingback endpoint都是机遇CORS(跨源资源共享)机制的，需要实现相应的安全协议；

#### 计量系统
- 计量系统是一个通过访问限制来控制页面展示周期的模块，比如页面某个模块的免费访问次数为10次，在超过该计量后，会显示付费踢墙等操作；
- 计量数据的记录依赖于READER _ID，且该信息存储于服务端；
- 该信息在每次Pingback endpoint中更新；

#### 首次点击免费
- 第一次点击免费策略；

#### ACCESS如何与自己项目结合
- 要实现这一整套机制，同样需要由上述第二点的六个模块组成，并实现其各个流程；
- 协同前端+后端+CDN配合使用，来进行文档和数据的筛选和存储；
- 制定一套稳固的策略，包括展示策略，安全策略，以及通信策略；

#### 附件
- [官方文档](https://www.ampproject.org/zh_cn/docs/reference/components/amp-access#access-url-variables);
- [案例页面](https://ampbyexample.com/components/amp-access/)
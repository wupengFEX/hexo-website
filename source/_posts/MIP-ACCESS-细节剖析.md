---
title: MIP ACCESS 细节剖析
date: 2017-03-16 18:19:56
tags: [MIP, Access, 设计方案, AMP]
---

> `MIP ACCESS`是一种页面访问权限控制机制，能够允许网页发布者在页面元素中定义内容标记，并结合用户访问情况进行综合评价，从而展现或隐藏页面中内容，直至用户登录、订阅或付费后才能够查看隐藏内容的一种全新阅读模式。

### 什么是MIP ACCESS
`MIP ACCESS`是一种页面访问权限控制机制，能够允许网页发布者在页面元素中定义内容标记，并结合用户访问情况进行综合评价，从而展现或隐藏页面中内容，直至用户登录、订阅或付费后才能够查看隐藏内容的一种全新阅读模式。

### 优势在哪里？
- 方式新颖：页面中任何元素都能加入权限控制标记，并根据标记解析情况进行展示或隐藏，打破了传统阅读只展示前n字的模式；
- 形式多样：页面内容可以是任何元素，包括纯文本、图片、视频等；
- 数据驱动：内容展现与否直接与发布者配置的数据相关联，根据解析情况决定是否展示相应元素；
- 配置灵活：发布者可以根据不同的需求配置不同接口，如数据请求接口、访问记录接口、登录页面、登出页面等；

### DEMO演示

在说具体实现和细节之前，我们列举了四个`DEMO`认识一下`ACCESS`到底是一个怎样的存在吧！

- `DEMO1`演示了`ACCESS`的基本功能，该案例提供了1篇首次点击免费文章和3篇免费文章，首次点击免费是指在第一次访问该域名下具有`ACCESS`功能的页面时免费查看的功能；如果访问文章数目超过4篇时，页面中配置有`ACCESS`属性的模块将会隐藏，登录后方可查看全部；

  <img src="/img/articles/mip-access/mip-access.gif" width="30%">

- `DEMO2`演示的是登录功能，案例制定的策略是登录后所有文章均免费查看；

  <img src="/img/articles/mip-access/mip-access-login.gif" width="30%">

- `DEMO3`演示了`ACCESS`登出功能，登出后受到权限限制；

  <img src="/img/articles/mip-access/mip-access-logout.gif" width="30%">

- `DEMO4`演示了重置数据的功能，重置会删除后端数据，由各自策略而定，在重置成功后所有页面的浏览记录均被删除；

  <img src="/img/articles/mip-access/mip-access-reset.gif" width="30%">

### 名词解释
在讲具体细节之前，大家先熟悉熟悉这些专有名词吧！
- `Access Runtime`: MIP Javascript运行环境；
- `Access Content Markup`: 模块中以属性形式定义的，规定访问权限的标示；
- `Authorization endpoint`: 授权接口，返回`markup`解析数据；
- `Pingback endpoint`: 计量接口，存储访问数据；

### 使用方式
- 开发者实现接口：所有接口的请求都依据`cors`方案，包括`Authorization`接口(返回解析`DOM`元素展示与否的数据)、`Pingback`接口，登陆相关接口相关逻辑；
- 引入`mip`脚本；

    ```
    <script type="text/javascript" src="https://mipcache.bdstatic.com/static/v1/mip.js"></script>
    ```

- 定义`script`配置标签，并配置以下信息：

    ```
    <script id="mip-access" type="application/json">
    {
      "authorization": "https://publisher.com/mip-access/api/mip-authorization.json?rid=READER_ID&url=CANONICAL_URL",
      "pingback": "https://publisher.com/mip-access/api/mip-pingback?rid=READER_ID",
      "login": "https://publisher.com/mip-access/login/?rid=READER_ID&url=CANONICAL_URL",
      "authorizationFallbackResponse": {
          "error": true,
          "access": false
        },
        "type": "client"
    }
    </script>
    ```

    - authorization：授权接口，返回mip-access表达式中需要进行计算的数据；<br>
    - pingback：计量接口，每次访问页面之后，通过该url发送请求到开发者服务器，由其对数据进行管理，如每访问一次计数减1；<br>
    - noPingback：是否允许计量；<br>
    - login：登陆相关接口，可以是一个map，如下:<br>

        ```
        "login": {
             "login": "https://publisher.com/login.html?rid={READER_ID}",
             "logout": "https://publisher.com/logout.html?rid={READER_ID}"
        }
        ```

    - authorizationFallbackResponse：如果Authorization接口请求失败，需要在这里配置相关接口参数作为backup；

        ```
        "authorizationFallbackResponse": {
            "error": true,
            "access": false
        }
        ```

    - authorizationTimeout：Authorization接口请求超时时间，默认为3s；

- 以`mip-access`属性来书写表达式

    ```
    <div mip-access=“access AND subscriber”>…</div>
    ```

### 实现细节

<img src="/img/articles/mip-access/mip-access.png">

上图为纯前端方式实现`ACCESS`的时序图，下面就以这个引子来说一下`ACCESS`的工作流程吧！
- 首先用户在访问页面时，请求会发送到`CDN`（页面嵌套在`Super Frame`中）或者开发者（单个`MIP`页面）自己的服务器上，此时返回`html`文档给用户，这样做是为了让用户能够第一时间看到页面，缩短请求的白屏时间；
- 在`html`文档返回之后启动`MIP Runtime`服务，并将页面中以`mip-access-hide`属性标记的所有`DOM`元素隐藏起来，同时根据开发中提供的`Authorization`接口发起请求；
- `Authorization`接口如果请求成功，则通过其返回的数据解析`html`文档中的`markup`，解析结果为`true`则展示元素，否则隐藏；如果请求失败，`MIP Runtime`会寻找`html`文档中以`mip-access`为标示的`script`，获取其中定义好的`JSON`数据，并以`authorizationFallbackResponse`字段中的内容作为解析需要的数据；如果`authorizationFallbackResponse`未定义则解析失败；
- 页面加载完成后如果开发者配置了`noPingback: true`的选项，则不会发起`Pingback`（该请求主要为了让`server`记录访问数据）请求；否则发起请求并将数据传递给开发者`server`进行保存，待下次访问根据状态返回相应的数据；

### 适用范围
目前来说，纯前端的`ACCESS`实现方案适用与一些不涉及用户信息和收费业务相关的简单页面，通过该方式可以自由化的配置页面中元素的展现方式；出于安全考虑和后续的需要，我们也会根据需求量来以前端+`server`的处理方式过滤`html`文档；

### 写在最后
有任何问题可以到 [github issues](https://github.com/mipengine/mip-extensions/issues) 提问。
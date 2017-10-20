---
title: MIP ACCESS 实现方案
date: 2017-01-11 12:34:05
tags: [MIP, Google, Access]
---

> 作者寄语：这是一套基于Google AMP Access的解决方案，其中example代表开发方，如AMP，可以参考改方案进行细节实施！

- <a href="#前言">前言</a>
- <a href="#名词解释">名词解释</a>
- <a href="#方案解读">方案解读</a>
- <a href="#总体流程">总体流程</a>
- <a href="#请求方案">请求方案</a>
- <a href="#解析支持方式">解析支持方案</a>
- <a href="#文档merge方案">文档merge方案</a>
- <a href="#事件分析">事件分析方案</a>

<h3 id='前言'> 一、前言 </h3>
 h3-ACCESS能够允许发布者对页面内容进行访问权限的控制，通过内容标记和用户访问情况进行综合评价，从而实现对付费墙和订阅功能的支持。 

<h3 id='名词解释'> 二、名词解释 </h3>
- Reader: 访客，浏览example文档的用户
- Publisher: 发布商
- Access Runtime: 执行文档的javascript运行环境
- Access Content Markup: 模块中以属性形式定义的，规定访问权限的标示
- Authorization endpoint: 授权端点
- Pingback endpoint: 计量系统调用(类似统计)的端点
- Example: 表示开发方

<h3 id='方案解读'> 三、方案解读 </h3>
- **开发者**
  - 实现接口，所有接口的请求方式走cors，可参见<a href="#请求方案">请求方案</a>。包括授权接口(返回需要用到的markup)、计量接口（统计访问信息），登陆接口（将访客id与用户id对应，从而记录注册用户数据）相关逻辑；
  - 引入example定义的脚本；
  - 定义script配置标签，并配置以下信息：
   ```
    <script id="example-access" type="application/json">
      {
        "authorization": "https://rocky-sierra-1919.herokuapp.com/example-access/api/example-authorization.json?rid=READER_ID&url=CANONICAL_URL&ref=DOCUMENT_REFERRER&_=RANDOM",
        "pingback": "https://rocky-sierra-1919.herokuapp.com/example-access/api/example-pingback?rid=READER_ID&ref=DOCUMENT_REFERRER&url=CANONICAL_URL",
        "login": "https://rocky-sierra-1919.herokuapp.com/example-access/login/?rid=READER_ID&url=CANONICAL_URL",
        "authorizationFallbackResponse": {
            "error": true,
            "access": false
          },
          "type": "client"
      }
      </script>
     ```
     1.authorization：授权接口，返回example-access表达式中需要进行计算的数据；
     2.pingback：计量接口，每次访问页面之后，通过该url发送请求到开发者服务器，由其对数据进行管理，如每访问一次计数减1；
     3.noPingback：是否允许计量；
     4.login：登陆相关接口，可以是一个map，如下；
       ```
        ”login": {
           "signin": "https://publisher.com/signin.html?rid={READER_ID}",
           "signup": "https://publisher.com/signup.html?rid={READER_ID}"
         }
       ```
     5.authorizationFallbackResponse：如果授权接口请求失败，需要在这里配置相关接口参数作为backup；

       ```
        "authorizationFallbackResponse": {
          "error": true,
          "access": false
        }
       ```
     6.authorizationTimeout：授权接口请求超时时间，默认为3s；
  - 以example-access属性来书写表达式
	```<div example-access=“expression”>…</div>```

- **example前端**
  - 计量系统：负责每个模块展示策略统计数据的操作；在每次访问页面后，待页面加载完成，登录/登出后调用站长配置的pingback接口，将数据发送给站长服务器，由其控制计数的变化，如果计数达到站长要求的情况，则用户需登录，付费或进行其他操作，并满足要求之后才能继续访问页面，或页面中的模块；
  - 登录系统：目前只支持站长提供登录地址自行管理数据的方式，后续会调研登录的整套流程和标准，加以扩充；

- **example后端**
	- 解析系统：负责expression解析。
	- example后端需要在用户请求后返回一个过滤了example-access元素的文档，使得页面第一时间展示，过滤具体方案参照<a href="#文档merge方案">文档merge方案</a>；
	- 然后example后端调用授权接口获取相关数据，通过markup和数据共同决定是否要展示html元素，并将需要展示的返回给前端，前端进行替换加入到当前页面；

<h3 id='总体流程'> 四、总体流程 </h3>
- 1.整体时序图（如果是付费页面，通过后端数据标示，必须做到登录强相关） 
    <img src="http://upload-images.jianshu.io/upload_images/2483150-baa1c6bc470d5f3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width = "600" align=center />

   - 用户访问cache页面；
   - cache拿到的document会被server抓取到；
   - **server所要做的第一件事就是替换所有通过example-access定义的dom元素，然后返回给前端进行展示，这样做的目的是为了能让用户第一时间看到页面；**
   - **server所要做的第二件事是请求站长提供的授权接口authorization，根据返回授权数据和文档中的markup共同决定哪些模块需要显示，计算markup方式可参见<a href="#解析支持方式">解析支持方案</a>。并把需要显示的组装成html文档返回给前端，然后access runtime将其进行merge进当前页，具体merge方式可参见<a href="#文档merge方案">文档merge方案</a>；**
   - 页面加载完成后access runtime调用计量接口，将相应访问统计数据进行操作，该操作策略由站长决定，如访问次数减1，目的是再次访问该站点页面时是否有权限访问，如果没有需要通过登录，付费等操作来继续访问。整个页面的事件方案可参见<a href="#事件分析">事件分析方案</a>；
   - 登录：用户点击登录之后会调用站长提供的登录页面，如果登录成功则回调给access runtime，并刷新当前页面，页面逻辑从第一步开始；

- 2.流程图 
    <br><img src="http://upload-images.jianshu.io/upload_images/2483150-8f23bde37364b354.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width = "600" align=center />

- 3.ID系统（ID不会重复，ID是针对某个站点整站来说的）
    <br><img src="http://upload-images.jianshu.io/upload_images/2483150-f36c6456fec5e97f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width = "600" align=center />

- 4.计量系统
    <br><img src="http://upload-images.jianshu.io/upload_images/2483150-09ef1ec239419813.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width = "600" align=center />

- 5.解析系统
获取所有example-access属性的dom，然后根据自定义的表达式进行展示和隐藏。

<h3 id='请求方案'> 五、请求方案 </h3>
- 通过cors方式进行所有请求，目前百度有流量的浏览器都支持该协议；
- 请求
 - 通过`http header`中的`origin`判断必须是百度`cdn`域名或者`publisher`的`origin`;
 - 如果`http origin`不存在，`example`提供`example-Same-origin: true`来标示当前请求来自于同域名下的请求（同域名下请求不带`origin`）；如果该字段存在则允许，否则禁止访问；
 - `__example_source_origin`参数为同域名下`url`，可以通过其来替代`http header origin`不存在的情况；
- 返回
 - `Access-Control-Allow-Origin:<origin>`，允许哪些网站进行请求；不建议设置为*；
 - `example-Access-Control-Allow-Source-Origin: <source-origin>`，允许`source-origin`读取`authorization response`，通过__example_source_origin定义，他用来决定是否能够读取授权接口；
 - `Access-Control-Expose-Headers:example-Access-Control-Allow-Source-Origin`，允许`http`返回中包含该字段;

<h3 id='解析支持方式'> 六、解析支持方式 </h3>
[参见amp方式](https://github.com/ampproject/amphtml/blob/master/extensions/amp-access/0.1/access-expr-impl.jison)

<h3 id='文档merge方案'> 七、文档merge方案 </h3>
- 首先server返回首屏html文档时，将example-access标记的元素替换为example定义的一个空元素，并在元素中加入section_id属性作为标记，为的是之后合并匹配；
- 同时server将替换前的dom元素保存在后端，并发起授权请求接口；
- 拿到授权接口后执行markup表达式，将需要显示的元素组装成文档，返回给前端；
- 拿到需要merge的文档后前端通过便利这些dom并通过section_id进行匹配，匹配到的话就将其替换为返回的新dom节点，从而达到更新的效果；

<h3 id='事件分析'> 八、事件分析 </h3>
参考[AMP Analytics](https://github.com/ampproject/amphtml/blob/master/extensions/amp-access/amp-access-analytics.md)
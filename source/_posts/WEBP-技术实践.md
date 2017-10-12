---
title: webp 技术实践
date: 2017-10-12 13:36:46
tags: [MIP, WEBP]
---

不论是 PC 还是移动端，图片一直占据着页面流量的大头，开发者们也一直在图片的大小和质量之间做着权衡。而今 webp 技术的出现，也正为解决这样一个技术瓶颈，本文介绍 webp 的同时也会介绍 [MIP](https://www.mipengine.org/) 对于 webp 技术的实践之路。

## 什么是 webp？

[webp](https://developers.google.com/speed/webp/) 是由 Google 收购 On2 Technologies 后发展出来的格式，以BSD授权条款发布。目前已经在不同厂商之间进行了尝试，如Google、Facebook、ebay、百度、腾讯、淘宝等。

## 优势

webp 的效果显而易见，在支持有损、无损、透明图片压缩的同时，据统计，webp 无损压缩后比 PNG 图片体积减少了 26%，有损图片比同类 JPEG 图像体积减少了 25-34%，以下总结了 webp 在不同指标上所能获得的提升对比：

- 体积 & 流量
	- 业界改造给出的数据
	改造 webp 之后图片体积会降低很多，具体可参照 [webp 体积测试链接](https://isparta.github.io/compare-webp/index.html#12345)，同时也可参照如下图所示
	![业界 webp 测试](/img/articles/webp/webptest.png)
	- MIP 本地测试
	![本地 webp 测试](/img/articles/webp/webptest1.png)
	- 总结
		- 从以上测试可知，如果将体积换算成带宽，webp 不同模式下都会节省大量流量；
		- 科技博客 Gig‍‍‍aOM 曾报道：谷歌的 Chrome 网上应用商店采用 WebP 格式图片后，每天可以节省几 TB 的带宽。Google+ 移动应用采用 WebP 图片格式后，每天节省了 50TB 数据存储空间。
- 速度
	- 这一部分主要取决与网络时间，所以没有确定数据，不过可以参考业界的数据；
	- 科技博客 Gig‍‍‍aOM 曾报道：YouTube 的视频略缩图采用 WebP 格式后，网页加载速度提升了 10%；谷歌的 Chrome 网上应用商店采用 WebP 格式图片后，页面平均加载时间大约减少 1/3。

## 兼容性

目前来说，webp 的支持程度也在不断上升，据 2017.10.12 [can i use](http://caniuse.com/#search=webp) 显示，全球 webp 的支持程度已经达到 73.64%，

![webp 支持程度图解](/img/articles/webp/caniuse.png)

而也正是因为这种天然的图片体积优势和发展趋势，MIP 团队也决定进行初步的实践尝试，以提升页面用户体验。

## 如何判断浏览器支持程度？

webp 的判断方法官方也在[文档](https://developers.google.com/speed/webp/faq#how_can_i_detect_browser_support_for_webp)中进行了总结，大致分为 3 种方式：

### HTML5 picture

这种方法不进行 webp 支持程度的判断，而是利用 html5 picture 元素的特性，允许开发者列举出多个图片地址，浏览器根据顺序展示出第一个能够展现的图片元素，如

```
<picture>
	<source type="image/webp" srcset="images/webp.webp">
	<img src="images/webp.jpg" alt="webp image">
</picture>
```

上面的示例在浏览器不支持 webp 图片的情况下自动回退到 jpg 格式进行展示，但 picture 的支持程度还不是很完善，开发者可以根据需求决定是否进行使用。

![webp 支持程度图解](/img/articles/webp/caniuse1.png)

### 嗅探

嗅探的方式是指直接向浏览器中插入一段 base64 的 webp 图片，检测图片是否能够正常加载，依据该方法进而判断支持程度，如官方给出的嗅探函数：

```
// check_webp_feature:
//   'feature' can be one of 'lossy', 'lossless', 'alpha' or 'animation'.
//   'callback(feature, result)' will be passed back the detection result (in an asynchronous way!)
function check_webp_feature(feature, callback) {
    var kTestImages = {
        lossy: "UklGRiIAAABXRUJQVlA4IBYAAAAwAQCdASoBAAEADsD+JaQAA3AAAAAA",
        lossless: "UklGRhoAAABXRUJQVlA4TA0AAAAvAAAAEAcQERGIiP4HAA==",
        alpha: "UklGRkoAAABXRUJQVlA4WAoAAAAQAAAAAAAAAAAAQUxQSAwAAAARBxAR/Q9ERP8DAABWUDggGAAAABQBAJ0BKgEAAQAAAP4AAA3AAP7mtQAAAA==",
        animation: "UklGRlIAAABXRUJQVlA4WAoAAAASAAAAAAAAAAAAQU5JTQYAAAD/////AABBTk1GJgAAAAAAAAAAAAAAAAAAAGQAAABWUDhMDQAAAC8AAAAQBxAREYiI/gcA"
    };
    var img = new Image();
    img.onload = function () {
        var result = (img.width > 0) && (img.height > 0);
        callback(feature, result);
    };
    img.onerror = function () {
        callback(feature, false);
    };
    img.src = "data:image/webp;base64," + kTestImages[feature];
}
```

其中包含了对有损、无损、透明图、动图等 webp 图片的嗅探，这是一种最为保险的方法。不过缺点也很明显，在图片类型不一且量级较大的情况下，前端并不能知道哪些图片是有损，无损，抑或是透明的，也没用办法对其中一种特定类型做特定策略，所以即使知道不支持该类型的 webp，然而我们也没有办法主观的去做容错。所以这种方法只适合于图片类型单一，如开发者知道所有图片都是有损的，或是动图等，有针对性的去处理。

同时在处理的过程中，为了提高嗅探效率，嗅探之后可以将结果以本地存储的方式进行保存，如cookie ，方便下次直接进行调用。

### Request Header

这种方式是较为符合标准的解决方案，浏览器在支持 webp 图片格式的情况下，会在请求的 http header accept 中携带 webp/image 的字段，后端接收到请求之后可以按照该形式来判断是否返回 webp 图片内容。

MIP 在实践中采用的是该方法，当用户访问 MIP Cache 上的页面时，不需要开发者替换图片，MIP Cache 根据 http header 自动决定是否返回 webp 图片内容。不过说到坑，依然也有，国内浏览器层出不群，大部分都向标准化的方向靠近，但仍然需要一定的时间来改善，所以过程中我们就发现了这样的问题，虽然 http header accept 中包含了 webp/image 的字段，但实际上是不支持 webp 格式的（华为 MT7 自带浏览器），具体体现在动图（animation）的 feature 上。

其实解决方案也很简单，就是在 webp 图片加载失败后发起原图请求，让图片能够正确的展示在页面上。具体可以通过 img onerror 函数进行判断。

## 转换工具？

webp 的转换工具很多，主要包含了命令行和可视化界面两种：

### 命令行

官方对于每一种 webp 格式也分别提供了对应的[转换工具](https://developers.google.com/speed/webp/download)，主要包含了cwebp、dwebp、vwebp、webpmux、gif2webp 等几种，开发者可以择优选择！

### 可视化

页面也提供了不同可视化的软件进行 webp 格式图片转换，如：[iSparta](http://isparta.github.io/)。

webp 作为一种新型图片格式，不但能够节省流量，减少图片体积，一定程度上也可以优化用户体验，同时 MIP 团队也在这方面不断迭代前行，致力于打造极致的用户体验！

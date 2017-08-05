---
title: 如何构建开源项目 CI 机制
date: 2017-07-017 19:50:00
tags: [开源,构建]
---

> 很久没有发布新文章了，最近几天一直在搞开源项目持续集成机制这一块的事情，今天算是告一段落了，把自己在这几天遇到的坑和用到的工具，技术整理成了一片文章，一是作为总结，二是作为分享，希望有类似经历的开发者可以参考！

#### 一、什么是 CI?
CI 是 Continuous integratio 的缩写，中文名称是持续集成。他作为开源项目必不可少的一个环节，不但可以规范编码方式，也可以直观的提供给开发者一种项目质量的信息。相信很多人也都看到过持续集成之后的结果，他是以 README.md 的形式体现在项目中的，比如最近做的 [MIP项目](https://github.com/mipengine/mip)。

<p align='center'>
    <a href="https://github.com/mipengine/mip">
        <img src='/img/articles/mip-ci/mipci.jpeg' width=500 title='MIP 项目持续集成' alt='MIP 项目持续集成'>
    </a>


#### 二、为什么要做持续集成？
- **提高代码质量**

    前端开源项目持续集成的核心模块是单元测试，他是通过输入（Input）来执行我们所开发的代码模块，检测输出（Output）是否和预期一致的一个环节。在这一部分，最直观的结果就是代码覆盖率的指标，覆盖率越高，代码被运行的越全面，出现问题的几率越少，目前来说单元测试的框架也比较多，如较为流行的 [mocha](http://mochajs.org/)，同时我们可以借助 [circleci](https://circleci.com/) 或 [travis](https://travis-ci.org/) 等平台来帮助我们在线跑单元测试。另外单元测试的语法在一定程度上可以帮助我们规避不优雅的代码写法。对于单元测试语法，这里不讲解，大家可以通过断言库进行了解，如 [chai](http://chaijs.com/)；

- **自动化覆盖多个浏览器**

    我们所开发的单元测试，可以在多个浏览器上进行 build，可以通过这样的结果来查看自己代码在不同浏览器上是否存在问题，解决了手动测试的时间成本，可以使用的工具有 [saucelabs](https://saucelabs.com/) 或 [browserstack](https://www.browserstack.com/)；

- **实时获取代码运行状态和质量信息**
    通过在线构建平台及其生成的 badge （徽章）我们可以及时知道在提交代码后，代码构建是否通过，覆盖率提高还是降低了，具体哪个地方引起了问题，以帮助我们在出现问题后及时修改；

- **对外展示的信息**
    从开源项目的使用者来说，通过 badge 可以直观的看出一个开源项目的好坏，能够给他们一种心理暗示说我是不是敢用这个项目，项目是否活跃等。当然，如果项目质量太差的话还是不要把 badge 挂出来了，会吓走开发者的...

#### 三、持续集成构建利器

在这篇文章里会结合 Mocha + Chai + Karma + Travis + Coveralls + SauceLabs 这几种利器和大家介绍一些开源项目中如何把持续集成跑起来。

- **Mocha + Chai + Karma**

    Mocha 是一种单元测试技术，Chai 是单元测试的断言库，Karma 主要目的是为开发人员提供一个有效的测试环境，三者搭配使用会很方便的构建出一个单元测试的环境。他们既可以在本地跑单测试，也可以结合 travis，circleci等平台进行构建，下图是通过以上三种技术在本地跑单元测试的截图：

    ![](/img/articles/mip-ci/unit-test.jpeg)

- **Travis**

    [Travis](https://travis-ci.org/) 是一个跑单元测试的平台，在与 GitHub 关联之后，在每次提交代码之后就会自动触发 travis 进行代码的构建，并最终产出单测结论的 badge，如下图中的绿色图标

    ![](/img/articles/mip-ci/travis-1.jpeg)

- **Coveralls**

    [Coveralls](https://coveralls.io/) 是汇总代码覆盖率的一个平台，在单元测试完成之后，会将单元测试产出的结果上报到这个平台上，并由其最终汇总出一个总的覆盖率，并产出相应 badge。同时在该平台上我们可以看到每一个文件的覆盖率、每一次提交之后覆盖率的增减、以及具体增减是由谁引起的等等信息，我们可以根据这些信息有针对性的进行修改。

    ![](/img/articles/mip-ci/coveralls-1.jpeg)

    ![](/img/articles/mip-ci/coveralls-2.jpeg)

- **Sauce Labs**

    [Sauce Labs](https://saucelabs.com/) 在不同浏览器上运行单元测试或者网站，并产出兼容性列表的一个平台。我们可以配置自己需要的浏览器，但是通过他提供的[浏览器支持列表](https://saucelabs.com/platforms)看出，目前只能配置 Chrome, firefox, ie, safari, android, iphone等选项，在每一次单元测试完成后会将生成的数据上报到 sauce labs 平台上，我们可以在这里查看具体构建信息。

    ![](/img/articles/mip-ci/saucelabs.jpeg)

#### 四、基于这套工具如何构建持续集成环境

- **安装 mocha, karma, chai**
`npm i --save-dev mocha karma chai`

- **配置 karma.conf.js**
具体配置可以在[官方网站](http://karma-runner.github.io/1.0/config/configuration-file.html)进行查看

- **构建单元测试用例**
不在细说，大家了解单测语法后自行进行编码即可

- **本地执行单测**
本地执行单元测试，看是否可以 run 起来，如果到这一步没有问题那我们就可以将本地单元测试这套机制和各个平台进行关联

- **关联 Travis 平台**
    - 在 travis 平台上启用你需要的仓库

        ![](/img/articles/mip-ci/travis-2.jpeg)

    - 创建 travis.yml 配置文件，以下是我的一些配置，比较重要的是before_script，script和after_script，这一部分决定了你需要执行什么操作。具体信息也可以参考 [MIP travis 配置](https://github.com/mipengine/mip/blob/master/.travis.yml)。

        ![](/img/articles/mip-ci/travis-3.jpeg)

    - 然后往仓库中 commit 就可以触发 travis 的构建功能

- **关联 Coveralls 平台**
    - 在 Coveralls 上启用需要的仓库
        ![](/img/articles/mip-ci/coveralls-3.jpeg)

    - 安装 karma-coveralls
        `npm i --save-dev karma-coveralls`

    - 将 karma-coveralls 加入到 karma plugins 配置项中，他会自动生成上报所需要的 lcov 文件，具体信息可以参考 [MIP Cover 配置](https://github.com/mipengine/mip/blob/master/scripts/unit/karma.cover.conf.js)

        ![](/img/articles/mip-ci/coveralls-4.jpeg)
    - 配置上报参数，在 karma 配置文件的 reporters 中加入 coverage, coveralls 两个值即可。通过该参数，travis 构建后会自动将 lcon 数据上报到 coveralls，如果发现 travis 执行完成后在 coveralls 上没有数据，一般都是上报出现了问题，可以在 travis 中查看错误信息

- **关联 sauce labs 平台**
    - 配置 karma 参数，包括 browsers，customLaunchers，sauceLabs等，具体信息可以参考 [MIP Sauce 配置](https://github.com/mipengine/mip/blob/master/scripts/unit/karma.sauce.conf.js)
    - 配置上报参数，同样，上报是必不可少的一步，只有这一步，平台才能拿到数据，配置方法很简单，在 karma 配置文件的 reporters 中加入 saucelabs 即可

- **获取 badge**
    - 将相关 badge 加入到项目 readme 中，每一个平台都会生成 badge，可以获取 markdown, html等多种格式

#### 五、坑列表
- 单元测试可以通过，但coveralls, saucelabs平台上没有数据
毋庸置疑，基本上都是上报出现了问题，在 travis log 中查看具体错误信息

- Sauce Labs Safari 与 Coveralls 同时使用时构建超时
Saucelabs 测试浏览器包括 Safari，且执行了 Coveralls 的上报，这个时候会卡死在 Safari 单测那一步，直到超时，查看 GitHub issue 发现，这是 travis 的 bug，在使用了 istanbul 等库后就会出现该问题，解决办法是将 coveralls 和 saucelabs 任务分开执行，如 mip 将其区分为了 [karma-sauce.conf.js](https://github.com/mipengine/mip/blob/master/scripts/unit/karma.sauce.conf.js) 和 [karma-cover.conf.js](https://github.com/mipengine/mip/blob/master/scripts/unit/karma.cover.conf.js)，并通过travis 分别执行他们

    ```
    script:
      - npm run test
      - npm run test:sl
      - npm run test:co
    ```

上面的一些步骤基本就可以完成持续集成环境的构建，当然坑都是自己踩了才知道，所以动手吧骚年，有问题欢迎在评论区交流！
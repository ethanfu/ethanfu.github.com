---
layout: post
title: "portable-config-maven-plugin 与 resource filter 方式对比"
description: ""
category: tool
tags: [maven,tool]
---
{% include JB/setup %}

在这个插件项目的主页上有介绍为什么需要这个插件，不过可能好多人还和我一样有着不少的疑问，这里就尽量把这事解释的更明白一些，让疑惑少一点，如果有适合你的场景，那么这可能就是你想要的。

## 简单介绍
开发的时候总是有各种环境，典型的如dev、test、production。最原始的情况下我们处理这个情况就是打包的时候手动修改各个文件，然后在不同环境打包之后部署。由于部署是经常的事情，大部分程序员应该接受不了这个情况，可能就会使用ant来完成，准备多套环境的文件，打包的时候来完成文件替换。

后来有了 maven，来解决这个问题的话主要是用 [resource filter](http://maven.apache.org/shared/maven-filtering/) 的方式来做，主要就是利用占位符配合不同的profile执行文本的一些替换，来适应不同的环境的打包。但是看名字也能看出来，他能处理的就是resource，也就是说如果你适应不同环境的时候需要改变的文件全部都会build到classes的目录下的话，那这个插件还是比较完美的。不仅仅是支持打包的时候支持不同环境，只要你被允许，你可以在本地使用不同的profile，来运行你的应用，并且支持jetty:run的方式。

>这里解释下这里为什么单独谈到 jetty:run 。首先 jetty 支持 war 的 overlays 特性，这点 Tomcat 的插件不支持。其次就是运行的时候webapp直接指向了 /src/main/webapp ,这样你jsp等静态资源修改肯定是直接生效的，开发所必须的，当然这个特性 Tomcat 插件也一样支持的。由于overlays我暂时必须选择jetty，如果你的项目不需要这个特性，那么我说的 jetty:run ,你也可以理解为 tomcat6:run 等都是等价的。

再后来就又有了新的插件，叫 [portable-config-maven-plugin](https://github.com/juven/portable-config-maven-plugin) ，这个插件的主要优点分以下几点来说吧：

1. 不需要占位符，是根据规则匹配之后完成替换，你的版本库中存的再也不是占位符了
2. 可以替换的不仅仅是resource，神马文件你随意，格式的话有限制，主要包括properties、xml以shell，shell部分主要是变量替换，不过我觉得这个特征反而是最有竞争力的

大体基本就这些，具体的一些对比的情况我列举几个应用场景来做阐述，这样的话大家应该会更明白一些。

## 几个场景

### 场景1

**场景描述：** 项目有历史原因，配置文件等信息都放在 `/src/main/webapp/WEB-INF/conf` 这样的目录下，并且不能修改，因为修改可能牵扯的面积太广，可能让公司好多人都会不适应，成本太高没必要。在这个项目背景下你开发还得继续可以`jetty:run`来运行，毕竟jsp等静态资源的改变一般是无论如何都受不了去deploy的。

我们分别讨论使用上述描述的两种方式来应对这个场景的情况，先讨论使用 [resource filter](http://maven.apache.org/shared/maven-filtering/) 的方式。

**resource filter:**  这个方式只能针对于resource资源(不了解的参考下面的特殊说明)，那么就是说我们这些配置文件完全不能使用该插件。也就是说这种场景，使用 [resource filter](http://maven.apache.org/shared/maven-filtering/) 的方式必须得配合war的filter来做，这样war打包后的文件是被替换的，不过由于存在占位符，你如果直接 jetty:run ，那项目一定是无法启动的了，占位符在那里呢。即便你把jetty的webapp目录更改到target下面的那个文件夹，那说明还必须得执行package还行，jsp等静态资源是没办法及时生效的，所以这个时候其实是多么迫切的需要一个不需要占位符来实现多个环境的插件啊。

> resource资源，这个估计可能会有人不太明白，这里说下我的理解，按照 maven 的解释，resource 主要分为 main resource 和 test resource 。区分两种 resource 主要是为了源代码和单元测试来区分开，来应对部分配置可能会不一样的情况。那么这样看来我的理解就是最终只要这个文件是resource，最终文件都会在编译的时候都会被放在 target/classes 和war项目的 WEB-INF/classes 中。这样的话我们 /src/main/webapp/WEB-INF/conf 的任意文件都不能作为 resource，做了的话那么这些文件都被打包到上面说的目录了，当然有个targetPath的属性配置，那个是配置webapp项目的根目录，不是你的resource放到哪里去的设置。

**portable-config-maven-plugin:** 这个就利用到了我上面提到的比较有竞争力的特征了，当然第一个特征也是必须的，结合起来处理这个场景简直是几乎完美，当然我只是用了几乎，因为还有点残缺，残缺部分等会说。

因为可以替换任意文件，那么我们这个场景下的目录自然不是问题。支持的文件类型足以支撑大部分需求，我们主要就是配置 xml 以及 properties 文件，shell 好多项目可能不太需要替换，因为可能就没有。不过这个插件主要就是在package 的时候去替换war文件内容，并不能替换源码目录的， `/src/main/webapp/` 下面的任意文件都是不会有任何改变的，如果你使用 jetty:run 的方式来运行项目，依赖于该插件的不使用占位符替换而是直接配置dev开发环境的方式是没有问题的。只是不能在本地直接使用其他profile来运行项目而已，使用其他环境就应该部署到相应的环境。

上面提到了几乎完美，这里也把我的认识说下，基于这个插件开发的时候，团队的版本库中配置文件的配置应该是个人dev环境，这样项目只要checkout下来每个人都是直接可以运行。这样就有个约定就是团队每个开发人员的数据库名称以及用户名密码等信息都保持完全一致，如果团队违反这个约定的话只能是每个人根据自己实际环境来调整下了。但是在打包去build的时候这个插件是完美支持这个场景的，如果你也是这个场景，那么这个插件几乎是你的最省时省力还省心的选择，推荐你使用。

### 场景2
**场景描述:** 项目完全符合 maven 约定，并且为了适配各个环境需要修改的所有的配置文件都在 classpath 下面，运行肯定和上个场景是一样的，开发嘛，要求的是少一些麻烦。

依然和上面一样，两种方式一个一个说：

**resource filter:** 这个方式在处理这个场景还是非常棒的，即便你团队每个人的开发环境都不一样，也一样轻松应对。
**portable-config-maven-plugin:** 这个场景下其实相比上一种方式应该要落下风的，区别就是占位符和不是占位符，个人感觉本质上区别不大。

### 场景3
**场景描述:** 假设一个上述两个场景的结合体，在这种情况下其实使用的话需要根据个人喜好了，不过最简单有效的方式应该还是 portable-config-maven-plugin 来处理你第一个场景的方式，resource的使用filter来做，现在 portable-config-maven-plugin 还暂时不支持resource部分的替换，不过可能很快就支持了，如果支持的话应该只是使用 portable-config-maven-plugin 就足够了。

## 最后

我现在遇到的问题典型是上面描述的场景3，一个结合体，给了我不少困扰，不过还好能把这些搞清楚了，并且还又在我最需要的时候 [juven xu](http://juvenxu.com/) 还发布了一个很合适的插件，非常感谢。好似在我发文之际这个插件又发布了一个新版本，大家有需要的就赶紧试试吧。如果有对这个插件的意见或者反馈就赶紧去[项目](https://github.com/juven/portable-config-maven-plugin)的页面去交流，有大家的使用和交流反馈产品才会变得更加完善。
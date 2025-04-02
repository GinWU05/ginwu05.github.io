---
title: Kibana二开经历分享
date: 2025-04-02 14:47:21
categories:
  - 技术
  - 前端开发
tags:
  - Kibana
  - 二次开发
  - 经验分享
---

## 前言

这只是一次经历分享，大家当故事听听就行，知识密度不会那么高。
不是kibana、elasticsearch的技术教程，是一个以技术主题的职场故事。
这个任务给我的印象实在是深，是技能面和人力调度最广的一次。
几年前的事情了，有些细节可能有所偏差，错误之处还请指出。

<!-- more -->

## 背景

2021年，我是工龄2.5年的前端开发，公司卖网络安全产品给其他公司客户。彼时已经是第二条产品线了，前端还是做后台管理web(react+umi)。
基层研发团队完整，高级研发人员有：创始人（老板，架构师级别），技术总监(公司技术天花板)，部门老大（团队负责人，网络安全专家）。
公司有自己维护的机房，使用VMware vSphere虚拟化技术灵活分配服务器系统资源，给整个研发团队使用，由运维同事管理。
至少3个项目环境：开发、测试、生产。

那天，PM(产品经理)找我：我们要把kibana的界面，嵌入到我们的后台管理。

我：...（短暂的思考） 为啥？ 意义是什么？ 谁做的技术选型？

PM: 老板决定的，因为我们现在的产品数据读写性能太差了，数据库要换成elasticsearch；然后老板也不希望前端再自己手动写界面，直接用kibana的界面就行了。

内心os：从要解决的痛点分析，kibana 不一定是一个好的方案，但这是老板的决定，我似乎没办法拿出一个更好的方案。我可以做，但效益不一定有预期那么好。

## 起步

好的，任务明确，部门前端两个，我自己工龄不如同事高，他做事情比较稳，但我更喜欢研究折腾玩一些新玩意。我愿意做这个任务，跟同事简单讨论，他表示ok。
开干呗，但我不知道啥是kibana、elasticsearch，肯定得先查资料，看文档，先简单了解一下。

```
Kibana 是一个强大的数据分析和可视化工具，它允许用户通过 ES|QL 进行数据查询和分析，支持多种数据源和类型，提供了一系列的特性和工具来加速数据洞察、解决问题、监控系统、进行安全检查以及提升搜索引活性。
---
Elasticsearch 是一个开源的分布式 RESTful 搜索和分析引擎，它不仅是 Elastic Stack 的核心，而且是一个高性 forming 数据存储和向量基数据库，能够满足各种使用场景的需求，包括 AI 搜索和分析。
```

emm... 要我自己讲，kibana是运维日志面板工具，可以展示来自Elasticsearch的数据；Elasticsearch是一个快速搜数据的引擎，类似数据库能存存储和查询数据。

叠甲：这是一个简单粗暴的理解，对于陌生的概念工具，这样理解可以快速上手。

**预期实现效果：自家产品的后台管理web，嵌入kibana页面，kibana是连接了Elasticsearch实例。**

方向也明确，嵌入可以用html的`iframe`技术；但kibana页面，需要其服务实例，我可以看文档资料，自己搭起来；
如何连接Elasticsearch实例，这需要跟后端要一下，但我也最好提供配置文件，需要啥值让后端直接告诉我，这样他们比较省事。

跟PM沟通了一下，这个任务没那么简单，我不好预估时间，而且这些都是新概念，我需要一些时间成本学习。而且个人认为，就算实现了老板的想法，效果不一定有老板预期那么理想。
但老板在公司有完全的决策权，我不会反驳他。还是会按照他的想法，去推动任务。你知道这个风险还有我的意见就行。

PM:放心去做就行。

能行，开搞。

## 创建实例

[Install Kibana with Docker](https://www.elastic.co/guide/en/kibana/7.17/docker.html#run-kibana-on-docker-for-dev)

我玩过服务器还有docker，跑个kibana实例并不难。直接在产品的测试环境服务器（虚拟机）执行吧。预估我后面的操作，都是可以撤回且不影响产品的测试环境。
连接Elasticsearch实例，需要配置`ELASTICSEARCH_HOSTS`的值，直接带着文档资料， 找后端同事让其提供就行。

就这样顺利有了kibana实例，也连接了Elasticsearch实例。工作电脑能访问产品的测试环境。

![kibana首页参考](https://www.elastic.co/guide/en/kibana/7.17/images/analytics-home-page.png)

歪头... 我搞这个实例，是需要搞页面，嵌入到自家产品的后台管理web。
但我现在得到了一个管理平台。如果老板的选型没问题，我应该是可以借助管理平台，配置出页面。
老板既然这么提了，估计是已经看过实际案例，我不需要质疑他，继续往下推动就完事。

## 配置页面

继续翻文档，了解kibana的概念，得出我需要的“零件”，组装成页面：
- 配置索引模式（Index Pattern）
- 使用 Discover 查看数据
- 创建可视化图表（Visualizations）
- 创建仪表板（Dashboard）
- 构建仪表盘（Dashboard）

[visualize-and-analyze](https://www.elastic.co/guide/en/kibana/7.17/introduction.html#visualize-and-analyze)

所以，可视化图表（Visualizations）就是页面的组成元素，仪表盘（Dashboard）就是页面。

[share-a-direct-link](https://www.elastic.co/guide/en/kibana/7.17/reporting-getting-started.html#share-a-direct-link)

Dashboard可以用链接的方式分享，这是**这次任务需要的kibana页面**。

我已经掌握配置kibana的页面了，现在需要回归需求文档，看看需要我配置出具体如何的kibana页面，展示什么数据。
然后用iframe引用dashboard的链接，嵌入到自家产品的后台管理web就行了。

（此处省略一些我了解Elasticsearch的细节，还有产品项目的具体的业务index pattern。 也花费了不少时间，需要找后端了解我页面上的xx模块，这里的数据从哪个index pattern获取）

配置出页面后，iframe嵌入也是小问题，没啥好讲的。隐约记得有跨域问题，我改过前端nginx的配置，为kibana配置了反向代理，当然请求了后端、运维同事的协助。他们在这块比较专业。

```nginx
location /kibana {
    proxy_pass http://192.168.1.100:5601;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

至此开发任务到这里收尾，找PM验收，然后上线就是把测试环境的东西，搬到生产环境，就完成上线了。

## 样式改造

PM（产品经理）：老板说，我们的kibana页面太丑了。
我：额... 那要怎么调整。
PM：ui已经设计好了，你按照这个设计稿调整就行。
我：（打开设计稿链接，沉思了一小会...）

**要改外部的url的样式，要么在自家产品页面，叠个样式层，直接覆盖；kibana是开源的，要么在kibana的源码上改，然后重新打包。**

内心OS：前者技术成本小，但支持度可能有限，而且kibana在iframe中，自家产品的样式层，可能无法影响到kibana的样式；
后者技术成本大，但有了源码，我可以自由更改，做任何样式调整。

我：改吧。我想办法。

PM:加油。😉

我更倾向用第一种方案，方便快速。但可能没办法完全还原设计稿。第二种方案，成本高，但样式肯定能还原。
优先用第一种方案，看老板的反馈，如果不行，就只能用第二种方案了。

（花了两三天时间，改了自家产品的样式，但效果不理想，PM反馈老板说还是丑。）

## 二次开发

没办法了，只能去碰kibana的源码了。虽然我并不想这么做。还是跟PM讲我的技术方案，风险告知...

PM:继续加油。😉

...


## 总结

我没有提到时间节点，但总体大概花费了两个月，中间省略了我很多走弯路的细节。
中间也有一些跟PM沟通，比如确定效果，某些功能是否需要。
这些不重要，故没提及。

<!--
https://www.elastic.co/guide/en/kibana/7.17/docker.html#run-kibana-on-docker-for-dev

-->
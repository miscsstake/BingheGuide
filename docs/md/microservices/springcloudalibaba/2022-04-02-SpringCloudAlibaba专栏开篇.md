---
layout: post
category: binghe-code-springcloudalibaba
title: 第01章：专栏开篇
tagline: by 冰河
tag: [springcloud,springcloudalibaba,binghe-code-springcloudalibaba]
excerpt: 很多小伙伴留言说，冰河你能不能写一些关于Java8的文章呢，看书看不下去，看视频进度太慢。好吧，看到不少读者对Java8还是比较陌生的，那我就写一些关于Java8的文章吧，希望对大家有所帮助。
lock: need
---

# SA实战 ·《SpringCloud Alibaba实战》第01章-专栏开篇啦


作者：冰河
<br/>星球：[http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs)
<br/>博客：[https://binghe.gitcode.host](https://binghe.gitcode.host)
<br/>文章汇总：[https://binghe.gitcode.host/md/all/all.html](https://binghe.gitcode.host/md/all/all.html)

**大家好，我是冰河~~**

今天是2022年清明假期的前一天，前段时间一直有小伙伴问我啥时能出一个SpringCloud Alibaba实战的系列文章。最近一段时间，冰河手上的事情确实是太多了，有点忙不过来。

细心的小伙伴可能会发现，我在 **冰河技术** 公众号的发文频率降低了，貌似原创频率也比之前低了点，确实是这段时间太忙了导致了，后面再和小伙伴们聊聊冰河最近都在干啥吧。不过，最近终于能够腾出时间为小伙伴们分享一期《SpringCloud Alibaba》实战的专栏了。

说的有点远了，扯回正题吧。哦对了，忘记跟大家说了，标题中的SA是SpringCloud Alibaba的简称，这是冰河嫌SpringCloud Alibaba这个名字太长了，自己姑且算是给它起了个小名（简称）吧，大家能够看懂就行。

![](https://binghe.gitcode.host/assets/images/microservices/springcloudalibaba/sa-2022-04-02-001.jpg)

## 专栏整体结构

废话不多说了，我们先来看看《SpringCloud Alibaba实战》专栏的整体结构吧，先上图。

![](https://binghe.gitcode.host/assets/images/microservices/springcloudalibaba/sa-2022-04-03-002.png)

从上图，大家可以看到，专栏从整体上分为十个大的篇章，分别为 **专栏设计、微服务介绍、微服务环境搭建、服务治理、服务容错、服务网关、链路追踪、消息服务、服务配置、分布式事务**。 

每个大的篇章下面会按照实际需要划分为一个或者多个章节。并且整个专栏会以一个典型电商系统的用户、商品、订单模块为例，贯穿整个专栏，每个章节都会以SpringCloud Alibaba提供的组件进行实现。让大家在学习《SpringCloud Alibaba实战专栏》的时候，真正能够深入理解以微服务开发实际项目，并且能够实战开发项目，进而能够灵活运用到公司的实际项目中。

只是停留在理论基础是远远不够的，学到的知识能够拿来实战，解决实际问题才是最重要的。

## 专栏形式

整个专栏以案例驱动的实行进行讲解，并且将案例贯彻整个专栏的始末。这里，经过不断思考，选择了大家都比较熟悉的电商系统的用户、商品和订单模块作为案例进行讲解。

并且每个章节总体上会按照下图的形式进行讲述。

![](https://binghe.gitcode.host/assets/images/microservices/springcloudalibaba/sa-2022-04-03-003.png)



每个章节的内容形式上先抛出某个问题，针对这个问题给出解决的思路，再探讨业界的一些通用解决方案，针对本专栏的定位结合阿里组件解决问题，接下来就是代码实战，最后针对当前章节给出其他必要的总结和说明。

## 适应群体

学习《SpringCloud Alibaba实战》最好是有一定的Java编程基础和SpringBoot基础，最好还具备一些SpringCloud的基础知识。当然，如果你对SpringCloud Alibaba比较熟悉了，学习本专栏也一定不会让你失望。本专栏几乎会涵盖SpringCloud Alibaba的各种组件，并且会将这些组件运用到实战中开发实际项目，相信学完此专栏，大家都能够对SpringCloud Alibaba有个全新的认识。

## 资源安排

本专栏涉及到的资源：包含源代码和画图源文件，以及涉及到的环境搭建使用的安装包等，都会放到 **冰河技术** 知识星球中，目前， **冰河技术** 知识星球内容正在完善中，后续会向大家开放。

## 写在最后

正所谓：**纸上得来终觉浅，绝知此事要躬行** 。学习技术停留在理论的基础上是远远不够的，我们一定要将学到的技术和知识运用到实际的项目当中，解决实际的问题才是最重要的。大家在学习本专栏的过程中，遇到任何问题都可以向冰河咨询。

切记：本专栏以实战内容为主，在学习的过程中一定要多动手，多实操，多思考，多总结。好了，让我们一起开始《SpringCloud Alibaba实战》专栏的学习吧。

**好了，今天就到这儿吧，我是冰河，我们下期见~~**

> 如果你觉得冰河写的还不错，请微信搜索并关注「 **冰河技术** 」微信公众号，跟冰河学习高并发、分布式、微服务、大数据、互联网和云原生技术，「 **冰河技术** 」微信公众号更新了大量技术专题，每一篇技术文章干货满满！不少读者已经通过阅读「 **冰河技术** 」微信公众号文章，吊打面试官，成功跳槽到大厂；也有不少读者实现了技术上的飞跃，成为公司的技术骨干！如果你也想像他们一样提升自己的能力，实现技术能力的飞跃，进大厂，升职加薪，那就关注「 **冰河技术** 」微信公众号吧，每天更新超硬核技术干货，让你对如何提升技术能力不再迷茫！


![](https://img-blog.csdnimg.cn/20200906013715889.png)
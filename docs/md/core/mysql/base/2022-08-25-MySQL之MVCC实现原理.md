---
layout: post
category: binghe-mysql-base
title: 【置顶】MySQL之MVCC实现原理
tagline: by 冰河
tag: [mysql,binghe-mysql-base]
excerpt: 【置顶】MySQL之MVCC实现原理
lock: need
---

# 【置顶】MySQL之MVCC实现原理

**大家好，我是冰河~~**

有很多小伙伴不太了解到底什么是MVCC，今天就跟大家一起来透彻的分析下，好了，咱们直接肝正文吧。

## 什么是MVCC

MVCC，全称Multi-Version Concurrency Control，即多版本并发控制。MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。

我们知道，一般情况下我们使用MySQL数据库的时候使用的是Innodb存储引擎，Innodb存储引擎是支持事务的，那么当多线程同时执行事务的时候，可能会出现并发问题。这个时候需要一个能够控制并发的方法，MVCC就起到了这个作用。

## MySQL的锁和事务隔离级别

在理解MVCC机制的原理之前，需要先理解MySQL的锁机制和事务的隔离级别，抛开MyISAM存储引擎不谈，就Innodb存储引擎来说，分别有行锁和表锁两种锁，表锁就是一次操作锁住整张表，这样锁的粒度最大，但是性能也最低，不会出现死锁。行锁就是一次操作锁住一行，这样锁的粒度小，并发度高，但是会出现死锁。

Innodb的行锁又分为共享锁（读锁）和排它锁（写锁），当一个事务对某一行加了读锁时，允许其他事务对这一行进行读操作，但是不允许进行写操作，也不允许其他事务对这一行执行加写锁，但是可以加读锁。

当一个事务对某一行加了写锁时，不允许其他事务对这一行进行写操作，但是可以读，同时不允许其他事务对这一行加读写锁。
下面来看一下MySQL的事务隔离级别，分为以下四种：

**读未提交：** 一个事务可以读到其他事务还没有提交的数据，会出现脏读。举个例子，有一张工资表，事务A先开启，然后执行查询id为1的员工的工资，假设此时的工资为1000，此时，事务B也开启，执行了更新操作，将id为1的员工工资减少了100，但是并未提交事务。此时再执行事务A的查询操作，可以读到事务B已经更新的数据，如果此时事务B发生回滚，事务A读到的就是“脏”数据。当事务A执行更新操作的话还可能产生幻读的情况。

**读已提交：** 一个事务只能读到另一个已经提交的事务修改过的数据，并且其他事务每对该数据进行一次修改并提交后，该事务都能查询得到最新值。还是同样的例子，这次的事务隔离级别为读已提交的情况下，事务B不提交事务的情况下，事务A无法读到事务B更新后的数据，也就避免了脏数据产生。但是，当事务B提交之后，事务A再执行相同的数据，会发现数据变了，这就是所谓的不可重复读，意思就是同一个事务中多次执行相同的查询得到的结果不一致，同时，幻读的情况还是存在。

**可重复读：** 一个事务第一次读过某条记录后，即使其他事务修改了该记录的值并且提交，该事务之后再读该条记录时，读到的仍是第一次读到的值，而不是每次都读到不同的数据，这就是可重复读，这种隔离级别解决了不可重复，但是还是会出现幻读。

**串行化：** 这种隔离级别因为对同一条记录的操作都是串行的，所以不会出现脏读、幻读等现象，但是这也就不是并发事务了。

## MySQL的undo log

MVCC底层依赖MySQL的undo log，undo log记录了数据库的操作，因为undo log是逻辑日志，可以理解为delete一条记录的时候，undo log会记录一条对应的insert记录，update一条记录的时候，undo log会记录一条相反的update记录，当事务失败需要回滚操作时，就可以通过读取undo log中相应的内容进行回滚，MVCC就利用到了undo log。

## MVCC的实现原理

MVCC的实现，利用到了数据库的隐式字段，undo log和ReadView。首先来看隐式字段，其实MySQL在表中的每行记录的后面，都隐式的记录了DB_TRX_ID（最近修改(修改/插入)事务ID），DB_ROLL_PTR（回滚指针，指向这条记录的上一个版本），DB_ROW_ID（自增ID，如果数据表没有主键，则默认以此ID简历聚簇索引）这几个隐藏的字段。

undo log分为两种，分别为insert undo log，在insert新记录时产生的undo log, 只在事务回滚时需要，并且在事务提交后可以被立即丢弃，还有update undo log，事务在进行update或delete时产生的undo log; 不仅在事务回滚时需要，在快照读时也需要；所以不能随便删除，只有在快速读或事务回滚不涉及该日志时，对应的日志才会被purge线程统一清除。MVCC利用到的是update undo log。

实际上undo log记录的是一个版本链，假设数据库中有一条记录如下：

![2022-08-25-001](https://binghe.gitcode.host/assets/images/core/mysql/base/2022-08-25-001.png)

现在有一个事务A修改了这条记录，把name改为tom，这个时候的操作流程为：

（1）事务A首先对该行记录加上行锁

（2）将该行记录拷贝到undo log中，作为一个旧的版本

（3）拷贝完之后将该行name修改为tom，然后将该行的DB_TRX_ID的值改为事务A的id，此时假设事务A的id为1，将该行的DB_POLL_PTR指向拷贝到undo log的那条记录

（4）事务提交后，释放锁

此时的情况如下：

![2022-08-25-002](https://binghe.gitcode.host/assets/images/core/mysql/base/2022-08-25-002.png)

此时又有一个事务B来修改这条记录，把age改为28，这时候的操作流程为：

（1）事务B对改行记录加上行锁

（2）将该行记录拷贝到undo log中，作为一个旧的版本，此时发现undo log已经有记录了，那么新的一条undo log作为链表的表头插入到该行记录的undo log的最前面。

（3）拷贝完后将该行的age改为28，然后将该行的DB_TRX_ID的值改为事务B的id，此时假设事务B的id为2，将该行的DB_POLL_PTR指向拷贝到undo log的那条记录。

（4）事务提交后释放锁。

此时的情况如下：

![2022-08-25-003](https://binghe.gitcode.host/assets/images/core/mysql/base/2022-08-25-003.png)

从上面我们可以看到，不同的事务或者相同的事务对同一行记录进行的修改，会使得该行记录的undo log形成一个版本链，undo log的链首就是最近一次的旧记录，而链尾就是最早一次的旧记录。

现在我们来假设一种情况，先假设事务A和事务B都没有提交，这时候有一个事务C，修改了name为tom的记录，把age改成了30，然后把事务提交，事务C的id为3，同样的，会插入一条记录到undo log中，此时的undo log版本链链首记录的DB_TRX_ID为3。

现在有一个事务D，查询name为tom的记录，此时将会启用快照读，快照是事务开始由查询操作触发的一个数据快照，不加锁的读在可重复读隔离级别下默认就是快照读，相对于快照读还有一个叫做当前读，更新操作都是当前读。在快照读时会产生一个读视图（Read view），在该事务执行快照读的那一刻，会生成数据库当前的一个快照，记录并且维护当前活跃的事务的ID，因为事务的ID都是自增的，所以越新的事务ID越大。读视图遵循可见性算法，而是否可见则需要做一些判断，读视图中除了记录当前活跃的事务ID以外，还记录了当前创建的最大事务ID，快照读时需要和Read view做比较来获得可见性结果。

Read view主要是把当前事务的ID，和系统中的活跃事务的ID作比较，比较的规则如下：

首先，Read view中会有一个Read view生成时刻系统中活跃的事务ID的数组，暂称为id_list。

然后Read view中会记录一个id_list中最小的事务ID，暂称为low_id。

最后Read view中还会记录一个Read view生成时刻系统中尚未分配的事务ID，也就是当前最大的事务ID+1，暂称为high_id。

* 当前事务ID如果小于low_id，则当前事务可见。
* 当前事务ID如果大于high_id，则当前事务不可见。
* 当前事务大于low_id小于high_id，再判断是否在id_list中，如果在，说明活跃的事务还没提交，当前事务不可见，但是对于活跃的事务本身可见，如果不在id_list中，则当前事务可见。

如果可见性结果为不可见的话，需要通过DB_ROLL_PTR到undo log中取出该记录的DB_TRX_ID进行比较，通过遍历版本链，直到找到满足特定条件的DB_TRX_ID, 那么这个DB_TRX_ID所在的旧记录就是当前事务能看见的最新老版本。

<p align="right"><font size="1">转自：blog.csdn.net/MortShi/article/details/121890348<font></p>

**好了，如果文章对你有点帮助，记得给冰河一键三连哦，欢迎将文章转发给更多的小伙伴，冰河将不胜感激~~**

## 关于星球

**冰河技术** 知识星球《RPC手撸专栏》已经开始了，我会将《RPC手撸专栏》的源码放到知识星球中，同时在微信上会创建专门的知识星球群，冰河会在知识星球上和星球群里解答球友的提问。

### 星球提供的服务

冰河整理了星球提供的一些服务，如下所示。

加入星球，你将获得： 

1.学习从零开始手撸可用于实际场景的高性能、可扩展的RPC框架项目

2.学习SpringCloud Alibaba实战项目—从零开发微服务项目 

3.学习高并发、大流量业务场景的解决方案，体验大厂真正的高并发、大流量的业务场景 

4.学习进大厂必备技能：性能调优、并发编程、分布式、微服务、框架源码、中间件开发、项目实战 

5.提供站点 https://binghe.gitcode.host 所有学习内容的指导、帮助 

6.GitHub：https://github.com/binghe001/BingheGuide - 非常有价值的技术资料仓库，包括冰河所有的博客开放案例代码 

7.可以发送你的简历到我的邮箱，提供简历批阅服务 

8.提供技术问题、系统架构、学习成长、晋升答辩等各项内容的回答 

9.定期的整理和分享出各类专属星球的技术小册、电子书、编程视频、PDF文件 

10.定期组织技术直播分享，传道、授业、解惑，指导阶段瓶颈突破技巧

### 如何加入星球

* **链接** ：打开链接 [http://m6z.cn/6aeFbs](http://m6z.cn/6aeFbs) 加入星球。
* **回复** ：在公众号 **冰河技术** 回复 **星球** 领取优惠券加入星球。

**特别提醒：** 苹果用户进圈或续费，请加微信 **hacker_binghe** 扫二维码，或者去公众号 **冰河技术** 回复 **星球** 扫二维码加入星球。

![sa-2022-04-21-007](https://binghe.gitcode.host/assets/images/microservices/springcloudalibaba/sa-2022-04-28-008.png)

**如果图片二维码过期，去公众号 冰河技术 回复 星球 扫二维码加入星球, 好了，今天就到这儿吧，我是冰河，我们下期见~~**

## 写在最后

> 如果你觉得冰河写的还不错，请微信搜索并关注「 **冰河技术** 」微信公众号，跟冰河学习高并发、分布式、微服务、大数据、互联网和云原生技术，「 **冰河技术** 」微信公众号更新了大量技术专题，每一篇技术文章干货满满！不少读者已经通过阅读「 **冰河技术** 」微信公众号文章，吊打面试官，成功跳槽到大厂；也有不少读者实现了技术上的飞跃，成为公司的技术骨干！如果你也想像他们一样提升自己的能力，实现技术能力的飞跃，进大厂，升职加薪，那就关注「 **冰河技术** 」微信公众号吧，每天更新超硬核技术干货，让你对如何提升技术能力不再迷茫！


![](https://img-blog.csdnimg.cn/20200906013715889.png)

## 加群交流

本群的宗旨是给大家提供一个良好的技术学习交流平台，所以杜绝一切广告！由于微信群人满 100 之后无法加入，请扫描下方二维码先添加作者 “冰河” 微信(hacker_binghe)，备注：`学习加群`。



<div align="center">
    <img src="https://binghe.gitcode.host/images/personal/hacker_binghe.jpg?raw=true" width="180px">
    <div style="font-size: 9px;">冰河微信</div>
    <br/>
</div>



## 公众号

分享各种编程语言、开发技术、分布式与微服务架构、分布式数据库、分布式事务、云原生、大数据与云计算技术和渗透技术。另外，还会分享各种面试题和面试技巧。

<div align="center">
    <img src="https://img-blog.csdnimg.cn/20210426115714643.jpg?raw=true" width="180px">
    <div style="font-size: 9px;">公众号：冰河技术</div>
    <br/>
</div>


## 星球

加入星球 **[冰河技术](http://m6z.cn/6aeFbs)**，可以获得本站点所有学习内容的指导与帮助。如果你遇到不能独立解决的问题，也可以添加冰河的微信：**hacker_binghe**， 我们一起沟通交流。另外，在星球中不只能学到实用的硬核技术，还能学习**实战项目**！

关注 [冰河技术](https://img-blog.csdnimg.cn/20210426115714643.jpg?raw=true)公众号，回复 `星球` 可以获取入场优惠券。

<div align="center">
    <img src="https://binghe.gitcode.host/images/personal/xingqiu.png?raw=true" width="180px">
    <div style="font-size: 9px;">知识星球：冰河技术</div>
    <br/>
</div>
---
   layout:     post
   title:      Axon Framework框架
   subtitle:   开发积累
   date:       2018-11-22
   author:     guchaolong
   header-img: img/post-bg-re-vs-ng2.jpg
   catalog: true
   category: 框架
   tags:
       - Axon
---
>Axon学习笔记  
> [资料](https://skyao.gitbooks.io/leaning-axon/content/concept/)
>


1. ## 介绍
Axon FrameWork：用来构建基于 __命令查询责任分离设计模式__ 的可伸缩、可扩展、可维护的框架  
命令查询责任分离（CQRS):Command Query Responsibility Sgregation

1. ## 概念
    DDD:领域驱动设计  
    1. 领域对象 Domain Object
    1. 实体 Entity
    1. 领域事件 Domain Event
    1. 聚合 Aggregate（是一个抽象概念，不是一个实体）
    1. 聚合根 Aggregate Root 一个实体
    1. 存储库 Repository
    关于聚合根
    ![aggregate](../_posts_images/aggregate.png)
    
1. Axon
    1. Command  
        1. 一个简单的对象POJO ，包含Command Handler 执行这个command时需要的数据
        2. 名字表示命令的意图    
    1. Command Bus
        1. 接收Command，并将他们路由给Command Handler，每个Command Handler负责一个特定的Command类型，根据command的内容执行相应的操作
        2. Command Handler可以在Command Bus上订阅和退订特定类型的command
    3. Command Handler
        1. 负责处理command，每个handler只负责处理一种类型的command
        2. Command Handler 从 repository 中获取领域对象(Aggregates)并执行他们的方法来修改他们的状态,Aggregates 的状态变更导致 Domain Events 的生成。Domain Events 和 Aggregates 组成 domain model / 领域模型
    



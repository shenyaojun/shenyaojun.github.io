---

layout:     post
title:      非Hadoop生态高性能OLAP分析引擎ClickHouse
subtitle:   BI+OLAP分析首选
date:       2022-05-23
author:     就是我啦
header-img: img/post-bg-ioses.jpg
catalog: 	  true
tags:
    - OLAP    
    - ClickHouse    
    - 宽表    
    - 列式存储
    - BI
    - MPP
    - Yandex
---

# 非Hadoop生态高性能OLAP分析引擎ClickHouse

> 最迷惑人也最没用的就是MPP概念，MPP的原意是“大规模并行处理”，但由于一些历史原因，现在当人们说到MPP架构时，它们实际上指代的是“分布式数据库”，而Hadoop架构指的则是以Hadoop项目为基础的一系列分布式计算和存储框架。不过由于MPP的字面意思，现实中还是经常有人纠结两者到底有什么联系和区别，两者到底是不是同一个层面的概念。

```sh
# ClickHouse，大数据量，十亿百亿起步，千亿万亿正常
Hadoop 组件太多了，ClickHouse用起来就方便的多
```

### ClickHouse 简介

**Click** flow data ware**house**

ClickHouse 是由俄罗斯的第一大搜索引擎 **Yandex** 公司开源的**列存数据库**。令人惊喜的是，ClickHouse 相较于很多商业 MPP 数据库，比如 Vertica，InfiniDB 有着极大的性能提升。除了 Yandex 以外，越来越多的公司开始尝试使用 ClickHouse 等列存数据库。对于一般的分析业务，结构性较强且数据变更不频繁，可以考虑将需要进行**关联的表打平成宽表**，放入 ClickHouse 中。
相比传统的大数据解决方案，ClickHouse 有以下的优点：

* 配置丰富，只依赖于 Zookeeper
* **不依赖于大数据生态**
* **列式存储**
* 线性可扩展性，可以通过添加服务器扩展集群
* 容错性高，不同分片间采用异步多主复制
* **单表性能极佳**，采用向量计算，支持采样和近似计算等优化手段
* 功能强大支持多种表引擎

其他特点：

* 单次数据量大的查询有优势，但**QPS不要太高**
* Join性能差，**尽量少join**，可以预先打平成宽表
* 适合**OLAP**，不适合OLTP
* 不支持事务
* 不擅长删除
* 不擅长更新
* 硬件层次优化：向量化执行引擎



这个B站视频，概念讲的很好

[https://www.bilibili.com/video/BV1g341157kn](https://www.bilibili.com/video/BV1g341157kn)

[https://www.bilibili.com/video/BV11y4y1C7Us](https://www.bilibili.com/video/BV11y4y1C7Us)

[https://www.bilibili.com/video/BV1Yh411z7os](https://www.bilibili.com/video/BV1Yh411z7os)

[https://www.bilibili.com/video/BV1Gg411g7Cv](https://www.bilibili.com/video/BV1Gg411g7Cv)

重点是理解MergeTree

### 插曲

ClickHouse现在已经是一家美国公司了

https://clickhouse.com/#quick-start

![image-20220523130940920](/img/images/image-20220523130940920.png)

[https://clickhouse.com/blog/we-stand-with-ukraine/](https://clickhouse.com/blog/we-stand-with-ukraine/)

公司都是有国籍的

### 竞品

好像不多，kylin、Doris、[StarRocks](https://blog.csdn.net/qq_20949471/article/details/124741703?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-4-124741703-null-null.pc_agg_new_rank&utm_term=doris%E5%BC%80%E6%BA%90%E4%BA%8B%E4%BB%B6+starrocks&spm=1000.2123.3001.4430)等，还有真假开源之争，真乱。相比而言，ClickHouse还纯粹些。

### Dashboard展示

ClickHouse主要是数据的存储和分析计算，而展示就需要一些Dashboard工具，比如Grafana等

[https://www.freesion.com/article/2168894698/](https://www.freesion.com/article/2168894698/)



### 总结

1. 构建大数据平台已经是大部分企业信息化应用的迫切需求
2. ClickHouse给了用户在Hadoop生态圈之外的另一个选择
3. OLAP实时处理+批处理+离线处理
4. 需要配合一个好的Dashboard展示工具




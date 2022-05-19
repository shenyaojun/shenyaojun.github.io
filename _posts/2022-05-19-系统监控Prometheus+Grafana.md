---

layout:     post
title:      系统监控Prometheus+Grafana
subtitle:   系统监控新套件
date:       2022-05-19
author:     就是我啦
header-img: img/post-bg-os-metro.jpg
catalog: 	  true
tags:
    - 系统监控    
    - Prometheus        
    - Grafana      
    - Zabbix    
    - Cacti
---

# 系统监控Prometheus+Grafana

> 系统监控利器，运维和**SRE**（SRE是指Site Reliability Engineer (网站可靠性工程师)。他是软件工程师和系统管理员的结合，一个SRE工程师基本上需要掌握很多知识：算法，数据结构，编程能力，网络编程，[分布式系统](https://baike.baidu.com/item/分布式系统/4905336)，可扩展架构，故障排除）至宝
>
> 与运维工程师不同的是。SRE工程师不但负责传统运维工程师的工作，他们也负责软件工程师的一些工作——参与某些软件或系统的开发。SRE团队相信**通过开发软件系统来维护系统的正常运行**，以此来代替传统运维工程师的人工操作，这样做大大节省了人力开销。

```sh
# 这个东西用的少，原来的理解有欠缺，这次专门整理一下
```

## 系统监控概念

亲自好使的Docker搭建过程：

[https://blog.csdn.net/ichen820/article/details/124575537](https://blog.csdn.net/ichen820/article/details/124575537)

[https://blog.csdn.net/kkonetwo/article/details/105414920](https://blog.csdn.net/kkonetwo/article/details/105414920)

还有这个B站视频，概念讲的很好

[https://www.bilibili.com/video/BV1wK4y1H79c](https://www.bilibili.com/video/BV1wK4y1H79c)

[https://www.bilibili.com/video/BV1AK4y1T7QQ/](https://www.bilibili.com/video/BV1AK4y1T7QQ/)

* SNMP监控时代   Cacti、Nagios、Zabbix
* 当今的监控系统   Prometheus+Grafana、国产OpenFalcon、夜莺
* 未来的监控系统  DataOps  AIOps

### 监控系统功能组件

- 指标数据的采集
- 指标数据的存储
- 指标数据趋势分析可视化
- 告警

### 监控体系

- 系统层监控
  - 系统监控: CPU, Load, Memory, Swap, Disk Io, Process, ....
  - 网络监控: 网络设备, 工作负载, 网络延迟, 丢包率....
- 中间件即基础类设施监控
  - 消息中间件: Kafka, RocketMq和RabbitMQ....
  - web服务器: Tomcat, Nginx....
  - 数据库及缓存数据库: Mysql, MogoDb, Redis, ElasticSearch....
  - 存储系统: Ceph....
- 应用层监控
  - 用于衡量应用程序的状态和性能.
- 业务层监控
  - 用于衡量应用程序的价值, 例如电子商务网站上的销售量
  - QPS, DAU日活 转化率.
  - 业务接口: 登录数, 注册数, 订单量, 搜索量和支付量等.

### 云原生时代的可观测性

可观测性系统

- 指标监控
  - 随时间推移产生的一些于监控相关的可聚合数据点.
- 日志监控
  - 离散式的日志或事件.
- 链路监控
  - 分布式应用调用链路跟踪

CNCF将可观测性和数据分析归类一个单独的类别, 且划分了四个子类

- 日志系统
  - 以 ElasticStack为代表
- 监控系统
  - 以 Prometheus 为代表
- 分布式调用链路跟踪系统
  - 以 Zipkin, jaeger 等为代表
- 混沌工程系统
  - 以 ChaosMonkey 和 ChaosBlade 等为代表



# Prometheus简介

Prometheus是由SoundCloud开发的**开源监控报警系统**和**时序列数据库**(TSDB)。
 Prometheus使用**Go**语言开发，是Google BorgMon监控系统的开源版本。 2016年由Google发起Linux基金会旗下的**原生云基金会**(Cloud Native Computing Foundation), 将Prometheus纳入其下第二大开源项目。 Prometheus目前在开源社区相当活跃。
 Prometheus和Heapster(Heapster是K8S的一个子项目，用于获取集群的性能数据)相比功能更完善、更全面。Prometheus性能也足够支撑上万台规模的集群。

## 基本原理

Prometheus的基本原理是**通过HTTP协议**周期性抓取被监控组件的状态，任意组件只要**提供对应的HTTP接口**就可以接入监控。不需要任何SDK或者其他的集成过程。
 这样做非常适合做虚拟化环境监控系统，比如VM、Docker、Kubernetes等。输出被监控组件信息的HTTP接口被叫做**exporter**。目前互联网公司常用的组件大部分都有exporter可以直接使用，比如Varnish、Haproxy、Nginx、MySQL、Linux系统信息(包括磁盘、内存、CPU、网络等等)。

## 架构涉及

![img](/img/images/2178672-0f8e643345fa1da8674.png)



### 常用的exporter整理

- node-exporter: 用来监控运算节点上的宿主机的资源信息，需要部署到所有运算节点
- kube-state-metric：prometheus采集k8s资源数据的exporter，能够采集绝大多数k8s内置资源的相关数据，例如pod、deploy、service等等。同时它也提供自己的数据，主要是资源采集个数和采集发生的异常次数统计
- cAdvisor （Container Advisor） ：用于监控正在运行的容器资源使用和性能信息。
   [https://github.com/google/cadvisor](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgoogle%2Fcadvisor)
- Blackbox_exporter：监控业务容器存活性。可以提供 http、dns、tcp、icmp 的监控数据采集



### 总结

6. 监控系统已经成为企业信息化基础设施不可或缺的一部分
6. 一般企业因为IT力量的缺乏，目前有能力搭建的并不多
6. 简单使用容易，想深入使用就要下功夫了


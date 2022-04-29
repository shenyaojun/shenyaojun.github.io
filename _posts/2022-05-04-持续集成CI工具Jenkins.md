---

layout:     post
title:      持续集成CI/CD工具Jenkins
subtitle:   可以不用，但不能不了解
date:       2022-05-04
author:     就是我啦
header-img: img/post-bg-map.jpg
catalog: 	  true
tags:
    - 持续集成    
    - 持续部署        
    - CI      
    - CD
    - Jenkins    
    - Devops
    - SonarQube
    - Harbor
    - Docker
---

# 持续集成CI/CD工具Jenkins

> 持续集成的目的就是**快速迭代**
>
> 同时，保持**高质量**
>
> 可以和测试自动化结合，代码**集成到主干之前，必须先通过自动化测试**

```yacas
Typora今天也不好用了，报beta版本过期，重新安装了0.11版本，现在凑合能用了
看来要考虑换了
```

一个很好的B站课程：

[https://www.bilibili.com/video/BV1AK4y1n7Rp](https://www.bilibili.com/video/BV1AK4y1n7Rp)

感觉看前75讲就够了，后面kubernetes一般还用不着

## 概念

最初是**瀑布模型**，后来是**敏捷开发**，现在是**DevOps**，这是现代开发人员构建出色产品的技术路线。随着DevOps的兴起，出现了**持续集成（Continuous Integration）、持续交付（Continuous Delivery） 、持续部署（Continuous Deployment）** 的新方法。传统的软件开发和交付方法正在迅速变得过时。从历史上看，在敏捷时代，大多数公司会每月，每季度，每两年甚至每年发布部署/发布软件。然而，现在，**在DevOps时代，每周，每天，甚至每天多次是常态**。当SaaS正在占领世界时，尤其如此，您可以轻松地动态更新应用程序，而无需强迫客户下载新组件。很多时候，他们甚至都不会意识到正在发生变化。开发团队通过软件交付流水线（Pipeline）实现自动化，以缩短交付周期，大多数团队都有自动化流程来检查代码并部署到新环境。



* Build great things at any scale.
* The leading open source automation server, Jenkins provides **hundreds of plugins** to support building, deploying and automating any project.
* Jenkins是一个**开源的、提供友好操作界面的持续集成(CI)工具**，起源于Hudson（Hudson是商用的），主要用于持续、自动的构建/测试软件项目、监控外部任务的运行（这个比较抽象，暂且写上，不做解释）。Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行。通常与版本管理工具(SCM)、构建工具结合使用。常用的版本控制工具有SVN、GIT，构建工具有Maven、Ant、Gradle。

![image-20220429173008754](/img/images/image-20220429173008754.png)

![image-20220429173838602](/img/images/image-20220429173838602.png)

![img](/img/images/20192451-376effcf69f2581d.png)

## 优点
* 通过自动化测试可以提早拿到回归测试的结果，避免将一些问题提交到交付生产中。
* 发布编译将会更加容易，因为合并之初已经将所有问题都规避了。
* 减少工作问题切换，研发可以很快获得构建失败的消息，在开始下一个任务之前就可以很快解决。
* 测试成本大幅降低，你的CI服务器可以在几秒钟之内运行上百条测试。
* 你的QA团队花费在测试上面的时间会大幅缩短，将会更加侧重于质量文化的提升上面。

## 持续集成需要一组工具和流程

* Git：开发人员开发代码，开发完提交到Git
* Jenkins从Git上拉取代码
* Jenkins调用SonarQube做代码审查
* Jenkins调用maven编译、打包
* 消息通知及测试报告：集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知，生成JUnit/TestNG测试报告
* Jenkins调用Docker生成镜像
* Jenkins调用Harbor上传镜像
* Jenkins在生产机器上拉取镜像，部署应用
* 分布式构建：支持Jenkins能够让多台计算机一起构建/测试

基本都是工具类的，重点在多熟悉多练，但作为一般企业IT，能用到这一步的不多。首先，大部分应用都是外包供应商开发，CI一般还是适合**Devops**的

### 总结

- DevOps 的整体目标是促进开发和运维人员之间的配合，并且通过**自动化的手段**缩短软件的整个交付周期，提高软件的可靠性

- 工具带来方便，也带来了复杂性，工具都有其适用条件

- 能自动化的就尽量自动化，这是IT的基本精神

  

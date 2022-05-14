---

layout:     post
title:      信号树ZooKeeper
subtitle:   信号树+通知机制
date:       2022-05-14
author:     就是我啦
header-img: img/post-bg-debug.png
catalog: 	  true
tags:
    - ZooKeeper    
    - 分布式锁        
    - 注册中心      
    - 配置管理    
    - Curator
---

# 信号树ZooKeeper

> 理解还是要真动手用用
>
> 自身有集群功能，leader和follower，半数以上即可正常运行，自带HA

```sh
# 这个东西用的少，原来的理解有欠缺，这次专门整理一下
```

## Zookeeper概念

* Zookeeper是Apache Hadoop项目下的一个子项目，是一个**树形目录服务**

* Zookeeper翻译过来就是动物园管理员，它是用来管Hadoop（大象）、Hive（密封）、Pig（小猪）的管理员，简称ZK

* Zookeeper是一个**分布式**的、**开源**的分布式应用程序的**协调服务**
  Zookeeper提供的主要功能包括

* **配置管理**

* **分布式锁**

* **集群管理**

  总之，Zookeeper的核心就是**<u>信号树+通知机制</u>**

  它还是**kafka所依赖的必备组件**

  

  - **短暂/临时(Ephemeral)**：当客户端和服务端断开连接后，所创建的Znode(节点)**会自动删除**
  - **持久(Persistent)**：当客户端和服务端断开连接后，所创建的Znode(节点)**不会删除**

## 资料

[https://www.bilibili.com/video/BV1nF411E7G6](https://www.bilibili.com/video/BV1nF411E7G6?p=34)

[https://zhuanlan.zhihu.com/p/62526102](https://zhuanlan.zhihu.com/p/62526102)



测试程序：

```java
package com.syj.zk;

import org.apache.zookeeper.*;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.util.List;

public class zkClient {
    private  String connectString = "zkserver01:2181";
    private int sessionTimeout = 2000;
    private ZooKeeper zkCli;

    @Before
    public void init() throws IOException {
        zkCli = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {

                System.out.println("--------------------start");

                List<String> children = null;
                try {
                    children = zkCli.getChildren("/", true);
                    for (String child : children) {
                        System.out.println(watchedEvent.toString() +"**********"+child);
                    }
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println("--------------------end");

            }
        }
        );

    }

    @Test
    public void create() throws InterruptedException, KeeperException {
        zkCli.create("/atguigu","ss.avi".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    @Test
    public void getChildren() throws InterruptedException, KeeperException {
        List<String> children = zkCli.getChildren("/", true);

        for (String child : children) {
            System.out.println(child);
        }

        Thread.sleep(Long.MAX_VALUE);

    }
}

```





Curator分布式锁测试程序

```java
package com.syj.zk.curator;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.retry.ExponentialBackoffRetry;

public class CuratorLockTest {
    public static void main(String[] args) {

        //创建分布式锁1
        InterProcessMutex lock1 = new InterProcessMutex(getCuratorFramework(), "/locks");

        //创建分布式锁2
        InterProcessMutex lock2 = new InterProcessMutex(getCuratorFramework(), "/locks");

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    lock1.acquire();
                    System.out.println("线程1 获取到锁");

                    lock1.acquire();
                    System.out.println("线程1 再次获取到锁");

                    System.out.println("线程1开始干活");
                    Thread.sleep(5 * 1000);

                    System.out.println("线程1干活结束");
                    lock1.release();
                    System.out.println("线程1 释放锁");

                    lock1.release();
                    System.out.println("线程1 再次释放锁");

                } catch (Exception e) {
                    e.printStackTrace();
                }

            }
        }).start();


        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    lock2.acquire();
                    System.out.println("线程2 获取到锁");

                    lock2.acquire();
                    System.out.println("线程2 再次获取到锁");

                    System.out.println("线程2开始干活");
                    Thread.sleep(5 * 1000);

                    System.out.println("线程2干活结束");
                    lock2.release();
                    System.out.println("线程2 释放锁");

                    lock2.release();
                    System.out.println("线程2 再次释放锁");

                } catch (Exception e) {
                    e.printStackTrace();
                }

            }
        }).start();

    }

    private static CuratorFramework getCuratorFramework() {

        ExponentialBackoffRetry policy = new ExponentialBackoffRetry(3000, 3);
        CuratorFramework client = CuratorFrameworkFactory.builder().connectString("zkserver01:2181")
                .connectionTimeoutMs(2000)
                .sessionTimeoutMs(2000)
                .retryPolicy(policy).build();

        //启动客户端
        client.start();

        System.out.println("客户端启动成功");

        return client;

    }
}

```



### Redis和Zookeeper分布式锁方案的比较

可以参考这个：

[https://baijiahao.baidu.com/s?id=1730176923760559058&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1730176923760559058&wfr=spider&for=pc)

一个利用Redis 的SetNx，一个利用zookeeper的临时顺序节点，总结起来：

（1）如果**不严格要求分布式锁安全**，可以考虑在Sentinel、Cluster模式下使用**Redis实现分布式锁**。例如多个客户端同时获取锁并不会导致严重的业务问题，或者只是要求性能优化避免多个客户端同时操作等场景，都可以使用Redisson提供的分布式锁。
（2）如果**严格要求分布式锁安全**，则可以使用**ZooKeeper**、Etcd等组件实现分布式锁。
当然，建议使用**Redisson、Curator**等成熟框架实现分布式锁，避免重复编码，也减少错误风险。



### 总结

作为大数据家族Hadoop的核心组件，Zookeeper天生就是适合分布式集群的

1. ZooKeeper **本身就是一个分布式程序**（只要**半数以上**节点存活，ZooKeeper 就能正常服务）。
2. 为了保证高可用，最好是以**集群形态**来部署 ZooKeeper，这样只要集群中大部分机器是可用的（能够容忍一定的机器故障），那么 ZooKeeper 本身仍然是可用的（**HA**）。
3. ZooKeeper 将**数据保存在内存中**，这也就保证了 高吞吐量和低延迟（但是内存限制了能够存储的容量不太大，此限制也是保持znode中存储的数据量较小的进一步原因）。
4. ZooKeeper 是**高性能的**。 在**“读”多于“写”**的应用程序中尤其地高性能，因为“写”会导致所有的服务器间同步状态。（“读”多于“写”是协调服务的典型场景。）
5. ZooKeeper有临时节点的概念。 当创建临时节点的客户端会话一直保持活动，瞬时节点就一直存在。而当会话终结时，瞬时节点被删除。持久节点是指一旦这个ZNode被创建了，除非主动进行ZNode的移除操作，否则这个ZNode将一直保存在Zookeeper上。
6. ZooKeeper 底层其实只提供了两个功能：①**管理（存储、读取）用户程序提交的数据**；②为用户程序提供数据节点**监听服务**



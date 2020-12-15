---
layout: post
title: ZooKeeper
date: 2020-09-08
Author: shope
categories: ZooKeeper
tags: [ZooKeeper]
comments: true
---
## 1、ZooKeeper是什么？

​		它是一个**分布式协调框架**，是Apache Hadoop 的一个子项 目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同 步服务、集群管理、分布式应用配置项的管理等。

## 2、ZooKeeper文件系统数据结构

**2.1、Zookeeper维护一个类似文件系统的数据结构：**

![ZooKeeper_Data_Struction.PNG](https://i.loli.net/2020/12/15/RWLyS2qiT9zZmxM.png)

每个子目录项都被称作为 **znode**(目录节点)，和文件系统类似，我们能够自由的增加、删除 znode，在一个znode下增加、删除子znode。

**2.2、节点类型：**

- **PERSISTENT­-持久化目录节点**：客户端与zookeeper断开连接后，该节点依旧存在，只要不手动删除该节点，他将永远存在。

- **PERSISTENT_SEQUENTIAL**-持久化顺序编号目录节点：客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给改节点名称进行顺序编号。

- **EPHEMERAL-临时目录节点**：客户端与Zookeeper断开连接后，改节点被删除。

- **EPHEERAL_SEQUENTIAL**-临时顺序编号目录节点：客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号。

- **Container-容器节点**：如果Container节点下面没有znode（子节点），则Container节点在未来会被Zookeeper**自动清除**，定时任务默认60s检查一次。

- **TTL节点**：默认禁止，之恶能通过配置zookeeper.extendedTypeEnabled=true 开启，不稳定。

  

**2.3 、监听通知机制**

- 如果注册的是对某个**节点**的监听，则当这个节点被删除，或者被修改时，对应的客户端将被通知
-  如果注册的是对某个**目录**的监听，则当这个目录有子节点被创建，或者有子节点被删除，对应的客户端将被通知
- 如果注册的是对某个目录的递归子节点进行监听，则当这个目录下面的任意子节点有目录结构 的变化（有子节点被创建，或被删除）或者根节点有数据变化时，对应的客户端将被通知。

需要注意的是通知是一次性的，只要触发过一次，第二次就不会再继续发通知，监听被移除了。递归监听，每个节点只会触发一次监听。

**每个节点都有一个状态：**

- **Looking**：此状态为集群中还未有Leader，所有的Follow都为Looking状态。
- **Following**：表明该节点是Follow节点，除了只能给客户端提供读取数据服务，还可以参与Leader选举。
- **Leading**：表示节点为Leader节点，提供读写数据的服务。
- **Observing**：表示为Observer节点，仅提供读请求，不可以参与Leader的选举和投票。



**ZooKeeper集群写数据机制：**只有Leader节点才写数据，在Leader写入数据后，会让Follow节点去同步数据，Follow同步成功会响应给Leader，最后统计同步成功的Follow节点的数量，在集群中超过半数的Follow节点写入数据成功该条数据才算成功写入。**Observer节点不计算为过半数量**。

**ZooKeeper选票逻辑：**

在第X个周期（每个周期都只选出一个Leader）：

**第一次投票**：每个Follow都会选举自己，给自己投一票，发出投票信息，每个节点收到后会和自己的投票对比，然后选择**zxid**最大的为Leader(zxid最大表示机器的数据是最新的)，如果zxid一样大，则默认选择myid大的为Leader。每台Follow仅有一票，所以没有任何一台Follow所得票数过半，至此第一次投票失败。

**第二次投票：**在第一次投票后（经过选票PK）每台Follow都有了**心目中的Leader**，然后大家都会对自己心目中的Leader投票，所得票数过半的Follow会成为Leader。

**选票的数据结构：**

```java
public class Vote {
    //	这个暂时不知道
    private final int version;
	//	被选的Leader id
    private final long id;
    //	被选的Leaders 的事务ID
    private final long zxid;
	//	逻辑时钟 每进入新一轮的投票后都会 +1
    private final long electionEpoch;
	//	选举周期
    private final long peerEpoch;
	//	节点状态
    private final ServerState state;
}

 public enum ServerState {
    LOOKING,0
    FOLLOWING,
    LEADING,
    OBSERVING
   }
```

**选票PK规则：**

![Zookper-Election-Rule.PNG](https://i.loli.net/2020/12/15/R4BNFrujqh8cv2e.png)

<a href="https://www.processon.com/diagraming/5fbe79107d9c08478997a0e2 "><font color="gree">ZooKeeper源码学习思路图</font></a>
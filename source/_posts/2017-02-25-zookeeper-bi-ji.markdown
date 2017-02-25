---
layout: post
title: "zookeeper 笔记"
date: 2017-02-25 17:30
comments: true
categories: Server
description: zookeeper
---

ZooKeeper 是一个开源的分布式协调服务，是分布式数据一致性的解决方案。

### 集群角色

在 ZooKeeper 中，有三种角色： Leader，Follower，Observer

一个 ZooKeeper 集群同时只会有一个Leader，其他都是 Follower 或 Observer。
Leader 服务器为客户端提供读和写服务，Follower 和 Observer 都能提供读服务，不能提供写服务。区别在于，Observer 不参与 Leader 选举过程，也不参与写操作的过半写成功策略，因此 Observer 可以在不影响写性能的情况下提升集群的读性能。

### 会话

客户端和 ZooKeeper 服务器会与服务器建立一个 TCP 连接，通过这个连接，客户端能够通过心跳检测和服务器保持有效的会话，也能够向 ZooKeeper 服务器发送请求并接受响应，同时还能通过该连接接收来自服务器的 Watch 事件通知。


### 数据节点

ZooKeeper 中的数据节点是指数据模型中的数据单元，称为 ZNode。ZooKeeper将所有数据存储在内存中，数据模型是一棵树（ZNode Tree），由斜杠进行分割的路径，就是一个ZNode。每个ZNode上都会保存自己的数据内容，同时会保存一系列属性信息。每个ZNode不仅本身可以写数据，还可以有下一级文件或目录。

在ZooKeeper中，ZNode可以分为持久节点和临时节点两类。持久节点是指一旦这个 ZNode 被创建了，除非主动进行 ZNode 的移除操作，否则这个 ZNode 将一直保存在 ZooKeeper上。临时节点的生命周期跟客户端会话绑定，一旦客户端会话失效，那么这个客户端创建的所有临时节点都会被移除。

ZooKeeper 允许用户为每个节点添加一个特殊的属性：SEQUENTIAL。一旦节点被标记上这个属性，那么在这个节点被创建的时候，ZooKeeper就会自动在其节点后面追加上一个整型数字，这个整型数字是一个由父节点维护的自增数字。

### 版本

ZooKeeper 的每个 ZNode 上都会存储数据，对于每个ZNode，ZooKeeper都会为其维护一个叫作Stat的数据结构，Stat中记录了这个ZNode的三个数据版本，分别是 version（当前ZNode的版本, cversion（当前ZNode子节点的版本）和 aversion（当前ZNode的ACL版本）。

### 事务

在 ZooKeeper 中，能改变 ZooKeeper 服务器状态的操作称为事务操作。包括数据节点创建与删除、数据内容更新和客户端会话创建与失效等操作。对应每一个事务请求，ZooKeeper都会为其分配一个全局唯一的事务ID，用ZXID表示，通常是一个64位的数字。每一个ZXID对应一次更新操作，从这些ZXID中可以间接地识别出ZooKeeper处理这些事务操作请求的全局顺序。

### Watcher

ZooKeeper 允许用户在指定节点上注册一些 Watcher，并且在一些特定事件触发的时候，ZooKeeper 服务端会将事件通知到感兴趣的客户端上去。该机制是 ZooKeeper 实现分布式协调服务的重要特性。

###  ACL

ZooKeeper 采用 Access Control Lists 策略来进行权限控制。ZooKeeper 定义了如下5种权限。

```
CREATE: 创建子节点的权限。
READ: 获取节点数据和子节点列表的权限。
WRITE：更新节点数据的权限。
DELETE: 删除子节点的权限。
ADMIN: 设置节点ACL的权限。
注意：CREATE 和 DELETE 都是针对子节点的权限控制。
```

### ZAB 原子广播协议

ZooKeeper Atomic Broadcast（ZAB，ZooKeeper原子广播协议）的协议作为其数据一致性的核心算法。

所有事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被称为Leader服务器，而剩下的其他服务器则成为 Follower 服务器。Leader 服务器负责将一个客户端事务请求转换成一个事务 Proposal（提案）并将该 Proposal分发给集群中所有的 Follower 服务器。之后 Leader 服务器需要等待所有 Follower 服务器的反馈，一旦超过半数的 Follower 服务器进行了正确的反馈后，Leader 就会再次向所有的 Follower 服务器分发 Commit 消息，要求对刚才的 Proposal 进行提交。

### 应用场景

* 数据发布与订阅-配置中心

发布者将数据发布到 ZooKeeper 节点上，供订阅者进行数据订阅，进而达到动态获取数据的目的，实现配置信息的集中式管理和动态更新。全局配置信息就可以发布到 ZooKeeper 上，让客户端（集群的机器）去订阅该消息。

客户端想服务端注册自己需要关注的节点，一旦该节点的数据发生变更，那么服务端就会向相应的客户端发送 Watcher 事件通知，客户端接收到这个消息通知后，需要主动到服务端获取最新的数据（推拉结合）。

* 命名服务，即生成全局唯一的ID。

* 分布式协调通知

ZooKeeper 中特有 Watcher 注册与异步通知机制，实现对数据变更的实时处理。不同的客户端都对ZK上同一个ZNode 进行注册，监听 ZNode 的变化（包括ZNode本身内容及子节点的），如果 ZNode 发生了变化，那么所有订阅的客户端都能够接收到相应的 Watcher 通知，并做出相应的处理，是一种通用的分布式系统机器间的通信方式。

* 心跳检测

基于 ZK 临时节点的特性，可以让不同的进程都在 ZK 的一个指定节点下创建临时子节点，不同的进程直接可以根据这个临时子节点来判断对应的进程是否存活。通过这种方式，检测和被检测系统直接并不需要直接相关联，而是通过 ZK 上的某个节点进行关联，大大减少了系统耦合。

* 分布式锁

分布式锁是控制分布式系统之间同步访问共享资源的一种方式。分布式锁又分为排他锁和共享锁两种。 排他锁又称为写锁或独占锁，共享锁又称为读锁。

把 ZooKeeper 上一个 ZNode 看作是一个锁，获得锁就通过创建ZNode的方式来实现。所有客户端都去/x_lock节点下创建临时子节点/x_lock/lock。ZooKeeper会保证在所有客户端中，最终只有一个客户端能够创建成功，那么就可以认为该客户端获得了锁。同时，所有没有获取到锁的客户端就需要到/x_lock节点上注册一个子节点变更的 Watcher 监听，以便实时监听到 lock 节点的变更情况。


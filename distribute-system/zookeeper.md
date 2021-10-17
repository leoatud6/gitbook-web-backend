---
description: From Apache Hadoop
---

# Zookeeper

## Overview

distribute-->分布式协调框架同步

Zookeeper = **文件系统** + **监听通知机制**

* 文件系统： 类似linux: 根目录，每个里面还可以继续有文件 不同节点还会有自己的类型： PERSISTENT, EPHEMERAL
* 监听通知机制：client注册监听它关心的目录节点，当目录节点发生变化时，通知client (类比redis) client listen the node of zookeeper: if anything changed, will notify the client 监听者模式 observer pattern

zookeeper连接是异步的，需要等待

## Zookeeper Core

![本身是一个目录结构系统（类比linux）](<../.gitbook/assets/image (6).png>)

default server** **port: **2181**

Springboot package: **zkClient**

default client port: **7001**

{% embed url="https://如果client运行时占用8080可以更改zoo.cfg" %}

```
admin.serverPort = xxxx
```

{% hint style="info" %}
[Zookeeper & dubbo](https://www.jianshu.com/p/9f3f8b0d6a08) server default会占用8080， 需要手动config
{% endhint %}

#### zookeeper集群一般是**奇数**：** 3个至少**

![应对脑裂，这样一个down掉了，另外两个可以足够时间选举出来，超过半数的模式选举 redis主从架构](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FFPhvTpLxh1hOF0hmV3qs%2Ffile.png?alt=media)



### ZooKeeper: only CP, not A, leader choose in cluster

### BASE: 最终一致性，最终可用

#### default选举算法: FastLeaderElection

 leader and follower



## Operation

```bash
bin/zkServer.sh start
ps -ef | grep zookeeper    #check zookeeper process?
./zkServer.sh status    #cannot load server start 
```


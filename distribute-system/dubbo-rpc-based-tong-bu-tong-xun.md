# Dubbo (RPC based 同步通讯)

## 发展方向，历史，Why

![](<../.gitbook/assets/image (4).png>)

### ORM: single jar，单体框架，可以多放几个server

### MVC: 垂直框架，每个负责单独应用，每次修改需要重新deploy

### RPC: 远程获取调用，前后分离，server之间建立连接，底层websocket

（消息传递： serialization / deserialization， 因此json处理object--> faster）

### SOA: 超大规模，利用dispatcher进行流量分配，以及server启动，cluster



## Dubbo Core 

![](<../.gitbook/assets/image (7).png>)

* Config: 配置层 
* Proxy: 通过代理的方式生成 
* Registry: 注册中心，完成注册功能 
* Cluster: load balancer应用 
* Monitor: 监控数据，tool 
* Protocol: 核心RPC远程调用 A<—>B 
* Exchange: 数据互联互通 
* Transport: Netty框架所在 
* Serialize: object序列化层

底层： **netty **JAVA NIO 异步事务驱动网络应用框架

TCP：通讯底层使用—>**NIO **（Dubbo, ES, RPC, MQ）\
CPU core \* 2 = thread number 

Reactor设计模式：基于事件驱动

![Consumer: Web / Provider: service / Container: dubbo](<../.gitbook/assets/image (5).png>)

需要结合注册中心**ZooKeeper**使用， port： **`2181`**

{% content-ref url="zookeeper.md" %}
[zookeeper.md](zookeeper.md)
{% endcontent-ref %}



### 使用

```
//导入依赖： dubbo-spring-boot-starter

dubbo.application.name = 
dubbo.registry.address = 127.0.0.1:2181
dubbo.registry.protocol = zookeeper

dubbo.protocol.name = dubbo
dubbo.protocol.prot = 20880

dubbo.monitor.protocol = registry


//使用注解
@EnableDubbot(scanXXX = "xxx") -> on the application startup

//暴露服务
@Service  --> replace springboot @service

//消费服务
@Reference   --> replace @Autowired
```

![eg of configuration](<../.gitbook/assets/image (39).png>)

### 集群容错 （实际开发可以整合hystrix @EnableHystrix）

Failover cluster: default—>一个不行尝试另一个 

Failfast cluster： 失败立即报错 

FailSafe cluster：失败后忽略，记录log 

Failback cluster：失败后自动回复 

Forking cluster： 并行调用多个服务器，太浪费资源 

Broadcast cluster: 同步的时候需要使用



### Load Balance

Random: 基于权重，可以设置

RoundRobin: 轮询

LeastActive

ConsistentHash

## Concepts

#### 灰度发布： 20%更新系统， 80%使用旧的，再慢慢过渡

#### 服务降级：简单处理 or 懒处理

#### 本地存根：stub, 正式调用前留一个copy, 类似proxy










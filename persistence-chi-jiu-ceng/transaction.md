# Transaction

## Overview

分布式三个特性：**CAP: consistency 、 avaliability、 partition-tolerance **

一个分布式系统最多只可能满足两项 P是必须要满足，**C和A**需要牺牲一个

## 实现

#### Atomikos—>垮库开源框架 

将basicDatasource变成atomikosDataSource —>增加原子性

#### TCC

Try confirm cancel: 两阶段补偿性



## Isolation level

* Default(spring default —> depending on the database) 
* READ_UNCOMMITTED ： lowest level 会有安全问题 
* READ_COMMITTED: oracle default 
* REPEATABLE_READ： mysql default，可以解决脏读，不可以重复读 
* SERIALIZABLE: 隔离性最高，性能最差


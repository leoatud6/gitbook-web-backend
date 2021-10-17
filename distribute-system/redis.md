# Redis (remote dictionary server)

## Overview

RDB 

AOF： append only file (recommend) 

DataType: String, Set, ZSet, Hash, List

Key-value storage 

单线程，高可用性，QPS high，所有操作原子性， written in C

![](<../.gitbook/assets/image (42).png>)

![](<../.gitbook/assets/image (43).png>)

### Distributed Lock for Redis

* setnx: add distributed lock --> only one thread can access, others wait
* set expired time for lock: not allow one thread occupied resource too long (set key value nx ex 10)
* how to define key for lock : token + key + currentTime?  --> don't forget expire time



## 分布式🔐

* setnx ==> set if absent --> check if the lock exist, if yes --> return
* must delete the lock --> use try{} finally {delete key}
* set expired time, in case the hardware issue/down
* add UUID, check if value is UUID when delete the lock/key
* 锁续命

![](<../.gitbook/assets/image (49).png>)

\==> Redisson (Redis 's son)

```java
String lock_key = UUID.generateUUID();
RLock redissonLock = redisson.getLock(lock_key);

try{
    redissonLock.lock();
    
    //***
    //   business logic
    //***

} finally{
    redissonLock.unlock();
}
```

![](<../.gitbook/assets/image (50).png>)





## Redission (Redis + JUC)

分布式锁

应对高并发处理

key为一个string，记得设置过期时间expire time

```java
Long num = stringRedisTemplate.opsForValue().increment(“lock”, 1);    
//加锁

stringRedisTemplate.opsForValue().increment(“lock”, -1);    
//解锁

stringRedidTemplate.expire(“lock”, 10, TimeUnit.SECONDS); 
// 10s之后不管如何停掉


//普通分布式锁
RLock lock = redission.getLock(“lock”);
Try{
        lock.lock();
        //process contents
} finally{
        lock.unlock();
}


//======================================REDIS========================================

//Jedis / Jedis Client
@Configuration
public class RedisConfig{

        @Value("${spring.redis.host:disabled}")
        private String host;
        
        @Value("${spring.redis.port:0}")
        private int port;
        
        @Value("${spring.redis.database:0}")
        private int database;
}

//usage
Jedis jedis = redisUtil.getJedis();
jedis.set("key","value");
jedis.close();

```



### Problems

![](<../.gitbook/assets/image (47).png>)






















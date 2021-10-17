# 大厂面试

### CAS / ABA  (compareAndSet)

 volatile --> ABA (不在乎中间过程)

 AtomicInteger --> AtomicIntegerIncrement()  / compareAndSet(x1, x2)

 AtomicReference\<T> : self define class

#### ABA: 类似修改version number

#### SpinLock自旋：用循环替代阻塞（消耗资源）

**`不会立刻阻塞（blockIng)， 而是采用spin/while loop的方式尝试获取锁`**

![](<../.gitbook/assets/image (141).png>)



### CountDownLatch / CyclicBarrier / Semaphore 具体使用方法

```java
//CountDownLatch    ===================================================================
CountDownLatch latch = new CountDownLatch(6); //秦灭六国

for(int i=0;i<6;i++){
    new Thread(()->{
        //business logic
        latch.countDown();
    }, Thread.currentThread.getName()).start();
}

latch.await();
System.out.println("秦灭六国，统一")；


//CyclicBarrier  （相当于CountDownLatch反着用，要有n个await()） ======================
CyclicBarrier barrier = new CyclicBarrier(7, ()->{
    //business logic after get the barrier
    System.out.println("所有人都到了，开始吃饭吧")； //等人到期了开始吃饭
});

for(int i=0;i<7;i++){
    new Thread(()->{
        //need to handle exception here
        barrier.await();
    }, Thread.currentThread.getName()).start();
}

//一个做减法（执行await后面内容），一个做加法（count await，然后执行）


//Semaphore ： 多个Thread抢多个resource, 停车问题    ==================================================================
Semaphore s = new Semaphore(3); //create 3 parking lots

new Thread(()->{
    s.acquire();//占1个车位， 还剩2两个

    //business logic
 
    s.release();//让出1个车位
}).start();
```



### BlockingQueue (AQS foundation)

```java
BlockingQueue<String> q = new ArrayBlockingQueue<>(3); //set the size for the queue


add() / remove() : size有限制的话，会抛出异常
offer() / poll() : 有bool的返回值，并且不会抛异常，顶多返回null

q.offer("hwllo", 2, TimeUnit.second); //可以进行超时检查之类的
 
```



### ArrayList (ConcurrentModificationException) 

```java
List<T> list = new Vector<T>();  //vector too old
List<T> list = Collections.synchronizedList(new ArrayList<>());
List<T> list = new CopyOnWriteArrayList<>();  //use this
```

```java
 Empty list --> size default = 10
 
 CopyOnWriteArrayList 底层： 
 private transient volatile Object[] array;
 add() ：里面有ReentrantLock
```



### HashSet(ConcurrentModificationException) 底层HashMap

```java
Set<T> set = Collections.synchronizedSet(new HashSet<>());
Set<T> set = new CopyOnWriteArraySet<>();  //use this
```

```java
CopyOnWriteArraySet 底层：
private final CopyOnWriteArrayList<E> al;

HashSet 底层： HashMap (map = new HashMap<>())
初始值16个， factor:0.75
只关心Key, Value是object常量
```



### HashMap(ConcurrentModificationException)

```java
Map<K, V> map = Collections.synchronizedMap(new HashMap<>());
Map<K, V> map = new ConcurrentHashMap<>();
```



### String.intern()

native method， meta data存在的话直接返回，不存在的话创建并返回

“Java”特殊，每次都是新的object （jdk JVM run的时候有"Java" String, 在classloader里面加载过了，是一个Version的keywords）



### AQS(JUC面试重点：AbstractQueuedSynchronizer) 抽象队列同步器

**int** 信号灯代表是否抢到lock  +  FIFO 先进先出的**Queue**

![](<../.gitbook/assets/image (133).png>)

![](<../.gitbook/assets/image (132).png>)

```java
private transient volatile Node head;
private transient volatile Node tail;
private volatile int state;  
```

#### ReentrantLock default NofairLock

` （公平锁可以保证获取顺序，非公平锁不保证顺序）`

CAS： compareAndSetState(x,x) //check and set state

![](<../.gitbook/assets/image (134).png>)

![](<../.gitbook/assets/image (135).png>)







### LockSupport (类似于Semaphore(1))

#### Reentrantlock: **针对recursion的，外部获取lock后内部自动获取 （synchronize同理 ）**

lock() & unlock() 必须次数相同，加锁几次，解锁几次-->unlock要用finally{} 

![lockSupport class methods](<../.gitbook/assets/image (130).png>)

![object: wait/notify / condition: await/signal](<../.gitbook/assets/image (131).png>)

#### Wait()/notify() --> 必须在synchronized()里面，一定要先wait()，再notify()

#### Awaiy()/signal()-->必须与lock.lock()/unlock()组队使用，否则报错，同样是有顺序的

```java
Lock lock = new ReentrantLock();
Conditon con = lock.newCondition();

try{
    lock.lock();
    con.await();
}finally{
    lock.unlock();
}
```

#### LockSupport ::: park() / unpark(Thread t) 方法不需要synchronized or lock

```java
Thread a = new Thread(()->{
    LockSupport.park()
},"a");
a.start();

Thread b = new Thread(()->{
    LockSupport.unpark(a);
},"b");

//先unpark() , 再park() -->没毛病
//内部permit, 只有0和1两个state, 最多只有一个permit
```



### Spring AOP

```java
@After
@Before
@AfterReturning
@AfterThrowing
@Around
```

#### Spring4 / Springboot1.x.x:

```
org.Junit.test  --> 
@SpringbootTest
@RunWith(SpringRunner.class)

执行顺序：：：
@before @after @afterReturning
@before @after @afterThrowing
```

#### Spring5 / Springboot2.x.x:

```
org.junit.jupiter.api.test -->
@SpringbootTest

执行顺序：：：after变到最后了
@before @afterReturning @after
@before @afterThrowing  @after

```

#### 循环依赖：a-->b --> c-->a （circular dependency）

![](<../.gitbook/assets/image (139).png>)

```
如果有circular dependency-->要用setter injection (不要用construtor injection)
而且必须都要singleton
```

![](<../.gitbook/assets/image (136).png>)

#### 三层缓存（DefaultBeanSingletonRegistry)

![ 顺序为1-》3--》2](<../.gitbook/assets/image (138).png>)

```java
refresh(); //加载bean

factoryRefresh(); //先把beanFactory 初始化-->再将bean初始化
 
doGetxxx();
doGetBean(); //spring里面所有“do”开头的，都是real business logic

//三大缓存，四大方法
singletonObjects (256)  //已经完整创建的bean， 经历了完整生命周期
earlySingletonObjects (16)  //正在创建过程？！
singletonFactories (16)  //（半成品）三级缓存只有壳子，没有具体populate各种属性之类的

getSingleton();  //先去一级缓存找有木有
doCreate();  //造出来bean后-->首先放进三级缓存，删掉二级缓存，填充注册
populateBean();
addSingleton();

RootBeanDefinition;  //spring里面的的bean的class （类似java里面object）

```



### 强，软，弱，虚Reference

Cache Design的时候用的到，尤其是大量Cache, 内存又不是无限大

![](<../.gitbook/assets/image (142).png>)

1. 强引用Strong（Default）: 只要有一个指向，就不会GC，default，即使OOM也不会回收
2. WeakReference: 不管内存够不够，GC直接回收  --> WeakHashMap(Cache)
3. SoftReference: 内存够留着，不够的话GC
4. PhantomReference: 要与ReferenceQueue一起使用， GC之前有个preDestroy() , 在ReferenceQueue里面还有





### Redis

single-thread Redis: Memory operation, so very fast

setNx: no expire time,  -->如果避免dead lock,需要expire time, 但是setNX和expire time 不能保证atomic ==> Redisson

#### redis cluster-->解决雪崩问题

#### 集群模式

* 哨兵: master & slave, sentinel --> vote for master and master controls
* 主从
* cluster

#### 传统数据类型

* string
* list
* hash
* set
* zset



###





### ThreadLocal (vs sychonized)

一般都是静态代码块出现

#### 原理

ThreadLocal的实现原理是，在每个线程中维护一个Map，键是ThreadLocal类型，值是Object类型。当想获取ThreadLocal的值时，就从当前线程中拿出Map，然后在把ThreadLocal本身作为键从Map中拿出值返回。

#### 优缺点

\*\*优点：\*\*提供线程内的局部变量。每个线程都自己管理自己的局部变量，互不影响

\*\*缺点：\*\*内存泄漏问题。可以看到ThreadLocalMap中的Entry是继承WeakReference的，其中ThreadLocal是以弱引用形式存在Entry中，如果ThreadLocal在外部没有被强引用，那么垃圾回收的时候就会被回收掉，又因为Entry中的value是强引用，就会出现内存泄漏。虽然ThreadLocal源码中的会对这种情况进行了处理，但还是建议不需要用TreadLocal的时候，手动调remove方法。



#### @Transactional 必须要标记在public method上面才有用

#### Spring Bean lifecycle: instantiation, populate, initialization, destruction



###










































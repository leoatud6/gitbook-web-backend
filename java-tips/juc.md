---
description: 并发包
---

# JUC

## Foundation

### CAS: compare and swap

 compareAndSet(A, B)

_`Unsafe是实现CAS的核心类，Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。Unsafe类提供了硬件级别的原子操作。  -->全部都是native方法`_



### Synchronized VS ReentrantLock

* 一个JVM keywords, 一个是class
* 一个不可以停掉，必须run完或者抛异常，一个可以用unlock()停掉
* synchronized只能unfair lock， lock可选unfair\&fair
* sychnornized没有condition





### Wait , Await, Sleep

* sleep: 不释放lock, 线程独有
* wait：释放lock, must use notify()/notifyAll() to terminate -->必须搭配synchronized使用
* await / signal : 对于conditon的方法 , await不需要synchronized代码块包围， 
  1. wait()是Object超类中的方法，而await()是ConditionObject类里面的方法. 所在的超类不同使用场景也不同，wait一般用于Synchronized中，而await只能用于ReentrantLock锁中

1. await = wait
2. signal = notify
3. signalAll = notifyAll

#### Condition: 可以通过lock获取（ReentrantLock）

```java
private Lock lock = new ReentrantLock(); 
private Condition condition = lock.newCondition();
```

 使用**`ReentrantLock`**比直接使用`synchronized`更安全，可以替代`synchronized`进行线程同步。

### AQS （AbstractQueuedSynchronizer）-->涉及到自旋的内容

```java
private transient volatile Node head; //队首
private transient volatile Node tail;//尾
private volatile int state;//锁状态，加锁成功则为1，重入+1 解锁则为0
```

![](<../.gitbook/assets/image (121).png>)





## starter

![](<../.gitbook/assets/image (12).png>)

## 内存可见性

每个线程一个cache有相应内容，copy from main memory，如果有多线程同时write & read，会产生问题。对于共享数据：static

## Volatile

相当于所有操作都是在main memory中完成

因此每次获取的都是最新的数据 保证内存中的数据可见 还是不能保证原子性

不像synchronize



## CountDownLatch

同步辅助，闭锁

在完成一组其他线程中执行的操作之前，一个或多个线程一直等待 

countDownLatch: 其他线程完成的一个标志 

latch.await() latch.countdown() 倒计时而已，new的时候传入一个倒计时的数

## 创建线程方式

* implement runnable
* extend thread
* implement callable
* use executorpool

## Callable

```java
//always put callable in the futureTask
FutureTask<String> task = new FutureTast<>(calltest);
new Thread(task).start();
try{
    String s = task.get();
}
```

## Reentrantlock

Lock（）之后要执行的内容必须放在try里面 

Unlock（）必须放在**finally**里面，无论如何必须解锁

```java
//always using try - catch

lock.lock();
try{
    ...
} finally{
    lock.unlock();
}
```

## Condition / 线程按序交替

lock.newCondition(); //get conditon must use Reentrantlock

应用：循环打印ABC案例

await = wait / signal = notify / signalall = notifyall

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FXrzd8OKUBF9pBO87qpuW%2Ffile.png?alt=media)

## ReadWriteLock 乐观锁

```java
lock.readLock().lock();
lock.writeLock().lock();
//涉及到两个lock
```

## ForkJoinPool / 分开合并框架 / 工作窃取

 ForkJoinPool extends Executorservice: belongs to thread pool

 work-stealing: 当前Thread空闲，其他Thread阻塞还有task时，当前Thread会窃取其他task同时执行 Full use of CPU, 一般CPU几个Core，就有几个thread

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FRsvZ7pyBSC28peqsAyqU%2Ffile.png?alt=media)

```java
Instant end = Instant.now();
System.out.println(Duration.between(end, start));
//打印间歇时间
```

![](<../.gitbook/assets/image (20).png>)


















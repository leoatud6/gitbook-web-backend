# J2SE

## Lambda

Lambda本身使用策略模式， 将一系列算法封装

![](<../.gitbook/assets/image (13).png>)

```java
(parameters) -> expression
OR
(parameters) -> {statement;}

//作为参数，传递给方法，本身无return，需要return必须使用花括号
```

## Final

method, class, variable

* final class: everything is final inside the class, cannot be inheritance
* 主要是防止future修改
* 一般private method默认为final method
*
*

## Static

only keep one copy

 减少创建，共享使用

static method 没有this

static不可用于局部变量

## Volatile

 不适用缓存，所见即所得，

Java提供了volatile关键字来保证可见性

当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。

通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性

## Native



## Transient



## Inner class




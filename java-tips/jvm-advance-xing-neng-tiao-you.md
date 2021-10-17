# JVM Advance

### JVM

```
jps
jstat
jinfo
jmap
jhat
jstack

-Xms20M：表示设置JVM启动内存的最小值为20M，必须以M为单位
-Xmx20M：表示设置JVM启动内存的最大值为20M，必须以M为单位。将-Xmx和-Xms设置为一样可以避免JVM内存自动扩展。大的项目-Xmx和-Xms一般都要设置到10G、20G甚至还要高
-verbose:gc：表示输出虚拟机中GC的详细情况
-Xss128k：表示可以设置虚拟机栈的大小为128k
-Xoss128k：表示设置本地方法栈的大小为128k。不过HotSpot并不区分虚拟机栈和本地方法栈，因此对于HotSpot来说这个参数是无效的
-XX:PermSize=10M：表示JVM初始分配的永久代（方法区）的容量，必须以M为单位
-XX:MaxPermSize=10M：表示JVM允许分配的永久代（方法区）的最大容量，必须以M为单位，大部分情况下这个参数默认为64M
-Xnoclassgc：表示关闭JVM对类的垃圾回收
-XX:+TraceClassLoading表示查看类的加载信息
-XX:+TraceClassUnLoading：表示查看类的卸载信息
-XX:NewRatio=4：表示设置年轻代（包括Eden和两个Survivor区）/老年代 的大小比值为1：4，这意味着年轻代占整个堆的1/5
-XX:SurvivorRatio=8：表示设置2个Survivor区：1个Eden区的大小比值为2:8，这意味着Survivor区占整个年轻代的1/5，这个参数默认为8
-Xmn20M：表示设置年轻代的大小为20M
-XX:+HeapDumpOnOutOfMemoryError：表示可以让虚拟机在出现内存溢出异常时Dump出当前的堆内存转储快照
-XX:+UseG1GC：表示让JVM使用G1垃圾收集器
-XX:+PrintGCDetails：表示在控制台上打印出GC具体细节
-XX:+PrintGC：表示在控制台上打印出GC信息
-XX:PretenureSizeThreshold=3145728：表示对象大于3145728（3M）时直接进入老年代分配，这里只能以字节作为单位
-XX:MaxTenuringThreshold=1：表示对象年龄大于1，自动进入老年代,如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代被回收的概率。
-XX:CompileThreshold=1000：表示一个方法被调用1000次之后，会被认为是热点代码，并触发即时编译
-XX:+PrintHeapAtGC：表示可以看到每次GC前后堆内存布局
-XX:+PrintTLAB：表示可以看到TLAB的使用情况
-XX:+UseSpining：开启自旋锁
-XX:PreBlockSpin：更改自旋锁的自旋次数，使用这个参数必须先开启自旋锁
-XX:+UseSerialGC：表示使用jvm的串行垃圾回收机制，该机制适用于单核cpu的环境下
-XX:+UseParallelGC：表示使用jvm的并行垃圾回收机制，该机制适合用于多cpu机制，同时对响应时间无强硬要求的环境下，使用-XX:ParallelGCThreads=设置并行垃圾回收的线程数，此值可以设置与机器处理器数量相等。
-XX:+UseParallelOldGC：表示年老代使用并行的垃圾回收机制
-XX:+UseConcMarkSweepGC：表示使用并发模式的垃圾回收机制，该模式适用于对响应时间要求高，具有多cpu的环境下
-XX:MaxGCPauseMillis=100：设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。
-XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低响应时间或者收集频率等，此值建议使用并行收集器时，一直打开

```

## 尚硅谷

### 性能调优步骤：

![](<../.gitbook/assets/image (118).png>)

### 性能调优指标：

* 响应时间 response time
* throughput
* concurrency thread?
* memory usage
* 相互间关系

![](<../.gitbook/assets/image (119).png>)



### MemoryLeak example:

![](<../.gitbook/assets/image (157).png>)

```java
//1. static generic memory leak
    static List list = new ArrayList<>();

    public void localMethod(){
        list.add(new Object()); //object can never been recycle
    }

//2. singleton, once you don't want to use it, it already there and cannot destroy

//3. 一个class的内部class，被其他所引用，这个class can never been destroy

//4. 各种connection, must use finally to close
        try {
            Connection conn = null;
        }catch(Exception ex) {
            
        }finally{
            conn.close();
        }
//5. local variable become class level variable
     solution: 如果不用了，可以手动设置msg = null; （null的内容可以被GC）
     
//7. 使用Map缓存的时候，经常会忘记。
solution1 使用固定size的LRU等cache
solution2 WeakHashMap map = new WeakHashMap();

```






















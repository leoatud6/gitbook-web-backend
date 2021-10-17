# JVM Basic

[https://www.youtube.com/watch?v=KbmZRzl16D4\&list=PLmOn9nNkQxJHNSznBikUX5sMv_XwDUYz4\&index=1](https://www.youtube.com/watch?v=KbmZRzl16D4\&list=PLmOn9nNkQxJHNSznBikUX5sMv_XwDUYz4\&index=1)



### 四大GC算法，七大GC

标记整理，清除，复制，分代

* Serial
* Parallel
* CMS(ConcMarkSweep) （互联网公司选择）
* G1（heap分块，分开进行gc）（既提高吞吐量，又减少GC时间。内存区域不再是新、老代，而是一个一个Region。没有内存碎片。可以预测停顿。）
* ParNew
* ParallelOld
* SerialOld (淘汰)



### OOM：

![](<../.gitbook/assets/image (140).png>)

* reduce full GC times, and part GC frequency
* tools: jstack, jconsole, jmap, jinfo --> VisualVM
* print GC info when running

![8 / 11 --> LTS](<../.gitbook/assets/image (109).png>)

OOM: out of memory --> heap not enough --> 性能调优原因

![](<../.gitbook/assets/image (110).png>)

#### 字节码 bytecode --> 使用合适的compiler--> get JVM support bytecode

#### 目前计算机领域三个难题：

* CPU
* OS
* Compiler

 不同语言之间可以通过API相互调用，Java使用native method调用C的内容

 **G1** GC default GC --> GMS (JDK9)

 **ZGC** (JDK11) _`ZGC性能远超过G1`_

Open JDK(维护期间半年) vs Oracle JDK(持续更新3年，商用款付费) **JDK11一致**

### HotSpot JVM

#### Overall process:

1. classloader --> loading bytecode = class file
2. Runtime data area(heap + stack + method area + program counter + native method stack)
3. **execution engine** (interpreter + JlT compiler(compile and cache) + GC)
4. native interface (with C++ ==> local method)

#### Stack-based instruction architecture 基于栈的指令集架构 (另外一种基于寄存器,受到硬件束缚)

javap -v filename.class   //反编译

#### JVM lifecycle (start, execute, exit)

 父类加载要早于子类（Object first load） 

 java运行的是process --> after run, JVM exit

![JVM整体结构](<../.gitbook/assets/image (112).png>)

### ClassLoader 类加载器

![](<../.gitbook/assets/image (114).png>)

1. Loading  （xx.class 有特定的文件标识） (bootstrap(write in C/C++, cannot print in Java, --> null) + extension + system  + self define...)
2. Linking (验证validation，准备prepare，解析resolve)
3. Initialization

final value在准备阶段就已经赋值，static不同。

```java
ClassLoader loader = Class.forName("java.lang.String").getClassLoader();
Thread.currentThread().getContextClassLoader();
ClassLoader.getSystemClassLoader().getParent();
```

#### 双亲委派机制：有上层，找上层，没有的话再用自定义的加载（如果重写String class）(可以避免核心类被篡改，也避免重复加载)

### Runtime Data Area / Environment  内存

```java
Runtime runtime = Runtime.getRuntime(); //singleton
```

![](<../.gitbook/assets/image (116).png>)

![](<../.gitbook/assets/image (115).png>)

#### PC Register: 程序计数器  -->每个thread一个

 pc register既没有GC，也没有OOM（stack有可能OOM的） 

#### Stack: stack frame --> 每个thread一个， 每个stack frame代表一个method

* local variables: reference / return address
* operand stack
* dynamic linking
* return address
* others... (minors)

![](<../.gitbook/assets/image (117).png>)

> 所有安全问题都是因为共享而产生的，同一个资源被不同thread访问，会产生安全问题。（stack是private的，不存在安全问题）





![](<../.gitbook/assets/image (113).png>)

























class loader --> memory model --> runtime engine

JVM引擎执行：字节码->无关优化->相关优化->寄存器分配->目标代码生成

```bash
#De compile java class code:
#(Public static void main(String args[]))
Javac className.java # compile
Javap -c className.class # decompile  : 反汇编
    
VM OPTIONS: 
    -Xms512m -Xmx512m -XX:+PrintGCDetails

```

## VM Option

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2Fky0WzZQFm6UMBiwI430V%2Ffile.png?alt=media)

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2F1jPUiNJpLTuaTqrpCwKO%2Ffile.png?alt=media)

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FvRv58lHs9FmT2v76kvd8%2Ffile.png?alt=media)

## ClassLoader

将class文件从disk加载到memory中 类加载器：

一共4种 :

* Bootstrap (C++) Extension: 
* Java AppClassLoader: 
* Java: classpath 
* selfDefined: in java.lang.classloader

## Native （native方法没有方法体）

native method is from system level, not JVM level， 类似于DLL

无法override string等java.lang package，保护原生内容

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FYRf7fxDqxIkFy2wBS0kt%2Ffile.png?alt=media)

![](<../.gitbook/assets/image (123).png>)





**Memory Knowlegde --> Monitoring --> Optimizing **

####  stack管运行， heap管存储

![](<../.gitbook/assets/image (124).png>)

## Heap \<Core knowledge in JVM>

调优的核心就是减少GC次数，因为GC也是需要消耗资源的  --> STW(stop the world)

JVM启动的时候heap创建（size fix）， 研究heap是为了避免OOM

 物理上不连续，逻辑上连续（virtual memory）

  TLAB: heap里面分给每个thread的小块儿，方便concurrency

 对象可以分配在stack中，所以heap不是存储所有object的

 大GC--> Stop The World (跟user thread冲突)

 

![ eden, s0, s1, old generation, permanent generation](<../.gitbook/assets/image (126).png>)

![](<../.gitbook/assets/image (127).png>)

```bash
-XX:+printGCDetails
-Xmx:10M        // -XX:InitialHeapSize   //default 1/4
-Xms:5M         // -XX:MaxHeapSize       //default 1/64
-Xmn:100M       // 设置新生代空间大小
//一般情况max and min设置成一样的值，避免频繁释放和初始化

jps 
jstat -gc 20184

-XX:NewRatio=4  // 新生代1，老年代4，一共5
-XX:SurvivorRatio
jinfo -flag SurvivorRatio 20183 

```

大部分instance都是Eden创建的，Eden销毁的

**`Eden : S0 : S1 ==>  8：1：1`**

**` YGC/Minor GC: 针对新生代的   Full GC/Major GC:针对old generation`**

  年龄计数器到达15（default）的时候放进old generation

![](<../.gitbook/assets/image (128).png>)



### 对象创建过程（内存分配策略）

 最后实在放不下-->byebye-->OOM

![](<../.gitbook/assets/image (129).png>)

* Minor GC: Eden, S0, S1 (S full不会触发GC)
* Major GC：比minor gc慢十倍以上
* Full GC: including meta data， System.gc()-->触发full GC / 老年代不足 /方法区不足

#### TLAB： thread local allocation buffer 

 Eden里面每个thread有单独的一份儿buffer, local thread，方便多个线程共同使用。--》因此head区域不全都是共享的

### 优化heap (对象未发生逃逸：仅仅在local method中使用)

#### Stack allocation: (stack没有gc) 需要开启：XX:+DoEscapeAynalysis (需要-server模式)

#### 同步省略/锁消除：如果用stack里面object当成锁，可以省略synchronized keywords

#### XX:+ElimiateAllocation

 

 


















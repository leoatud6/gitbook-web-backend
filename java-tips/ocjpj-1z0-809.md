---
description: Passed 01/28/2019
---

# OCJPJ 1Z0-809

#### Mac JDK version setup:

```
/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
```

####

#### 函数式接口：

![](<../.gitbook/assets/image (10).png>)

####

#### Function Interface:

![](<../.gitbook/assets/image (14).png>)



#### Generic:

![将类型作为变量传递](<../.gitbook/assets/image (15).png>)



#### Optional: (kill null pointer exception)

```java
public static<T> Optional<T> empty(){
    Optional<T> t = (Optional<T>) EMPTY;
}

//create optional class
public void createOptional(){
    
    Optional<Node> node1 = Optional.empty();
    
    Optional<Node> node2 = Optional.of(new Node()); //不接受null
    
    Optional<Node> node3 = Optional.ofNullable(new Node()); //不接受null
}
```

![](<../.gitbook/assets/image (16).png>)



#### Stream: (**`内部迭代`**VS_`外部迭代`_)

eg: 遍历集合的高级方式 （forEach?）

![](<../.gitbook/assets/image (17).png>)

![](<../.gitbook/assets/image (18).png>)

![](<../.gitbook/assets/image (19).png>)

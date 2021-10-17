# J2EE

## Overview

于是推出了J2EE， 像Java bean 一样， 这还是一个规范， 但是比Java bean 复杂的多， 其中有：

**JDBC**:  Java 数据库连接

**JNDI** :  Java 命名和目录接口， 通过一个名称就可以定位到一个数据源， 连jdbc连接都不用了

**RMI**：  远程过程调用，  让一个机器上的java 对象可以调用另外一个机器上的java 对象 

**JMS** :   Java 消息服务，  可以使用消息队列了

**JTA**：  Java 事务管理， 支持分布式事务， 能在访问、更新多个数据库的时候，仍然保证事务， 还是分布式。

**Java mail** : 收发邮件

J2EE 后来改成了Java EE。当然最重要的是， java bean 变成了 \*\*Enterprise Java bean \*\*, 简称 **EJB**。\


## Java Web Foundation

[https://github.com/h2pl/Java-Tutorial](https://github.com/h2pl/Java-Tutorial) for more resources

**SSH**: struts + spring + hibernate

#### **Project Structure**: 

* WEB-INF: src / classes/ lib /JSP/ web.xml
* META-INF

#### **Servlet**(Thread NOT safe, singleton but multi-thread) 

* init(): been called once --> when URL hit
* service(): including doGet() / doPost() --> core: MethodType
* destroy(): when not using
* GC

![](<../.gitbook/assets/image (180).png>)

![servlet configuration is in the web.xml file](<../.gitbook/assets/image (181).png>)

1. Web Client 向Servlet容器（Tomcat）发出Http请求；
2. Servlet容器接收Web Client的请求；
3. Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中；
4. Servlet容器创建一个HttpResponse对象；
5. Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给 HttpServlet对象；
6. HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息；
7. HttpServlet调用HttpResponse对象的有关方法，生成响应数据；
8. Servlet容器把HttpServlet的响应结果传给Web Client；

![](<../.gitbook/assets/image (182).png>)

> servlet use lots of listener, observer pattern

#### Cookie & Session

还有一点与 Session 关联的 Cookie 与其它 Cookie 没有什么不同，这个配置的配置可以通过 web.xml 中的 session-config 配置项来指定。

*   基于 URL Path Parameter，默认就支持 ()

    ```
     /path/Servlet?name=value&name2=value2&JSESSIONID=value3
     请注意如果客户端也支持 Cookie 的话，Tomcat 仍然会解析 Cookie 中的 Session ID，并会覆盖 URL 中的 Session ID。

    ```
* 基于 Cookie，如果你没有修改 Context 容器个 cookies 标识的话，默认也是支持的
* 基于 SSL，默认不支持，只有 connector.getAttribute("SSLEnabled") 为 TRUE 时才支持



### Java Logging

![](<../.gitbook/assets/image (185).png>)

* 门面型日志框架：

1. JCL：　　Apache基金会所属的项目，是一套Java日志接口，之前叫Jakarta Commons Logging，后更名为Commons Logging
2. SLF4J：  是一套简易Java日志门面，**本身并无日志的实现**。（Simple Logging Facade for Java，缩写Slf4j）

* 记录型日志框架:

1. JUL：　　JDK中的日志记录工具，也常称为JDKLog、jdk-logging，自Java1.4以来的官方日志实现。
2. Log4j：　 一个具体的日志实现框架。
3. Log4j2：   一个具体的日志实现框架，是LOG4J1的下一个版本，与Log4j 1发生了很大的变化，Log4j 2不兼容Log4j 1。
4. Logback：一个具体的日志实现框架，和Slf4j是同一个作者，但其性能更好。


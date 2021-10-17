---
description: 'SOA: Serice Oriented Architecture     SSH: Struts  + Spring + Hibernate'
---

# Spring

![video tutorial index](<../.gitbook/assets/image (1).png>)



## OAuth2

![](<../.gitbook/assets/image (219).png>)

![](<../.gitbook/assets/image (220).png>)

```java
先去第三方portal注册一个自己公司的账号：
app key & app secret 
授权call back page： 验证成功或失败后跳回的地址

第三方验证成功后返回的时候会在url上带一个类似UUID的code, code=xxxxx
use CODE to exchange the Access Token （can only use once）
需要手动拿这个code, app key, app secret --> get TOKEN

拿到access token基本就拿到了用户的所有信息，可以访问所有用户对外开放的内容



if first time login using OAuth2--》register a new account for this user
如果这个账号注册了，直接查出来信息使用，否则需要重新注册

同一个账户的uid是一样的
```

## Self defined SpringContext Module

```java
/**
 * Util to get bean in anywhere of the project
 */
@Component
public class SpringContextUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public static ApplicationContext getApplicationContext(){
        return applicationContext;
    }

    public static Object getBean(String name){
        return getApplicationContext().getBean(name);
    }

    public static <T> T getBean(Class<T> clazz){
        return getApplicationContext().getBean(clazz);
    }

    public static <T> T getBean(String name, Class<T> clazz){
        return getApplicationContext().getBean(name, clazz);
    }

}


//in the main class
    static ApplicationContext ctx;

    public static void main(String[] args) {
        Constants.ConstantPath.initialize(isRelease);
        //create propertie file
        ctx = new SpringApplicationBuilder(MainApplication.class).headless(false).run(args);
        //start the mainview later
        AppStarterUtil.start();
    }

    //need springboot way to destroy everything
    public static void exitAapplication(){
        int exitCode = SpringApplication.exit(ctx, new ExitCodeGenerator() {
            @Override
            public int getExitCode() {
                return 0;
            }
        });
        System.exit(exitCode);
    }
```

## IOC/DI

通过java reflection, proxy pattern

```java
//bean.xml
<Bean id=“self define” =“reference”>
<property name=“name of this object” ref=“实际创建的对象”>


//有参数构造
<constructor-arg index=“” value=“">
<constructor-arg name=“” value=“”>
<constructor-arg type=“java.lang.string” value=“”>
 
//team work
<import resource=“”>

//injection values
<property name=“” value=“”>
<property name=“array”>
    <array> / <list> / <set>
        <value> xx </value>
        <value> xx </value>        
        <value> xx </value>
    <map>
        <entry key=“” value=“”>

<property name=“”><null/><property>

<property name=“”>
    <prop>
        <prop key=“” >value<prop>
        <prop key=“” >value<prop>        
        <prop key=“” >value<prop>
        
//p namespace
Xmlns:p=“http://www.springframework.org/schema/p"

//c constuctor 
Xmlns:c=“http://www.springframework.org/schema/c”



```

###

### Spring Core Containers: 

core / beans / context / SpEL

### Data Access:

JDBC / ORM / OXM / **JMS **/ Transactions

### Web:

Websocket / servlet / web / portlet

### Point Cut:

AOP / Aspects / Instrumentation / Messaging

## ApplicationContext

config : **applicationContext.xml**

This is the IOC context, container

All the beans stored in context, 任何bean可以从这个container里面获取

(ClassPathXmlApplicationContext, BeanFactroy, FileSystemXmlApplicationContext(绝对路径), XmlWebApplicationContext)

_Xml: applicationContext.xml 代替： @Configuration_

_\<context: component-scan : base-package="xx,xx,xx" /> 代替： @ComponentScan _

_需要相应更改： @contextConfiguration(“classpath:applicationContext.xml")_

## Bean

id /  class

Property name / reference

#### 装配方式：

1. 隐式发现机制，@autowired （annotation）
2. Java中显式
3. Xml中显示

#### 注入方式（Injection method）:

1. constructor
2. setter
3. member (any method)

第三方组件只能通过Java装配或者xml装配实现，@Autowired无法使用。

#### Bean Scope:  @scope(scopeName = "xx") @Lazy

1. **si**ngleton: 一旦加载就会创建，积极加载，不管获取与否
2. prototype: 不会磨粉加载，每次加载拿到不同的object
3. session：会话范围内
4. request: 每次请求时重新创建，后面版本使用
5. application: 整个应用范围内

#### 创建方式：

1. static factory
2. dynamic factory, need to create factory first
3. no args
4. with args

## XML装配

default id = class name，多个相同bean的时候需要通过id区分

```markup
    <constructor-arg ref=“xx” />
    <constructor-arg name=“xx”/ index=“x”/type=“java.lang.String" value=“xx” /> 
    
    <list>/<set>/<array>
    <value> </value> //single or primitive type
    <ref bean=“” /> //generic object DI
    <map>
        <entry=“” value=“”> for value DI
        <entry=“” value-ref=“”> //for generic object DI
    </map>
    
    setter injection
    <property name=“” value=“”/>
    <property name=“” ref=“object” />
    
    <util:set / list / map… >
    <ref bean=“” />
```

## JUnit Test

目录必须与main folder一致， 注意junit test 和 springContext必须版本一致

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AppConfig.class)
```

## Web Application Three Layers

1. Dao layer: @Repository 
2. Service layer: @Service
3. Web controller layer: @Controller / @RestController

## AOP (底层 = 代理模式)

```java
<aop:aspectj-autoproxy />
<context: package-scan>
```

@Aspect: 声明这是切面组件

@Pointcut(execution("xx.xx.xx")) : 切入点

@Before("pointcut method name) 

应用：log, security, transaction, exception

#### 五个切入点：

1. **before advice **
2. **After returning advice **
3. **After throwing advice **
4. **After finally advice **
5. **Around advice: 包围连接点，最强大的一种**

```java
<aop:config>
    <aop:pointcut id=“pointcut” expression=“”/>
    <aop:advisor advice-ref=“” pointcut-ref=“pointcut”/>
    <aop:pointcut id=“pointcut” expression=“execution(* package_path.*.*(..))” />  
```

## Proxy

类比： 对于租客来说，房东是不透明的，直接跟中介打交道

应用： cache, firewall, synchronized

proxy与host implement同一个接口，有代理走代理

#### dynamic proxy: 

InvocationHandler --> invoke()

```java
handler.getClass().getClassLoader();
interface.getClass().getInterfaces();
```

#### reflection:

```java
    dom4j + java reflection
    userService = Class.forName(“com.service.UserService”);
    userService.setName(“dd”);
```

## Web

all servlets created by **tomcat**

自己创建一个servlet, extends from HttpServlet



## SPI

Service provider interface --> 核心： decouple

spring中使用大量SPI

![“META-INF/services/”](<../.gitbook/assets/image (8).png>)

![](<../.gitbook/assets/image (9).png>)

### Logging

Spring里面需要手动增加

### Interface: log4j2

### Implementation: logback + slf4j

also --> log4j.properties

{% hint style="info" %}
注意， log system引入的包会重合，需要去重
{% endhint %}


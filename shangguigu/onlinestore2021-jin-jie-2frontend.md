# onlinestore2021\_进阶2\_性能提升（压测、缓存）

## Thymeleaf渲染页面

### 动静分离

![](<../.gitbook/assets/image (90).png>)

每个service管理自己的页面，microservice

```bash
#关闭缓存可以看到事实效果-->配合devtool 一起工作
spring:
    thymeleaf:
        cache: false

#配合redirect controller使用: view resolver会帮忙拼串
thymeleaf default prefix : classpath:templates/
thymeleaf default surfix : .html

#必须指定，否则无法访问静态资源
spring:
  mvc:
    static-path-pattern: /static/**
    
#实时动态更新
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>

之后control + F9 --> rebuild the project即可
一定要在缓存关闭的情况下才有


```

### Controller (主要handle redirect request, 因此不用@RestController)

```java
//Spring MVC source code : 以下四个位置的资源都可以default寻找
ResourceProperties {
	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { 
									"classpath:/META-INF/resources/",
										"classpath:/resources/", 
										"classpath:/static/", 
										"classpath:/public/" };
}

=== 主页内容 ===
//因为有的要redirect， 因此不用@RestController
@Controller
public class IndexController {

    === business logic here, find category ===
    //可以添加多种路径
    @GetMapping({"/", "index.html"})
    public String indexPage() {
        return "index";
    }
}

@GetMapping({"/", "index.html"})
public String indexPage(Model model) {  //model from spring ui
    
    List<CategoryEntity> categoryEntityList = categoryService.getLevel1Categorys();
    //this is the model body for the response, SpringMVC
    model.addAttribute("categorys", categoryEntityList);
    return "index";
}
			
```

```markup
<!--thymeleaf start here!-->

首先添加名称空间
<html lang="en" xmlns:th="http://www.thymeleaf.org">

Simple expressions:
VariableExpressions: ${...} 
SelectionVariableExpressions: *{...} 
MessageExpressions: #{...} 
LinkURLExpressions: @{...} 
Fragment Expressions: ~{...}


<li th:each="category:${categorys}">
    <a href="#" class="header_main_left_a" th:attr="ctg-data=${category.catId}">
    <b th:text="${category.name}">家用电器</b></a>
 </li>
 
 
遍历的时候首先写上名字，下文中才能够使用 
<li th:each="brand:${result.brands}">
    <a href="/static/search/#">
        <img th:src="${brand.brandImg}" alt="">
        <div th:text="${brand.brandName}">
            华为(HUAWEI)
        </div>
    </a>
</li>


```

### Pagenation (front + end)

```javascript
===Pagenation desplay===
// callback function
//page_a是被绑定的class, jQuery的写法
$(".page_a").click(function () {
    var pn = $(this).attr("pn");
    var href = location.href+"";
    if(href.indexOf("pageNum")!=-1){
        location.href = replaceParamVal(href, "pageNum", pn);
    }else{
        location.href = location.href + "&pageNum=" + pn;
    }
    //return false是为了禁用默认行为
    return false;
});

//replace pathvariable  (without repeat, remove repeat)
function replaceParamVal(url, paramName, replaceVal){
    var oUrl = url.toString();
    var re = eval('/('+paramName+'=)([^&]*)/gi');
    var nUrl = oUrl.replace(re, paramName+"="+replaceVal);
    return nUrl;
}


//another example
$(".sort_a").click(function(){
    //others颜色变回去
    $(".sort_a").css({"color":"#333","border-color":"#CCC","background":"#FFF"});
    //被点击的颜色改变
    $(this).css({"color":"#FFF","border-color":"#e4393c","background":"#e4393c"});
    return false;
});

//各种三元符号之类的：：：
<div class="filter_top_left" th:with="p = ${param.sort}">
    <a th:class="${(!#strings.isEmpty(p)&& #strings.startsWith(p,'hotScore') &&#strings.endsWith(p,'desc'))?'sort_a desc':'sort_a'}"
       th:attr="style=${(#strings.isEmpty(p) || #strings.startsWith(p,'hotScore'))?
       'color:#FFF;border-color:#e4393c;backgroun:#e4393c':'color:#333;border-color:#CCC;backgroun:#FFF'}"
       sort="hotScore" href="/static/search/#">综合排序 [[${(!#strings.isEmpty(p)&& #strings.startsWith(p,'hotScore') &&#strings.endsWith(p,'desc'))?'↓':'↑'}]]</a>

//看是否checkbox被选中
$("#showHasStcok").change(function (){
    alert($(this).prop("checked"));
    return false;
});

```

### bread crunches 面包屑导航

#### 面包屑的内容就是url？之后所有的内容

```javascript
@GetMapping("/list.html")
public String listPage(SearchParam param, Model model, HttpServletRequest request){
    //httpservlet request自带查询内容，可以用作是面包屑
    String queryString = request.getQueryString();
    param.set_queryString(queryString);
    ....
}

//在处理url的时候注意编码方式，不同语言等，还有就是空格%20的替换
String encode = "";
try {
    encode = URLEncoder.encode(attr, "UTF-8");
    encode = encode.replace("+","%20");
} catch (UnsupportedEncodingException e) {
    e.printStackTrace();
}


//使用远程调用feign的，可以加上cache
@Cacheable(value="attr", key = "'attrinfo:'+#root.args[0]")

```

[https://www.json.cn//](https://www.json.cn)

## Nginx 域名访问环境

nginx--> gateway --> detail service server

#### nginx代理给gateway的时候，会丢失request的host信息（也会丢掉其他很多东西，cookies之类的）： proxy_setheader Host $host

```bash
#enable auto run
docker update nginx --restart=always
docker restart nginx

#nginx 配置文件结尾一定要加上“;”

#gateway server 的 routing rules
# gulimall_host
- id: gulimall_host_route
    uri: lb://gulimall-product
    predicates:
      - Host=**.gulimall.com


#add set header for nginx to Gateway
location / {
    proxy_set_header Host $host;
    proxy_pass http://gulimall.com;
}

```

![nginx有外网地址，对外暴露。内网只有内网可以访问。](<../.gitbook/assets/image (91).png>)

![nginx/conf/nginx.conf](<../.gitbook/assets/image (92).png>)

## Pressure Test 压力测试(Jmeter、Gatling)

keywords: 重复，并发，量级，随机变化

#### 内存泄露， 并发，同步 （潜在问题）

#### 性能指标：

* Response Time (RT)
* HPS: hits per second
* TPS: transaction per second 完整的一个查询，like下单，上架
* QPS: query per second
* Max response time:
* Min response time:
* 90% response time:

#### 衡量标准：

* 吞吐量thourghput
* 响应时间response time
* 错误率error rate

#### Windows本身提供给TCP/IP的端口号：1024-5000， 4 mins to recollect them。 如果短时间请求多了port会占满。（Jmeter address already in use (Windows only?!)-->需要set注册表，添加两个项目，手动增加最大连接数和回收数）

#### CPU密集型（计算类） VS IO密集型（查询类）

### 性能监控

####  JVM主要优化 heap + Metaspace(after Java8) --> GC

#### young GC(fast) VS full GC(slow)

 创建对象->看看新生代放得下嘛-->放不下再看老年代-->还放不下就full GC-->还放不下就OOM（out of memory）   (速度逐渐变慢)

  每次小GC，一定要把Edem清理干净，（放入幸存者区 或者 老年区）

### Jconsole & Jvisualvm(after JDK1.6)

直接cmd运行jconsole即可

 安装Visualvm plugins

![](<../.gitbook/assets/image (95).png>)

### 压测（JMeter）

![](<../.gitbook/assets/image (96).png>)

```bash
#docker for nginx
docker stats 

nginx主要是计算型的，占用CPU很大，memory基本不变

```

#### 中间件越多，性能损失越大，大多损失在中间network的传输中

#### 对于service具体服务可以做的：enable cache， disable/limit log， optimized DB(查询id变为index)，调整程序运行内存(memory-->Heap)

1. 中间件：越少越好
2. business logic：DB (use ES, Redis)、 template render (CPU, enable cache)、static resources（put into Nginx） 
3. nginx：动静分离：所有static的内容放在nginx
4. 具体serviceImpl：对需要查询的数据做cache,，Java代码中反映：将多次查询变为一次查询

```java
-Xmx1024m 
-Xms1024m 
-Xmn512m   ：新生代和伊甸园区内存

===extract method===
private List<CategoryEntity> getParent_cid(List<CategoryEntity> selectList, Long parent_cid) {
    List<CategoryEntity> collect = selectList.stream().filter(i -> {
        //filter里面放的是条件，满足条件的可以后续被collect
        return i.getParentCid() == parent_cid;
    }).collect(Collectors.toList());
    return collect;
    //return baseMapper.selectList(new QueryWrapper<CategoryEntity>().eq("parent_cid", v.getCatId()));
}
```

## Cache == Cash 缓存 & 分布式🔐

 数据库仅仅承担“落盘”工作，LRU: map

 缓存数据特点：

* 即时性，数据一致性要求不高的
* 访问量达，且更新频率不高（read heavy write rare）

### Local Cache: 

#### 存在很多问题，在cluster deployment中， 还有数据一致性问题（集群中不同机器的cache不同）

![](<../.gitbook/assets/image (97).png>)

![应该使用这种分布式缓存](<../.gitbook/assets/image (98).png>)

### Distributed Cache （Redis）:

缓存中间件：Redis: 不同的Redis, 每个Redis存不同的内容，但是只有一份儿。

```java
import spring-boot-starter-data-redis

spring:
  redis:
    host: 192.168.4.52
    port: 6379

use springboot StringRedisTemplate

//docker start redis
docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf

#enter into redis client
docker exec -it redis redis-cli
docker restart redis

#auto start
docker update redis --restart=always 


//增加cache level的成品
@Override
public Map<String, List<Catelog2Vo>> getCatalogJson() {
    String catalogJSON = redisTemplate.opsForValue().get("catalogJSON");
    if(StringUtils.isEmpty(catalogJSON)){
       //empty --> use DB
        Map<String, List<Catelog2Vo>> catalogJsonFromDb = getCatalogJsonFromDb();
        //put into cache
        redisTemplate.opsForValue().set("catalogJSON", JSON.toJSONString(catalogJsonFromDb));
        //can directly return
        return catalogJsonFromDb;
    }
    return JSON.parseObject(redisTemplate.opsForValue().get("catalogJSON"), new TypeReference<Map<String, List<Catelog2Vo>>>(){});
}

//坑
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {}
Lettuce底层操作Redis会有OutOfDirectMemory error
推荐使用Jedis的底层客户端
封装后：redisTemplate即可


redis command:
get xxx
set xxx xxx NX
set xxx xxx xxx EX NX
ttl xxx
del xxx
```

### 1. 缓存穿透 penetration (查一个不存在的数据)

 大并发同时给server -->所有都去差DB --> DAO崩溃

 **原因**：没有将null result放在Redis, 查询永不存在的数据

** 解决：**put null result, and set expire time

### 2. 缓存雪崩 avlanche（多个数据有相同的过期时间）

 大并发同时给server查这些刚刚过期的-->同时去查DB-->雪崩

 **原因：**大面积同时失效

 **解决：**过期时间set random time

### 3. 缓存击穿 breakdown (访问hotkey)

** 原因：**某一个高频热点key失效

 **解决：**add lock （只能让一个人查DB，然后放到Redis）

```java
//set one day expired
redisTemplate.opsForValue().set("catalogJSON", JSON.toJSONString(catalogJsonFromDb), 1, TimeUnit.DAYS);

分布式中，sychnorized没用，因为local lock只能锁住当前service, cluster中没有用
本地锁JUC等，没有卵用==》need 分布式锁


```

![db query后，先写进cache，再返回](<../.gitbook/assets/image (99).png>)

### SetEX,NX  == setIfAbsent(set if not exist-->分布式核心)

set lock以后，执行完业务逻辑一定要delete lock -->如果没有成功delete lock, 就dead lock了

\--> set auto expired time

#### add lock & delete lock -->必须要原子操作

```java
set lock haha NX

//call it self, count as retry, 自旋
//set expire time (atomic)
public Map<String, List<Catelog2Vo>> getCatalogJsonFromDbWithRedisLock() {
    String uuid = UUID.randomUUID().toString();
    Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", uuid, 300, TimeUnit.SECONDS);
    if(lock){
        Map<String, List<Catelog2Vo>> dataFromDb;
        try{
            dataFromDb= getDataFromDb();
        }finally{
            //此处应当使用lua脚本保证delet key的原子性
            String script = "if redis.call('get', KEYS[1] == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
            redisTemplate.execute(new DefaultRedisScript<Long>(script, Long.class), Arrays.asList("lock"), uuid);
        }
        return dataFromDb;
    }else{
        //add lock failed--> retry
        //Thread.sleep(100);
        return getCatalogJsonFromDbWithRedisLock();
    }
}

```

#### ==>最终结论还是：使用redisson (lue的script， 保证执行的原子性)

**RedLock**: Redis distributed locks

```java
//redisson
@Configuration
public class MyRedissonConfig {

    static String redisAddress = "redis://192.168.4.52:6379";

    @Bean(destroyMethod = "shutdown")
    public RedissonClient redisson() throws IOException {
        Config config = new Config();
        config.useSingleServer().setAddress(redisAddress);
        return Redisson.create(config);
    }
}
```

### Redisson (Reentrant lock)

所有的lock尽量使用reentrant lock， 避免dead lock出现。

Redisson就是reentrant lock

#### unlock()，一定要放在finally里面，必须解锁

#### redisson lock自动续期如果没完成（default 30s for the first time, every time use time/3 续期至满，直到业务结束）-->看门狗watchdog

```java

RLock lock = redissonClient.getLock("lock");
lock.lock();
lock.unlock();

//如果手动设置时间加锁，不会自动续期
//正常使用时，self define the expire time
lock.lock(10, TimuUnit.Second);


ReadOnWriteLock lock;
lock.readLock().lock();
lock.writeLock().unlock();
更改数据用write lock， 读数据用read lock==》一定能读到最新的数据
写🔐没有释放，读🔐就必须等待。两个用的同一把锁。
读🔐没有释放，写🔐就必须等待。

try{}catch(){}
finally{
    //解锁必须放在🔐finally里面
    lock.unlock();
}


```

#### ReadWriteLock(只要有write，就需要等待)

* **read + write:  need to wait**
* read + read: no different
* write + write: normal 
* **write + read: need to wait**

### **Semaphore (限流，总量n)+ CountDownLatch + ReadWriteLock**

** 区分阻塞和非阻（阻塞的话就一直等，非阻塞就是try ）**

```java
@ResponseBody
@GetMapping("/go")
public String go() {
    RSemaphore park = redissonClient.getSemaphore("park");
    park.release();
    return "go ok";
}

@GetMapping("/park")
@ResponseBody
public String park() throws InterruptedException {
    RSemaphore park = redissonClient.getSemaphore("park");
    park.acquire(); //return (boolean)tryAcquire();
    return "park ok";
}

@GetMapping("/lockdoor")
@ResponseBody
public String lockDoor() throws InterruptedException {
    RCountDownLatch door = redissonClient.getCountDownLatch("door");
    door.trySetCount(5);
    door.await();

    return "...";
}

@GetMapping("/gogogo/{id}")
@ResponseBody
public String gogogo(@PathVariable("id") Long id) {
    RCountDownLatch door = redissonClient.getCountDownLatch("door");
    door.countDown();//count down, -1
    return id + " gogogo...";
}
```

**锁的粒度，越细越好，加上具体要查询的内容的名字和ID，用String单独表示。**

### 缓存数据一致性（涉及到DB的write, update）

#### 双写模式（改完数据库，write to Cache）--> dirty data

#### 失效模式（改DB时删Cache，等待下次DB查询）

#### 最终一致性（eventually consistence）

 实际方案要根据数据的特性以及系统的特性来区分，过度开发也不好，速度慢，花时间。

### Canal解决缓存数据一致

![ali开源的中间件](<../.gitbook/assets/image (100).png>)

## Spring Cache

![cacheManager](<../.gitbook/assets/image (101).png>)

可以有多种实现，可以redission， concuranthashMap...

```java
import spring-boot-starter-cache

#spring.cache.type=redis

@Cacheable (trigger cache)
@CacheEvict (delete cache)
@CachePut (update cache)  双写模式使用这个注解
@Caching (regroup, multible operation) 
@CachecConfig (share the same config 4 cache)


@EnableCaching
public class MainApplication{}

缓存的分区，按照业务类型分类
@Cacheable({"category", "product"})

default 过期时间 -1： never expire  ==> need to reset
default serialization: jdk         ==> need to set JSON
default key: auto-generate: name + SimpleKey[]  ==> need to custmized

PART SOLUTION: on properties file
spring.cache.redis.time-to-live=60000  //1mins=60000ms
@Cacheable(value = "category", key = "#root.method.name") //change key SpEL语法
spring.cache.redis.key-prefix=CACHE_
spring.cache.redis.use-key-prefix=true
spring.cache.redis.cache-null-values=true

@Configuration
@EnableCaching
@EnableConfigurationProperties(CacheProperties.class)  //可以使用配置文件内容
public class MyCacheConfig {

    @Autowired
    CacheProperties cacheProperties; //autowire配置文件内容

    @Bean
    public RedisCacheConfiguration getRedisCacheConfiguration(CacheProperties cacheProperties){

        RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig();
        cacheConfig = cacheConfig.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()));
        cacheConfig = cacheConfig.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericFastJsonRedisSerializer()));
        
        //following is from properties file and redisConfigSource
        CacheProperties.Redis redisProperties = cacheProperties.getRedis();
        if(redisProperties.getTimeToLive()!=null){
            cacheConfig = cacheConfig.entryTtl(redisProperties.getTimeToLive());
        }
        if(redisProperties.getKeyPrefix()!=null){
            cacheConfig = cacheConfig.prefixCacheNameWith(redisProperties.getKeyPrefix());
        }
        if(redisProperties.isCacheNullValues()){
            cacheConfig = cacheConfig.disableCachingNullValues();
        }
        if(redisProperties.isUseKeyPrefix()){
            cacheConfig = cacheConfig.disableKeyPrefix();
        }
        return cacheConfig;
    }
}

@Caching：可以组合多个注解使用:::
//同时清空两个cache
@Caching(evict = {
        @CacheEvict(value = "categroy", key="#root.methodName"),
        @CacheEvict(value = "category", key = "'anotherName'")
})

//清空所有与category相关的分区，缓存分区的作用
@CacheEvict(value = "category", allEntries=true)

//添加sync option， 可以进行加锁，解决击穿问题
@Cachable(value="xx", key="xx", sync=true)

常规数据（read heavy write rear），完全可以使用spring-cache
特殊数据： 读写锁，cancal， 等，特殊对待

```




























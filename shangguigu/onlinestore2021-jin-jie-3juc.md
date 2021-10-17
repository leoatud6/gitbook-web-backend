# onlinestore2021\_进阶3\_JUC\&Auth

## 异步

1. extend from thread
2. implement runnable
3. implement callable + FutureTask
4. use thread pool (仅此方式可以控制资源，不用瞬间资源耗尽，性能稳定)

```java
//callable + FutureTask
FutureTask<Integer> futureTask = new FutureTask<>(new CallClass());
new Thread(futureTask).start();
//get到return value, 阻塞等待
Integer integer = futureTask.get();


ExecutorService executorService = Executors.newFixedThreadPool(10);
executorService.execute(new Runnable());
executorService.submit(new Callable());  //has return value


/**
 * corePoolSize: 一直存在的thread, eg:10
 * maxPoolSize: 最大允许同时运行的thread, eg:200
 * keepAliveTime: 等待n时间后释放空闲的thread, 释放的内容是max-core （core不会释放）
 * BlockingQueue<Runnable>: workQueue, 很多任务的时候，任务放在queue里面保存， size = Integer.Max
 * threadFactory: Executors.defaultThreadFactory()
 * handler: queue full后，discard(older) / abort / callerRun / rejected
 *          eg: RejectedExecutionHandler, 可以拒绝新的任务
 *
 */
new ThreadPoolExecutor(5,
        200,
        10,
        TimeUnit.SECONDS,
        new LinkedBlockingDeque<>(10000),
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.AbortPolicy());

使用threadpool之后，程序不会结束，除非销毁线程池
service.shutdown();
```

### CompletableFuture 异步编排

 多个异步现成互相有依赖关系

####  function interface needed.

* runAsync();
* supplyAsync();

#### 异步有顺序执行：

* thenRun();
* thenRunAsync()
* thenAccept();
* thenApply();

#### 可以无限嵌套自己，前两个任务都结束后才能执行第三个任务：

* runAfterBoth();
* runAfterBothAsync();
* thenAcceptBoth();   //with return value
* thenCombineAsync();    //with return value

#### 一个完成后，进行下一个：(A,B其中一个完成就可以进行下一个)

* runAfterEitherAsync();  //不感知结果，自己也无返回值
* acceptEitherAsync();    //只接收上一个的result，可以感知结果，自己无返回值
* applyToEitherAsync();  //既可以感知结果，也可以返回结果

#### 多任务组合：最后执行完使用get()进行阻塞，等待所有的完成。

* allOf();    //等待所有任务完成
* anyOf();  //任何一个任务完成

```java
//no return value
CompletableFuture.runAsync(()->{
    //task here
    System.out.println(Thread.currentThread().getId());
},executorService);
executorService.shutdown();

//future return value, need to return 
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    System.out.println(Thread.currentThread().getName());
    return 10;
}, executorService).handle((res, thr) -> {  //after the method, can add handler
    if (res != null)
        return res * 2;
    else if (thr != null)
        return 0;
    else
        return -1;
});
System.out.println(future.get());

```

### 实战应用

#### 不同的任务，有互相依赖的，需要级联执行。最后要用allOf().get() 等待所有异步任务执行完

```java
@Override
public SkuItemVo item(Long skuId) throws ExecutionException, InterruptedException {

    SkuItemVo skuItemVo = new SkuItemVo();

    CompletableFuture<SkuInfoEntity> infoFuture = CompletableFuture.supplyAsync(() -> {
        //1. basic information : skuInfo
        SkuInfoEntity info = this.getById(skuId);
        skuItemVo.setInfo(info);
        return info;
    }, executor);

    CompletableFuture<Void> attrFuture = infoFuture.thenAcceptAsync((result) -> {
        //3. salesAttrs, and other attrs: spu attrs
        Long infoSpuId = result.getSpuId();
        List<SkuItemSaleAttrVo> saleAttrVos = skuSaleAttrValueService.getSaleAttrsBySpuId(infoSpuId);
        skuItemVo.setSaleAttrs(saleAttrVos);
    }, executor);

    CompletableFuture<Void> descFuture = infoFuture.thenAcceptAsync((result) -> {
        //4. spuInfo
        Long spuId = result.getSpuId();
        SpuInfoDescEntity spuInfoDescEntity = spuInfoDescService.getById(spuId);
        skuItemVo.setDesp(spuInfoDescEntity);
    }, executor);

    CompletableFuture<Void> skuGroupAttrFuture = infoFuture.thenAcceptAsync((result) -> {
        //5. spu 规格参数info
        Long catalogId = result.getCatalogId();
        List<SpuItemAttrGroupVo> attrGroupVos = attrGroupService.getAttrGroupWithAttrsBySpuId(result.getSpuId(), catalogId);
        skuItemVo.setGroupAttrs(attrGroupVos);
    }, executor);

    //2. skuImg
    CompletableFuture<Void> imageFuture = CompletableFuture.runAsync(() -> {
        List<SkuImagesEntity> images = imagesService.getImagesBySkuId(skuId);
        skuItemVo.setImages(images);
    }, executor);

    //wait for all finish, use get() to wait all finish: must have this get method
    CompletableFuture.allOf(attrFuture, descFuture, skuGroupAttrFuture, imageFuture).get();

    return skuItemVo;
}
```

## onlinestore详情页

```java
//string 处理
<a th:href="|http://item.gulimall.com/${product.skuId}.html|">

```

```markup
mapper里面只要有嵌套结果集（一个bean里面有另外一个static class），都要自己手动封装
<resultMap id="SpuItemAttrGroupVo" type="com.onlinestore.product.vo.SkuItemVo.SpuItemAttrGroupVo">
    <result column="attr_group_name" property="groupName"></result>
    <collection property="attrs" ofType="com.onlinestore.product.vo.SkuItemVo.SpuBaseAttrVo">
        <result column="attr_name" property="attrName"></result>
        <result column="attr_value" property="attrValue"></result>
    </collection>
</resultMap>
<select id="getAttrGroupWithAttrsBySpuId"
        resultType="SpuItemAttrGroupVo">
    各种left join的语句
</select>
```

```java
//返回的时候带上MODEL, 然后thymeleaf里面可以通过model的内容渲染便利，
//封装好vo即可
@GetMapping("/{skuId}.html")
public String skuItem(@PathVariable("skuId") Long skuId, Model model){
    System.out.println("ping " + skuId + " .html");
    SkuItemVo vo = skuInfoService.item(skuId);
    model.addAttribute("item", vo);
    return "item";
}
```

### 不同属性之间的页面跳转（很多细节）

![](<../.gitbook/assets/image (106).png>)

```java
$(".sku_attr_value").click(function () {
    var skus = new Array();

    $(this).addClass("clicked");
    var curr = $(this).attr("skus").split(",");
    skus.push(curr);
    $(this).parent().parent().find(".sku_attr_value").removeClass("checked");
    $("a[class='sku_attr_value checked']").each(function () {
        skus.push($(this).attr("skus").split(","));
    });

    var filterOrigin = skus[0];
    for (var i = 1; i < skus.length; i++) {
        filterOrigin = $(filterOrigin).filter(skus[i]);
    }
    //最后需要跳转的地址
    location.href = "http://item.gulimall.com/"+filterOrigin[0] + ".html";
});
```

### 创建Executor ThreadPool + 添加Properties

```java
//@EnableConfigurationProperties(ThreadPoolConfigProperties.class) 如果配置文件Bean有@Component就不需要这句了
@Configuration
public class MyThreadConfig {
    @Bean
    public ThreadPoolExecutor getThreadPool(ThreadPoolConfigProperties pool){
        return new ThreadPoolExecutor(
                pool.getCoreSize(),
                pool.getMaxSize(),
                pool.getKeepAliveTime(),
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(10000),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );
    }
}


@ConfigurationProperties(prefix = "gulimall.thread")
@Component
@Data
public class ThreadPoolConfigProperties {
    Integer coreSize;
    Integer maxSize;
    Integer keepAliveTime;
}

gulimall.thread.core-size=20
gulimall.thread.keep-alive-time=10
gulimall.thread.max-size=200

//之后在任何地方就都可以使用了
@Autowired
ThreadPoolExecutor executor;
```

## Auth/OAuth

### 社交登陆&单点登录SSO

 templates folder(thymeleaf)下的页面，非index.html，都需要redirect到达。

```java
//直接跳转到templates里面的login.html
@Controller
public class LoginController {
    @GetMapping("/login.html")
    public String loginPage(){
        return "login";
    }
}
```

### 发送验证码（front-end）

```java
//send sms code
$(function () {
    $("#sendCode").click(function () {
        //send code to SMS

        //countdown
        if (!$(this).hasClass("disabled")) {
            timeoutChangeStyle();
        }
    });
});
var num = 10;
function timeoutChangeStyle() {
    $("#sendCode").attr("class", "disabled");
    if (num == 0) {
        $("#sendCode").text("Send SMS");
        num = 10;
        $("#sendCode").attr("class", "");
    } else {
        var str = num + "s send again";
        $("#sendCode").text(str);
        setTimeout("timeoutChangeStyle()", 1000);
    }
    num--;
};

//jQuery send get 
$.get("/sms/sendcode?phone=" + $("phoneNum").val());

```

### 发送验证码（back-end）

 使用aliyun的service, 需要传入phone number & code (UUID self generate)

 在专门的third party module中处理，Feign远程调用

#### 验证码都是存在Redis里面的。

#### lots of details: 

* 防止重发，重新刷新页面后 --> 接口防刷
* 验证码再次校验-->code放在Redis里面_**redisTemplate.opsForValue().set(AuthConstant.SMS_CODE_CACHE_PREFIX+phone, code, 300, TimeUnit.SECONDS);**_

```java
@Data
public class UserRegistVo {

    @NotEmpty(message = "please input username")
    @Length(min = 6, max = 18, message = "must between 6 - 18")
    private String username;
    @NotEmpty(message = "please input password")
    @Length(min= 3, max =18, message = "must between 3 - 18")
    private String password;
    @Pattern(regexp = "^[1][3-9][0-9]{9}$", message = "phone number wrong format")
    @NotEmpty(message = "please input phone number")
    private String phone;
    @NotEmpty(message = "please input code")
    private String code;
}


//首先尽量避免使用forward，推荐使用redirect, forward不改变请求方式
//同时方法中使用RedirectAttributes
//BindingResult中的内容可以给thymeleaf使用
@PostMapping("/regist")
public String regist(@Valid UserRegistVo vo, BindingResult result, RedirectAttributes attributes) {
    //back to main page
    if (result.hasErrors()) {
        Map<String, String> map = result.getFieldErrors().stream().collect(Collectors.toMap(FieldError::getField, FieldError::getDefaultMessage));
        attributes.addFlashAttribute("errors", map);
        return "redirect:/reg.html";
    }
    return "redirect:/login.html";
}



```

#### post not support exception: 如果使用的是forward, 要注意method是来源于原请求的，同是get的话，不会过。

#### 重定向携带数据使用session原理， 只要跳到下个页面，session里面数据就删掉HttpSession

#### 令牌机制token：一旦验证完成，就删掉（redis中）

#### JSON接受数据一定要@RequestBody



## Spring MVC 映射空方法

#### viewController

```java
//相当于手动在Controller里面set Redirect
@Configuration
public class GulimallWebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/login.html").setViewName("login");
        registry.addViewController("/reg.html").setViewName("reg");
    }
}
```

## Register --> Feign to Member Module

### 自定义exception

```java
public class PhoneDupException extends RuntimeException{
    public PhoneDupException() {
        super("Phone number Duplicate");
    }
}

//operation
@Override
public void checkPhoneUnique(String phone) {
    throw new PhoneDupException();
}
```

### 对已有member data进行查询--> count>0

```java
@Override
public void checkPhoneUnique(String phone) throws PhoneDupException {
    Integer mobile = this.baseMapper.selectCount(new QueryWrapper<MemberEntity>().eq("mobile", phone));
    if (mobile > 0) {
        throw new PhoneDupException();
    }
}
```

### 所有的password都要encrypt

 可逆 VS 不可逆

 MD5， message digest algorithm5 -->网上有破解的

 MD5 + Salt

```java
//spring 自带encrypt
BCryptPasswordEncoder encrpter = new BCryptPasswordEncoder();
String encodeString = encrpter.encode(vo.getPassword());
encrpter.matches(input, origin);
```

### 219截止








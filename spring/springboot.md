# SpringBoot

[https://www.yuque.com/atguigu/springboot](https://www.yuque.com/atguigu/springboot)

[https://gitee.com/leifengyang/springboot2](https://gitee.com/leifengyang/springboot2)

## SpringBoot 2.x.x (2021) 基础入门  ===========================

why： 对完整生态的全面支持 ==》 Server-less, Cloud deploy

 microservice (distributed)--> http light weight --> auto deployment

#### 响应式编程，导入WebFlus (instead of SpringMVC)

![](<../.gitbook/assets/image (147).png>)

####   Challenges:

* 容错机制，network, server down
* 服务监控
* 远程调用
* 服务发现
* logging
* async task
* load balancing

![](<../.gitbook/assets/image (145).png>)

#### Deploy: cloud-native  -->线上主要是Devs problem

* 服务自愈
* 弹性收缩
* 服务隔离
* 流量治理
* 灰度发布
* 自动化部署

### Maven

#### parent starter: 主要做依赖管理，dependencies version control

1. 引入springboot start dependencies
2. Auto-configuration 配置
3. ConfigurableApplicationContext = SpringApplication.run(xx.class);

#### 默认配置都是最终映射到MultipartProperties 上，配置文件会最终绑定到每个class上--> in container

```markup
//maven code
maven pom packaging default : jar  自带运行环境，直接可以运行
<packaging>jar</packaging>
  
maven plugin --> pack to jar auto
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>

default tomcat starting port: 8888

修改version版本
<properties>
    <mysql.version>5.1.43</mysql.version>
</properties>


spring-boot-start-*:
一旦引入后，所有它需要的内容直接引入（如果有overlap, need to exclude）
所有类似starter的最底层依赖：spring-boot-starter
一旦定义好了版本号，之后不需要声明版本号

如果引入非spring管理的，需要自己定义version

spring-boot-autoconfigure:
负责所有的自动配置
所有场景的自动配置都在这里
不一定全部都生效

```

###

### SpringBoot2 @Configuration Full VS Lite(如无依赖，推荐使用)

```java
//引导程序
SpringApplication.run(MainApplication.class, args);

//添加组件
@Configuration //this is configure file

//Springboot 2 new update： full VS lite
// full: proxyBeanMethods = true: 如果外部调用这个里面的method，只会从container中寻找组件
// lite: proxyBeanMethods = false: 如果外部调用，可以重新创建，类似普通的code
//        lite模式spring不会检查bean是否在container中，因此运行会比较快
//        因此如果只是用来register, 没有其他组件依赖这个的话，尽量用lite
//configClass本身也是一个bean，在container里面
@Configuration(proxyBeanMethods = true) //是不是代理bean的方法
The default is true meaning each @Bean method will get proxied through CgLib. 
Each call to the method will pass through the proxy and assuming singleton scoped beans, 
it will return the same instance each time the method is called.

When setting it to false no such proxy method will be created and each call to the method will create a new instance of the bean. 
It will act just as a factory method. This is basically the same as the so called Bean Lite Mode, or @Bean methods on non-@Configuration annotated classes.

//properties file here
```

###

### 正常注册@Bean @Component @Controller @Service @Repository

###

### 添加组件@ComponentScan @Import @ImportResource(xml file) @ConditionalOnMissingBean

```java
//default scan the MainClass package
@ComponentScan(packages = "xx.xx.xx")  //put reference here

//import = 导入，array: 给容器中自动创建所需要的组件--> default name = class name
@Import({User.class, DBHelper.class})

@Conditional 
@ConditionalOnMissingBean //if not exist--> create for you

//批量导入组件，spring-->springboot / xml-->@
@ImportResource
@ImportResource("classpath:bean.xml")
```

###

### Properties file binding (配置绑定)

```java
//eg: db pool

//option 1:
//一定要首先加载到container中@component, 每个值可以自动绑定
@ConfigurationProperties(prefix = "xxx")
//必须同时加载容器中，需要有相应的class
@Component  

//option 2:
//一定要在配置类中写
@EnableConfigurationProperties(xx.class) //确定class文件， 同时也自动@Component到容器了
```



### @SpringBootApplication （如果用户配置了，用用户的，否则使用底层default的）

1. Springboot first init all autoconfiguration bean
2. all beans will be validate base on the properties/conditions
3. all validated bean will be add into the container
4. **用户配置优先**，但是底层也有default (**option1**)
5. 每个autoconfiguration都有properties, 有default value， 但是我们可以customized (**option2**)

> xxxAutoConfiguration --> Component --> xxxProperties getValue --> application.properties getValue

```java
默认扫描：主程序所在的包和default path， 就可以自动扫描所有的file了
scanBasePackages=???
@ComponentScan @EnableAutoConfiguration

//@SpringBootApplication相当于三个注解
@SpringBootConfiguration  ==> @Configuration
@ComponentScan("com.defualt.current")  ==> 有自定义的filter
@EnableAutoConfiguration

    @AutoConfigurationPackage //将某个程序所在包下的所有组件导入container
        @import(Resgiterar.class)  //批量注册
        
    @Import(AutoConfigurationImportSelector.class) //selectImport()
        autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
        给容器中批量导入组件
        Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {}
        
        从meta data开始加载：Factorie_Resource_Location = "META-INF/spring.factories";
        SpringbootAutoConfigure.jar: 核心包。里面有各种autoConfiguration
        配置文件写好了，spring一旦启动，自动加载127
        
        虽然所有场景默认加载，但是按需配置： @ConditionalOnClass(xx.class)
 
//具体分析WEB
@ConditionalOnWebApplication(type = Type.SERVLET)   //这里决定了是用MVC， 而非WebFlux响应式编程
@ConditionalOnClass(DispatcherServlet.class)        //默认必须有的class
@AutoConfigureAfter(ServletWebServerFactoryAutoConfiguration.class)
public class DispatcherServletAutoConfiguration {

    //这个有意思：文件上传解析器，
	  @Bean
		@ConditionalOnBean(MultipartResolver.class)
		@ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME)
		public MultipartResolver multipartResolver(MultipartResolver resolver) {
			// Detect if the user has created a MultipartResolver but named it incorrectly
			// 防止用户自己配置的不符合规范（名字错了）， 返回出去的名字是复合规范的
			// 给@Bean 方法传入参数，参数的值会从container里面找
			return resolver;
		}
}     
                      
//具体分析AOP
@Configuration(proxyBeanMethods = false)
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
public class AopAutoConfiguration {}
//注意有条件加载， 并且是lite import, default就配置了

如果启动spring报错，要用
@SpringBootApplication(exclude="xxAutoConfiguration.class")
```

![一旦有红色的，说明没有加载，说明这个class就不会使用](<../.gitbook/assets/image (146).png>)

### Springboot使用

1. 引入场景依赖（Spring-boot-start-xxx）
2. 具体内容\[**debug=true**], 开启自动配置报告，negative match都是不生效的
3. 确定配置文件是否需要修改
4.  额外的自定义器xxxCustomizer

### 开发技巧

1. **lombok**: 加上依赖，安装插件plugin，编译的时候才生成  （@ToString @AllArgsxxx @Data @**Slf4j**(注入log，可以直接使用log)）
2. **devtools**: hot deployment  VS JRebel付费的， real hot deploy
3. **spring initailizr**: IDE & [https://start.spring.io/](https://start.spring.io)

## SpringBoot 2.x.x (2021) 核心功能 ===========================

### 1 Properties File 

#### Yaml/Yml (String can use ' ' / " " , or not)

```yaml
//数据类型不同，格式不同
person:
  userName: zhangsan, 'zhangsan \n', "zhangsan \n" #单引号作为字符串输出，双引号作为换行输出
  boss: true
  birth: 1991/12/5
  age: 29
  interets:                      #list
    - basketball
    - scorrll
    - dance
  animal: [cat, dog, fish]       #list
  score: {english:80, math:90}   #map
  salarys:                       #list or set
    - 999.98
    - 1919.99
  pet:
    name: duoduo
    weight: 16
  allPets:
    sick:
      - {name: duoduo, weight: 17} #object
      - name: phoebe    #another object
        weight: 7
    health:
      - {name: chenchen, weight: 80}
      - {name: caicai, weight: 40}

```

#### spring-boot-configuration-processor

```markup
add type ahead :::
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-configuration-processor</artifactId>
		<optional>true</optional>
	</dependency>
	
	
need to exclude this process in maven when pack jar:::
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.springframework.boot</groupId>
							<artifactId>spring-boot-configuration-processor</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
		</plugins>
```

### 2 Web Development 

SpringBoot对SpringMVC做了很多自动配置，大多数情况可以@Autowired, @EnableWebMvc

1. ViewResolver
2. static resource path handling
3. converter, formatConverter...
4. HttpMessageConverter
5. MessageCodesResolver -->internationalized
6. index.html --> static welcome page
7. Favicon support
8. ConfigurableWebBindingInitializer

### 2.1 static resource access

![都可以存放static file： ResouceProperties.class中有详细定义](<../.gitbook/assets/image (148).png>)

```java
//static mapping: /**
"spring.mvc.static-ath-pattern = /resources/**"  //you can add prefix yourself


//如果使用如下path, 相当于customized path, 需要找res/**
spring:
  mvc:
    static-path-pattern: /res/**

//也可以更改default static path
spring:
  resources:
    static-locations: [classpath:/haha]  //必须从haha path下存放static file

//如果访问，一定会返回“aaa”:请求进来首先看controller, 如果找不到，则-->/**
@RestController
public class ResourceController {
    @RequestMapping("/banner.jpg")
    public String test(){
        return "aaa";
    }
}
```

{% hint style="info" %}
webJars: 将之前的静态资源编程jar，可以直接放入maven (jQuery, angular...)
{% endhint %}

### 2.2 Welcome page

* static path: index.html (不可以配置prefix)
* controller handle /index

#### favicon.ico: default static path即可

```java
WebMvcAutoConfiguration --> SpringMVC

@EnableConfigurationProperties: 说明配置类和文件有绑定

@Configuration(proxyBeanMethods = false) //only confirguration has this
@Import(EnableWebMvcConfiguration.class)
@EnableConfigurationProperties({ WebMvcProperties.class,  //==> SpringMVC
                                ResourceProperties.class, 
                                WebProperties.class })
@Order(0)
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {}

//如果一个类只有一个有参构造器，说明这个类参数的所有内容都要从container中确定

//核心处理规则在这里： 资源处理的default rules
protected void addResourceHandlers(ResourceHandlerRegistry registry) {
	super.addResourceHandlers(registry);
	if (!this.resourceProperties.isAddMappings()) {
		logger.debug("Default resource handling disabled");
		return;
	}
	ServletContext servletContext = getServletContext();
	addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
	addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
		registration.addResourceLocations(this.resourceProperties.getStaticLocations());
		if (servletContext != null) {
			registration.addResourceLocations(new ServletContextResource(servletContext, SERVLET_LOCATION));
		}
	});
}

//能够access static resource的所有规则，如果false,啥都木有(core to get all rules)
spring.resources.add-mappings=true 

//缓存过期时间
spring.resources.cache.period=10000 


handlerMapping: 处理器映射（SpringMVC core component）
欢迎页已经在上面的class中写死了
```

### 2.3 Request handling

### 2.3.1 RequestMapping, HandlerMapping

![HTTP请求method代表具体操作(http method is core, method = verb)](<../.gitbook/assets/image (149).png>)

#### form表单提交：只有get & post (delete和put也会default-->get)

![Only Form](<../.gitbook/assets/image (150).png>)

**HiddenHttpMethodFilter: 用来配置**

```java
开启form的RESTful

使用以下方式可以成功提交表单，"_method" + "hidden", 必须用post 

REST风格需要手动开启:::
使用场景：需要form提交的时候，页面交互的时候 --> only adapt to the form
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true

<form action="/user" method="post">
    <input name="_method" type="hidden" value="DELETE"/>  //大小写都可以
    <input value="REST-PUT submit" type="submit" />
</form>

//工作原理：：：
doFilterInternal()
 PUT/DELETE/PATCH
HttpMethodRequestWrapper.class
 @override
 getMethod(), return input value
 

//customize method name --> "_method"
<input name="_m" type="hidden" value="DELETE"/>  

@Bean
 public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
     HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
     hiddenHttpMethodFilter.setMethodParam("_m");
     return hiddenHttpMethodFilter;
 }
```

**DispatchServlet （httpServlet(doGet(), doPost()), Servlet）**: 处理所有请求的开始 

```java
Object
    HttpServlet
        FrameworkServlet (doGet() / doPost() --> processRequest())
            DispatchServlet


processRequest(){
try {
			doService(request, response);
		}
}

//dispatchServlet:::
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
//core method:::
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {

		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.--> find the corresponding handler
				mappedHandler = getHandler(processedRequest);


//handlerMapping: default have 5 
//new request: cycling for these 5 handlerMapping
RequestMappingHandlerMapping
WelcomePageHandlerMapping //所有index request-->based on this rules
BeanNameUrlHandlerMapping
RouterFunctionMapping
SimpleUrlHanlderMapping


//RequestMappingHandlerMapping： 只保存的controller里面的内容
@RequestMapping 所有handler的内容
里面保存了每个controller可以处理的请求，所有的mapping都在handlerMapping中

SpringBoot帮我配置了handlerMapping:
				RequestMapping
				WelcomePage
如果需要customize handlerMapping，可以自己定义（比如版本号不同的话，进入到不同的方法）

```

### 2.3.2 RequestParam 

1. annotation 
2. Servlet API (ServletRequest, HttpSession, HttpMethod, WebRequest, Principla, InputStream, Reader, Locale, TimeZone, ZoneId)  <--**ServletRequestMethodArgumentResolver(核心处理类)**
3. 复杂参数 (Model, ServletResponse, SessionStatus, Errors)
4. customized parameter

```java
//1===annotation===============================================================
@PathVariable 可以单独封装参数，也可以使用map
@RequestHeader 相当于携带request header里面的内容了
    //url= xxx/car/1  eg: http://localhost:8888/car/3/owner/lisi
    @GetMapping("/car/{id}/owner/{username}")
    public Map<String, Object> getCar(@PathVariable("id") Integer id,
                                      @PathVariable("username") String username,
                                      @PathVariable Map<String, String> pv,
                                      @RequestHeader("User-Agent") String userAgent,
                                      @RequestHeader Map<String, String> header){
        Map<String, Object> map =new HashMap<>();
        map.put("id", id);
        map.put("name", username);
        map.put("pv",pv);
        map.put("userAgent", userAgent);
        map.put("headers", header);
        return map;
    }

@RequestParam 封装Map的时候不需要带小括号
@CookieValue make sure have cookie first
    //http://localhost:8888/car?age=18&inters=cat&inters=duoduo
    @GetMapping("/car")
    public String getCar(@RequestParam("age")Integer age,
                         @RequestParam("inters") List<String> inters,
                         @RequestParam Map<String, String> params,
                         @CookieValue("_ga") String _ga,
                         @CookieValue("_ga") Cookie cookie){
        Map<String, Object> map = new HashMap<>();
        map.put("age", age);
        map.put("inters", inters);
        map.put("params", params);
        map.put("_ga", _ga);
        map.put("cookie", cookie);
        return map.toString();
    }

@RequestBody ONLY POST METHOD //POST only, body 是post里面的，不是url上的
    @PostMapping("/save")
    public String getCar(@RequestBody String content){
        Map<String, Object> map = new HashMap<>();
        map.put("content", content);
        return map.toString();
    }
    
@RequestAttribute 一般测试forward, 是带有attributes的
    //http://localhost:8888/goto
    @Controller
    public class RequestController {
    
        //must be redirect/forward request
        @GetMapping("/goto")
        public String goToPage(HttpServletRequest request){
            request.setAttribute("msg", "already successd");
            request.setAttribute("code", 200);
            return "forward:/success";
        }
    
        @ResponseBody
        @GetMapping("/success")
        public Map success(@RequestAttribute("msg") String msg,
                           @RequestAttribute("code") Integer code,
                           HttpServletRequest request){
            Map<String, Object> map = new HashMap<>();
            map.put("msg", request.getAttribute("msg"));
            map.put("code", code);
            return map;
        }
    }



@MatrixVariable 
在requestParam前面加上很多path， 分号分割的path称为矩阵变量
// /cars/{path; low=43;brand=audi, tesla,yd}
// 可以对比queryString: /car/{path}?xxx=aa&xx=bb
矩阵变量使用场景：cookie禁用，session内容如何使用？
before：session.set(a,b)-> jsessionid-->cookie---> request carry
now：url rewrite：/abc;jessionid=xxx 将cookie的值使用矩阵变量传递

首先手动开启：两种方法注入WebMvcConfigurer
    @Configuration(proxyBeanMethods = false)
    public class WebConfig implements WebMvcConfigurer {
    
        //第一种写法
        @Override
        public void configurePathMatch(PathMatchConfigurer configurer) {
            UrlPathHelper helper = new UrlPathHelper();
            //set to false, not remove--> enable @MatrixVariable
    
        }
        
        //第二种写法
        @Bean
        public WebMvcConfigurer webMvcConfigurer(){
            return new WebMvcConfigurer() {
                @Override
                public void configurePathMatch(PathMatchConfigurer configurer) {
                    UrlPathHelper helper = new UrlPathHelper();
                    helper.setRemoveSemicolonContent(false);
                    configurer.setUrlPathHelper(helper);
                }
            };
        }
    }
@MatrixVariable测试代码：
    //http://localhost:8888/cars/sell;low=455;brand=byd,tesla,dsds
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand) {
        Map<String, Object> map = new HashMap<>();

        map.put("low", low);
        map.put("brand", brand);
        return map;
    }


//2===servlet API===============================================================
ServletRequestMethodArgumentResolver    //以下内容为所支持的参数
return WebRequest.class.isAssignableFrom(paramType) || 
ServletRequest.class.isAssignableFrom(paramType) || 
MultipartRequest.class.isAssignableFrom(paramType) || 
HttpSession.class.isAssignableFrom(paramType) || 
pushBuilder != null && pushBuilder.isAssignableFrom(paramType) || 
Principal.class.isAssignableFrom(paramType) && !parameter.hasParameterAnnotations() || 
InputStream.class.isAssignableFrom(paramType) || 
Reader.class.isAssignableFrom(paramType) || 
HttpMethod.class == paramType || 
Locale.class == paramType || 
TimeZone.class == paramType || 
ZoneId.class == paramType;


//3===复杂参数===============================================================
Map, Model = 相当于在Request里面放入Attribute ==> request,setAttribute(x,x);
【RedirectAttributes】
【ServletResponse】
//Map类型的参数，返回值， Map和Model的处理是一样的
mavContainer.getModel() --> BindingAwareModelMap (Model & Map)

    @GetMapping("/params")
    public String testParam(Map<String, Object> map,   //相当于request里面添加attribute
                            Model model,               //相当于request里面添加attribute
                            HttpServletRequest request,
                            HttpServletResponse response){
        map.put("hello","world666");
        model.addAttribute("world", "hello666");
        request.setAttribute("message", "hwlloworld");
        Cookie cookie = new Cookie("c1", "v1");
        cookie.setDomain("localhost");
        //response里面添加cookie,浏览器会帮忙保存cookie
        response.addCookie(cookie);
        return "forward:/success";
    }

    @ResponseBody
    @GetMapping("/success")
    public Map success(@RequestAttribute(value="msg", required = false) String msg,
                       @RequestAttribute(value = "code", required = false) Integer code,
                       HttpServletRequest request){

        Object msg1 = request.getAttribute("message");
        Map<String, Object> map= new HashMap<>();
        map.put("reqMethod_msg", msg1);
        System.out.println("you entered success page");
        return map;
    }
    

//4===self defined parameter===============================================================
ServletModeAttributeMethodProcessor
WebDataBinder //将自定义的内容化放进这个class里面，可以customized放入自己的内容
    xxxFormater/Converters //格式化器将数据类型转换, 使用反射机制
    GenericConversionService //具体找哪个数据类型可以转换 JavaBean-->Integer
    //拿到所有自定义类的属性值，然后一个一个map到相应property上面
    BindingResult //还有validation功能在此时间

    @PostMapping("/saveuser")
    public Person saveUser(Person person){
        return person;
    }
    
    //自定义converter组件， 在WebConfig里面添加自定义Configurer
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
  
            @Override
            public void addFormatters(FormatterRegistry registry) {
                registry.addConverter(new Converter<String, Person>() {  //add your class
                    @Override
                    public Person convert(String s) { //return your class
                        if(!StringUtils.isEmpty(s)){
                            //business logic
                        }
                        return null;
                    }
                });
            }
    };

```

![](<../.gitbook/assets/image (151).png>)

#### Springboot default disable matrix --> UrlPathHelper-->removeSemicolonContent

![](<../.gitbook/assets/image (152).png>)

**SpringMVC自定义组件:**

* @EnableWebMvc : 全面接管
* @Configuration + WebMvcConfigurer: 自定义规则 (两种方法，_**一个是implement: 因为jdk8有interface default method, no need to implement all methods,**_ 另一个是@Bean)
* @WebMvcRegistrations: 改变默认底层组件

`一般Springboot V2的configuration (proxyBeanMethods= false), default if true`

**`UrlPathHelper`**

### 2.3.3 How Request Been Resolved

#### HandlerMethodArgumentResolver : 处理上述所有类request(参数解析原理)

```java
doDispatch()   [dispatchServlet]
handler        [handlerMapping]
HandlerAdapter  大的反射工具，适配器 [interface]
    RequestMappingHandlerAdapter  ==>@RequestMapping , return ModelAndView
    HandlerFunctionAdapter
    HttpRequestHandlerAdapter
    SimpleControllerHandlerAdapter

invokeHandlerMethod(request, response, handlerMethod); //执行目标方法 【RequestMappingHandlerAdapter】
argumentResolvers 【*26】 参数解析器：确定将要执行的目标方法的参数 
    supourtsParameter() ->boolean
    resolveArgument() -> returnValuesHandler 【返回值处理器 * 14】

invokeForRequest();         【ServletInvocableHandlerMethod】
getMethodArgumentValues();  【ServletInvocableHandlerMethod】获取方法参数值

getArgumentResolver();      【HandlerMethodArgumentResolverComposite】Request进来后，挨个判断，二十六个哪个支持这个参数
//先遍历for看哪个resolver可用，再具体解析
```

![Request: 参数解析器，与参数类型有关系](<../.gitbook/assets/image (153).png>)

![Response: 返回值处理器](<../.gitbook/assets/image (154).png>)

{% hint style="info" %}
"HEAD" request: 不是真正处理的，如果有缓存，是大头兵用来cache的 (option request?!)
{% endhint %}

{% hint style="info" %}
SpringBoot第一次Run的时候比较慢，但是内部有缓存机制(cache in memory)，之后会变快 
{% endhint %}

对于Redirect的Request， request里面携带的Attribute (如果是Map / Model) --> HttpServletRequest

目标方法执行完后，所有内容放在**ModelAndViewContainer**, 包括要去的页面地址，还有操作的各种数据

#### 处理派发结果：

1. processDispatchResult(); 
2. renderMergedOutputModel(); 如何处理得到的内容进行渲染
3. exposeModelAsRequestAttributes(); 暴露请求作为关键方法，model中的所有数据遍历并且放在请求域中

#### 自定义类的request: url = xxx.xx?a=112\&b=19\&c=string

* WebDataBinder = binderFactory.createBinder(webRequest, attribute, name);
* web数据绑定器，将请求参数（String）转化成指定的类型，再放到JavaBean中
* 利用Converters将请求数据转换成指定类型，再次封装到JavaBean中

{% hint style="info" %}
interface里面的Default method，只需要实现需要的，可以有多个default method，不需要实现全部method
{% endhint %}

![](<../.gitbook/assets/image (155).png>)



### 2.4 Response and Content

![](<../.gitbook/assets/image (156).png>)

本节主要讨论直接响应出去(data)-->@ResponseBody-->xxxProcessor-->xxxConverter，可以自定义相应内容的format.

* 相应内容type可以放在request: accept里面
*   可以开启 **contentnegotiation-favor-paramter=true,** 使用**url = ?format=xml/json**进行协商



### 2.4.1 Response and Content -- JSON

 首先要添加**web-starter**依赖, web场景啊自动引入了**json/jackson/jackson-datatype-jdk8**

#### (1) Jackson.jar + @ResponseBody

@ResponseBody可以自动返回JSON数据

 **原理：**returnValueHandler \* 15 【返回值解析器】 （首先确定是否支持这种type: supportReturnType()）

```java
1. ModelAndView
2. Model
3. View
4. ResponseEntity
5. ResponseBodyEmitter
6. HttpEnity
7. HttpHeader
8. Callable
9. DeferredResult
10. ListenableFuture
11. CompletionStage
12. WebAsyncTask
13. @ModelAttribute
14. @ResponseBody --> RequestResponseBodyMethodProcessor

//找到processor之后，使用messageConverter转换成JSON数据类型

```

![浏览器default使用requestHeader的方式告知可以接受的数据类型](<../.gitbook/assets/image (170).png>)

#### (2) HTTPMessageConverter原理 (MappingJackson2MessageConverter)

 是否可以将Bean class转为JSON数据？！ MessageConverter是**可逆**的。 (can read / write)

![default messageConventer on the system](<../.gitbook/assets/image (171).png>)

![](<../.gitbook/assets/image (172).png>)

### 2.4.2 Response and Content -- 内容协商

> 通过遍历所有messageConverter最终找到数据类型一致的，并且进行convert

#### request里面可以定义response的type:` application/json , application/xml, */*`

"accept" = " xxx"  (http协议)   （自己的project需要引入xml的包jackson-dataformat-xml）

HttpMessageConverter: canRead?-->read() / canWrite?-->write();

#### The Browser Request header is fixed: 需要修改setFavorParameter --> true

```yaml
#允许URL中定义format, 因为Browser中无法确定accept内容， default.
Spring:
  mvc:
    contentnegotiation:
      favor-parameter: true

http://localhost:8888/test/person?format=json
http://localhost:8888/test/person?format=xml
```

![](<../.gitbook/assets/image (175).png>)

### 2.4.3 自定义MessageConverter

![](<../.gitbook/assets/image (176).png>)

需要手动添加一个MessageConverter

```java
WebMvcConfigurer
    default void configureMessageConverters(){}
    default void extendMessageConverters(){}  //使用这个扩展自定义的

//add new converter in webmvconfigurer
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {
        @Override
        public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
            converters.add(new CustomMessageConverter());
        }
}}

//need to write your own message converter based on the httpMessageConverter
//only focus on write, so default settings for read
public class CustomMessageConverter implements HttpMessageConverter {
    @Override
    public boolean canRead(Class aClass, MediaType mediaType) {
        return false;  //not support read
    }

    @Override
    public boolean canWrite(Class aClass, MediaType mediaType) {
        return aClass.isAssignableFrom(Person.class);
    }

    //core method here: server need to know what can read/write
    // application/x-custom
    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return MediaType.parseMediaTypes("application/x-custom");
    }

    @Override
    public void write(Object p, MediaType mediaType, HttpOutputMessage httpOutputMessage) throws IOException, HttpMessageNotWritableException {
        String data = ((Person)p).getName() + ";" + ((Person)p).getAge();
        OutputStream body = httpOutputMessage.getBody();
        body.write(data.getBytes());
    }

    @Override
    public Object read(Class aClass, HttpInputMessage httpInputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }
}
```

#### 想要使用Url内容协商，需要更改mediaType (default only json+xml(need add xml jar))

```java
ContentNegotiationManager //defined format = xml / json in URL can be used
    默认添加header (accept=???)， 使用配置后支持URL=xml/json

//自定义内容协商策略
//http://localhost:8888/test/person?format=gg
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {
        @Override
        public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
            //Map<String, MediaType> mediaTypes
            HashMap<String, MediaType> hashMap = new HashMap<>();
            hashMap.put("json", MediaType.APPLICATION_JSON);
            hashMap.put("xml", MediaType.APPLICATION_XML);
            hashMap.put("gg",MediaType.parseMediaType("application/x-custom"));  //自定义参数
            ParameterContentNegotiationStrategy strategy = new ParameterContentNegotiationStrategy(hashMap);
            configurer.strategies(Arrays.asList(strategy));
        }
    }
}
```

### 2.5 View resolver & Template engine 

![Springboot不支持JSP](<../.gitbook/assets/image (177).png>)

* freemarker
* groovy-template
* thymeleaf (推荐)   --> server side java template engine  (并发不好)

![](<../.gitbook/assets/image (178).png>)

#### thymeleaf自动配置好了：

* ThymeleafProperties
* SpringTemplateEnegine
* ThymeleafViewResolve

```java
//JAVA CODE : model里面set value
    @GetMapping("/atguigu")
    public String atguigu(Model model){
        model.addAttribute("msg", "hello world at guigu");
        model.addAttribute("link", "http://facebook.com");
        return "success";
    }

//Thymeleaf
<h1 th:text=${msg}>hhahaha</h1>
<h2>
    <a href="google.com" th:href=${link}>go to google</a>   //go to link 
    <a href="google.com" th:href=@{link}>go to google 2</a> //认为是个地址，仅仅是String + xxx
</h2>


//更改访问路径
//http://localhost:8888/world/atguigu
server:
  servlet:
    context-path: /world  //以后所有请求都要以/world开始
    
```

#### In order to prevent refresh page, must be using GET, so let other POST to redirect to GET

防止表单重复提交：**REDIRECT**

```java
//use HttpSession(save login status) & Model(save info for thymeleaf use)
@PostMapping("/login")
public String main(User user, HttpSession session, Model model){
    if(StringUtils.hasText(user.getUsername()) && "admin".equals(user.getPassword())){
        //success
        session.setAttribute("loginUser", user);
        //login success redirect to main --> avoid multiple times access
        return "redirect:/main.html";
    }else{
        model.addAttribute("msg", "wrong password or username" );
        return "login";  //return to login page
    }
}

@GetMapping("/main.html")
public String mainPage(HttpSession session, Model model){
    //check if you login or not--> use session: filter / intercepter
    Object loginUser = session.getAttribute("loginUser");
    if(loginUser!=null)
        return "main";
    else{
        model.addAttribute("msg", "please login again");
        return "login";
    }
}
```

#### JS/HTML的common content可以抽取

```java
quote from other file
    <div th:include="common :: commonheader"></div>
    <div th:replace="common :: #commonscript"></div>

//任何重复性内容都可以被替换
```

### 2.5.1 解析原理

```java
ModelAndViewContainer : all data go into ModelAndViewContainer 

processDispatchResult() // core method to process result and distribute
 render(mv, request, response);
   locale
   resolve--> model
   getView object based on request
      View: define how to rend the front end
        ViewResolver: resolve view object
        get Redirect object: RedirectView
        ContentNegotiationViewResolver
        view.render() --> 
          get url first
          response.sendRedirect(url)  ==> redirect view
      
  
  if return value starts "forward":
    new InternalResourceView(forwardUrl);
      request.getRequestDispatcher(path).forward(request, response);
  if return value starts "redirect":
    new RedirectView();
  else
    return regular string 
      new ThymeleafView(); 
      
  //so you can customized the view resolver

```

![](<../.gitbook/assets/image (186).png>)



### 2.6 Interceptor

```java
//原生HandlerInterceptor interface
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
    }
}
```

**自己实现步骤：**

1. 定制自己的intercepter, implement HandlerInterceptor
2. 定制自己的WebMvcConfig
3. 将自己的intercepter放入WebMvcConfig

```java
//自定义interceptor内容
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //login check business logic
        HttpSession session = request.getSession();
        Object loginUser = session.getAttribute("loginUser");
        if(loginUser!=null){
            return true; //pass 放行
        }
        //拦截住
        //拦截住。未登录。跳转到登录页
        request.setAttribute("msg","请先登录");
        //        re.sendRedirect("/");
        request.getRequestDispatcher("/").forward(request,response);
        return false;
    }

//自定义WebConfig内容，需要加入自定义的interceptor
@Configuration(proxyBeanMethods = false)
public class AdminConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")  //拦截所有request, including static resource
                .excludePathPatterns("/","/login","/css/**","/js/**","/font/**","/images/**"); //后面static内容手动添加
    }
}
```



### 2.6.1 Interceptor原理

> preHandle正向进行，postHandle倒叙进行，页面渲染完成后，afterCompletion倒叙触发
>
> 任何一步有异常，直接触发当前handler的afterCompletion

1、根据当前请求，找到**HandlerExecutionChain【**可以处理请求的handler以及handler的所有 拦截器】

2、先来**顺序执行** 所有拦截器的 preHandle方法

* 1、如果当前拦截器prehandler返回为true。则执行下一个拦截器的preHandle
* 2、如果当前拦截器返回为false。直接    倒序执行所有已经执行了的拦截器的  afterCompletion；

**3、如果任何一个拦截器返回false。直接跳出不执行目标方法**

**4、所有拦截器都返回True。执行目标方法**

**5、倒序执行所有拦截器的postHandle方法。**

**6、前面的步骤有任何异常都会直接倒序触发** afterCompletion

7、页面成功渲染完成以后，也会倒序触发 afterCompletion

![](<../.gitbook/assets/image (187).png>)



### 2.7 File Upload

```markup
<form role="form" th:action="@{/upload}" method="post" enctype="multipart/form-data">
表单上传的固定写法： 必须使用POST

跟文件上传相关的settings: MultipartServlet
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=20MB
```

```java
@PostMapping("/upload")
public String upload(@RequestParam("email") String email,
                     @RequestParam("username") String username,
                     @RequestPart("headerImg") MultipartFile headerImg, //注意multipart
                     @RequestPart("photos") MultipartFile[] photos){ //@RequestPart()

    log.info(email, username, headerImg.getSize(), photos.length);

    if(!headerImg.isEmpty()){
        String filename = headerImg.getOriginalFilename();
        //headerImg.transferTo(new File("H"));
    }
    if(photos.length>0){
        for(MultipartFile photo : photos){
            if(!photo.isEmpty()){
                String filename = headerImg.getOriginalFilename();
                //headerImg.transferTo(new File("H"));
            }
        }
    }
    return "main";

```

![](<../.gitbook/assets/image (189).png>)



### 2.8 Exception/Error Handling

![](<../.gitbook/assets/image (190).png>)

**Browser**: whitepage / **Postman**: JSON with error message

Request with carry a cookies: JSEESIONID=XXXX after login

![](<../.gitbook/assets/image (191).png>)

![put error page under: templates/error/xxx.html](<../.gitbook/assets/image (192).png>)

![you can use message/error/path here to display in the customized page](<../.gitbook/assets/image (193).png>)

![springboot autoconfiguration jar--> error handling](<../.gitbook/assets/image (194).png>)

```java
ErrorMvcAutoConfiguration
    DefaultErrorAttributes --> id: errorAttributes
        implements ErrorAttributes, HandlerExceptionResolver
        all error attributes defined here:::
            if (!options.isIncluded(Include.EXCEPTION)) {
                errorAttributes.remove("exception");
            }

            if (!options.isIncluded(Include.STACK_TRACE)) {
                errorAttributes.remove("trace");
            }
    
            if (!options.isIncluded(Include.MESSAGE) && errorAttributes.get("message") != null) {
                errorAttributes.put("message", "");
            }
    
            if (!options.isIncluded(Include.BINDING_ERRORS)) {
                errorAttributes.remove("errors");
            }
    BasicErrorController --> id: basicErrorController
        process /error requests
        ModelAndView & Return String
        View --> id: error (render default error page, whitePage)
            BeanNameViewResolver --> find the corresponding View page
    DefaultErrorViewResolver:::
        static {
        		Map<Series, String> views = new EnumMap<>(Series.class);
        		views.put(Series.CLIENT_ERROR, "4xx");
        		views.put(Series.SERVER_ERROR, "5xx");
        		SERIES_VIEWS = Collections.unmodifiableMap(views);
        }
        error/4xx||5xx.html      
```

1、执行目标方法，目标方法运行期间有**任何异常都会被catch**、而且标志当前请求结束；并且用 **dispatchException**

2、进入视图解析流程（页面渲染？）

processDispatchResult(processedRequest, response, mappedHandler, **mv**, **dispatchException**);

3、**mv** = **processHandlerException**；处理handler发生的异常，处理完成return ModelAndView；**遍历**所有的 **handlerExceptionResolvers，看谁能处理当前异常【HandlerExceptionResolver处理器异常解析器--> have DefaultErrorAttributes】**

{% hint style="info" %}
find 404.html first, if not exist, find 4xx.html
{% endhint %}

#### ****

#### **定制错误处理逻辑**

* 自定义错误页
*
  * error/404.html   error/5xx.html；有精确的错误状态码页面就匹配精确，没有就找 4xx.html；如果都没有就触发白页
* @**ControllerAdvice**+@**ExceptionHandler**处理全局异常；底层是 **ExceptionHandlerExceptionResolver 支持的**
* @**ResponseStatus**+自定义异常 ；底层是 **ResponseStatusExceptionResolver ，把responsestatus注解的信息底层调用** **response.sendError(statusCode, resolvedReason)；tomcat发送的/error**
* Spring底层的异常，如 参数类型转换异常；**DefaultHandlerExceptionResolver 处理框架底层的异常。**
*
  * response.sendError(HttpServletResponse.**SC_BAD_REQUEST**, ex.getMessage());
  * ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1354552/1606114118010-f4aaf5ee-2747-4402-bc82-08321b2490ed.png)
* 自定义实现 HandlerExceptionResolver 处理异常；可以作为默认的全局异常处理规则
*
  * ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1354552/1606114688649-e6502134-88b3-48db-a463-04c23eddedc7.png)upper two is for customized annotation, last one is for lower level default handling
* **ErrorViewResolver**  实现自定义处理异常；
*
  * response.sendError 。error请求就会转给controller
  * 你的异常没有任何人能处理。tomcat底层 response.sendError。error请求就会转给controller
  * **basicErrorController 要去的页面地址是** **ErrorViewResolver**  ；

```java
//self defined ExceptionHandler

@ControllerAdvice //for error handling
public class GlobalExceptionHandler {
    @ExceptionHandler({ArithmeticException.class, NullPointerException.class})
    public String handlerArithException(){
        return "login"; //modelAndView or path of model
    }
}


@ResponseStatus(value = HttpStatus.FORBIDDEN, reason = "too many user accounts")
public class UserTooManyException extends RuntimeException{
    public UserTooManyException(String message){
        super(message);
    }
}

//self defined resolver (resolve can: return page & return mv)
@Order(value= Ordered.HIGHEST_PRECEDENCE)  //lower score, higher priority
@Component
public class CustomerHandlerExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request,
                                         HttpServletResponse response,
                                         Object handler,
                                         Exception ex) {
        //can add self defined information, or just return mv
        return new ModelAndView();
    }
}

```



### 2.9 Web Component Injection (Servlet, Filter, Listener)

### 2.9.1 Use servlet API

* @ServeletComponentScan --> must have in the main class
* @WebServlet
* @WebFilter
* @WebListener

```java
1. need to use @WebServlet
2. need to use @ServletComponentScan in the main class

@WebServlet(urlPatterns = "/myservlet")  //not passing interceptor
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("66666");
    }
}


@WebFilter(urlPatterns = {"/css/*", "/images/*"})
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        //first execute
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
        //last execute
    }
}


@WebListener
public class MyServeletListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {

    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {

    }
}
```

### 2.9.2 Use Registration Bean (Better)

* **ServletRegistrationBean**
* **FilterRegistrationBean**
* **ServletListenerRegistrationBean**

```java
//default proxyBeanMethods = true --> SingletonBean --> result will be reused
@Configuration(proxyBeanMethods = true)  
public class MyRegistConfig {

    @Bean
    public ServletRegistrationBean myServlet(){
        MyServlet myServlet = new MyServlet();
        return new ServletRegistrationBean(myServlet, "/my", "my02/");
    }

    @Bean
    public FilterRegistrationBean myFilter(){
        MyFilter myFilter = new MyFilter();
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(myFilter);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/my","/css/*"));
        //return new FilterRegistrationBean(myFilter, myServlet()); //only filter myservlet content
        return filterRegistrationBean;
    }

    @Bean
    public ServletListenerRegistrationBean myListener(){
        MyServeletListener listener = new MyServeletListener();
        return new ServletListenerRegistrationBean(listener);
    }
    
}
```

![](<../.gitbook/assets/image (195).png>)

### 2.10 Embedded Web Container

No need to handle Tomcat or container myself

```markup
ServletWebServerApplicationContext  (ApplicationContext)
    TomcatServletWebServerFactory (default)
    UnderTow...
    Jetty...

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-undertow</artifactId>
</dependency>

WebServerFactoryCustomizer:
 	xxxCustomizer
```

![](<../.gitbook/assets/image (196).png>)

### 2.11 Customized Principles

#### 1、定制化的常见方式

* 修改配置文件；
* **xxxxxCustomizer；**
* **编写自定义的配置类   xxxConfiguration；+** **@Bean替换、增加容器中默认组件；视图解析器**
* **Web应用 编写一个配置类实现 WebMvcConfigurer 即可定制化web功能；+ @Bean给容器中再扩展一些组件**

```
@Configuration
public class AdminWebConfig implements WebMvcConfigurer
```

* @EnableWebMvc + WebMvcConfigurer —— @Bean  可以全面接管SpringMVC，所有规则全部自己重新配置； 实现定制和扩展功能
*
  * 原理
  * 1、WebMvcAutoConfiguration  默认的SpringMVC的自动配置功能类。静态资源、欢迎页.....
  * 2、一旦使用 @EnableWebMvc 、。会 @Import(DelegatingWebMvcConfiguration.**class**)
  * 3、**DelegatingWebMvcConfiguration** 的 作用，只保证SpringMVC最基本的使用
*
  *
    * 把所有系统中的 WebMvcConfigurer 拿过来。所有功能的定制都是这些 WebMvcConfigurer  合起来一起生效
    * 自动配置了一些非常底层的组件。**RequestMappingHandlerMapping**、这些组件依赖的组件都是从容器中获取
    * **public class** DelegatingWebMvcConfiguration **extends** **WebMvcConfigurationSupport**
*
  * 4、**WebMvcAutoConfiguration** 里面的配置要能生效 必须  @ConditionalOnMissingBean(**WebMvcConfigurationSupport**.**class**)
  * 5、@**EnableWebMvc  **导致了 **WebMvcAutoConfiguration  没有生效。**
* ... ...

#### 2、原理分析套路

**场景starter** **- xxxxAutoConfiguration - 导入xxx组件 - 绑定xxxProperties --** **绑定配置文件项**

****

### 3 DAO 

> spring-boot-starter-data-xxx

### 3.1 SQL-Datasource

#### Springboot-starter-data-jdbc: HikariDataSource(connection pool)

![ not including the driver (mysql / oracle / mariadb) --> must self import](<../.gitbook/assets/image (197).png>)

```markup
if version is not match --> use version overlap: 
<mariadb.version>3.0.0</mariadb.version>

DataSourceAutoConfiguration
DataSourceTransactionManagerAutoConfiguration
JdbcTemplateAutoConfiguration
JndiDataSourceAutoConfiguraation
XADataSourceAutoConfiguration (distributed)

DataSourceProperties:::
spring.datasource.xxx

spring:
  datasource:
    url: jdbc:mysql://localhost:3308/facs2db
    username: root
    password: dbpiovan
    driver-class-name: org.mariadb.jdbc.Driver
  jdbc:
    template:
      query-timeout: 3
```

```java
@Autowired
JdbcTemplate template;

@Test
void contextLoads() {

	List<Map<String, Object>> maps = template.queryForList("select * from tblcomponents");
	maps.forEach((m)->{
		System.out.println(m.toString());
	});
}
```

### 3.2 SQL-Druid Datasource

#### HikariDataSource VS Druid (Alibaba)

Druid datasource已经绑定spring.datasourece， 所有可以set的，都可以放在properties file

```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.17</version>
</dependency>

//default: ConditionOnMissingBean  --> if using starter, then  @Deprecated
@ConfigurationProperties("spring.datasource")
@Bean
public DataSource myDataSource(){
    DruidDataSource dataSource = new DruidDataSource();
//        dataSource.setUrl();  --> not fixed, so use @ConfigurationProperties
//        dataSource.setUsername();
//        dataSource.setPassword();
    return dataSource;
}

//druid 监控page
@Bean
public ServletRegistrationBean statViewServlet(){
    StatViewServlet servlet = new StatViewServlet();
    ServletRegistrationBean<StatViewServlet> registrationBean = new ServletRegistrationBean<>(servlet, "/druid/*");
    return registrationBean;
}
```

```yaml
//just use spring-starter-druid
druid:
  aop-patterns: com.atguigu.admin.*  #springbean监控
  filters: stat,wall,slf4j  #所有开启的功能

  stat-view-servlet:  #监控页配置
    enabled: true
    login-username: admin
    login-password: admin
    resetEnable: false

//stat filter settings
  web-stat-filter:  #web监控
    enabled: true
    urlPattern: /*
    exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

  filter:
    stat: #sql监控
      slow-sql-millis: 1000
      logSlowSql: true
      enabled: true
    wall: #防火墙
      enabled: true
      config:
        drop-table-allow: false

```

### 3.3 SQL-Mybatis

Third party nameing: xxx-spring-boot-starter

![choose the right version](<../.gitbook/assets/image (198).png>)

* Global Setting file
* **SqlSessionFactory**: @Bean
* **SqlSession**
* **Mapper**

> global setting xml file + mapper xml file (need both)  --> separate SQL with JAVA code

```java
MyBatisProperties.class
mybatis.xx --> all the settings

SqlSessionTemplate -> SqlTemplate

@AutoConfiguriedMapperScannerRegistrar

@Mapper--> auto scan and add into the container

```

```yaml
# 配置mybatis规则、使用MyBatisPlus则此项配置无效
mybatis:
  #config-location: classpath:mybatis/mybatis-config.xml #global setting file
  mapper-locations: classpath:mybatis/mapper/*.xml      #mapping file
  configuration:  # 指定mybatis全局配置文件中的相关配置项, everything in this configuration part
    map-underscore-to-camel-case: true  #normally need to set ture, so can match the name with table  name
```

```markup
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.admin.mapper.AccountMapper">

<!--    public Account getAcct(Long id); -->
    <select id="getAcct" resultType="com.atguigu.admin.bean.Account">
        select * from  account_tbl where  id=#{id}
    </select>
<!--    -->
</mapper>

@Mapper  //must have this mapper
public interface AccountMapper {
    public Account getAcct(Long id);
}
```

![](<../.gitbook/assets/image (199).png>)

```java
@MapperScan("xx.xx.xx.mapper") --> replace @Mapper
//only use annoation: better:
//@Mapper
public interface CityMapper {

    @Select("select * from city where id=#{id}")
    public City getById(Long id);


    @Insert("insert into  city(`name`,`state`,`country`) values(#{name},#{state},#{country})")
    @Options(useGeneratedKeys = true,keyProperty = "id")
    public void insert(City city);

}


//similar to the following file, for the complicated SQL, or join ...
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.admin.mapper.CityMapper">

<!--    public void insert(City city);-->
<!--    <insert id="insert" useGeneratedKeys="true" keyProperty="id">-->
<!--        insert into  city(`name`,`state`,`country`) values(#{name},#{state},#{country})-->
<!--    </insert>-->

</mapper>

//learn to use mixed
```

### 3.4 SQL-Mybatis-Plus

**mybatis plus includings mybatis already**

> mybatis-plus-boot-starter

![](<../.gitbook/assets/image (200).png>)

```java
public interface UserMapper extends BaseMapper<User>{
    //mybatis plus already handled here
}

@TableField(exist=false) //not map to the db, ignore them
private String userName;
```

#### Service layer can extends from IService (Mybatis-plus) directly::: (IService<> + ServiceImpl\<x,y>)

```java
public interface UserService extends IService<User>{

}

@Service
public class UserServiceImpl extends ServiceImplm<UserMapper, User> implements UserService{
    //can use lots of mapper stuff
}
```

### 3.5 NoSQL(Redis=6379)

RedisAutoConfiguration--> RedisProperties

**LettuceConnectionConfiguration **(Netty in the lower level) &&& **JedisConnectionConfiguration**

```java
spring.redis.xx

//generic
RedisTemplate<ObjectKey, ObjectValue>
//most used
StringRedisTemplate: default k/v=string

```

Use cloud Redis, charge in hour rate

0.0.0.0/0  ==> everyone can access

```yaml
redis:
  host: r-bp1nc7reqesxisgxpipd.redis.rds.aliyuncs.com  //at least need the url 
  port: 6379
  password: lfy:Lfy123456
  client-type: jedis
  jedis:
    pool:
      max-active: 10
#    url: redis://lfy:Lfy123456@r-bp1nc7reqesxisgxpipd.redis.rds.aliyuncs.com:6379
#    lettuce:
#      pool:
#        max-active: 10
#        min-idle: 5
```

**Filter**: servlet defined: can use without spring

**Interceptor**: spring defined, only spring can use, also can use @Interceptor

### 4 Unit Test 

after 2.2.0: **JUnit5**（Junit **platform**(cross platform) + Junit **Jupiter**(Core) + Junit **Vintage**(cross older version)）

Version2.4之后又不同了，需要手动添加Vintage(before2.4)

```java
spring-boot-starter-test 

Jupiter5.7.0 (Core)

@Test(must from JUnit5)
//所有内容自动具有spring的属性

@Test :表示方法是测试方法。但是与JUnit4的@Test不同，
       他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
@ParameterizedTest :表示方法是参数化测试，下方会有详细介绍

@RepeatedTest :表示方法可重复执行，下方会有详细介绍

@DisplayName :为测试类或者测试方法设置展示名称

@BeforeEach :表示在每个单元测试之前执行
@AfterEach :表示在每个单元测试之后执行
@BeforeAll :表示在所有单元测试之前执行
@AfterAll :表示在所有单元测试之后执行

@Tag :表示单元测试类别，类似于JUnit4中的@Categories

@Disabled :表示测试类或测试方法不执行，类似于JUnit4中的@Ignore
@Timeout :表示测试方法运行如果超过了指定时间将会返回错误
@ExtendWith :为测试类或测试方法提供扩展类引用（same with @RunWith）


```

### 4.1 Assertion

* 简单断言
* 数组断言
* 组合断言
* 异常断言
* 超时断言
* 快速失败

断言（assertions）是测试方法中的核心部分，用来对测试需要满足的条件进行验证。**这些断言方法都是 org.junit.jupiter.api.Assertions 的静态方法**。JUnit 5 内置的断言可以分成如下几个类别：

**检查业务逻辑返回的数据是否合理。**

**所有的测试运行结束以后，会有一个详细的测试报告；**

用来对单个值进行简单的验证。如：

| 方法              | 说明                 |
| --------------- | ------------------ |
| assertEquals    | 判断两个对象或两个原始类型是否相等  |
| assertNotEquals | 判断两个对象或两个原始类型是否不相等 |
| assertSame      | 判断两个对象引用是否指向同一个对象  |
| assertNotSame   | 判断两个对象引用是否指向不同的对象  |
| assertTrue      | 判断给定的布尔值是否为 true   |
| assertFalse     | 判断给定的布尔值是否为 false  |
| assertNull      | 判断给定的对象引用是否为 null  |
| assertNotNull   | 判断给定的对象引用是否不为 null |

 **assertAll**: must all pass, or false

```java
Assertions: assertAll / assertNull / fail / assertEquals

Assumptions: assumptTure / False / That --> only terminate but not throw error/exceptions
```

must pass all unit test before release

```java
Nested Test:::
嵌套测试情况下，外层的Test不能驱动内层的Before(After)Each/All之类的方法提前/之后运行
内层的Test可以驱动外层的Before(After)Each/All之类的方法提前/之后运行

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }
    }
```

* @ValueSource
* @NullSource
* @EnumSource
* @CvsFileSource
* @MethodSource

![](<../.gitbook/assets/image (201).png>)

```java
add different value/source to test the case
@ParameterizedTest
 
@ParameterizedTest
@DisplayName("参数化测试")
@ValueSource(ints = {1,2,3,4,5})
void testParameterized(int i){
    System.out.println(i);
}

JUnit4 --> JUnit5 : migration tips
```

### 5 Monitoring 

### 5.1 SpringBoot Actuator

![](<../.gitbook/assets/image (202).png>)

JAX-RS: Jakarta RESTful Web Services 

JMX(jconsole) + HTTP

```yaml
# management 是所有actuator的配置
# management.endpoint.端点名.xxxx  对某个端点的具体配置
management:
  endpoints:
    enabled-by-default: true  #默认开启所有监控端点  true
    web:
      exposure:
        include: '*' # 以web方式暴露所有端点, expose everything, json return value

  endpoint:   #对某个端点的具体配置
    health:
      show-details: always
      enabled: true

    info:
      enabled: true

    beans:
      enabled: true

    metrics:
      enabled: true
```

![test paths](<../.gitbook/assets/image (203).png>)

```java
Endpoint:
    Health: monitoring
    Metrics: runtime data
    Loggers: log data
```

```java
implements AbstractHealthIndicator
    @override
    doHealthCheck()//must implement yourself


info:
  appName: boot-admin
  appVersion: 1.0.0
  mavenProjectName: @project.artifactId@
  mavenProjectVersion: @project.version@
```

![](<../.gitbook/assets/image (204).png>)

### 5.2 Spring Boot Admin Server

![](<../.gitbook/assets/image (205).png>)

need additional server project

```java
@EnableAdminServer
main(){}

server running:::
http://localhost:8888/applications

been monitored project:::
must follow the rules to create
```

### 6 How it works

### 6.1 Profile

application-profile --> application-\[dev].yml

many properties file,

```java
spring.profiles.active=prod

default and environment --> loading, but environment should be domainated
```

![can use cmd to start with specific profile](<../.gitbook/assets/image (207).png>)

```java
@Profile("production")  //implement here
@Configuration
public class ProductionConfiguration{}
```

```markup
Profile groups:
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq
```

### 6.2 Outer configuration

![](<../.gitbook/assets/image (208).png>)

![](<../.gitbook/assets/image (209).png>)

### 6.3 Self-defined Starter

![](<../.gitbook/assets/image (210).png>)

```java
Module1:xxx-spring-boot-starter
Module2:xxx-spring-boot-starter-autoconfiguration  

Module2:
  need to implement detailed content
  must have XXXServiceAutoConfiguration class
  
  @Configuration
  @ConditionalOnMissingBean(HelloService.class)
  @EnableConfigurationProperties(HelloProperties.class)  //default put HelloProperties into container
  public class HelloServiceAutoConfiguration{
  
  }

Module1 must dependency Module2

use Maven to clean & install
  


```

### 6.4 crash the SpringBoot

1. create application
2. run application

{% hint style="info" %}
headless mode: not rely on others

getSpringFactories --> springt.factories file

ioc--> initalizer --> listener --> refresh() \[core] --> starter() --> runner


{% endhint %}

#### SpringBoot启动过程

_创建 **SpringApplication**_

*
  * 保存一些信息。
  * 判定当前应用的类型。ClassUtils。Servlet
  * **bootstrappers：初始启动引导器（**List\<Bootstrapper>**）：去spring.factories文件中找** org.springframework.boot.**Bootstrapper**
  * 找 **ApplicationContextInitializer**；去**spring.factories找** **ApplicationContextInitializer**
*
  *
    * List\<ApplicationContextInitializer\<?>> **initializers**
*
  * **找** **ApplicationListener  ；应用监听器。**去**spring.factories找** **ApplicationListener**
*
  *
    * List\<ApplicationListener\<?>> **listeners**

_运行 **SpringApplication**_

*
  * **StopWatch**
  * **记录应用的启动时间**
  * **创建引导上下文（Context环境）createBootstrapContext()**
*
  *
    * 获取到所有之前的 **bootstrappers 挨个执行** intitialize() 来完成对引导启动器上下文环境设置
*
  * 让当前应用进入**headless**模式。**java.awt.headless**
  * **获取所有** **RunListener（运行监听器）【为了方便所有Listener进行事件感知】**
*
  *
    * getSpringFactoriesInstances 去**spring.factories找** **SpringApplicationRunListener**.
*
  * 遍历 **SpringApplicationRunListener 调用 starting 方法；**
*
  *
    * **相当于通知所有感兴趣系统正在启动过程的人，项目正在 starting。**
*
  * 保存命令行参数；ApplicationArguments
  * 准备环境 prepareEnvironment（）;
*
  *
    * 返回或者创建基础环境信息对象。**StandardServletEnvironment**
    * **配置环境信息对象。**
*
  *
    *
      * **读取所有的配置源的配置属性值。**
*
  *
    * 绑定环境信息
    * 监听器调用 listener.environmentPrepared()；通知所有的监听器当前环境准备完成
*
  * 创建IOC容器（createApplicationContext（））
*
  *
    * 根据项目类型（Servlet）创建容器，
    * 当前会创建 **AnnotationConfigServletWebServerApplicationContext**
*
  * **准备ApplicationContext IOC容器的基本信息**  **prepareContext()**
*
  *
    * 保存环境信息
    * IOC容器的后置处理流程。
    * 应用初始化器；applyInitializers；
*
  *
    *
      * 遍历所有的 **ApplicationContextInitializer 。调用** **initialize.。来对ioc容器进行初始化扩展功能**
      * 遍历所有的 listener 调用 **contextPrepared。EventPublishRunListenr；通知所有的监听器contextPrepared**
*
  *
    * **所有的监听器 调用** **contextLoaded。通知所有的监听器 contextLoaded；**
*
  * **刷新IOC容器。**refreshContext
*
  *
    * 创建容器中的所有组件（Spring注解）
*
  * 容器刷新完成后工作？afterRefresh
  * 所有监听 器 调用 listeners.**started**(context); **通知所有的监听器** **started**
  * **调用所有runners；**callRunners()
*
  *
    * **获取容器中的** **ApplicationRunner**
    * **获取容器中的  CommandLineRunner**
    * **合并所有runner并且按照@Order进行排序**
    * **遍历所有的runner。调用 run** **方法**
*
  * **如果以上有异常，**
*
  *
    * **调用Listener 的 failed**
*
  * **调用所有监听器的 running 方法  **listeners.running(context); **通知所有的监听器** **running**
  * **running如果有问题。继续通知 failed 。调用所有 Listener 的** **failed；通知所有的监听器** **failed**

```java
ApplicationContextInitilizer
ApplicationListener
SpringApplicationRunListener
ApplicationRunner
SpringApplicationRunListener


resources/META-INF/spring.factories  --> save all the component list

@Order(n) //n represent the priority

```

## SpringBoot 1.x.x (Before 2020)  ===========================

### Banner

```java
SpringApplication application = new SpringApplication(Application.class);
application.setBannerMode(Banner.Mode.OFF);
application.run(args);
```

### Properties File

```markup
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/jdbc
spring.datasource.username=root
spring.datasource.password=Achen888?

spring.jpa.database=mysql
spring.jpa.show-sql=true
spring.jpa.generate-ddl=true
spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.naming_strategy=org.hibernate.cfg.ImprovedNamingStrategy
```

### Swing with Springboot: how to start application

```java
public static void main(String[] args){

    var ctx = new SpringApplicationBuilder(SwingApp.class).headless(false).run(args);
    
    EventQueue.invokeLater(()->{
        var ex = ctx.getBean(SwingApp.class);
        ex.setVisible(true);
    });
}
```



![@RequestParam()   VS   @PathVariable](<../.gitbook/assets/image (108).png>)

@**ResponseBody **: return string value will be the **JSON **content










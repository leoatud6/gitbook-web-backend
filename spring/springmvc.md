---
description: 完全替代Struts2,
---

# SpringMVC

## Overview

Spring core: ** Beans, core, context, expression**

MVC core: 收到请求，传递参数， 跳转页面 （收参，传值，跳转）

```java
@Controller // define action component
@RequestMapping //url
@RequestParam //参数
@RequestBody //读取request请求的body部分
@ResponseBody
```

#### 前端控制器 （web.xml） -->**DispatchServlet**

这个servlet只会去找handler，不会访问static resource，除非设置servletmapping url pattern : _.action 前端filter请求，只会处理.action的url access _

_Servlet url-pattern: / (类似 /_)， 等价于一个filter， filter to ALL 作用：调用springmvc all, 启动mvc factory : 因此需要有init param: contextConfigLocation classpath:mvc.xml

#### 后端控制器（正常的servlet） --> controller

先写controller： @controller 

@RequestMapping(“/xx”) method也可以用这个@ Controller return的内容即为url, 会与**prefix **and **surfix**拼接，然后redirect到 指定页面 controller里面的方法叫做handler: must have return String —> 返回的是一段url 因此也可以传参，访问时 url?id=1, id与code中传入参数名字一样即可 %20: space

## Forward VS Redirect（forward带有之前信息(method not change), redirect重新去新的URL）

C—>View 

C—>Controller （**功能之间的衔接**）一定是redirect (no use @RestController)

**Redirect**: /xx/xx/xx.jsp //如果返回中有这两个Keywords， 需要加上extension 

**Forward**: /xx/xx/xx.jsp



**Forward**，可以重复执行，url不会改变，**因此增删改之后不能用forward **

**Redirect**, **url会改变，换成新的url**，再刷新是new link



C得到数据后，转发V，并向V传递数据，

V渲染数据后，返回用户有数据的页面 

Forward: Request作用域 

Redirect：session作用域 session里面包含requests

## Model & ModelAndView

里面的数据在jsp中都可以通过requestScope.xx获取 

**model: **可以渲染很多工作，不仅仅是JSP，目的：data 与 view 解耦 

**modelAndView**: setViewName(“forward:/data.jsp”) 即为url springmvc大部分传值都在request里面，而非session

## Get VS Post

GET VS POST： get会将request的具体data放在url中 

post会将request的具体data放在**body**中，**不会暴露**

## AJAX: asyc javascript and xml

AJAX 就是用 JS 向服务端发起一个请求，并获取服务器返回的内容 不需要刷新页面，直接用javascrpit发送请求，（不必等待结果，可以继续做其他的事儿） 

Ajax: 在jsp中使用script，需要jquery.js 

JQuery就是javascript的一篇代码： 对于操作页面的包，还可以优化Ajax



## Exception Resolve

handler里面有很多try catch, 每个catch都有相应的logic去处理 

Dao/ Service里面的exception, 必须无条件上抛到Controller 

高内聚：逻辑单一，每个class只单单处理一个业务 Return ModelAndView : 只有这里会用到ModelAndView

## HandlerInterceptor

目的：抽取C中的冗余功能 ==> HandlerInterceptor 接口 

比如不登录不让做一些事儿 

三个周期：：：preHandler —> postHandler —> afterCompletion 

过滤器filter： for spring 

拦截器：for handler、controller中

## SpringBoot整合Spring MVC

![two factories， core](<../.gitbook/assets/image (21).png>)

```java
在mvc.xml里面：<context:component-scan base-package=“” /> 
    Use-default-filters=“false”  ==> include-filter expression=“Controller"
在applicationContext.xml里面： <context:component-scan base-package=“” />
    Use-default-filters=“true” ==> exclude-filter expression=“Controller"
```

#### mybatis applicationContext

* 配置dataSource / connectionPool \~
* 配置SqlSessionFactoryBean, 引用dataSource，作为POJO \~
* 配置MapperScannerConfigurer, 作为DAO \~
* 配置DataSourceTransactionManager, tx ,引用dataSource\~

#### @CrossOrigin("url")

![](<../.gitbook/assets/image (22).png>)

## Others

@GetMapping @PostMapping @PutMapping @DeleteMapping === @ReqeustMapping

本身基于MVC分离模式

![DispatchServlet is core (本身无处理能力)](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FsHsHhvyQwwX8Ayf1NNzI%2Ffile.png?alt=media)

![how tomcat works](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FO9NToWCGpxsjtQs4hoIL%2Ffile.png?alt=media)

## Interceptor

interceptor使用反射，相当于**proxy，代理**模式

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FbExdZuxh1dFDOoh3DY1o%2Ffile.png?alt=media)

## Filter

filter因为implements了很多filter，每过一个filter都要调用一下filter的方法做过滤，责任链模式

filter在外，handler/interceptor在内（使用反射，已经抵达class内了） 

**filter仅仅对web使用，interceptor可以对任何组件使用，直接IoC即可。**

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2FCvUNrHTXUpCzO8KSJR5u%2Ffile.png?alt=media)



## SpringMVC工作原理

1. 用户向服务端发送一次请求，这个请求会先到前端控制器**DispatcherServlet**。
2. DispatcherServlet接收到请求后会调用**HandlerMapping**处理器映射器。由此得知，该请求该由哪个Controller来处理。
3. DispatcherServlet调用**HandlerAdapter**处理器适配器，告诉处理器适配器应该要去执行哪个Controller。
4. HandlerAdapter处理器适配器去执行Controller并得到**ModelAndView**，并层层返回给DispatcherServlet。
5. DispatcherServlet将ModelAndView交给**ViewReslover**视图解析器解析，然后返回真正的视图。
6. **DispatcherServlet**将模型数据填充到视图中。
7. DispatcherServlet将结果响应给用户。



DispatchServlet <-- FrameworkServlet <-- HttpServletBean <-- HttpServlet <-- GenericServlet <-- Servlet

![](<../.gitbook/assets/image (105).png>)



## DoPost and DoGet from FacsGatewayClient

```java
 /**
     * http://localhost:39320/iotgateway/browse
     * @return
     */
    public String browsePing() {
        return doGet(HTTP + webServerUrl + ":" + webServerPort + BROWSE);
    }


    //main method being used
    public String writeBean(KepwareIotBean bean){
        //logger.info("HTTP + webServerUrl + \":\" + webServerPort + WRITE ::::" + JSON.toJSONString(bean));
        return doPost(HTTP + webServerUrl + ":" + webServerPort + WRITE, bean);
    }

    /**
     * url: https://localhost:39320/iotgateway/write
     * contains body of the POST: [{"id": "Channel1.Device1.Tag1","v": "123"}]
     *
     * @param url
     * @param bean, beanList
     * @return
     */
    public String doPost(String url, KepwareIotBean bean) {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        HttpPost httpPost = new HttpPost(url);
        CloseableHttpResponse response = null;

        try {
            //set request paramaters
            StringEntity params = new StringEntity(JSON.toJSONString(bean));
            params.setContentType(new BasicHeader(org.apache.http.protocol.HTTP.CONTENT_TYPE, "application/json"));
            httpPost.setEntity(params);
            //execute
            response = httpclient.execute(httpPost);
            //if response is 200, return value, else, return null
            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                HttpEntity entity = response.getEntity();
                String result = EntityUtils.toString(entity, "UTF-8");
                EntityUtils.consume(entity);
                httpclient.close();
                return result;
            }
            httpclient.close();
        } catch (IOException e) {
            logger.error("DoPost request failed" , e);
            return null;
        }
        return null;
    }


    /**
     * https://localhost:39320/iotgateway/read?ids=Channel1.Device1.Tag1&ids=Channel2.Device2.Tag2
     *
     * If throw exception, than failed
     * @param url
     * @return
     */
    public String doGet(String url) {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        // create get request
        HttpGet httpGet = new HttpGet(url);
        CloseableHttpResponse response = null;
        try {
            // execute
            response = httpclient.execute(httpGet);
            // check if return is 200
            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                HttpEntity entity = response.getEntity();
                String result = EntityUtils.toString(entity, "UTF-8");
                EntityUtils.consume(entity);
                httpclient.close();
                return result;
            }
            httpclient.close();
        } catch (IOException e) {
            logger.info("Fail to ping IoT server at: " + "[ " + url + " ]", e);
            return "Fail to ping IoT server at: " + "[ " + url + " ]";
        }
        return "No Response from IoT server at : "  + "[ " + url + " ]" ;
    }

    private void urlBuilder(String p1, String p2){
        URIBuilder builder;
        try {
            builder = new URIBuilder(HTTP + webServerUrl + ":" + webServerPort + READ);
            builder.setParameter(p1, p2);
            //HttpPost post = new HttpPost(builder.build());
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }

    }
```

```java
public class HttpClientUtil {

    @Test
    public void send() throws InterruptedException {
        String urlpost = "http://127.0.0.1:39320/iotgateway/write";
        String urlget = "http://127.0.0.1:39320/iotgateway/read";

        for (int i = 0; i < 2; i++) {

            Thread.sleep(2000);
            //System.out.println(doPost(urlpost));
           // System.out.println(doGet(urlget));
        }

    }

    public static String doPost(String url){
        List<HashMap> list = new ArrayList<>();
        HashMap<String, String> map = new HashMap<>();
        map.put("id", "Channel1.iottest.blender2");
        map.put("v","99.66");
        list.add(map);

        map = new HashMap<>();
        map.put("id", "Channel1.iottest.blender1");
        map.put("v","77.6");
        list.add(map);

        CloseableHttpClient httpclient = HttpClients.createDefault();

        HttpPost httpPost = new HttpPost(url);
        try {
            StringEntity params = new StringEntity(JSON.toJSONString(list));
            params.setContentType(new BasicHeader(HTTP.CONTENT_TYPE, "application/json"));

            httpPost.setEntity(params);

            System.out.println("entity is : " + params.toString());
            System.out.println("json is : " + JSON.toJSONString(list));
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }

        CloseableHttpResponse response = null;

        try {
            response =  httpclient.execute(httpPost);
            if(response.getStatusLine().getStatusCode() == HttpStatus.SC_OK){
                System.out.println("connection accept =============================================================================");
                HttpEntity entity = response.getEntity();
                String result = EntityUtils.toString(entity, "UTF-8");
                EntityUtils.consume(entity);
                httpclient.close();
                return result;
            }
            httpclient.close();
        } catch (IOException e){
            e.printStackTrace();
            return null;
        }
        return null;
    }


    public static String doGet(String url) {
        // 创建Httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
        // 创建http GET请求
        HttpGet httpGet = new HttpGet(url);
        CloseableHttpResponse response = null;
        try {
            // 执行请求
            response = httpclient.execute(httpGet);
            // 判断返回状态是否为200
            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                HttpEntity entity = response.getEntity();
                String result = EntityUtils.toString(entity, "UTF-8");
                EntityUtils.consume(entity);
                httpclient.close();
                return result;
            }
            httpclient.close();
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }

        return null;
    }


    public static void download(String url, String fileName) {

        // 创建Httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
        // 创建http GET请求
        HttpGet httpGet = new HttpGet(url);
        CloseableHttpResponse response = null;
        try {
            // 执行请求
            response = httpclient.execute(httpGet);
            // 判断返回状态是否为200
            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                HttpEntity entity = response.getEntity();

                // String result = EntityUtils.toString(entity, "UTF-8");
                byte[] bytes = EntityUtils.toByteArray(entity);
                File file = new File(fileName);
                //  InputStream in = entity.getContent();
                FileOutputStream fout = new FileOutputStream(file);
                fout.write(bytes);

                EntityUtils.consume(entity);

                httpclient.close();
                fout.flush();
                fout.close();
                return;
            }
            httpclient.close();
        } catch (IOException e) {
            e.printStackTrace();
            return;
        }

        return;
    }
}
```



## ===尚硅谷 SpringMVC===

**source code:** [https://s8jl-my.sharepoint.com/personal/atguigu_s8jl_onmicrosoft_com/\_layouts/15/onedrive.aspx?id=%2Fpersonal%2Fatguigu%5Fs8jl%5Fonmicrosoft%5Fcom%2FDocuments%2F%E5%B0%9A%E7%A1%85%E8%B0%B7SpringMVC%E6%96%B0%E7%89%88\&originalPath=aHR0cHM6Ly9zOGpsLW15LnNoYXJlcG9pbnQuY29tLzpmOi9nL3BlcnNvbmFsL2F0Z3VpZ3VfczhqbF9vbm1pY3Jvc29mdF9jb20vRXQ3d21kNDNsT0pHZ3k0eHJrcVU4dE1CS3loanZPbmhyQnQyYTNGNFY4aFo0QT9ydGltZT1lc1JhMnNqeTJFZw](https://s8jl-my.sharepoint.com/personal/atguigu_s8jl_onmicrosoft_com/\_layouts/15/onedrive.aspx?id=%2Fpersonal%2Fatguigu%5Fs8jl%5Fonmicrosoft%5Fcom%2FDocuments%2F%E5%B0%9A%E7%A1%85%E8%B0%B7SpringMVC%E6%96%B0%E7%89%88\&originalPath=aHR0cHM6Ly9zOGpsLW15LnNoYXJlcG9pbnQuY29tLzpmOi9nL3BlcnNvbmFsL2F0Z3VpZ3VfczhqbF9vbm1pY3Jvc29mdF9jb20vRXQ3d21kNDNsT0pHZ3k0eHJrcVU4dE1CS3loanZPbmhyQnQyYTNGNFY4aFo0QT9ydGltZT1lc1JhMnNqeTJFZw)

* HandlerMapping:
* HanlderInterceptor
* ViewResolver
* **LocalResolver**: 本地化，国际化
* **MultipartResolver**: file upload resolver
* **HandlerExceptionResolver**: exception handler

Jar package need:

* _spring-aop_
* _spring-context_
* _spring-core_
* _spring-expression_
* commons-logging
* **spring-web**
* **spring-webmvc**

****

![xml version to create the servlet](<../.gitbook/assets/image (212).png>)

![ViewResolver process /WEB-INF/view/xxx.jsp --> access path](<../.gitbook/assets/image (213).png>)

**@RequestMapping(value="xxx") : xxx put into ViewResolver(map path to resource)**

**@RequestMapping(method="xxx") : GET HEAD POST PUT PATCH DELETE OPTIONS TRACE**

AOP & OOP (horizontal and vertical)

![](<../.gitbook/assets/image (214).png>)

**405**: Method not support

**4xx**: client problem: request sender problem

![this is RESTful, same url, different method](<../.gitbook/assets/image (215).png>)

```java
//RESTful
@RequestMapping(value="/test", method=RequestMethod.GET)
//same url, different method is the core of RESTful

//must have username
@RequestMapping(value="/test", params={"username", "!age"})
//cannot have age

@RequestMapping(value="/test", params={"username", "age!=12"})
//age cannot equal to 12

@RequestMapping(value="/test", headers={"xx=xx"})
//must be in the same header

@RequestMapping(value="/*/ant??/test")
//path = /*/antXX/test (X can be any char, x can be anything)
/*
    *: anything in the same layer
    ?: any character
    **: anything, any layer
*/

//placeholder usage
@RequestMapping(value="/test/{id}/{username}")
public String test(@PathVariable("id") Integer id, @PathVariable("username") String name){}


```

```java
Listener --> Filter --> (Interceptor) Servlet --> REQUEST PROCESS

HiddenHttpMethodFilter::: Filter POST method request
    doFilterInternal()
        doFilterChain (ResponibleChain pattern)
            filterChain.doFilter((servlet)wraper, response)
作用：在form中转化为相应的请求
1. must be POST
2. 参数 = _method

```

![\*/ filter every request](<../.gitbook/assets/image (216).png>)

#### 使用form发送PUT请求：(form defualt using POST-->hidden HTTP)

![\_method + POST ==> PUT](<../.gitbook/assets/image (217).png>)

**AJAX： activeX plugin**

主要是解决兼容性问题

 页面不跳转，并且产生一些interaction

### Request 请求数据

![](<../.gitbook/assets/image (218).png>)

* form中名称与controller方法中参数name一致，则可以看作是request的参数自动赋值

```java
@RequestParam(value="xx", required=false, defaultValue="admin")
input参数的由来，就是request里面的参数
value="" : mapping
required="" : necessary or not?
defaultValue="" : if not exist

@RequestHeader() 
请求头的信息，header， same with RequestParam
@RequestHeader("Accept-language") 

@CookieValue()
session must rely on cookie, so, in the cookies, there must have a session ID
正常只能获得client variable， cookie属于浏览器？必须用cookieValue


```




























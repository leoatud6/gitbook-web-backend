# Good Resources

## 纯源码

[http://www.iocoder.cn/JUC/good-collection/?vip](http://www.iocoder.cn/JUC/good-collection/?vip)

### JUC source code 

[https://www.jianshu.com/p/af9c0f404a93](https://www.jianshu.com/p/af9c0f404a93)



### ToolBoxs

[http://www.jayh.club/#/99.tools/01.%E8%87%AA%E5%AE%9A%E4%B9%89Markdown%E7%A5%9E%E5%99%A8Typora%E7%9A%84%E4%B8%BB%E9%A2%98%E6%A0%B7%E5%BC%8F](http://www.jayh.club/#/99.tools/01.%E8%87%AA%E5%AE%9A%E4%B9%89Markdown%E7%A5%9E%E5%99%A8Typora%E7%9A%84%E4%B8%BB%E9%A2%98%E6%A0%B7%E5%BC%8F)









### HttpClient

```java
public class HttpClientUtil {
    public static String doGet(String url)   {
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
        }catch (IOException e){
            e.printStackTrace();
            return null;
        }

        return  null;
    }
```

### HttpPost (Kepware Project)

```java
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
            httpclient.close();  //need to close everytime
            return result;
        }
        httpclient.close();
    } catch (IOException e) {
        logger.error("DoPost request failed" , e);
        return null;
    }
    return null;
}
```

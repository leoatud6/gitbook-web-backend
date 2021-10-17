# onlinestore2021\_基础1\_architecture and project setup

![](<../.gitbook/assets/image (35).png>)

springboot / redis / mysql --> default known

![micro-service ](<../.gitbook/assets/image (36).png>)

## Conclusion

![](<../.gitbook/assets/image (83).png>)

## Vagrant download mirror

[https://app.vagrantup.com//boxes/search](https://app.vagrantup.com/boxes/search) --> download package

```bash
vagrant init centos/7
vagrant up
vagrant ssh
#after modify the ip mapping address
vagrant reload

```

![](<../.gitbook/assets/image (37).png>)

端口转发 port forwarding --> need more mapping

## Docker installation

{% embed url="https://docs.docker.com/" %}

[https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)

```bash
docker build
docker pull
docker run
sudo systemctl start docker
sudo docker images
#enable startup
sudo systemctl enable docker  

#################MYSQL START###################
#download images
docker pull mysql:5.7

docker run -p 3306:3306 --name mysql 
-v /mydata/mysql/log:/var/log/mysql 
-v /mydata/mysql/data:/var/lib/mysql 
-v /mydata/mysql/conf:/ect/mysql 
`-e MYSQL_ROOT_PASSWORD=root -d mysql:5.7

docker ps

#enter container's bin bash
docker exec -it ce741cd07ace /bin/bash


#################REDIS START###################

docker run -p 6379:6379 --name redis 
-v /mydata/redis/data:/data 
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf 
-d redis redis-server /etc/redis/redis.conf

#enter into redis client
docker exec -it redis redis-cli
docker restart redis

#auto start
docker update redis --restart=always 

```

![](<../.gitbook/assets/image (38).png>)

appendonly yes : for redis persistence

{% embed url="https://raw.githubusercontent.com/redis/redis/6.0/redis.conf" %}

## Maven & Java path

```bash
which java   #usr/bin/java
java -version
export JAVA_HOME=/Library/Java/Home
echo $JAVA_HOME

#version 2
cd /Users/chenliu
ls -l #show list
ls -a #show all (include hiden)
#find bash_profile
open -e .bash_profile #open in app
"export M2_HOME=/Applications/apache-maven-3.6.3
export PATH=$PATH:$M2_HOME/bin"
source .bash_profile

```

## VSCode: also install many plugins

## DBeaver: replacement for Mysql workbench

## Git installation

```bash
brew install git
brew install git-gui

git config --global user.name "leoatud"
git config --global user.email "leoatud@gmail.com"
ssh-keygen -t rsa -C "leoatud@gmail.com"
#check public key
cat ~/.ssh/id_rsa.pub 
#test connection
ssh -T git@github.com
```

.gitignore --> need to setup in IDEA

## Renren open source

### renren fast

### renren fast vue: 

```bash
#first: install node js
npm install xxx
npm cache clean --force
rm -rf node_modules package-lock.json
npm install
npm run dev  / npm start
```

## SpringCloud Alibaba (need to choose correct version) 

![](<../.gitbook/assets/image (54).png>)

```java
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.2.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

* configuration center: Spring cloud config  /  **Nacos**
* registry center: Eruka  /  **Nacos**
* API gateway: Zuul   / ** SpringCloud gateway**

![](<../.gitbook/assets/image (53).png>)

![](<../.gitbook/assets/image (51).png>)

![](<../.gitbook/assets/image (52).png>)

### Nacos as Discovery center

```bash
 <dependency>
     <groupId>com.alibaba.cloud</groupId>
     <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
 </dependency>
 
 nacos server port: 8848
 #download and run
 
 #start the nacos
 sh startup.sh -m standalone
 
 #yml application properties
 spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        

#main 
@EnableDiscoveryClient
#must have a service name
spring: 
 application:
    name: service-provider

```

### Nacos as Config center

实际上就是propertie file的内容可以实时更新，在nacos中

#### 核心：程序先load bootstrap--> find nacos config center--> load properties (high prority)

![](<../.gitbook/assets/image (55).png>)

```bash
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>

#bootstrap.properties 优先于 application.properties
spring.application.name=gulimall-coupon
spring.cloud.nacos.config.server-addr=192.168.4.42:8848
#choose which namespace or group to use (optional)
spring.cloud.nacos.config.namespace=871f7460-6bae-46a2-bc33-055ab5ad6ad8
spring.cloud.nacos.config.group=Default


#annotation in restcontroller
@RefreshScope
@Value("${xx.xx.xx}") (getting from nacos instead of properties file)
在nacos里面，需要创建groupid= name + .properties

```

* 命名空间：namespace default public (dev, test, run, product)
* 配置集：
* 配置集ID：dataID(naming convention: gulimall-xxx.properties)
* 配置分组：1111,1212,618 .... 

![namespace, self define, default public](<../.gitbook/assets/image (56).png>)

### Feign (http-based)  (假装)

compare to Dubbo(RPC)

```java
<dependency>spring-cloud-starter-openfeign</dependency>

@EnableFeignClient(basePackages="reference")

@FeignClient("gulimall-ware")
public interface xxxService(){
    @PostMapping()
    ...xxx...xxx
}



//details as follow ==============================
@EnableFeignClients(basePackages = "com.onlinestore.member.feign")
@EnableFeignClients("com.onlinestore.member.feign")
一定要有开启！！！！！！！！！！

//how to write interface in member module
@FeignClient("gulimall-coupon")
public interface CouponFeignService {
    @RequestMapping("/coupon/coupon/member/list")
    public R memberCoupons();
}
```

## API Gateway (routing, circuit breaker, limiting, monitoring, load balancing)

Spring Cloud **Gateway **replace Zuul

**Netty**(api gateway==> fast)VS tomcat

* route
* predicate
* filter

![实际上就是URI分析--> route到指定proxy](<../.gitbook/assets/image (57).png>)

```java
1. Create new Gateway module --> using Gateway 
2. change springboot version and spring cloud version
3. @EnableDiscoveryClient
4. bootstrap.properties
5.(optional) @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})



http://localhost:89/?url=qq
//settings in application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: test_route
          uri: https://www.baidu.com
          predicates:
            - Query=url,baidu
            
        - id: qq_route
          uri: https://www.qq.com
          predicates:
            - Query=url,qq
```








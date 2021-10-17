# MyBatis

## Tk-mapper

```markup
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>${mapper-starter.version}</version>
            <!--            <exclusions>-->
            <!--                <exclusion>-->
            <!--                    <groupId>org.springframework.boot</groupId>-->
            <!--                    <artifactId>spring-boot-starter-jdbc</artifactId>-->
            <!--                </exclusion>-->
            <!--            </exclusions>-->
        </dependency>

@MapperScan(basePackage = "reference of mapper")  
          
@Column
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)

mapper interface extends Mapper<BEAN>


//all methods
selectAll
selectByPrimaryKey
select(bean)
selectOne(bean)
insert()
insertSelective()
updateByPrimaryKey(bean)
updateByPrimaryKeySelective(bean)
updateByExample(bean)
deleteByPrimaryKey
deleteByExample



```



```
#mybatis
mybatis.configuration.map-underscore-to-camel-case=true
//this will also applied to field
//一律使用包装类类型，否则报错
```



## Cache level

1. level one cache: default use: session scope

同一个session中同样的查询，只会generate一套sql语句 如果两个同样的sql中间有更新或者update，缓存会刷新，重新执行查询（实际上是返回的内容重新操作） 

2\. level two cache: default close: 跨session

不同的session内执行相同的内容，每次都会独立进行 可以手动开启二级缓存，但是要设置cache  在实例类中实现serializiable::: implements Serializble







## Lazy loading

```markup
//mapper.xml
//需要加入asm & cjlib jar
<settings>
    <setting name=“useGeneratedKeys” value=“true” />
    <setting name=“lazyLoadingEnabled” value=’true” />
    <setting name=“aggressiveLazyLoading” value=“false” />
    
    
```

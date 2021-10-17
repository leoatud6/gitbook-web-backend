# onlinestore2021\_基础3\_onlinestore_content

### Recursion & Stream to lookup

```java
	//=== bean add new features --> mybatis-
	@TableField(exist = false)
	private List<CategoryEntity> children;
	
//=== service implementation customized ===
//recursion find child-menus
private List<CategoryEntity> getChildren(CategoryEntity entity, List<CategoryEntity> all){

    List<CategoryEntity> child = all.stream().filter(categoryEntity -> {
        return categoryEntity.getParentCid() == entity.getCatId();
    }).map(item -> {
        item.setChildren(getChildren(item, all));
        return item;
    }).sorted((a, b) -> {
        return a.getSort() - b.getSort();
    }).collect(Collectors.toList());
    
    return child;
}  
```

### Disable ESLint

![](<../.gitbook/assets/image (70).png>)

### Vue life cycle methods

![](<../.gitbook/assets/image (71).png>)

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: admin_route
          uri: lb://renren-fast
          predicates:
            - Path=/api/**
          ## all front end request using "api"

###########
request: http://localhost:8080/renren-fast/captcha.jpg

```

### 跨域Cross Region:

浏览器对Javascript施加的安全限制， 不同网站。协议，域名，端口任何有不同，就有同源限制。

![cross region process](<../.gitbook/assets/image (73).png>)

如果是简单请求（head, get, post）， 且content-type为：text/plain /form-data/ application。 其他的需要发布**预见请求**， **OPTIONS **request deny first。

```javascript
data() {
  return {
    meuns: [],
    defaultProps: {
      children: "children",
      label: "name",   
      //props: the detail content for the element ui to display
    },
  };
},

<span class="custom-tree-node" slot-scope="{ node, data }">
<span>{{ node.label }}</span>
<span>
  <el-button type="text" size="mini" @click="() => append(data)">
    Append
  </el-button>
  <el-button type="text" size="mini" @click="() => remove(node, data)">
    Delete
  </el-button>
</span>
</span>

//: = v-bind : put following line into el-tree tag
default-expand-all:expand-on-click-node="false"

====http post====
addCategory(data) {
  console.log("submit data", this.category);
  this.http({
    url: this.http.adornUrl("/product/category/save"),   //url
    method: "post",                                      //method
    data: this.http.adornData(this.category, false),     //data binding
  }).then(({ data }) => {
    this.$message({
      message: "save successful",                        //success message
      type: "success",
    });
  });
}, 

==== 解构，仅仅取出对象中部分内容，其他内容保持不变 ====
var { catId, name, icon, productUnit } = this.category;
var data = {
  catId: catId,
  name: name,
  icon: icon,
  productUnit: productUnit,
};

//简化
var data = { catId, name, icon, productUnit };

==== recursive to find the maxLevel --> drag ====
this.maxLevel //is the memberField value
countNodeLevel(node) {
  //find all node, calculate max depth
  if (node.children != null && node.children.length > 0) {
    //need to iterater
    for (let i = 0; i < node.children.length; i++) {
      if (node.children[i].catLevel > this.maxLevel) {
        this.maxLevel = node.children[i].catLevel;
      }
      //nice once
      this.countNodeLevel(node.children[i]);
    }
  }
},
```



```java
@RequestMapping("/delete")
public R delete(@RequestBody Long[] catIds){
		categoryService.removeByIds(Arrays.asList(catIds));
    return R.ok();
}
//只要有RequestBody，就要用POST请求，因为需要请求体，GET是没有body的


//mybatis-plus逻辑删除，不是彻底physically删除
//logic delete, make in one value as delete, not physically delete
@TableLogic(value = "1", delval = "0")
private Integer showStatus;
	
```

### Mybatis-plus Fake Delete

![](<../.gitbook/assets/image (74).png>)

### File Storage System  (前端upload)

![](<../.gitbook/assets/image (75).png>)

#### OSS: 对象存储object storage service

![](<../.gitbook/assets/image (76).png>)

### Gateway (SpringCloud) Project (URL redirect, need Nacos)

```yaml
#同样也需要配置gateway api
spring:
  cloud:
    gateway:
      routes:
        - id: third_party_route
          uri: lb://third-party   #注意这里lb代表负载均衡，后面是service的name, 显示在nacos中
          predicates:
            - Path=/api/thirdparty/**
          filters:
            - RewritePath=/api/thirdparty/(?<segment>.*),/$\{segment}

        - id: product_route
          uri: lb://gulimall-product
          predicates:
            - Path=/api/product/**
          filters:
            - RewritePath=/api/(?<segment>.*),/$\{segment}
```

### 前后端:必须都校验发送到DB的内容，防止人为使用postman进行request.

#### 此步是为了避免脏数据写进DB

### JSR303 （后端校验）(分组校验)（self-defined annotation）

给Bean添加校验注解

```java
package javax.validation.constraints

@NotNull
@Email
@Future
@Max
@Min
@NotEmptu
@Past
@AssertFalse
...

@Valid (in contoller to enable the validation)

===BindingResult: 可以立刻get校验的结果===
@RequestMapping("/save")
public R save(@Valid @RequestBody BrandEntity brand, BindingResult result){
		if(result.hasErrors()){
		    Map<String, String> map = new HashMap<>();
		    result.getFieldErrors().forEach((t)->{
            String message = t.getDefaultMessage();
            String field = t.getField();
            map.put(field,message);
        });
		    return R.error(400, "not validate").put("data", map);
    }else {
        brandService.save(brand);
    }
    return R.ok();
}


===FOR THE BEAN===
@NotNull(message = "brand name must not be null")
private String name;
//    @NotNull
//    @URL(message = "logo must be an url")
private String logo;

private String descript;

private Integer showStatus;

@Pattern(regexp = "/^[a-zA-Z]$/", message = "must be only one character")
private String firstLetter;

@Min(value = 0, message = "must greater than 0")
private Integer sort;


===GROUP===
首先定义不同接口：AddGroup / UpdateGroup

@NotNull(message = "modified item must have id", groups = {UpdateGroup.class})
@Null(message = "new item cannot assign id", groups = {AddGroup.class})
private Long brandId;

@Validated({AddGroup.class})  放在Controller代替@Valid


=== self-defined ===
//must have three element in the interface:::
String message() default "{javax.validation.constraints.NotNull.message}";
Class<?>[] groups() default {};
Class<? extends Payload>[] payload() default {};

//all @ in the original interface must be copied
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
@Repeatable(NotNull.List.class)
@Documented
@Constraint(
    validatedBy = {ListValueConstriainsValidator.class}
)

//define的时候要有@
public @interface ListValue {}

//define another class: showStatus: only 0 or 1
public class ListValueConstriainsValidator implements ConstraintValidator<ListValue, Integer> {

    private Set<Integer> set = new HashSet<>();

    @Override
    public void initialize(ListValue constraintAnnotation) {
        int[] vals = constraintAnnotation.vals();
        //这里的vals就是可以的取值
        for (int v : vals)
            set.add(v);
    }

    //这里的integer就是需要校验的value
    @Override
    public boolean isValid(Integer integer, ConstraintValidatorContext constraintValidatorContext) {
        return set.contains(integer);
    }
}

//还需要resource里面定义一个ValidationMessages.properties的file
com.onlinestore.common.validation.ListValue=must summit [0] or [1]

//一个注解可以有多个校验器，因为有不同的type,(number, byte, boolean...)
```

![自定义注解三要素](<../.gitbook/assets/image (78).png>)

### Spring Controller Advice (JSR303 Extention)

* 专门定义一个ControllerExceptionAdvice, catch所有的异常，处理所有的exception
* **@Slf4j: 可直接使用log**
* **@RestControllerAdvice**  == @ReponseBody + @RestController + @ControllerAdvice()
* @在Common里面定义BizCodeEnum， 同样有getter， 之后错误代码和信息直接调用code

```java
//can handle all exceptions
@ExceptionHandler(value = MethodArgumentNotValidException.class)
public R handleValidException(MethodArgumentNotValidException e) {
    log.error("xxx", e.getMessage(), e.getClass());
    BindingResult result = e.getBindingResult();
    HashMap<String, String> map = new HashMap<>();
    result.getFieldErrors().forEach((t) -> {
        map.put(t.getField(), t.getDefaultMessage());
    });
    return R.error(BizCodeEnum.VALID_EXCEPTION.getCode(), BizCodeEnum.VALID_EXCEPTION.getMsg()).put("data", map);
}

@ExceptionHandler(value = Throwable.class)
public R handleException(Throwable e) {
    //self added, whatever you want, to carry the bottom of the exception
    return R.error();
}
```

### SKU, SPU

#### SPU： standard product unit 基本属性，规格包装

#### SKU: stock keeping unit 销售属性

### Vue (back to vue)

```java
===VUE===
//../：表示上级目录
import category from "../common/category";
// ./表示当前目录
import AddOrUpdate from "./attrgroup-add-or-update";


===父子组件===
event mechisam
//component 1
methods: {
  nodeclick(data, node, component){
      this.$emit("tree-node-click", data, node, component);
  },
//component 2
<category @tree-node-click="nodeclick"> </category>
methods: {
  nodeclick(data, node, component) {
    console.log("attr get click", data, node, component);
  },
当点击treenode的时候可以传递信息（data, node, components）

```

```java
//现成的工具类在service implementation里面
@Override
public PageUtils queryPage(Map<String, Object> params, Long catelogId) {
    if (catelogId == 0) {
        //check all
        IPage<AttrGroupEntity> page = this.page(new Query<AttrGroupEntity>().getPage(params),
                new QueryWrapper<AttrGroupEntity>());
        return new PageUtils(page);
    } else {
        //select * from attrgroup where catelog_id = ?
        //多字段匹配
        String key = (String) params.get("key");
        QueryWrapper<AttrGroupEntity> wrapper = new QueryWrapper<AttrGroupEntity>().eq("catelog_id", catelogId);
        if (!StringUtils.isEmpty(key)) {
            wrapper.and((obj) -> {
                obj.eq("attr_group_id", key).or().like("attr_group_name", key);
            });
        }
        IPage<AttrGroupEntity> page = this.page(new Query<AttrGroupEntity>().getPage(params),
                wrapper);
        return new PageUtils(page);
    }
}
/**
 SELECT attr_group_id,attr_group_name,sort,descript,icon,catelog_id 
 FROM pms_attr_group 
 WHERE (catelog_id = ? AND (attr_group_id = ? OR attr_group_name LIKE ?))
 **/
```

### MybatisPlus Pagenation

```java
//新增一个config file
@Configuration
@EnableTransactionManagement
@MapperScan(basePackages = "com.onlinestore.product.dao")
public class MybatisConfig {
    @Bean
    public PaginationInnerInterceptor paginationInnerInterceptor(){
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
        paginationInnerInterceptor.setOverflow(true);
        paginationInnerInterceptor.setMaxLimit(500L);
        return paginationInnerInterceptor;
    }
}

相同的表示：：：
@RequestMapping(value = "/catelog/list", method = RequestMethod.GET)
@GetMapping("/catelog/list")

//cascade update in service need 
@Transactional
public void cascadeUpdate(CategoryEntity category) {
    this.updateById(category);
    categoryBrandRelationService.updateCategory(category.getCatId(), category.getName());
}
```

### 分层

* PO： persistant object --> entity
* DO：domain object
* TO：transfer object
* DTO: data transfer object
* VO：value object / view object，接受页面传来的数据，封装对象；将业务处理完成的对象，封装成页面要用的数据
* BO：business object

```java
//可以使用Bean.Utils(from Spring)来copy properties from Entity to VO
//避免一个一个操作
//主要目的是“关联表格”的保存，redundant data who cares
@Transactional
@Override
public void saveAttr(AttrVo attr) {
    AttrEntity entity = new AttrEntity();
    BeanUtils.copyProperties(attr, entity);
    //save base information
    this.save(entity);
    //save releationship
    AttrAttrgroupRelationEntity relationEntity = new AttrAttrgroupRelationEntity();
    relationEntity.setAttrGroupId(attr.getAttrGroupId());
    relationEntity.setAttrId(entity.getAttrId());
    attrAttrgroupRelationDao.insert(relationEntity);
}

//使用各种类，就是不同程度封装bean/pojo， 一层又一层，每一层新增加一些信息内容。
//善用BeanUtils里面util的一些method进行copy and paste

//stream programming
List<AttrAttrgroupRelationEntity> collect = Arrays.asList(vos).stream().map((t) -> {
    AttrAttrgroupRelationEntity relationEntity = new AttrAttrgroupRelationEntity();
    BeanUtils.copyProperties(t, relationEntity);
    return relationEntity;
}).collect(Collectors.toList());
```

#### 大部分logic应该放在Service Impl里面， Controller deals with Service. Service 可以deal with other Services, 也可以deal with Dao

#### 所有通过dao查出来的数据做一个_**非空判断**_

#### 传来数据的pagenation，自带的PageUtil, Page

```java
IPage<AttrEntity> page = this.page(new Query<AttrEntity>().getPage(params), queryWrapper);
PageUtils pageUtils = new PageUtils(page);
return pageUtils;
```

### How to define Enum

```java
public enum AttrEnum{
    ATTR_TYPE_BASE(1, "base"),
    ATTR_TYPE_SALE(0, "sale");
    
    private int code;
    private String msg;

    AttrEnum(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }
}
```

### Mapper.xml

```markup
===如何写batch, 关联，for each in SQL===
<delete id="deleteBatchRelation">
    DELETE FROM UPDATE `pms_attr_attrgroup_relation` WHERE
    <foreach collection="entities" item="item" separator=" OR ">
        (attr_id=#{item.attrId} AND attr_group_id=#{item.attrGroupId})
    </foreach>
</delete>
```

#### VO应当用于controller layer，封装view、前端想要的内容。不掺杂DB entity的详细内容。 

```java
//请求变量不为空：
public R brandsList(@RequestParam(value="catId", required = true) Long catId) {}

@GetMapping("/brands/list")
public R brandsList(@RequestParam(value="catId", required = true) Long catId) {
   List<BrandEntity> vos =  categoryBrandRelationService.getBrandsByCatId(catId);

    List<BrandVo> collect = vos.stream().map((t) -> {
        BrandVo brandVo = new BrandVo();
        brandVo.setBrandId(t.getBrandId());
        brandVo.setBrandName(t.getName());
        return brandVo;
    }).collect(Collectors.toList());

    return R.ok().put("data", collect);
}
```

### 拿到Blob json后，可以用[bejson.com](http://bejson.com) transfer to 实体类

![](<../.gitbook/assets/image (79).png>)

### Feign Again

1. need to register to nacos
2. need to @EnableFeignClients(basePackages="feign package")
3. create interface in feign package
4. need to @FeignClient("which module")
5. FeignClient must use http to connect another Feign                                                                 (so FeignService--> AnotherController) 且一定是完整的路径 (@RequestBody转为Json)
6. @RequestBody传递数据，service同controller一致

![细节：因为是Json，只要是一一对应即可](<../.gitbook/assets/image (80).png>)

```java
//============service==============
@EnableFeignClients(basePackages = "com.onlinestore.product.feign")

@FeignClient("gulimall-coupon")
public interface CouponFeignService {

    @PostMapping("coupon/spubounds/save")
    R saveSpuBounds(@RequestBody SpuBoundTo spuBoundTo);
}

//===========controller===========
@PostMapping("/save")
public R save(@RequestBody SpuBoundTo spuBoundTo){
		spuBoundsService.save(spuBoundTo);
    return R.ok();
}

//======TO==========
需要手动更改，需要添加TRANSFER OBJECT into Common's module
```

#### TO: transfer object: Feign之间传输的内容是TO， json--> json。 TO可以放在common里面，因为是全局使用。内部数据传输。

### IntelliJ, compound start all application

![](<../.gitbook/assets/image (81).png>)

![](<../.gitbook/assets/image (82).png>)

```java
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

### 模糊查询 + 分页

```java
@Override
public PageUtils queryPage(Map<String, Object> params) {

    QueryWrapper<WareInfoEntity> wrapper = new QueryWrapper<>();
    String key =(String) params.get("key");

//模糊检索
    if(!StringUtils.isEmpty(key)){
        wrapper.eq("id", key).or()
                .like("name", key)
                .or().like("address", key)
                .or().like("areacode",key);
    }
    
    IPage<WareInfoEntity> page = this.page(new Query<WareInfoEntity>().getPage(params), wrapper);
    return new PageUtils(page);
}
```

### 在Transaction里面的内容，如果失败，则所有内容rollback， 但是如果是在try-catch里面，则不用rollback.

### 事务

```java
@EnableTransactionManagement (in the spring starter)
@Transactional (in the service implementation)
```


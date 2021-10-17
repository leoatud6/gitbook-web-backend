# onlinestore2021\_进阶1\_Elasticsearch

## ES ElasticSearch 

数据检索和分析

![](<../.gitbook/assets/image (87).png>)

CRUD 海量数据

* insert = **index** (database = index)
* table = **type**
* entry = **document** (**JSON**)
* column = properties

#### inverse index --> 相关性得分

#### 冷热分离

最好将冷数据写入一个索引中，然后热数据写入另外一个索引中，这样可以确保热数据在被预热之后，尽量都让他们留在 Filesystem OS Cache 里。

**join/nested/parent-child 搜索都要尽量避免，性能都很差的**

Vagrant how to set up [network](https://www.vagrantup.com/docs/networking/public_network)

* 9200:   ES对外界通讯的HTTP
* 9300：ES cluster状态下互通的port
* 5601：kibana的operation port

```bash
systemctl enable docker
systemctl start docker

#easilest way to setup vagrant to public network
Vagrant.configure("2") do |config|
  config.vm.network "public_network"
end

#check memory
free -m 

#create folder in vagrant machine
sudo mkdir -p /mydata/elasticsearch/config
sudo mkdir -p /mydata/elasticsearch/data

sudo echo "http.host:0.0.0.0">>/mydata/elasticsearch/config/elasticsearch.yml

#start docker es, must limit the memory, map the port
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx128m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/shared/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/shared/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/shared/elasticsearch/plugins \
-d elasticsearch:7.10.1

docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/shared/elasticsearch/config/elasticsearch.yml -v /mydata/elasticsearch/data:/usr/shared/elasticsearch/data -v /mydata/elasticsearch/plugins:/usr/shared/elasticsearch/plugins -d elasticsearch:7.10.1

#find logs of ES
docker logs elasticsearch

#-R: recursion
chmod -R 777 /mydata/elasticsearch/

#run kibana
docker run --name kibana 
-e ELASTICSEARCH_HOSTS=http://192.168.4.52:9200  (use m1)
-p 5601:5601 -d kibana:7.10.1

#always enable to start
systemctl enable docker
docker update 4bfb8d909bc8 --restart=always

#install analysizer-chinese
docker exec -it ae3cd7baf2da /bin/bash

```

### ES operation command (http-based)

```bash
# get basic information
GET /_cat/nodes
GET /_cat/health
GET /_cat/master
GET /_cat/indices  # show databases

# save content: which index, which id?: put必须有ID
PUT customer/external/1
{"name":"duoduo"}

#result
{
    "_index": "custmer",     #db
    "_type": "external",     #table
    "_id": "1",              #identifyier
    "_version": 4,           #save how many times?
    "result": "updated",     #first time:create, later:update
    "_shards": {             #for cluster use
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}

#如果使用POST->保存更新二合一，系统自动添加ID
POST customer/external/

#POST更新会对比原来的数据，如果没有变化，则相当于没有操作
#update:json里面带上doc
POST customer/external/1/_update
{"doc":{"name": "duoduo"}}
#如果不带_update, 不会检查原来数据，直接更新

#只有POST且带_Update, 才会对比原数据并且更新

#bulk操作
{"index":{"_id":"1"}}
{"name":"caicaizi"}
{"index":{"_id":"2"}}
{"name":"chenchen"}

#所有元内容，需要前面下划线

#add bulk data
PUT /bank/_bulk
#test data
https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json

```

### DSL--Basic -->Kibana: QueryDSL(domain-specific language)

#### must/must not/should --> will contribute the hit-score of the search

#### filter--> not contribute the hit-score

#### 查找精确字段，推荐使用term (replace match)

```bash
GET /bank/_search?q=*&sort=account_number:asc
GET /bank/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "account_number": "asc"
    }
  ]
}

====================================================
#paganation: from 0 to 2
GET /bank/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 2
}
====================================================
#only query _source content, content in hit ->_source
GET /bank/_search
{
  "query": {
    "match_all": {}
  },
  "size": 20,
  "_source":["balance","firstname","lastname"]
}

====================================================
#detail query, account_number=20
#全文检索，最后返回结果按照hit score排序
GET /bank/_search
{
  "query": {
    "match": {
      "account_number": "20"
    }
  }
}

#精确查找，而非同时包含
{"match_phrase" : "xxx & bbb"}

#multi match search, 任何一个匹配上都可
GET /bank/_search
{
  "query": {
    "multi_match": {
      "query": "mill",
      "fields": ["address", "city"]
    }
  }
}
====================================================
#bool search， everything must be TRUE
#should: 满足最好，不满足也可
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "gender": "M"
        }},{"match": {
          "address": "mill"
        }}
      ],
      "must_not": [
        {"match": {
          "age": "38"
        }}
      ],
      "should": [    #better have
        {"match": {
          "lastname": "Wallace"
        }}
      ]
    }
  }
}

====================================================
#filter不会进行任何hit-score calculate, 查询
结果相同
GET /bank/_search
{
  "query": {
    "bool": {
      "filter": [
        {"range": {
          "age": {
            "gte": 18,
            "lte": 30
          }
        }}
      ]
    }
  }
}

====================================================
#term = match, but for precious value query, number等非文本字段
#如果是address等，比较长的，可以分词的，要用match
GET /bank/_search
{
  "query": {
    "term": {
      "age": {
        "value": "38"
      }
    }
  }
}
```

### DSL--Aggregation(数据分析功能) (类似mysql里面的group)

#### "aggs", 同”query“同级

```bash
====================================================
GET /bank/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 30
      }
    }
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": "10"
      },
      "aggs": {
        "genderAgg": {
          "terms": {
            "field": "gender.keyword",
            "size": 10
          },
          "aggs": {
            "balancePerAgg": {
              "avg": {
                "field": "balance"
              }
            }
          }
        },
        "ageBalanceAgg":{
          "avg": {
            "field": "balance"
          }
        }
      }
    },
    "ageAvg": {
      "avg": {
        "field": "age"
      }
    },
    "balanceAvg":{
      "avg": {
        "field": "balance"
      }
    }
  },
  "size": 0
}
```

### DSL--Mapping 映射

#### type在V8版本以后会被移除，因此不要再指定type了

#### 更新映射：唯一办法是数据迁移（创建新的index, data copy from old）

```bash
#print the type of each element
GET /bank/_mapping
====================================================
#create mapping
PUT /my_index
{
  "mappings": {
    "properties": {
      "age":{"type": "integer"},
      "email":{"type": "keyword"},
      "name":{"type": "text"}
    }
  }
}

====================================================
#add new mapping
PUT /my_index/_mapping
{
  "properties":{
    "employee_id":{
      "type":"keyword",
      "index":"false"
    }
  }
}
====================================================
#create new bank index, note that "Keyword" type main 精确信息,different than text
PUT /newbank
{
  "mappings": {
    "properties": {
      "account_number": {
        "type": "long"
      },
      "address": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "balance": {
        "type": "long"
      },
      "city": {
        "type": "keyword"
      },
      "email": {
        "type": "keyword"
      },
      "employer": {
        "type": "keyword"
      },
      "firstname": {
        "type": "keyword"
      },
      "gender": {
        "type": "keyword"
      },
      "lastname": {
        "type": "keyword"
      },
      "state": {
        "type": "keyword"
      }
    }
  }
}

====================================================
#index 迁移
POST _reindex
{
  "source": {
    "index": "bank",
    "type": "account"
  },
  "dest": {
    "index": "newbank"
  }
}
```

### DSL--Tokenizer 分词器 -->中文[IK分词器](https://github.com/medcl/elasticsearch-analysis-ik)

#### 解压到plugin folder即可

```bash
docker exec -it xxxx /bin/bash
chmod -R 777 plugins
elasticsearch-plugin list  #check if installed
docker restart elasticsearch

====================================================
#default
POST _analyze
{
  "analyzer": "standard",
  "text": "我爱猫咪"
}

====================================================
#ik-smart
POST _analyze
{
  "analyzer": "ik_smart",
  #ik_max_word :max the words
  "text": "我爱猫咪"    #我，爱， 猫咪
}

```

### Elasticsearch-Rest-Client (JAVA)

![low level =JDBC , high level =Mybatis](<../.gitbook/assets/image (85).png>)

一定注意版本（专门在properties里面标注）

![](<../.gitbook/assets/image (88).png>)



```bash
9300: TCP （不用这个）
long connection
spring-data-elasticsearch-trasport-api.jar (version be carefule)

9200: HTTP
JestClient: 非官方，更新慢 (6.3.1, two years ago)
RestTemplate: springboot + resttemplate (麻烦)
HttpClient：同上
Elasticsearch-Rest-Client: 官方RestClient, 上手简单

```

```java
@Test
public void indexData1() {
    IndexRequest indexRequest = new IndexRequest("users");
    indexRequest.id("1");
    //method 1
    //indexRequest.source("username", "chenliu", "age", 30, "gender","male");
    
    //method 2
    User user = new User();
    String s = JSON.toJSONString(user);
    indexRequest.source(s, XContentType.JSON); //must be even parameters

    try {
    //execute index command
        IndexResponse index = client.index(indexRequest, COMMON_OPTIONS);
        System.out.println(index);
    } catch (IOException e) {
        e.printStackTrace();
    }
}


@Test
public void searchData(){
    SearchRequest searchRequest = new SearchRequest();
    searchRequest.indices("bank");

    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    searchRequest.source(sourceBuilder);

    try {
        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        SearchHits hits = search.getHits();
        System.out.println("hits number: " + hits.getTotalHits());

    } catch (IOException e) {
        e.printStackTrace();
    }
}

{
    //aggregation, cascade usage
    SearchRequest searchRequest = new SearchRequest();
    searchRequest.indices("bank");

    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    sourceBuilder.query(QueryBuilders.matchQuery("address", "mill"));
    TermsAggregationBuilder agg1 = AggregationBuilders.terms("ageAgg").field("age").size(10);
    sourceBuilder.aggregation(agg1);
    AvgAggregationBuilder agg2 = AggregationBuilders.avg("balanceAvg").field("balance");
    sourceBuilder.aggregation(agg2);
    searchRequest.source(sourceBuilder);
}


{
    //there are two layers of hits (for the real data)
    SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
    System.out.println(search);
    SearchHits hits = search.getHits();
    SearchHit[] hits1 = hits.getHits();
    for (SearchHit hit : hits1) {
        System.out.println("hits content: " + hit.getSourceAsString());
    }
}


{
    //get aggregation result
    Aggregations aggregations = search.getAggregations();
    for (Aggregation agg : aggregations) {
        String name = agg.getName();
    }
    Terms ageAgg = aggregations.get("ageAgg");
    List<? extends Terms.Bucket> buckets = ageAgg.getBuckets();
    for (Terms.Bucket bucket : buckets) {
        String keyAsString = bucket.getKeyAsString();
        System.out.println(keyAsString);
    }
}


{
    //save bulk data to ES
    //2. save data: bulkRequest, RequestOptions
    BulkRequest bulkRequest = new BulkRequest();
    for (SkuEsModel model : skuEsModels) {
        //save requst, one by one, and then add to bulk, 统一执行
        IndexRequest indexRequest = new IndexRequest(EsConstant.PRODUCT_INDEX);
        indexRequest.id(model.getSkuId().toString());
        indexRequest.source(JSON.toJSONString(model), XContentType.JSON);

        bulkRequest.add(indexRequest);
    }
    
    //异常处理可以统一往上抛，到上面layer处理。加上错误码handling
    try {
        BulkResponse bulk = restHighLevelClient.bulk(bulkRequest, GulimallElasticSearchConfig.COMMON_OPTIONS);
        //return handling
        boolean b = bulk.hasFailures();
        List<String> collect = Arrays.stream(bulk.getItems()).map(t -> {
            return t.getId();
        }).collect(Collectors.toList());
        log.error("product publish error", collect);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

#### Json operation tips:

```bash
JSON.parseObjct(String, Account.class);
JSON.getJsonString(object);

#alibaba fast json
```

## Nginx

```bash
docker run -p 80:80 --name nginx -d nginx
docker container cp nginx:/etc/nginx .  #copy folder to local
#change folder name
mv nginx config
#move config folder to nginx foler 
mv config nginx/

docker run -p 80:80 --name nginx -v /mydata/nginx/html:/usr/share/nginx/html -v /mydata/nginx/logs:/var/log/nginx -v /mydata/nginx/conf:/etc/nginx -d nginx

docker update nginx --restart=always
```

![设置分词器地址](<../.gitbook/assets/image (84).png>)

## ES项目应用

* logging system (ELK)
* front end information (for fast speed)

![work as the log storage](<../.gitbook/assets/image (86).png>)

#### 上架：content copy from Database/mysql to ES (cluster, 容量不够，数量来凑，运行在内存当中， 快)

```bash
#create the index for Onlinestore project
PUT /product
{
  "mappings": {
    "properties": {
      "skuId": {
        "type": "long"
      },
      "spuId": {
        "type": "keyword"
      },
      "skuTitle": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "skuPrice": {
        "type": "keyword"
      },
      "skuImg": {
        "type": "keyword",
        "index": false,
        "doc_values": false
      },
      "saleCount": {
        "type": "long"
      },
      "hasStock": {
        "type": "boolean"
      },
      "hotScore": {
        "type": "long"
      },
      "brandId": {
        "type": "long"
      },
      "catalogId": {
        "type": "long"
      },
      "brandName": {
        "type": "keyword",
        "index": false,
        "doc_values": false
      },
      "brandImg": {
        "type": "keyword",
        "index": false,
        "doc_values": false
      },
      "catalogName": {
        "type": "keyword",
        "index": false,
        "doc_values": false
      },
      #注意避免扁平化处理数据，type应该设置为nested
      "attrs": {
        "type": "nested",
        "properties": {
          "attrId": {
            "type": "long"
          },
          "attrName": {
            "type": "keyword",
            "index": false,
            "doc_values": false
          },
          "attrValue": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

### Mybatis ForEach

```bash
#mybatis foreach query
<select id="selectSearchAttrs" resultType="java.lang.Long">
    SELECT attr_id FROM `pms_attr` WHERE attr_id IN 
    <foreach collection="attrIds" item="id" separator="," open="(" close=")">
        #{id}
    </foreach>
    AND search_type = 1
</select>
```

### 增加泛型

```java
//change R to R<T>
public class R<T> extends HashMap<String, Object> {
	private static final long serialVersionUID = 1L;
	
	//new added
	private T data;
	
	//需要有getter & setter
}


//final solution
public <T> T getData(TypeReference<T> typeReference){
	Object data = get("data");
	T t = JSON.parseObject(JSON.toJSONString(data), typeReference);
	return t;
}

public R setData(Object data){
	put("data", data);
	return this;
}
```

### Feign Remote Call

#### 一般远程调用操作需要有try-catch，一旦失败，不影响其他的查询。

#### Feign的重试机制是通过while(true)里面的try-catch完成的，一旦catch, while 结束。最大重试次数是5次，也可以选择never retry。重试机制default close

## 检索功能Search (onlinestore)

 所有search condition：都在url上面：xx=?\&xx=?

```java
//search page所有的查询内容封装成对象：
@Data
public class SearchParam {

    private String keyworkd;
    private Long catalog3Id;

    //search ranking has lot
    private String sort;
    
    private Integer hasStock;
    private String skuPrice;
    private List<Long> brandId;

    private List<String> attrs;

    private Integer pageNum;
}

//同样，返回的内容也要封装，再想办法display到页面
@Data
public class SearchResult {
    //data from ES
    private List<SkuEsModel> products;
    //pagenation
    private Integer pageNum;
    private Long total;
    private Integer totalPages;
    //result of all brands
    private List<BrandVo> brands;
    //category
    private List<CatalogVo> catalogs;
    //attrs
    private List<AttrVo> attrs;
}
```

```java
//ES
GET product/_search
{
  "query": {},    //模糊匹配
  "sort": [        //sort
    {
      "skuPrice": {
        "order": "desc"
      }
    }
  ],
  "from": 0,      //pagenation
  "size": 20,
  "highlight": {  //加粗highlight查询的text
    "fields": {"skuTitle": {}}, 
    "pre_tags": "<b style='color:red'>",
    "post_tags": "</b>"
  }
}
```

#### 如果是nested的data，filter, aggregation等其他操作都需要nested

```java
//java中如何使用
@Autowired
RestHighLevelClient client;

@Override
public SearchResult search(SearchParam param) {

    //1.prepare request
    SearchRequest searchRequest = buildSearchRequest();
    SearchResult result = null;
    try {
        //2.run the request
        SearchResponse response = client.search(searchRequest, COMMON_OPTIONS);
        //3.get the response data and anaylsis
        result = buildSearchResult();

    } catch (IOException e) {
        e.printStackTrace();
    }
    return result;
}



// 搞懂下面的内容，基本ES-JAVA搞定
private SearchResult buildSearchResult(SearchResponse response, SearchParam param) {
    SearchResult result = new SearchResult();
    SearchHits hits = response.getHits();
    //1 product details ===================================================================
    List<SkuEsModel> esModels = new ArrayList<>();
    if (hits.getHits() != null && hits.getHits().length > 0) {
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();
            SkuEsModel skuEsModel = JSON.parseObject(sourceAsString, SkuEsModel.class);
        }
    }
    result.setProducts(esModels);

    //2 ===================================================================
    List<SearchResult.AttrVo> attrVos = new ArrayList<>();
    ParsedNested attr_agg = response.getAggregations().get("attr_agg");
    ParsedLongTerms attr_id_agg = attr_agg.getAggregations().get("attr_id_agg");
    for (Terms.Bucket bucket : attr_id_agg.getBuckets()) {
        SearchResult.AttrVo attrVo = new SearchResult.AttrVo();
        //id
        attrVo.setAttrId(bucket.getKeyAsNumber().longValue());
        //name
        String attr_name_agg = ((ParsedStringTerms) (bucket.getAggregations().get("attr_name_agg"))).getBuckets().get(0).getKeyAsString();
        attrVo.setAttrName(attr_name_agg);
        //values
        List<String> attr_value_agg_s = ((ParsedStringTerms) (bucket.getAggregations().get("attr_value_agg"))).getBuckets().stream().map(b -> {
            String keyAsString = b.getKeyAsString();
            return keyAsString;
        }).collect(Collectors.toList());
        attrVo.setAttrValue(attr_value_agg_s);

        attrVos.add(attrVo);
    }
    result.setAttrs(attrVos);


    //3 ===================================================================
    List<SearchResult.BrandVo> brandVos = new ArrayList<>();
    ParsedLongTerms brand_agg = response.getAggregations().get("brand_agg");
    for (Terms.Bucket bucket : brand_agg.getBuckets()) {
        SearchResult.BrandVo brandVo = new SearchResult.BrandVo();
        long brandId = bucket.getKeyAsNumber().longValue();
        String brand_img_agg = ((ParsedStringTerms) (bucket.getAggregations().get("brand_img_agg"))).getBuckets().get(0).getKeyAsString();
        String brand_name_agg = ((ParsedStringTerms) (bucket.getAggregations().get("brand_name_agg"))).getBuckets().get(0).getKeyAsString();

        brandVo.setBrandId(brandId);
        brandVo.setBrandImg(brand_img_agg);
        brandVo.setBrandName(brand_name_agg);
        brandVos.add(brandVo);
    }
    result.setBrands(brandVos);

    //4 catalog ===================================================================
    ParsedLongTerms catalog_agg = response.getAggregations().get("catalog_agg");
    List<SearchResult.CatalogVo> catalogVos = new ArrayList<>();
    List<? extends Terms.Bucket> buckets = catalog_agg.getBuckets();
    for (Terms.Bucket bucket : buckets) {
        SearchResult.CatalogVo vo = new SearchResult.CatalogVo();
        String keyAsString = bucket.getKeyAsString();
        vo.setCatalogId(Long.parseLong(keyAsString));
        //sub aggregation
        ParsedStringTerms catalog_name_agg = bucket.getAggregations().get("catalog_name_agg");
        String keyAsString1 = catalog_name_agg.getBuckets().get(0).getKeyAsString();
        vo.setCatalogName(keyAsString1);
        catalogVos.add(vo);
    }
    result.setCatalogs(catalogVos);

    //5: pagenation need to do it yourself  ===================================================================
    result.setPageNum(param.getPageNum());

    long total = hits.getTotalHits().value;
    result.setTotal(total);

    int totalPages = (int) (total % PRODUCT_PAGE_SIZE) == 0 ? (int) (total / PRODUCT_PAGE_SIZE) : (int) (total / PRODUCT_PAGE_SIZE + 1);
    result.setTotalPages(totalPages);
    //get from param

    return result;
}


private SearchRequest buildSearchRequest(SearchParam param) {
    SearchSourceBuilder builder = new SearchSourceBuilder();

    //1 search ====================================================================================================
    BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
    if (!StringUtils.isEmpty(param.getKeyword())) {
        //must
        boolQuery.must(QueryBuilders.matchQuery("skuTitle", param.getKeyword()));
    }
    if (param.getCatalog3Id() != null) {
        //filter#1
        boolQuery.filter(QueryBuilders.termQuery("catalogId", param.getCatalog3Id()));
    }
    //=====terms vs term： depending on list or value you are passing===
    if (param.getBrandId() != null && param.getBrandId().size() > 0) {
        //filter#2
        boolQuery.filter(QueryBuilders.termsQuery("brandId", param.getBrandId()));
    }
    //filter#3_attrs
    if (param.getAttrs() != null && param.getAttrs().size() > 0) {
        for (String attrStr : param.getAttrs()) {
            BoolQueryBuilder nestedBoolQuery = QueryBuilders.boolQuery();
            String[] s = attrStr.split("_");
            String attrId = s[0];
            String[] attrValues = s[1].split(":");
            nestedBoolQuery.must(QueryBuilders.termQuery("attrs.attrId", attrId));
            nestedBoolQuery.must(QueryBuilders.termQuery("attrs.attrValue", attrValues));
            //每一个都要生成一个nested query
            NestedQueryBuilder nestedQuery = QueryBuilders.nestedQuery("attrs", nestedBoolQuery, ScoreMode.None);
            boolQuery.filter(nestedQuery);
        }
    }
    //filter#4 has stock?
    if (param.getHasStock() != null)
        boolQuery.filter(QueryBuilders.termQuery("hasStock", param.getHasStock() == 1));
    //filter#5 price range (1_500 / _500 / 500_)
    if (!StringUtils.isEmpty(param.getSkuPrice())) {
        RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("skuPrice");
        String[] s = param.getSkuPrice().split("_");
        if (s.length == 2) {
            rangeQuery.gte(s[0]).lte(s[1]);
        } else if (s.length == 1) {
            if (param.getSkuPrice().startsWith("_")) {
                rangeQuery.lte(s[0]);
            } else {
                rangeQuery.gte(s[0]);
            }
        }
        boolQuery.filter(rangeQuery);
    }
    builder.query(boolQuery);


    //2 sort, highlight================================================================================================
    //sort
    if (!StringUtils.isEmpty(param.getSort())) {
        String sort = param.getSort();
        String[] s = sort.split("_");
        SortOrder asc = s[1].equalsIgnoreCase("asc") ? SortOrder.ASC : SortOrder.DESC;
        builder.sort(s[0], asc);
    }
    //pagenation
    builder.from((param.getPageNum() - 1) * PRODUCT_PAGE_SIZE);
    builder.size(PRODUCT_PAGE_SIZE);
    //highlight
    if (!StringUtils.isEmpty(param.getKeyword())) {
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.field("skuTitle");
        highlightBuilder.preTags("<b style='color:red'>");
        highlightBuilder.postTags("</b>");
        builder.highlighter(highlightBuilder);
    }
    System.out.println(builder.toString());

    //3 aggregation====================================================================================================
    //(1) brand_agg
    TermsAggregationBuilder brand_agg = AggregationBuilders.terms("brand_agg");
    brand_agg.field("brandId").size(20);
    //(1)-sub
    brand_agg.subAggregation(AggregationBuilders.terms("brand_name_agg").field("brandName").size(1));
    brand_agg.subAggregation(AggregationBuilders.terms("brand_img_agg").field("brandImg").size(1));
    builder.aggregation(brand_agg);
    //(2) catalog_agg
    TermsAggregationBuilder catalog_agg = AggregationBuilders.terms("catalog_agg");
    catalog_agg.field("catalogId").size(20);
    //(2)-sub
    catalog_agg.subAggregation(AggregationBuilders.terms("catalog_name_agg").field("catalogName").size(1));
    builder.aggregation(catalog_agg);
    //(3) attrs_agg
    NestedAggregationBuilder attr_agg = AggregationBuilders.nested("attr_agg", "attrs");
    TermsAggregationBuilder attr_id_agg = AggregationBuilders.terms("attr_id_agg").field("attrs.attrId");
    attr_id_agg.subAggregation(AggregationBuilders.terms("attr_name_agg").field("attrs.attrName").size(1));
    attr_id_agg.subAggregation(AggregationBuilders.terms("attr_value_agg").field("attrs.attrValue").size(20));
    attr_agg.subAggregation(attr_id_agg);
    builder.aggregation(attr_agg);


    System.out.println(builder.toString());
    SearchRequest searchRequest = new SearchRequest(new String[]{EsConstant.PRODUCT_INDEX}, builder);
    return searchRequest;
}
```

![](<../.gitbook/assets/image (103).png>)




















































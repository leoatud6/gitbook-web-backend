---
description: 'ELK: Elastic Search / Logstash / Kibana'
---

# Elastic Search

Lucene based

Search Engine : RESTful web

Space trade off time : need more memory / resource --> need to limit the runtime memory

ELK: elasticSearch  logstask kibana

#### sharding + cluster

![](<../.gitbook/assets/image (45).png>)

#### Reverse Indexed

use sentence/words to find IDs (multiple IDs)

keywords: Term Dictionary

* index ==> database
* type ==> table
* document ==> row
* field ==> column

### Jest (Java client for ES)

```bash
#properties file
server.port=8094 #this is predefine for java
spring.elasticsearch.jest.uris=http://es.gmall.com:9200

```

![](<../.gitbook/assets/image (46).png>)

```java
    @Autowired
    JestClient jestClient;

    public void saveSkuLsInfo(SkuLsInfo skuLsInfo){
        Index.Builder indexBuilder = new Index.Builder(skuLsInfo);
        indexBuilder.index("gmall1221_sku_info").type("_doc").id(skuLsInfo.getId());
        Index index = indexBuilder.build();
        try {
            jestClient.execute(index);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

![](<../.gitbook/assets/image (44).png>)






















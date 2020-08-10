[TOC]
# Elasticsearch

## 文档(Document)

#### 什么是文档？

文档就是类似数据库里面的一条长长的存储记录。文档是索引信息的基本单位。

文档被序列化为JSON格式，物理保存在一个索引中。

- 文档字段名：JSON格式由name/value pairs租车，对应的name就是文档字段名
- 文档字段类型：每个字段都有对应的字段类型：String、integer、long等，并支持数据&嵌套





## 入门案例

1. 分词器使用

   ik_samrt最粗粒度拆分

   ```json
   GET _analyze
   {
     "analyzer": "ik_smart",
     "text": "黄山黄河在我心中重千斤"
   }
   ```

   ![image-20200604170226542](https://gitee.com//kulalasmile/image/raw/master/img/image-20200604170226542.png)

   ik_max_word最细粒度拆分

   ```json
   GET _analyze
   {
     "analyzer": "ik_max_word",
     "text": "黄山黄河在我心中重千斤"
   }
   ```

   ![image-20200604170409391](https://gitee.com//kulalasmile/image/raw/master/img/image-20200604170409391.png)

2. 录入文档

   **PUT  /索引/_doc/id号**

   如果已经存在就全覆盖修改。:如果 只需要插入，不修改 着 在后面 加上_ create (这时候会提示已经存在),post 不能带_ create

   ```json
   PUT /test1/_doc/8
   {
     "name": "京东",
     "age": 23,
     "birthday":"1999-1-1"
   }
   ```

   ![image-20200604170647382](https://gitee.com//kulalasmile/image/raw/master/img/image-20200604170647382.png)

3. 修改

   **POST /索引/_update/id号**

   ```json
   POST /test1/_update/8
   {
     "doc": {
       "name": "京2东",
       "age": 23,
       "birthday":"1999-1-1"
     }
   }
   ```

   ![image-20200604171234072](https://gitee.com//kulalasmile/image/raw/master/img/image-20200604171234072.png)

4. 删除

   删除索引 **DELETE /索引**

   删除文档 **DELETE /索引/id号**

   ![image-20200604171542698](https://gitee.com//kulalasmile/image/raw/master/img/image-20200604171542698.png)

5. 查询

   查询所有

   ```json
   GET /test1/_search
   {
     "query": {
       "match_all": {}
     }
   }
   ```

   条件查询

   ```json
   GET /test1/_search
   {
     "query": {
       "match": {
         "name": "法外"
       }
     }
   }
   ```

   ![image-20200604171730682](https://gitee.com//kulalasmile/image/raw/master/img/image-20200604171730682.png)

   针对某个字段进行排序：

   ```json
   GET /test1/_search
   {
     "query": {
       "match": {
         "name": "法外"
       }
     },
     "sort": [
       {
         "age": {
           "order": "asc"
         }
       }
     ]
   }
   ```

   ![image-20200604171840083](https://gitee.com//kulalasmile/image/raw/master/img/image-20200604171840083.png)

   **分页查询：from：起始，size：查询总数**

   ```json
   GET /test1/_search
   {
     "query": {
       "match": {
         "name": "法外"
       }
     },
     "sort": [
       {
         "age": {
           "order": "asc"
         }
       }
     ],
     "from": 0,
     "size": 3
   }
   ```

   **布尔查询：**
   must（即and），所有条件都要符合 where id=1 and name=xxx

   ```json
   GET /test1/_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "match": {
               "name": "法外"
             }
           },
           {
             "match": {
               "age": "27"
             }
           }
         ]
       }
     }
   }
   ```

   should（即or），所有条件都要符合 where id = 1 or name = xx

   ```json
   GET /test1/_search
   {
     "query": {
       "bool": {
         "should": [
           {
             "match": {
               "name": "法外"
             }
           },
           {
             "match": {
               "age": "19"
             }
             
           }
         ]
       }
     }
   }
   ```

   must_not（即not），所有条件满足 where id is not null

   ```json
   GET /test1/_search
   {
     "query":{
       "bool": {
         "must_not": [
           {
             "match": {
               "name": "法外"
             }
           }
         ]
       }
     }
   }
   ```

   **过滤器filter，对数据进行过滤**

   gt：大于 gte：大于等于 lt：小于 lte：小于等于

   ```json
   GET /test1/_search
   {
     "query": {
       "bool": {
         "must": [
           {
             "match": {
               "name": "法外"
             }
           }
         ],
         "filter": [
           {
             "range": {
               "age": {
                 "gte": 0
               }
             }
           }
         ]
       }
     }
   }
   ```

   只显示指定结果

   ```json
   GET /test1/_search
   {
     "query": {
       "match_all": {}
     },
     "_source": ["name"]
   }
   ```

   ![image-20200605134949054](https://gitee.com//kulalasmile/image/raw/master/img/image-20200605134949054.png)

   聚合函数 求平均值和 总数 总量

   ```json
   GET /test1/_search
   {
     "query": {
       "match_all": {}
     },
     "aggs": {
       "sum": {
         "sum": {
           "field": "age"
         }
       },
       "total_count": {
         "value_count": {
           "field": "age"
         }
       },
       "pjz":{
         "avg": {
           "field": "age"
         }
       }
     }
   }
   
   ```

   ![image-20200605135809527](https://gitee.com//kulalasmile/image/raw/master/img/image-20200605135809527.png)

   分组查询

   ```json
   GET /test1/_search
   {
     "query": {
       "match_all": {}
     },
     "aggs": {
       "fz": {
         "terms": {
           "field": "age"
         }
       }
     }
   }
   ```

   ![image-20200605140012794](https://gitee.com//kulalasmile/image/raw/master/img/image-20200605140012794.png)

6. 查询所有索引

   ```
   GET /_cat/indices
   ```

   ![image-20200605133915923](https://gitee.com//kulalasmile/image/raw/master/img/image-20200605133915923.png)

7. 查看节点健康

   ？v 的意思 显示列出项 的title

   ?pretty 结果 json 格式化的方式输出

   ```
   GET /_cat/health?v
   ```

   ![image-20200605134029908](https://gitee.com//kulalasmile/image/raw/master/img/image-20200605134029908.png)

   



## SpringData Elasticsearch

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.5.RELEASE</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <groupId>org.example</groupId>
    <artifactId>elasticsearch-study</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
```



application.yml

```yaml
server:
  port: 8989

spring:
  data:
    elasticsearch:
      cluster-nodes: 127.0.0.1:9300
      cluster-name: my-application
```



实体类

```java
/**
 * 商品
 *
 * @author wangyufeng / wangyufeng@seerbigdata.com
 * @date 2020-06-05
 */
@Document(indexName = "item",type = "docs", shards = 1, replicas = 0)
public class Item {
    @Id
    private Long id;
    /**
     * 标题
     */
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String title;
    /**
     * 类别
     */
    @Field(type = FieldType.Keyword)
    private String category;
    /**
     * 品牌
     */
    @Field(type = FieldType.Keyword)
    private String brand;
    /**
     * 价格
     */
    @Field(type = FieldType.Double)
    private Double price;
    /**
     * 图片
     */
    @Field(index = false, type = FieldType.Keyword)
    private String images;

    public Item(){};

    public Item(Long id, String title, String category, String brand, Double price, String images) {
        this.id = id;
        this.title = title;
        this.category = category;
        this.brand = brand;
        this.price = price;
        this.images = images;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getCategory() {
        return category;
    }

    public void setCategory(String category) {
        this.category = category;
    }

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    public String getImages() {
        return images;
    }

    public void setImages(String images) {
        this.images = images;
    }

    @Override
    public String toString() {
        return "Item{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", category='" + category + '\'' +
                ", brand='" + brand + '\'' +
                ", price=" + price +
                ", images='" + images + '\'' +
                '}';
    }

}

```

> 映射

Spring Data通过注解来声明字段的映射属性，有下面的三个注解：

- @Document

  作用在类，标记实体类为文档对象，一般有四个属性

  - indexName：对应索引库名称
  - type：对应在索引库中的类型
  - shards：分片数量，默认5
  - replicas：副本数量，默认1

- `@Id` 作用在成员变量，标记一个字段作为id主键

- @Field

  作用在成员变量，标记为文档的字段，并指定字段映射属性：

  - type：字段类型，取值是枚举：FieldType
  - index：是否索引，布尔类型，默认是true
  - store：是否存储，布尔类型，默认是false
  - analyzer：分词器名称：ik_max_word



ItemRepository

```java
public interface ItemRepository extends ElasticsearchRepository<Item, Long> {

    /**
     * 找到之间的价格
     *
     * @param price1 price1
     * @param price2 price2
     * @return {@link List<Item>}
     */
    List<Item> findByPriceBetween(Double price1, Double price2);
}
```



测试创建索引

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class elaTest {
	@Test
    public void createIndex(){
        // 创建索引，会根据Item类的@Document注解信息来创建
        esTemplate.createIndex(Item.class);
        // 配置映射，会根据Item类中的id、Field等字段来自动完成映射
        esTemplate.putMapping(Item.class);
    }
}
```



增加

```java
	@Autowired
    ElasticsearchTemplate esTemplate;

    @Autowired
    private ItemRepository itemRepository; 	
	
	@Test
    public void testAdd(){
        Item item = new Item(1L,"小米手机","手机","小米",3488.00,"http:iamge.com");
        itemRepository.save(item);
    }
```



修改

```java
    @Test
    public void testUpdate(){
        Item item = new Item(1L,"redmi手机","手机","小米",3488.00,"http:iamge.com");
        itemRepository.save(item);
    }
```



批量新增

```java
    @Test
    public void indexList() {
        List<Item> list = new ArrayList<>();
        list.add(new Item(1L, "小米手机7", "手机", "小米", 3299.00, "http://image.leyou.com/13123.jpg"));
        list.add(new Item(2L, "坚果手机R1", "手机", "锤子", 3699.00, "http://image.leyou.com/13123.jpg"));
        list.add(new Item(3L, "华为META10", "手机", "华为", 4499.00, "http://image.leyou.com/13123.jpg"));
        list.add(new Item(4L, "小米Mix2S", "手机", "小米", 4299.00, "http://image.leyou.com/13123.jpg"));
        list.add(new Item(5L, "荣耀V10", "手机", "华为", 2799.00, "http://image.leyou.com/13123.jpg"));
        itemRepository.saveAll(list);
    }
```



删除

```java
    @Test
    public void testDelete(){
        itemRepository.deleteById(1L);
    }
```



根据id查询

```java
	@Test
    public void testQuery(){
        Optional<Item> optional = itemRepository.findById(2L);
        System.out.println(optional.get());
    }
```



按价格降序排序

```java
    @Test
    public void testFind(){
        Iterable<Item> price = this.itemRepository.findAll(Sort.by(Sort.Direction.DESC, "price"));
        price.forEach(item -> System.out.println(item));

    }
```



自定义范围查询

```java
 @Test
    public void queryByPriceBetween(){
        List<Item> priceBetween = this.itemRepository.findByPriceBetween(2000.00, 3500.00);
        priceBetween.forEach(item -> System.out.println(item));
    }
```



**高级查询**

条件查询

```java
	@Test
    public void testBaseQuery(){
        MatchQueryBuilder queryBuilder = QueryBuilders.matchQuery("title","小米");
        Iterable<Item> search = itemRepository.search(queryBuilder);
        search.forEach(System.out::println);
    }
```



自定义查询

基本的match query：

```java
	@Test
    public void testNativeQuery(){
        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
        queryBuilder.withQuery(QueryBuilders.matchQuery("title","小米"));
        Page<Item> items = this.itemRepository.search(queryBuilder.build());
        System.out.println("总数："+ items.getTotalElements());
        System.out.println("总页数"+ items.getTotalPages());

        items.forEach(System.out::println);
    }
```

NativeSearchQueryBuilder：Spring提供的一个查询条件构建器，帮助构建json格式的请求体

`Page`：默认是分页查询，因此返回的是一个分页的结果对象，包含属性：

- totalElements：总条数
- totalPages：总页数
- Iterator：迭代器，本身实现了Iterator接口，因此可直接迭代得到当前页的数据
- 其它属性：



分页查询

利用`NativeSearchQueryBuilder`可以方便的实现分页：

```java
 	@Test
    public void testNativeQueryPage(){
        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
        queryBuilder.withQuery(QueryBuilders.termQuery("category","手机"));
        queryBuilder.withPageable(PageRequest.of(0,3));
        Page<Item> items = this.itemRepository.search(queryBuilder.build());
        System.out.println("总条数" + items.getTotalElements());
        // 打印总页数
        System.out.println("总页数" + items.getTotalPages());
        // 每页大小
        System.out.println("每页大小" + items.getSize());
        // 当前页
        System.out.println("当前页" + items.getNumber());
        items.forEach(System.out::println);
    }
```



排序

```java
@Test
public void testNativeQuerySort(){
    NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
    queryBuilder.withQuery(QueryBuilders.matchQuery("category","手机"));
    queryBuilder.withSort(SortBuilders.fieldSort("price").order(SortOrder.DESC));
    Page<Item> items = this.itemRepository.search(queryBuilder.build());

    items.forEach(System.out::println);
}
```



聚合

```java
 	@Test
    public void testAgg(){
        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
        queryBuilder.withSourceFilter(new FetchSourceFilter(new String[]{""},null));
        queryBuilder.addAggregation(
                AggregationBuilders.terms("brands").field("brand"));
        AggregatedPage<Item> aggpage = (AggregatedPage<Item>) this.itemRepository.search(queryBuilder.build());
        StringTerms agg = (StringTerms) aggpage.getAggregation("brands");
        List<StringTerms.Bucket> buckets = agg.getBuckets();
        for (StringTerms.Bucket bucket : buckets){
            System.out.println(bucket.getKeyAsString());
            System.out.println(bucket.getDocCount());
        }
    }
```



嵌套聚合，求平均值

```java
	@Test
    public void testSubAgg(){
        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
        queryBuilder.withSourceFilter(new FetchSourceFilter(new String[]{""},null));
        queryBuilder.addAggregation(AggregationBuilders.terms("brands").field("brand").subAggregation(AggregationBuilders.avg("priceAvg").field("price")));
        AggregatedPage<Item> aggPage = (AggregatedPage<Item>) this.itemRepository.search(queryBuilder.build());
        StringTerms agg = (StringTerms) aggPage.getAggregation("brands");
        List<StringTerms.Bucket> buckets = agg.getBuckets();
        for (StringTerms.Bucket bucket : buckets){
            System.out.println(bucket.getKeyAsString() + ",共" + bucket.getDocCount() + "台");
            InternalAvg avg = (InternalAvg) bucket.getAggregations().asMap().get("priceAvg");
            System.out.println("平均售价:" + avg.getValue());

        }
    }
```


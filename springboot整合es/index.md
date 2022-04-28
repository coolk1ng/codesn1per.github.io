# SpringBoot整合ES


### 1. 添加依赖

```xml
				<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```

### 2. 配置客户端

```java
@Configuration
public class RestClientConfig extends AbstractElasticsearchConfiguration {

    @Value("${elasticsearch.host}")
    private String host;

    @Override
    @Bean
    public RestHighLevelClient elasticsearchClient() {
        final ClientConfiguration clientConfiguration = ClientConfiguration.builder()
                .connectedTo(host).build();
        return RestClients.create(clientConfiguration).rest();
    }
}
```

```yml
# ES主机端口
elasticsearch:
  host: 127.0.0.1:9200
```

### 3. 客户端对象

* ElasticsearchOperations
* RestHighLevelClient (推荐)

##### 1. ElasticsearchOperations

特点: 始终使用面向对象方式操作ES

相关注解:

```java
@Data
@Document(indexName = "product",createIndex = true)
public class Product {
    @Id
    private String id;
    @Field(type = FieldType.Keyword)
    private String title;
    @Field(type = FieldType.Double)
    private Double price;
    @Field(type = FieldType.Text)
    private String description;
}
```

* 创建doc

  ```java
  @Test
      public void testIndex(){
          Product product = new Product();
          product.setId(UUID.randomUUID().toString().replaceAll("-",""));
          product.setTitle("商品一号");
          product.setPrice(100.0);
          product.setDescription("我是一个100元的商品!!!!");
          elasticsearchOperations.save(product);
      }
  ```

* 查询单个doc

  ```java
  @Test
      public void testSearch(){
          Product product = elasticsearchOperations.get("c9c312e6196746ad8c9c9e5060f36ea8", Product.class);
          System.out.println(product);
      }
  ```

* 删除doc

  ```java
  @Test
      public void testDelete(){
          elasticsearchOperations.delete("c9c312e6196746ad8c9c9e5060f36ea8",Product.class);
      }
  ```

* 查询所有doc

  ```java
  @Test
      public void testFindAll(){
          SearchHits<Product> search = elasticsearchOperations.search(Query.findAll(), Product.class);
          for (SearchHit<Product> item : search) {
              System.out.println(item);
          }
      }
  ```

##### 2. RestHighLevelClient

* 创建索引和映射

  ```java
  @Test
      public void restHighLevelClient_test_indexAndMapping() throws IOException {
          CreateIndexRequest createIndexRequest = new CreateIndexRequest("product");
          createIndexRequest.mapping("{\n" +
                  "    \"properties\":{\n" +
                  "      \"title\":{\n" +
                  "        \"type\": \"keyword\"\n" +
                  "      },\n" +
                  "      \"price\":{\n" +
                  "        \"type\":\"double\"\n" +
                  "      },\n" +
                  "      \"description\":{\n" +
                  "        \"type\":\"text\"\n" +
                  "      }\n" +
                  "    }\n" +
                  "  }", XContentType.JSON);
          restHighLevelClient.indices().create(createIndexRequest, RequestOptions.DEFAULT);
      }
  }
  ```

* 删除索引

  ```java
  @Test
      public void restHighLevelClient_test_deleteMapping() throws IOException {
          AcknowledgedResponse acknowledgedResponse = restHighLevelClient.indices()
                  .delete(new DeleteIndexRequest("product"), RequestOptions.DEFAULT);
          System.out.println(acknowledgedResponse.isAcknowledged());
      }
  ```



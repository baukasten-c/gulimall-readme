# SpringBoot集成Elasticsearch8.6

1、SpringBoot集成Elasticsearch8.6

1-1、参考 https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/8.6/installation.html，导入依赖

```java
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
    <version>8.6.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.3</version>
</dependency>
```

1-2、参考 https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/8.6/connecting.html，编写配置类

```java
@Configuration
public class GulimallSearchConfig {
    @Bean
    public ElasticsearchClient elasticsearchClient(){
        RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200)).build();
        ElasticsearchTransport transport = new RestClientTransport(restClient, new JacksonJsonpMapper());
        return new ElasticsearchClient(transport);
    }
}
```

1-3、测试

```java
@Autowired
private ElasticsearchClient elasticsearchClient;
@Test
public void contextLoads() {
    System.out.println(elasticsearchClient);
}
```

1-4、报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694078380198-5e5dacc2-6173-4e1d-ac43-84ff9b448b72.png)

1-5、新增依赖

```java
<dependency>
    <groupId>jakarta.json</groupId>
    <artifactId>jakarta.json-api</artifactId>
    <version>2.0.1</version>
</dependency>
```

1-6、重新进行测试，再次报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694078520746-c0920d39-80d5-4126-a1b3-2e370276f052.png)

1-7、因为导入的 gulimall-common 中有数据库依赖，需要禁用数据源自动配置

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
```

1-8、再次测试，成功

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694078717573-f1d703fc-6233-4555-9e5d-2d974248dcfd.png)

2、测试存储数据到 es

2-1、参考 https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/8.6/indexing.html，编写代码

```java
@Autowired
private ElasticsearchClient elasticsearchClient;
@Test
public void indexData() throws IOException {
    User user = new User("张三", "男", 20);
    IndexResponse indexResponse = elasticsearchClient.index(i -> i
            .index("users")
            .id("1")
            .document(user));
    System.out.println(indexResponse);
}
@Data
@AllArgsConstructor
@NoArgsConstructor
class User{
    private String userName;
    private String gender;
    private Integer age;
}
```

2-2、报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694078870786-d0cce24f-fb5c-4643-b080-05270238b349.png)

2-3、将配置类 RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200)).build(); 中的 localhost 换为虚拟机ip

2-4、再次进行测试，成功

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694078999132-5f7f83aa-e254-4aff-a8e9-59768af9a135.png)

不必事先手动创建名为"users"的索引，如果该索引已经存在，它会将数据添加到该索引中，如果索引不存在，则会尝试创建该索引并添加数据

3、复杂查询

3-1、参考 https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/8.6/searching.html，查询数据

```java
@Autowired
private ElasticsearchClient elasticsearchClient;
@Test
public void searchData() throws IOException {
    SearchResponse<Bank> response = elasticsearchClient.search(s -> s
            .index("bank")
            .query(q -> q
                    .match(t -> t
                            .field("address")
                            .query("mill")
                    )
            ), Bank.class
    );
    System.out.println(response);
}
@Data
class Bank {
    private long accountNumber;
    private String address;
    private int age;
    private long balance;
    private String city;
    private String email;
    private String employer;
    private String firstname;
    private String gender;
    private String lastname;
    private String state;
}
```

3-2、报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694084484050-e3fdd60d-d774-422b-8339-f0dca57a0146.png)

3-3、将 Bank 变为静态类 static class Bank()，再次测试，报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694084490381-097e7df8-721e-4620-854d-6762c33829be.png)

3-4、在 accountNumber 上添加注解 @JsonProperty("account_number")，将 "account_number" 字段映射到正确的属性名

3-5、再次测试，成功

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694084503184-ab6a2683-24ab-44c6-90ba-5e9b1ca40a6f.png)

4、聚合查询

4-1、参考 https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/8.6/aggregations.html，编写代码

```java
@Autowired
private ElasticsearchClient elasticsearchClient;
@Test
public void searchData() throws IOException {
    SearchResponse<Bank> response = elasticsearchClient.search(s -> s
            .index("bank")
            .query(q -> q
                    .match(t -> t
                            .field("address")
                            .query("mill")
                    )
            )
            .aggregations("ageAgg", a -> a
                    .terms(h -> h
                            .field("age")
                            .size(10)
                    )
            )
            .aggregations("balanceAvg", a -> a
                    .avg(h -> h
                            .field("balance")
                    )
            ), Bank.class
    );
    System.out.println(response);
    List<Hit<Bank>> hits = response.hits().hits();
    for (Hit<Bank> hit: hits) {
        Bank bank = hit.source();
        System.out.println("bank：" + bank);
    }
    List<LongTermsBucket> ageAgg = response.aggregations().get("ageAgg").lterms().buckets().array();
    for (LongTermsBucket bucket: ageAgg) {
        System.out.println("年龄：" + bucket.key() + "===>" + bucket.docCount());
    }
    AvgAggregate balanceAvg = response.aggregations().get("balanceAvg").avg();
    System.out.println("平均薪资：" + balanceAvg.value());
}

@Data
static class Bank {
    @JsonProperty("account_number")
    private long accountNumber;
    private String address;
    private int age;
    private long balance;
    private String city;
    private String email;
    private String employer;
    private String firstname;
    private String gender;
    private String lastname;
    private String state;
}
```

4-2、测试，成功打印

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694084516682-c3b67f2e-1a2f-45cd-a333-c26c35edfafe.png)

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694084980912-15bb4515-d651-4a4c-8130-ed1173e36b0d.png)
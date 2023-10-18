# SpringCache

1、使用示例

```java
@Cacheable({"category"})
@Override
public List<CategoryEntity> getLevel1Categorys() {
    List<CategoryEntity> categoryEntities = baseMapper.selectList(new QueryWrapper<CategoryEntity>().eq("parent_cid", 0));
    return categoryEntities;
}
```

1-1、@Cacheable 代表当前方法的结果需要缓存。如果缓存中有，不调用方法，缓存中没有才调用方法。最后将方法的结果放入缓存

1-2、@Cacheable({"category"}) 每一个需要缓存的数据都需要指定缓存的分区(按照业务类型分)。存储同一类型的数据，都可以指定为同一分区，分区名默认就是缓存的前缀

1-3、缓存的key值(默认生成)：缓存分区名 : : 缓存的名字(默认SimpleKey[])，如：category : : SimpleKey[]

1-4、缓存的value值：默认使用 jdk 序列化机制，将序列化的数据存到 redis 中

1-5、缓存默认过期时间 ttl：-1，即永不过期，不符合缓存设计规范

2、自定义操作

2-1、指定缓存的key值：使用 SpEL 表达式

参考：https://docs.spring.io/spring-framework/reference/integration/cache/annotations.html#cache-spel-context

```java
@Cacheable(value = {"category"}, key = "#root.method.name")
```

2-2、指定缓存数据的存活时间：配置文档中修改存活时间

```java
spring:
  cache:
    type: redis
    redis:
      time-to-live: 3600000 #以毫秒为单位
```

2-3、指定缓存数据为json格式：自定义配置

3、相关注解

3-1、@Cacheable：触发将数据存储到缓存的操作

3-2、@CacheEvic：触发将数据从缓存中删除的操作(失效模式)

​         @CacheEvict(value = "category", allEntries = true)：指定删除某个分区下的所有数据 

3-3、@CachePut：不影响方法执行更新缓存(双写模式，需要有返回值)

3-4、@Caching：组合以上多个操作

```java
@Caching(evict = {
         @CacheEvict(value = "category", key = "'getLevel1Categorys'"),
         @CacheEvict(value = "category", key = "'getCatalogJson'")
})
```

3-5、@CacheConfig：在类级别共享缓存的相同配置

4、SpringCache避免大并发读情况下的缓存失效问题

4-1、缓存穿透：大量并发同时查询一个null数据。解决方案：缓存空数据

```java
spring:
  cache:
    type: redis
    redis:
      cache-null-values: true
```

4-2、缓存击穿：大量并发同时查询一个正好过期的数据。解决方案：加锁 ，默认无锁，可以使用sync = true来添加本地锁

```java
@Cacheable(value = {"category"}, key = "#root.method.name", sync = true)
```

4-3、 缓存雪崩：大量的key同时过期。解决：使用随机时间、过期时间

5、总结：常规数据(读多写少，对即时性、一致性要求不高的数据)，完全可以使用Spring-Cache
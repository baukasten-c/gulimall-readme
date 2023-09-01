# Jackson格式化失效

1、在 Nacos 中配置 Jackson 格式化时间

```java
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: Asia/Shanghai
```

2、发现时间并没有被格式化，查询资料后发现 Jackson 对 LocalDateTime 类型的数据不起作用，考虑到很少用到时间，因此只在实体类上相关变量上添加注解 @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Shanghai")

3、重启后，发现时间数据成功格式化
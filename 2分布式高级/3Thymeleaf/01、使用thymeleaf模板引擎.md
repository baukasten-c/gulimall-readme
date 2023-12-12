# 使用thymeleaf模板引擎

1、引入依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2、Nacos 中关闭 thymeleaf 缓存

```
spring:
  thymeleaf:
    cache: false
```

3、静态资源放入 static 文件夹，页面放入 templates 文件夹，可以按照路径直接访问

4、在 http://localhost:10000 下可以成功查看页面

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694248457190-f3305d67-a9fc-4b3f-a47b-93db8e83acb8.png)

5、热部署

5-1、方法一：引入 spring-boot-devtools 依赖，修改页面后使用 Ctrl + Shift + 9 重新自动编译

5-2、方法二：使用 JRebel 插件
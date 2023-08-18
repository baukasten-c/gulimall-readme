# 修改OSS访问路径

1、在网关中添加 gulimall-third-party 的路由

```java
- id: third_party_route
  uri: lb://gulimall-third-party
  predicates:
    - Path=/api/thirdparty/**
  filters:
    - RewritePath=/api/?(?<segment>.*), /$\{segment}
```

2、重启网关模块，访问 http://localhost:88/api/thirdparty/oss/policy 报错404

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1692240212481-4f0b815e-e6f5-49bb-a7f9-d7de67b53a24.png)

3、在 OssController 中添加 @RequestMapping("thirdparty") 即可

4、为什么视频中不需要加 @RequestMapping("thirdparty") 呢？

因为：视频中的 filters 为 - RewritePath=/api/thirdparty/(?<segment>.*), /$\{segment}，即将 /api/thirdparty/ 都重写为空，而我的代码只去掉了 /api
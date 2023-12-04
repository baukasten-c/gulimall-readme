# Docker部署Seata分布式事务

1、使用 @Transactional 即本地事务，在微服务架构中进行远程调用时会造成一些问题：远程调用方法顺利执行，而主方法因为网络原因或其他远程调用方法错误导致回滚，执行的远程调用方法并不会随之回滚，因此考虑使用分布式事务

2、Docker部署Seata

2-1、参考 https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E，根据 Spring Boot 和 Spring Cloud 的版本拉取对应的 Seata 版本镜像

```shell
docker pull seataio/seata-server:1.6.1
```

2-2、创建临时容器获取 resources 目录文件

```shell
docker run -d -p 8091:8091 -p 7091:7091  --name seata-server seataio/seata-server:1.6.1
docker cp seata-server:/seata-server/resources /mydata/seata
```

2-3、参考 application.example.yml 文件修改 application.yml 配置

```java
server:
  port: 7091
spring:
  application:
    name: seata-server
logging:
  config: classpath:logback-spring.xml
  file:
    path: ${user.home}/logs/seata
  extend:
    logstash-appender:
      destination: 127.0.0.1:4560
    kafka-appender:
      bootstrap-servers: 127.0.0.1:9092
      topic: logback_to_logstash
console:
  user:
    username: seata
    password: seata
seata:
  config:
    type: nacos
    nacos:
      server-addr: nacos启动ip:8848
      group: SEATA_GROUP
      username: nacos
      password: nacos
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: nacos启动ip:8848
      group: SEATA_GROUP
      cluster: default
      username: nacos
      password: nacos
  security:
    secretKey: SeataSecretKey0c382ef121d778043159209298fd40bf3850a017
    tokenValidityInMilliseconds: 1800000
    ignore:
      urls: /,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/api/v1/auth/login
```

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1701242057182-8e00afce-bf3e-4a38-ae51-e19055908658.png)

2-4、删除临时容器

```shell
docker stop seata-server
docker rm seata-server
```

2-5、挂载配置文件并启动 seata-server 实例

```shell
docker run --name seata-server \
	--restart=always \
    -p 8091:8091 \
    -p 7091:7091 \
    -e SEATA_IP=指定seata-server启动的IP(该IP用于向注册中心注册时使用) \
    -v /mydata/seata/resources:/seata-server/resources \
    seataio/seata-server:1.6.1
```

2-6、注册成功

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1701242317174-60ac8acb-45b9-4114-9106-311f8fc76653.png)

2-7、模块启动后连接失败，因为默认连接本地 seata

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1701251060646-51393757-7ef3-46bf-9f52-eec1417b26b1.png)

2-8、修改模块配置，重启模块后可以正常连接

```java
seata:
  service:
    grouplist:
      default: 虚拟机ip:8091
```
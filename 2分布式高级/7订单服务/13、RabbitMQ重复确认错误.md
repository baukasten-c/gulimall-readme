# RabbitMQ重复确认错误

1、项目启动后发现报错：

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1701660308894-1e5c7308-b2e3-411b-892f-42cbe9eb74f8.png)

2、错误原因：在 application.yml 配置文件中配置了手动确认模式，但配置失效，被注解注入的SimpleRabbitListenerContainerFactory 覆盖，其默认使用自动确认模式，而项目中又存在手动确认的代码，导致信息被重复确认

3、手动模式配置失败原因：未设置监听容器类型，默认为 simple，使用 SimpleRabbitListenerContainerFactory，且下面的 direct 配置不生效

```java
spring:
  rabbitmq:
    listener:
      direct:
        acknowledge-mode: manual 
```

4、解决方法：

4-1、方法一：使用默认配置

```java
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: manual
```

4-2、方法二：设置监听容器类型

```java
spring:
  rabbitmq:
    listener:
      type: direct
      direct:
        acknowledge-mode: manual 
```

4-3、方法三：在消费者类设置模式为手动确认

```java
@RabbitListener(queues = "xxxx", ackMode = "MANUAL")
```


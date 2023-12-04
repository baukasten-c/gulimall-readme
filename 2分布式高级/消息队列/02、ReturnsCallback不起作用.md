# ReturnsCallback不起作用

1、自定义 RabbitTemplate 解决循环依赖问题后，进行测试，发现投递失败信息确实并未进入队列，但是却并没有回调方法

```java
@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
    this.rabbitTemplate = rabbitTemplate;
    rabbitTemplate.setMessageConverter(messageConverter());
    initRabbitTemplate();
    return rabbitTemplate;
}
```

2、对比 RabbitTemplateConfigurer 中的 configure()，找出自定义 RabbitTemplate 与自动配置的 RabbitTemplate 有什么区别

```java
public void configure(RabbitTemplate template, ConnectionFactory connectionFactory) {
    PropertyMapper map = PropertyMapper.get();
    template.setConnectionFactory(connectionFactory);
    if (this.messageConverter != null) {
        template.setMessageConverter(this.messageConverter);
    }
    template.setMandatory(determineMandatoryFlag());
    RabbitProperties.Template templateProperties = this.rabbitProperties.getTemplate();
    if (templateProperties.getRetry().isEnabled()) {
        template.setRetryTemplate(new RetryTemplateFactory(this.retryTemplateCustomizers)
                .createRetryTemplate(templateProperties.getRetry(), RabbitRetryTemplateCustomizer.Target.SENDER));
    }
    map.from(templateProperties::getReceiveTimeout).whenNonNull().as(Duration::toMillis)
            .to(template::setReceiveTimeout);
    map.from(templateProperties::getReplyTimeout).whenNonNull().as(Duration::toMillis)
            .to(template::setReplyTimeout);
    map.from(templateProperties::getExchange).to(template::setExchange);
    map.from(templateProperties::getRoutingKey).to(template::setRoutingKey);
    map.from(templateProperties::getDefaultReceiveQueue).whenNonNull().to(template::setDefaultReceiveQueue);
}
```

3、发现比自定义 RabbitTemplate 多一步 setMandatory()

4、mandatory：交换器无法根据自身类型和路由键找到一个符合条件的队列时的处理方式

​	  true：RabbitMQ会调用Basic.Return命令将消息返回给生产者

​	  false：RabbitMQ会把消息直接丢弃

5、虽然在 Nacos 配置文件中配置了 mandatory 为 true，但如果不手动读取配置，还是会默认将错误消息直接丢弃

```java
spring:
  rabbitmq:
    publisher-returns: true #开启发送端抵达Queue队列确认
    template:
      mandatory: true #异步确认优先回调returnConfirm
```

6、configure 读取配置文件进行配置，但因为我们确定 mandatory 为 true，可以省略读取步骤

```java
template.setMandatory(determineMandatoryFlag());

private RabbitProperties rabbitProperties;
public RabbitTemplateConfigurer(RabbitProperties rabbitProperties) {
    Assert.notNull(rabbitProperties, "RabbitProperties must not be null");
    this.rabbitProperties = rabbitProperties;
}
private boolean determineMandatoryFlag() {
    Boolean mandatory = this.rabbitProperties.getTemplate().getMandatory();
    return (mandatory != null) ? mandatory : this.rabbitProperties.isPublisherReturns();
}
```

7、修改自定义 RabbitTemplate 后，重新进行测试，发现可以正常回调方法

```java
@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
    this.rabbitTemplate = rabbitTemplate;
    rabbitTemplate.setMessageConverter(messageConverter());
    initRabbitTemplate();
    return rabbitTemplate;
}
```


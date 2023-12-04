# RabbitTemplate报错循环依赖

1、RabbitMQ配置类

```java
@Configuration
public class MyRabbitConfig {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
    @PostConstruct
    public void initRabbitTemplate() {
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                System.out.println("confirm：correlationData[" + correlationData + "]=>ack:[" + ack +  "]=>cause:[" + cause + "]");
            }
        });
        rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
            @Override
            public void returnedMessage(ReturnedMessage returnedMessage) {
                Message message = returnedMessage.getMessage();
                int replyCode = returnedMessage.getReplyCode();
                String replyText = returnedMessage.getReplyText();
                String exchange = returnedMessage.getExchange();
                String routingKey = returnedMessage.getRoutingKey();
                System.out.println("Fail Message[" + message + "]=>replyCode[" + replyCode + "]" +
                        "=>replyText[" + replyText + "]=>exchange[" + exchange + "]=>routingKey[" + routingKey + "]");
            }
        });
    }
}
```

2、模块启动时报错循环依赖

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699422580631-9027f11d-3f4a-4edc-a9f2-950b2212c546.png)

3、根据报错原因，在 RabbitAutoConfiguration 类的 rabbitTemplate() 和 rabbitTemplateConfigurer() 处添加断点

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699432672296-d5a55cb4-fa4b-4865-8da6-ed6dc4367953.png)

4、循环依赖产生原因

4-1、通常情况下，Spring会根据Bean之间的依赖关系来初始化它们

4-2、初始化配置类 MyRabbitConfig 时，发现依赖于 RabbitTemplate

4-3、在自动配置类 RabbitAutoConfiguration 中初始化 RabbitTemplate 时，发现依赖于 RabbitTemplateConfigurer

```java
@AutoConfiguration
@ConditionalOnClass({RabbitTemplate.class, Channel.class})
@EnableConfigurationProperties({RabbitProperties.class})
@Import({RabbitAnnotationDrivenConfiguration.class, RabbitStreamConfiguration.class})
public class RabbitAutoConfiguration {
    @Configuration(proxyBeanMethods = false)
    @Import({RabbitConnectionFactoryCreator.class})
    protected static class RabbitTemplateConfiguration {
        @Bean
        @ConditionalOnMissingBean
        public RabbitTemplateConfigurer rabbitTemplateConfigurer(RabbitProperties properties, ObjectProvider<MessageConverter> messageConverter, ObjectProvider<RabbitRetryTemplateCustomizer> retryTemplateCustomizers) {
            RabbitTemplateConfigurer configurer = new RabbitTemplateConfigurer(properties);
            configurer.setMessageConverter((MessageConverter)messageConverter.getIfUnique());
            configurer.setRetryTemplateCustomizers((List)retryTemplateCustomizers.orderedStream()
                                                   .collect(Collectors.toList()));
            return configurer;
        }
        @Bean
        @ConditionalOnSingleCandidate(ConnectionFactory.class)
        @ConditionalOnMissingBean({RabbitOperations.class})
        public RabbitTemplate rabbitTemplate(RabbitTemplateConfigurer configurer, ConnectionFactory connectionFactory) {
            RabbitTemplate template = new RabbitTemplate();
            configurer.configure(template, connectionFactory);
            return template;
        }
    }
}
```

4-4、在自动配置类 RabbitAutoConfiguration 中初始化 RabbitTemplateConfigurer 时，发现依赖于 MyRabbitConfig 的 MessageConverter，并导致循环依赖

5、三种解决方法

5-1、将自定义 MessageConverter 与 RabbitTemplate 放入两个不同的配置文件中

```java
@Configuration
public class MyRabbitConfig {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @PostConstruct
    public void initRabbitTemplate() {
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                System.out.println("confirm：correlationData[" + correlationData + "]=>ack:[" + ack +  "]=>cause:[" + cause + "]");
            }
        });
        rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
            @Override
            public void returnedMessage(ReturnedMessage returnedMessage) {
                Message message = returnedMessage.getMessage();
                int replyCode = returnedMessage.getReplyCode();
                String replyText = returnedMessage.getReplyText();
                String exchange = returnedMessage.getExchange();
                String routingKey = returnedMessage.getRoutingKey();
                System.out.println("Fail Message[" + message + "]=>replyCode[" + replyCode + "]" +
                        "=>replyText[" + replyText + "]=>exchange[" + exchange + "]=>routingKey[" + routingKey + "]");
            }
        });
    }
}
@Configuration
public class MyMessageConverterConfig {
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

5-2、使用静态内部类

```java
@Configuration
public class MyRabbitConfig {
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
    @Component
    public static class rabbitTemplateConfiguration {
        @Autowired
        private RabbitTemplate rabbitTemplate;
        @PostConstruct
        public void initRabbitTemplate() {
            rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
                @Override
                public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                    System.out.println("confirm：correlationData[" + correlationData + "]=>ack:[" + ack +  "]=>cause:[" + cause + "]");
                }
            });
            rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
                @Override
                public void returnedMessage(ReturnedMessage returnedMessage) {
                    Message message = returnedMessage.getMessage();
                    int replyCode = returnedMessage.getReplyCode();
                    String replyText = returnedMessage.getReplyText();
                    String exchange = returnedMessage.getExchange();
                    String routingKey = returnedMessage.getRoutingKey();
                    System.out.println("Fail Message[" + message + "]=>replyCode[" + replyCode + "]" +
                            "=>replyText[" + replyText + "]=>exchange[" + exchange + "]=>routingKey[" + routingKey + "]");
                }
            });
        }
    }
}
```

5-3、自定义 RabbitTemplate

```java
@Configuration
public class MyRabbitConfig {
    private RabbitTemplate rabbitTemplate;
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        this.rabbitTemplate = rabbitTemplate;
        rabbitTemplate.setMessageConverter(messageConverter());
        rabbitTemplate.setMandatory(true);
        initRabbitTemplate();
        return rabbitTemplate;
    }
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
    public void initRabbitTemplate() {
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                System.out.println("confirm：correlationData[" + correlationData + "]=>ack:[" + ack +  "]=>cause:[" + cause + "]");
            }
        });
        rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
            @Override
            public void returnedMessage(ReturnedMessage returnedMessage) {
                Message message = returnedMessage.getMessage();
                int replyCode = returnedMessage.getReplyCode();
                String replyText = returnedMessage.getReplyText();
                String exchange = returnedMessage.getExchange();
                String routingKey = returnedMessage.getRoutingKey();
                System.out.println("Fail Message[" + message + "]=>replyCode[" + replyCode + "]" +
                        "=>replyText[" + replyText + "]=>exchange[" + exchange + "]=>routingKey[" + routingKey + "]");
            }
        });
    }
}
```


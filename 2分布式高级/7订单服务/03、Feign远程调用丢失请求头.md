# Feign远程调用丢失请求头

1、在购物车进行结算时，发现页面报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699609973220-6bc91117-bae4-4df4-96e8-54bd447b8b8f.png)

2、根据提示，发现是远程查询所有购物项信息( cartFeignService.getUserCartItems()) 时出问题

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699610324481-94679500-aff2-4db2-8495-b1b66b06baca.png)

3、在方法中，虽然当前用户一定已经登录，但是 userInfoTo 获取到的是临时用户信息，无法根据 userId 获取 redis 键值，报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699610261303-6b906272-8860-40b5-94d2-6eed59e341bd.png)

4、发生错误的原因：

4-1、在远程查询所有购物项信息时点击 step into 单步执行

```java
List<OrderItemVo> items = cartFeignService.getUserCartItems();
```

4-2、进入 ReflectiveFeign 类的 invoke() 方法，使用代理对象实际执行的方法

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699841876814-6dc6907c-2058-4ff2-a2db-6601c5e6493d.png)

4-3、进入 SynchronousMethodHandler 的 invoke() 方法，首先创建了一个新的请求模板

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699842125129-41f1d3e5-f1b9-462e-85df-3cfcdec80c80.png)

4-4、随后调用 SynchronousMethodHandler 的 executeAndDecode() 方法对模板进行处理

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699842423720-931259f4-8b58-4300-a351-ab85eec4d0e3.png)

4-5、在 SynchronousMethodHandler 的 targetRequest() 方法中，遍历所有请求拦截器的迭代器来包装新模板，实际调试中却发现：包装后的模板与新模板并没有什么区别

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699843045626-fd8b8bbf-74ca-4999-93f0-e21641fb2ff2.png)

4-6、在先前的设计中，当前用户的登录信息被存放在cookie中，每次请求都会携带

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699842769050-71a8fd62-daf7-459e-8d69-1646b5c61096.png)

4-7、cart 模块的拦截器中，就是通过 cookie 里存放的 sessionId 来判断用户是否登录的。现在请求头中不携带任何数据，无法获取登录用户信息，会被认为没有登录，导致后续的错误

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699842761653-d24abf7e-3861-4f10-926a-2c9a7dafbcb2.png)

5、解决措施：

5-1、在 order 模块创建一个拦截器，在新请求模板中添加 cookie

```java
@Configuration
public class MyFeignConfig {
    @Bean
    public RequestInterceptor requestInterceptor(){
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate requestTemplate) {
                ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                HttpServletRequest request = requestAttributes.getRequest();
                String cookie = request.getHeader("Cookie");
                requestTemplate.header("Cookie", cookie);
            }
        };
    }
}
```

5-2、验证：模板中有请求头数据，请求中有 sessionId

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699842526372-5e8c7231-6c68-43bf-93e2-7c4463fa04c3.png)

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699845824223-6dd86b64-1fc8-497b-8615-b39d99e454ed.png)


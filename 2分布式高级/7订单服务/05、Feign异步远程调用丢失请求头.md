# Feign异步远程调用丢失请求头

1、为了提高代码执行效率，将获取订单确认页所需数据相关代码改为异步形式

```java
@Override
public OrderConfirmVo confirmOrder() throws ExecutionException, InterruptedException {
    OrderConfirmVo orderConfirmVo = new OrderConfirmVo();
    MemberRespTo loginUser = OrderInterceptor.loginUser.get();
    CompletableFuture<Void> addressFuture = CompletableFuture.runAsync(() -> {
        List<MemberAddressVo> address = memberFeignService.getAddress(loginUser.getId());
        orderConfirmVo.setAddress(address);
    }, executor);
    CompletableFuture<Void> itemsFuture = CompletableFuture.runAsync(() -> {
        List<OrderItemVo> items = cartFeignService.getUserCartItems();
        orderConfirmVo.setItems(items);
    }, executor);
    orderConfirmVo.setIntegration(loginUser.getIntegration());
    CompletableFuture.allOf(addressFuture, itemsFuture).get();
    return orderConfirmVo;
}
```

2、重启运行后报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699858023198-68ca4874-e773-4a37-adf7-24aa1000208e.png)

3、单步调试后发现错误原因是：没有在当前线程的上下文中获取请求的相关属性

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699858095975-1578e359-4d35-4802-888b-d8ece9e8f4f8.png)

4、由代码可知，请求信息是从requestAttributesHolder 中获取的，而 requestAttributesHolder 类型为 ThreadLocal，即本地线程。异步任务直接使用线程池里的线程去执行方法，而不是继续使用原有线程，导致无法继续使用原线程中的数据

```java
public abstract class RequestContextHolder {
    private static final ThreadLocal<RequestAttributes> requestAttributesHolder = new NamedThreadLocal("Request attributes");
    private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder = new NamedInheritableThreadLocal("Request context");
    @Nullable
    public static RequestAttributes getRequestAttributes() {
        RequestAttributes attributes = (RequestAttributes)requestAttributesHolder.get();
        if (attributes == null) {
            attributes = (RequestAttributes)inheritableRequestAttributesHolder.get();
        }
        return attributes;
    }
}
```

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1699860643365-0358a449-ff82-4070-8743-5188fba05fac.png)

5、解决方法：

5-1、在每次异步任务执行前，向线程中共享之前的请求数据

```java
RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
CompletableFuture<Void> addressFuture = CompletableFuture.runAsync(() -> {
    RequestContextHolder.setRequestAttributes(requestAttributes);
    List<MemberAddressVo> address = memberFeignService.getAddress(loginUser.getId());
    orderConfirmVo.setAddress(address);
}, executor);
CompletableFuture<Void> itemsFuture = CompletableFuture.runAsync(() -> {
    RequestContextHolder.setRequestAttributes(requestAttributes);
    List<OrderItemVo> items = cartFeignService.getUserCartItems();
    orderConfirmVo.setItems(items);
}, executor);
```

5-2、使用 InheritableThreadLocal 类

5-2-1、ThreadLocal 的子类 InheritableThreadLocal 可以使让子线程继承父线程的信息。根据代码，当参数 inheritable 为 true 时，子线程可以共享父线程信息，系统默认 inheritable 为 false

```java
public abstract class RequestContextHolder {
    private static final ThreadLocal<RequestAttributes> requestAttributesHolder = new NamedThreadLocal("Request attributes");
    private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder = new NamedInheritableThreadLocal("Request context");
    public static void setRequestAttributes(@Nullable RequestAttributes attributes, boolean inheritable) {
        if (attributes == null) {
            resetRequestAttributes();
        } else if (inheritable) {
            inheritableRequestAttributesHolder.set(attributes);
            requestAttributesHolder.remove();
        } else {
            requestAttributesHolder.set(attributes);
            inheritableRequestAttributesHolder.remove();
        }
    }
}
```

5-2-2、因此，可以通过语句，在异步调用前将父线程中的请求信息绑定给子线程

```java
RequestContextHolder.setRequestAttributes(RequestContextHolder.getRequestAttributes(), true);
CompletableFuture<Void> addressFuture = CompletableFuture.runAsync(() -> {
    List<MemberAddressVo> address = memberFeignService.getAddress(loginUser.getId());
    orderConfirmVo.setAddress(address);
}, executor);
CompletableFuture<Void> itemsFuture = CompletableFuture.runAsync(() -> {
    List<OrderItemVo> items = cartFeignService.getUserCartItems();
    orderConfirmVo.setItems(items);
}, executor);
```


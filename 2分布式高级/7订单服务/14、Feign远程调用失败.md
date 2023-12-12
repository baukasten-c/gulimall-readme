# Feign远程调用失败

1、队列中的信息一直没有被确认，被重复发送给消费者

2、检查代码，发现是解锁库存方法中有错误，导致信息一直没有确认

```java
@RabbitHandler
public void handleStockLockedRelease(Long detailId, Message message, Channel channel) throws IOException {
    try{
        wareSkuService.unlockStock(detailId);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false); 
    }catch(Exception e){
        channel.basicReject(message.getMessageProperties().getDeliveryTag(), true); 
    }
}
```

3、检查解锁库存方法，发现是远程调用 order 模块时出现问题

```java
public void unlockStock(Long detailId){
    WareOrderTaskDetailEntity taskDetailEntity = wareOrderTaskDetailService.getById(detailId);
    if(taskDetailEntity != null){
        WareOrderTaskEntity taskEntity = wareOrderTaskService.getById(taskDetailEntity.getTaskId());
        String orderSn = taskEntity.getOrderSn();
        R r = orderFeignService.getOrderStatus(orderSn);
        if(r.getCode() == 0){
            OrderTo orderInfo = r.getData("data", new TypeReference<OrderTo>() {});
            if((orderInfo == null || orderInfo.getStatus() == OrderConstant.OrderStatusEnum.CANCLED.getCode()) && taskDetailEntity.getLockStatus() == 1){
                orderUnlockStock(taskDetailEntity.getSkuId(), taskDetailEntity.getWareId(), taskDetailEntity.getSkuNum(), detailId);
            }
        }else{
            throw new RuntimeException("远程调用服务失败");
        }
    }
}
```

4、联想之前代码，可以知道是 Feign 远程调用不携带请求头，而 order 模块又必须验证登录

5、可以同样创建一个拦截器，在新请求模板中添加 cookie，但考虑到远程调用方法本身其实并不需要验证登录，于是选择放行相关请求

```java
String requestURI = request.getRequestURI();
boolean match = new AntPathMatcher().match("/order/order/status/**", requestURI);
if(match){
    return true;
}
```


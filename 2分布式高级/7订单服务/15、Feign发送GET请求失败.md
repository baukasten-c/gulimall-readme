# Feign发送GET请求失败

1、使用支付宝进行支付时，Feign 接口一直报错

```java
@Override
public String getOrderPay(String orderSn) {
    PayTo to = new PayTo();
    OrderEntity orderEntity = this.getOrderByOrderSn(orderSn);
    String pay = null;
    if(orderEntity.getStatus() == OrderConstant.OrderStatusEnum.CREATE_NEW.getCode()){
        to.setOutTradeNo(orderSn);
        BigDecimal payAmount = orderEntity.getPayAmount().setScale(2, BigDecimal.ROUND_UP);
        to.setTotalAmount(payAmount.toString());
        List<OrderItemEntity> orderItems = orderItemService.list(new QueryWrapper<OrderItemEntity>().eq("order_sn", orderSn));
        to.setSubject(orderItems.get(0).getSpuName());
        GoodsDetail[] goodsDetails = orderItems.stream().map(item -> {
            GoodsDetail goodsDetail = new GoodsDetail();
            goodsDetail.setGoodsId(item.getSkuId().toString());
            goodsDetail.setGoodsName(item.getSkuName());
            goodsDetail.setQuantity(item.getSkuQuantity().longValue());
            goodsDetail.setPrice(item.getSkuPrice().toString());
            return goodsDetail;
        }).toArray(GoodsDetail[]::new);
        to.setGoodsDetail(goodsDetails);
        pay = thirdPartFeignService.pay(to);
    }
    return pay;
}
```

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1701761311128-f5688dae-a95a-433f-a48b-78bfc4efe7d8.png)

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1701761316185-4ab9e03b-d9bf-428d-8d5f-cfbc6f7558e1.png)

2、根据错误提示，第三方模块里没有对应的 POST 方法，但实际代码中使用的都是 GET 请求

```java
@FeignClient("gulimall-third-party")
public interface ThirdPartFeignService {
    @GetMapping("/thirdparty/pay/alipay")
    String pay(@RequestBody PayTo to);
}
```

3、原因：Feign远程调用将 GET 请求变为了 POST 请求。Feign 默认使用的连接工具实现类，只要发现有对应的 body 体对象，就会强制把 GET 请求转换成 POST 请求

4、解决方法一：直接用 POST 请求取代 GET 请求

5、解决方法二：手动拆分对象变为多个单一变量，并使用 @RequestParam 注解将参数拼接到请求 url 上

6、解决方法三：使用 @SpringQueryMap 注解直接传递对象，但是该注解使用反射获取属性名称和值，如果对象中有复杂变量，如：数组、枚举、日期等，则会报错

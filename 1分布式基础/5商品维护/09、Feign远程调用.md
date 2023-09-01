# Feign远程调用

1、在 gulimall-product 的 pom 文件中引入依赖

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2、在 gulimall-product 的启动类添加注解 @EnableFeignClients，开启 Feign 的功能

3、在 feign 包下编写 Feign 接口

```java
@FeignClient("gulimall-coupon")
public interface CouponFeignService {
    @PostMapping("/coupon/spubounds/save")
    R saveSpuBounds(@RequestBody SpuBoundTo spuBoundTo);
    @PostMapping("/coupon/skufullreduction/saveinfo")
    R saveSkuReduction(@RequestBody SkuReductionTo skuReductionTo);
}
```

4、在 gulimall-common 中编写 to 实体类，用于模块间数据的传输

4-1、SpuBoundTo，结合 gulimall-product 的 Bounds 和 gulimall-coupon 的 SpuBoundsEntity

```java
@Data
public class SpuBoundTo {
    private Long spuId;
    private BigDecimal buyBounds;
    private BigDecimal growBounds;
}
```

4-2、SkuReductionTo，结合 gulimall-product 的 Skus 和 gulimall-coupon 的 SkuFullReductionEntity、SkuLadderEntity

```java
@Data
public class SkuReductionTo {
    private Long skuId;
    private int fullCount;
    private BigDecimal discount;
    private int countStatus;
    private BigDecimal fullPrice;
    private BigDecimal reducePrice;
    private int priceStatus;
    private List<MemberPrice> memberPrice;
}
@Data
public class MemberPrice {
    private Long id;
    private String name;
    private BigDecimal price;
}
```

5、被远程调用方法的参数类型可以与 Feign 接口中方法的参数类型不同，因为有@RequestBody， 数据以 json 形式传递，会自动转换

```java
@RequestMapping("/save")
public R save(@RequestBody SpuBoundsEntity spuBounds)
@PostMapping("/saveinfo")
public R saveInfo(@RequestBody SkuReductionTo reductionTo)
```

6、开启两个模块，可以成功传输数据

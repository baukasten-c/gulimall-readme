# 查询sku是否有库存

1、修改老师代码，改为批量查询

```java
@Override
public List<SkuHasStockVo> getSkusHasStock(List<Long> skuIds) {
    List<Map<Long, Long>> skuStocks = baseMapper.getSkuStocks(skuIds);
    List<SkuHasStockVo> skuHasStockVos = new ArrayList<>();
    for (Map<Long, Long> skuStock : skuStocks) {
        Long skuId = skuStock.get("skuId");
        Long count = skuStock.getOrDefault("stock", 0L);
        SkuHasStockVo skuHasStockVo = new SkuHasStockVo(skuId, count > 0);
        skuHasStockVos.add(skuHasStockVo);
    }
    return skuHasStockVos;
}

<select id="getSkuStocks" resultType="java.util.Map">
    SELECT sku_id AS skuId, SUM(stock - stock_locked) AS stock
    FROM wms_ware_sku
    WHERE sku_id IN
    <foreach collection="list" item="skuId" open="(" separator="," close=")">
        #{skuId}
    </foreach>
    GROUP BY sku_id
</select>
```

2、好处

2-1、减少与数据库的交互次数，速度更快

2-2、对于错误 skuId，原本代码会报错，现在则不会有影响。更正：使用 Long 类型接收，原始代码也不会报错

3、测试，报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694143628197-813ac1ea-08e8-4a5e-bf47-0a1939c52114.png)

4、因为 SUM() 会将函数结果自动转换为 BigDecimal类型

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1694143631988-58644d44-c946-49c4-b064-938c991a661a.png)

5、修改代码，测试，可以正常运行

```java
@Override
public List<SkuHasStockVo> getSkusHasStock(List<Long> skuIds) {
    List<Map<Long, Long>> skuStocks = baseMapper.getSkuStocks(skuIds);
    List<SkuHasStockVo> skuHasStockVos = new ArrayList<>();
    for (Map<Long, Long> skuStock : skuStocks) {
        Long skuId = skuStock.get("skuId");
        boolean hasStock = skuStock.get("stock") != null ? true : false;
        SkuHasStockVo skuHasStockVo = new SkuHasStockVo(skuId, hasStock);
        skuHasStockVos.add(skuHasStockVo);
    }
    return skuHasStockVos;
}
```

6、P135再次测试发现问题，数据库中没有的 skuId 不会出现在 skuHasStockVos 中，最终会被存为 null，于是修改代码为：

```java
@Override
public List<SkuHasStockTo> getSkusHasStock(List<Long> skuIds) {
    Map<Long, Map> skuStocks = baseMapper.getSkuStocks(skuIds);
    List<SkuHasStockTo> skuHasStockTos = new ArrayList<>();
    for (Long skuId : skuIds) {
        boolean hasStock = false;
        if(skuStocks.containsKey(skuId)){
            BigDecimal stock = (BigDecimal) skuStocks.get(skuId).get("stock");
            hasStock = stock != null && stock.intValue() > 0;
        }
        SkuHasStockTo skuHasStockTo = new SkuHasStockTo(skuId, hasStock);
        skuHasStockTos.add(skuHasStockTo);
    }
    return skuHasStockTos;
}

@MapKey("skuId")
Map<Long, Map> getSkuStocks(List<Long> skuIds);

<select id="getSkuStocks" resultType="java.util.Map">
    SELECT sku_id AS skuId, SUM(stock - stock_locked) AS stock
    FROM wms_ware_sku
    WHERE sku_id IN
    <foreach collection="list" item="skuId" open="(" separator="," close=")">
        #{skuId}
    </foreach>
    GROUP BY sku_id
</select>
```


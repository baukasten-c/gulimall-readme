# attrValue检索失败

1、在 Elasticsearcch 中检索 attrValue，发现只有属性值完全相同才能匹配成功，与逻辑不符

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1698112899094-f676aef6-b228-4bbd-8426-dd2741a5f0af.png)

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1698112906004-b97af388-27bd-4f62-8829-4779a48afdfc.png)

2、因为上传到 Elasticsearch 中时使用的 SkuEsModel 实体类，其中设置 attrValue 为 String 类型，Elasticsearch 中又设置 attrValue 的类型为 keyword，最终导致这个问题的出现

3、解决方法

3-1、将 SkuEsModel 实体类中 attrValue 的类型设置为 List<String> 

3-2、将数据库中的 attrValue 字符串拆分为 List

```java
List<SkuEsModel.Attr> attrsList = baseAttrs.stream().filter(item -> idSet.contains(item.getAttrId()))
        .map(item -> {
            SkuEsModel.Attr attr = new SkuEsModel.Attr();
            BeanUtils.copyProperties(item, attr);
            List<String> attrValue = Arrays.asList(item.getAttrValue().split(";"));
            attr.setAttrValue(attrValue);
            return attr;
        }).collect(Collectors.toList());
```

3-3、删除 Elasticsearch 中的 gulimall_product，重新上架商品

4、再次进行检索，检索成功

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1698113670485-2504ddc7-8d1a-438f-b30f-4f68abf8e04a.png)
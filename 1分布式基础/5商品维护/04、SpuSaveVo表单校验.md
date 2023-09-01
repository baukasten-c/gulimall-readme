# SpuSaveVo表单校验

1、SpuInfoController 的 save() 加上 @Validated

```java
@RequestMapping("/save")
public R save(@Validated @RequestBody SpuSaveVo spuSaveVo){
    spuInfoService.saveSpuInfo(spuSaveVo);
    return R.ok();
}
```

2、SpuSaveVo 加上校验注解

```java
@Data
public class SpuSaveVo {
    @NotBlank(message = "商品名称不能为空")
    private String spuName;
    @NotBlank(message = "简单描述不能为空")
    private String spuDescription;
    @NotNull(message = "分类不能为空")
    private Long catalogId;
    @NotNull(message = "品牌不能为空")
    private Long brandId;
    @NotNull(message = "重量值不能为空")
    private BigDecimal weight;
    private int publishStatus;
    @NotEmpty(message = "商品详情图集不能为空")
    private List<String> decript;
    @NotEmpty(message = "商品图片集不能为空")
    private List<String> images;
    private Bounds bounds;
    private List<BaseAttrs> baseAttrs;
    private List<Skus> skus;
}
```


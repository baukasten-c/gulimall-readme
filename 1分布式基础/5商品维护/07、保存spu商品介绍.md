# 保存spu商品介绍

1、视频中，老师将商品介绍 decripts 通过 join 放到一条数据里，而商品图集 images 则是分成多条数据，那为什么不将 decripts 也分成多条数据呢？这样不是更清楚，更易于操作和维护吗？

2、于是我将代码改为

```java
public void saveSpuInfo(SpuSaveVo spuSaveVo) {
    List<String> decripts = spuSaveVo.getDecript();
    spuInfoDescService.saveSpuInfoDesc(spuInfoEntity.getId(), decripts);
}

@Override
public void saveSpuInfoDesc(Long id, List<String> decripts) {
    if(decripts == null || decripts.size() == 0){
        return;
    }
    List<SpuInfoDescEntity> spuInfoDescEntities = decripts.stream()
            .map(decript -> new SpuInfoDescEntity(id, decript))
            .collect(Collectors.toList());
    this.saveBatch(spuInfoDescEntities);
}
```

3、结果发现 this.saveBatch(spuInfoDescEntities); 报错，检查数据库后发现，pms_spu_info_desc 中，spu_id 就是主键，因此 spu_id 不能重复，decripts 也就不能不能存为多条数据

4、思考了一下为什么要这样设计数据库：pms_spu_info_desc 应该是直接借鉴了存文字版商品介绍的数据库设计，对于文字版，商品介绍再长也只是一段文字，作为一条数据即可
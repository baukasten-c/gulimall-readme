# attr_type为2

1、attr_type 为0表示销售属性，attr_type为1表示规格参数，attr_type为2既表示销售属性，又表示规格参数

2、修改 attr-add-or-update.vue 的属性类型下拉框

```vue
<el-form-item label="属性类型" prop="attrType">
  <el-select v-model="dataForm.attrType" placeholder="请选择">
    <el-option label="规格参数" :value="1"></el-option>
    <el-option label="销售属性" :value="0"></el-option>
    <el-option label="销售属性+基本属性" :value="2"></el-option>
  </el-select>
</el-form-item>
```

3、修改 ProductConstant，添加 ATTR_TYPE_BOTH(2, "既是销售属性又是基本属性");

4、修改 AttrServiceImpl 的 queryBaseAttrPage()

```java
QueryWrapper<AttrEntity> wrapper = new QueryWrapper<AttrEntity>().eq("attr_type", "base".equalsIgnoreCase(type) ?
                ProductConstant.AttrEnum.ATTR_TYPE_BASE.getCode() : ProductConstant.AttrEnum.ATTR_TYPE_SALE.getCode());
wrapper.or().eq("attr_type", ProductConstant.AttrEnum.ATTR_TYPE_BOTH.getCode());
```

5、重启，发现 attr_type 为2的数据既可以显示在规格参数页面，又可以显示在销售属性页面，而在发布商品页面只显示在销售属性
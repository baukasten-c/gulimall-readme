# sku图片保存

1、原本通过下方代码保存默认图片地址

```java
String defaultImg = "";
for (Images image : item.getImages()) {
    if (image.getDefaultImg() == 1) {
        defaultImg = image.getImgUrl();
    }
}
skuInfoEntity.setSkuDefaultImg(defaultImg);
```

2、但是如果有多个默认图片，就只会保存最后的数据，因此修改代码为

```java
List<String> imgUrls = sku.getImages().stream()
                        .filter(img -> img.getDefaultImg() == 1)
                        .map(Images::getImgUrl)
                        .collect(Collectors.toList());
skuInfoEntity.setSkuDefaultImg(String.join(",", imgUrls));
```

3、保存 sku 基本信息和 sku 图集时都需要遍历 sku.getImages()，思考可不可以合二为一

3-1、将保存 sku 图集时的遍历移动到保存 sku 基本信息里，无法实现，因为 skuId 此时还不存在

3-2、将保存 sku 基本信息时的遍历移动到保存 sku 图集里，然后只需要调用修改方法，将 skuDefaultImg 插入数据库即可，但是代码不如原来清晰，同时性能也没有提高很多

4、结论：两次遍历可以合并，但是m'y
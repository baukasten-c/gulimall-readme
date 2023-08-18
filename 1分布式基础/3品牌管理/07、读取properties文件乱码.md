# 读取properties文件乱码

1、编写自定义校验注解，设置错误信息读取自 ValidationMessages.properties 文件

2、使用 Postman 进行测试，发现信息为乱码，查看文件，文件中文字正常显示

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1692321021201-936e06d9-86a8-47ca-836e-bddd07ffaf9b.png)

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1692321057036-bd300179-3437-4308-9b3f-efeb31e9e1d6.png)

3、在 File-Settings-Editor-File Encodings 中勾选 Transparent native-to-ascii conversion 后即可正常显示

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1692321026478-98843b62-6a32-4474-9f98-5528b07040e9.png)

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1692321114937-048f7eb9-be06-495f-98c5-78bad4e9200b.png)

4、原因：读取文本文件时，默认情况下，Java 使用平台的默认字符编码（通常是 UTF-8）来解码文件内容。如果文件中包含了非 ASCII 字符，而代码或运行环境没有正确设置编码，就会导致乱码问题。勾选了这个选项后，文件中的内容 "显示状态只能为0或1" 中的非 ASCII 字符会被转换为相应的 Unicode 转义序列，这些转义序列会被正确地读取并显示，从而避免了乱码问题。
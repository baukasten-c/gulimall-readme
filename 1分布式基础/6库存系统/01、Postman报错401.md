# Postman报错401

1、通过 Postman 发送请求以实现领取采购单，但是却发现报错 401

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693819515496-3f2f7bdb-30ca-48e8-9fdf-281fa581b45a.png)

2、由提示信息知，请求缺少 token，查看其他请求，发现它们的 Cookie 中都带有相同 token

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693819753167-b57d46c7-cbc7-48f9-bfde-a250b76a2c8f.png)

3、在 Postman 中添加 token，测试，可以正常发送请求

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693819862670-7fb185c8-96e2-4052-8f00-b8ad189940f5.png)
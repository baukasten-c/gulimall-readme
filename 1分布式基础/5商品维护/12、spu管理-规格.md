# spu管理-规格

1、将 attrupdate.vue 添加到 product 中，点击规格按钮，报错400

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693878350351-5d97fff3-4454-4b54-b6a6-4ca44f8255e4.png)

2、参考 https://blog.csdn.net/huhuhutony/article/details/120290512，在 index.js 中添加路由，点击规格按钮，发现页面一片空白

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693878355947-601c0767-7107-4bfa-b6c2-6b13113c3057.png)

3、打印 this.$route.query，发现参数 catalogId 没有获取到，导致后面的操作都无法进行

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693878361501-3975c4b3-4df6-4b5b-9772-5c987ca06cfb.png)

4、原因是：之前将其他页面的 catalogId 都改为了 catelogId，因此现在参数名无法对应，修改后页面可以成功显示

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693878726660-9cdcb628-c71a-48db-a9a0-8365b63babc0.png)
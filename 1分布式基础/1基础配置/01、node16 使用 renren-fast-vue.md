# node16 使用 renren-fast-vue

1、原始版本为node18，一直出错，降低版本到16

2、使用node16依旧一直报错后，于是删除node16重新安装

2-1、在 设置-应用-安装的应用 中卸载node.js

2-2、需要同时删除node_cache和node_global两个文件夹

2-3、安装node16

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248351686-f26984ee-a88c-4f99-a83a-0c6fbf99d088.png)

3、在idea中启动后台

4、使用git克隆renren-fast-vue到本地

5、使用vs打开renren-fast-vue，修改package-lock.json中node-sass的版本为6.0.1

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248377001-68976949-749d-443f-9c0d-0a80a70d2ce9.png)

1)参考文章

https://blog.csdn.net/qq_24654501/article/details/119005722

 2)node node-sass 版本对应如图

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248391056-802fc0ed-eaa2-4b5c-aeea-aa1e2b8b2ae2.png)

3)我没有修改package.json，因为我的原始版本就是6.0.1，同时也不需要修改sass-loader

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248438787-940611bf-9ba3-410b-bb34-b2a92d8e6bd2.png)

6、执行npm install报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248461516-0e09c104-b26a-4e7a-9b0f-804bc4327c37.png)

 7、需要自己安装node-sass

npm i node-sass@6.0.1 --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248477554-7eb8bd41-1125-4979-a1fa-3e5524d3ed72.png)

 8、再次执行npm install，没有报错，执行npm run dev后出现界面

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248490510-46dc5ec0-fc99-4dc8-a95d-4a05cd959bbd.png)

注：忙了一天才成功，期间做了各种尝试，因此本文仅供参考。写文章时发现npm的版本降低了，不知道有没有影响。现在后怀疑只要把node-sass换为8.0以上版本，node18也可以成功


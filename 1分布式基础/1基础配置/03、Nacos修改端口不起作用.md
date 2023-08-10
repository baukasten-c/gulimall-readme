# 3、Nacos修改端口不起作用

1、在yml中设置server.port，运行一个模块后发现端口还是8080，且运行其他模块会报错8080端口被占用

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248552423-b173b8c4-808f-4d35-9999-2303cc7a2422.png)

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248565505-43b8892a-c91c-4fb3-99e5-d7da6aa072ff.png)

2、排查后发现配置都没有问题

3、发现问题出现在application.properties里，之前在里面设置了server.port=8080，把每个模块中的这句话注释掉即可

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1690248598049-8e9d7c6e-fc93-49fb-92ae-0f1f95b91690.png)

4、在一个普通的项目中，生成两个配置文件，一个yml，一个properties，分别设置两个不同的端口，启动项目，最后起效的是properties中的配置。 在只有yml和yaml情况下，以yml为准
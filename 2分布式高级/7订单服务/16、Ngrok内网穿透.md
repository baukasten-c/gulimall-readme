# Ngrok内网穿透

1、在 https://ngrok.com/download 下载 Ngrok

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1702021572252-a51f5e59-2c02-4db6-bae6-dff7f47f376d.png)

2、登录账户

3、添加账户信息，将 authtoken 添加到默认的 ngrok.yml 配置文件中

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1702281660006-48954631-6e79-41a1-910e-63895349c247.png)

```shell
ngrok config add-authtoken xxxxxxx
```

4、申请域名，使每次启动使用的访问地址都相同

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1702281831225-cfff1a0d-d65a-4d2f-9890-e5b759aa823e.png)

5、使用命令进行内网穿透

```shell
ngrok http http://order.gulimall.com:80 --domain xxxxxx    
```

6、内网穿透成功

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1702282266110-59b3b27f-02f8-4442-b982-15fd0a1fce02.png)


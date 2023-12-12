# wget安装ik分词器

1、在 https://github.com/medcl/elasticsearch-analysis-ik/releases 找到 Elasticsearch 对应版本的 ik 分词器

2、在 docker 中进入 /mydata/elasticsearch/plugins，输入命令创建 ik 文件夹

```shell
mkdir ik
cd ik
```

3、下载 ik 分词器，报错无法建立 SSL 连接

```shell
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.6.0/elasticsearch-analysis-ik-8.6.0.zip
```

4、修改命令(新增 --no-check-certificate，https 变 http)，重新执行，可以成功下载

```shell
wget --no-check-certificate http://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.6.0/elasticsearch-analysis-ik-8.6.0.zip
```

5、解压 ik 分词器

```shell
unzip elasticsearch-analysis-ik-8.6.0.zip
```

6、重启 elasticsearch

```shell
docker restart elasticsearch
```

7、刷新 kibana 页面，执行命令，可以进行分词，即安装成功

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693925380571-db5302cf-fe40-4d10-aba2-93a58b193f64.png)

8、但是不推荐使用 wget 进行下载，速度很慢
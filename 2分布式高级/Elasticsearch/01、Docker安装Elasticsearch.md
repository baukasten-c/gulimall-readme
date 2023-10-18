# Docker安装Elasticsearch

1、拉取镜像

```shell
docker pull elasticsearch:8.6.0
```

2、创建挂载点目录

```shell
mkdir -p /mydata/elasticsearch/config  创建目录
mkdir -p /mydata/elasticsearch/data
mkdir -p /mydata/elasticsearch/plugins
echo "http.host: 0.0.0.0" >/mydata/elasticsearch/config/elasticsearch.yml
//设置mydata/elasticsearch/文件夹中文件都可读可写
chmod -R 777 /mydata/elasticsearch/
```

3、启动 Elasticsearch

```shell
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:8.6.0
```

4、使用 docker ps 命令进行查看，发现并没有 elasticsearch

5、猜测原因是资源不足，删除 elasticsearch，修改命令后重新执行

```shell
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx256m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:8.6.0
```

6、使用 docker ps 命令进行查看，发现有 elasticsearch，但立刻再次查看，发现 elasticsearch 再次消失

7、使用  docker logs elasticsearch 命令查看问题

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693897672813-b938a2f1-f5b2-4a10-ad64-fa1bc50bda00.png)

8、参考 https://blog.csdn.net/tiancao222/article/details/131469295?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-131469295-blog-126546947.235%5Ev38%5Epc_relevant_anti_vip_base&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-131469295-blog-126546947.235%5Ev38%5Epc_relevant_anti_vip_base&utm_relevant_index=5，在 elasticsearch.yml 文件中，只要配置了xpack的安全选项，就能正常启动

```shell
//关闭密码安全验证
echo 'xpack.security.enabled: false' >> elasticsearch.yml
```

9、elasticsearch 终于成功启动

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1693898007005-fa73d104-7209-4ea9-9da6-aedee6790ed2.png)
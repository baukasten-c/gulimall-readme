# OSS上传测试

1、参考 https://blog.csdn.net/weixin_45089791/article/details/114531980，了解三种不同上传方式

1-1、普通上传：前端将文件流上传到服务端后，服务端进行处理，返回给前端地址，这种方式适合传统单体的系统架构，不将文件服务作为一个单独的服务进行部署。但在现在流行的分布式大环境下，传统的文件存储方式已经不适用

1-2、前端直传：优点：直传文件流不用过服务端，上传速度不受服务端带宽影响。由于文件不过服务端，所以也不占用服务端的系统资源。缺点：客户端通过JavaScript把AccesssKeyID和AccessKeySecret写在代码里面有泄露的风险

1-3、为解决直传的缺点，现在大多使用服务端签名后直传

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1691543381871-12fbdc13-5bd9-4a7f-a138-812f76bcd647.png)

2、参考 https://github.com/alibaba/aliyun-spring-boot/tree/master/aliyun-spring-boot-samples/aliyun-oss-spring-boot-sample#aliyun-spring-boot-oss-simple，将阿里云OSS与SpringBoot项目进行连接

2-1、在 guli-common 中添加依赖

```java
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>aliyun-oss-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

2-2、在Nacos中配置accessKeyId、secretAccessKey和region

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1691543998879-bf991994-6c26-4d13-ab30-ae6b34001106.png)

3、参考 [https://help.aliyun.com/zh/oss/deve](https://help.aliyun.com/zh/oss/developer-reference/simple-upload-11?spm=a2c4g.11186623.0.0.69b6df3aLdnjzI)[loper-reference/simple-upload-11?spm=a2c4g.11186623.0.0.69b6df3aLdnjzI](https://help.aliyun.com/zh/oss/developer-reference/simple-upload-11?spm=a2c4g.11186623.0.0.69b6df3aLdnjzI)，在 GulimallProductApplicationTests 中使用上传文件流代码

```java
@Test
void testUpload() {
    // 填写Bucket名称，例如examplebucket。
    String bucketName = "guli-mall--oss";
    // 填写Object完整路径，完整路径中不能包含Bucket名称，例如exampledir/exampleobject.txt。
    String objectName = "mountain.jpg";
    // 填写本地文件的完整路径，例如D:\\localpath\\examplefile.txt。
    // 如果未指定本地路径，则默认从示例程序所属项目对应本地路径中上传文件流。
    String filePath= "C:\\Users\\Lenovo\\Pictures\\v2-41e45a115573517209a67c6fd67fcc52_r.jpg";
    try {
        InputStream inputStream = new FileInputStream(filePath);
        // 创建PutObjectRequest对象。
        PutObjectRequest putObjectRequest = new PutObjectRequest(bucketName, objectName, inputStream);
        // 创建PutObject请求。
        PutObjectResult result = ossClient.putObject(putObjectRequest);
    } catch (OSSException oe) {
        System.out.println("Caught an OSSException, which means your request made it to OSS, "
        + "but was rejected with an error response for some reason.");
        System.out.println("Error Message:" + oe.getErrorMessage());
        System.out.println("Error Code:" + oe.getErrorCode());
        System.out.println("Request ID:" + oe.getRequestId());
        System.out.println("Host ID:" + oe.getHostId());
    } catch (FileNotFoundException e) {
        throw new RuntimeException(e);
    } finally {
        if (ossClient != null) {
        	ossClient.shutdown();
        }
    }
}
```

4、测试，发现报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1691544492129-011286bb-3ef3-4016-9c4c-a2344cb98700.png)

5、参考 https://blog.csdn.net/qq_42569573/article/details/113093682，在 guli-common 中添加依赖

```java
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>4.6.0</version>
</dependency>
```

6、可以正常进行上传

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1691544650593-2122a845-ecee-4a6b-8802-75f332f083ea.png)
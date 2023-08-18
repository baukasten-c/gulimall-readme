# gulimall-third-party模块

1、将阿里云OSS等服务统一放置在第三方服务模块，按视频创建一个新 module 即可

2、设置 gulimall-third-party 依赖 gulimall-common ，同时排除数据库相关配置

```java
<dependency>
    <groupId>com.atguigu</groupId>
    <artifactId>gulimall-common</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <exclusions>
        <exclusion>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

3、将oss设置从 gulimall-common 转移到 gulimall-third-party 中

```java
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>aliyun-oss-spring-boot-starter</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.aliyun</groupId>
        <artifactId>aliyun-java-sdk-core</artifactId>
        <version>4.6.0</version>
    </dependency>
</dependencies>  
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2021.0.5.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

4、将 third-party 注册到Nacos

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1692176995750-8cc04b28-05c4-4de2-82e4-95f1619c1662.png)

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1692177000640-f74425a8-72b1-44ac-a573-8fbaa0a612ce.png)

5、重新启动服务，运行 RenrenApplication 时报错

> Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled. 
>
> 2023-08-16 15:28:01.427 ERROR 11380 --- [           main] o.s.b.d.LoggingFailureAnalysisReporter   :  
>
> *************************** APPLICATION FAILED TO START *************************** 
>
> Description: Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured. Reason: Failed to determine a suitable driver class  
>
> Action: Consider the following: If you want an embedded database (H2, HSQL or Derby), please put it on the classpath. If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active).

6、因为在 gulimall-third-party 中没有将数据库相关依赖全部排除，排除后即可成功启动

```java
<exclusion>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</exclusion>
```

7、运行 GulimallGatewayApplication 时报错

> Caused by: java.lang.IllegalArgumentException: Oss endpoint can't be empty. 
>
> at org.springframework.util.Assert.isTrue(Assert.java:121) ~[spring-core-5.3.24.jar:5.3.24] 
>
> at com.alibaba.cloud.spring.boot.oss.autoconfigure.OssContextAutoConfiguration.ossClient(OssContextAutoConfiguration.
>
> java:54) ~[aliyun-oss-spring-boot-starter-1.0.0.jar:1.0.0] 
>
> at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_152] 
>
> at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_152] 
>
> at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_152] 
>
> at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_152] 
>
> at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:154) ~[spring-beans-5.3.24.jar:5.3.24] ... 20 common frames omitted

8、在网上搜索到解决方法多种多样，我的是把下面的代码从 third-party 模块的 pom 文件中删除即可

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2021.0.5.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

9、运行测试代码，可以成功上传文件

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1692178509927-f612e396-26ec-4dcb-a9d1-95e6abd8fac4.png)

注：为了截图，我把代码恢复，但是上面的两个启动时报错这次都没有再报错
## MinIO对象存储服务

### 简介： 

> [Minio](https://so.csdn.net/so/search?q=Minio&spm=1001.2101.3001.7020) 是一个基于Apache License v2.0开源协议的对象存储服务，虽然轻量，却拥有着不错的性能。它兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据。 
>
> 例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几 kb 到最大 5T 不等。

### 说明：

> [Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)如果想安装软件 , 必须先到 [Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020) 镜像仓库下载镜像。

###  3、创建目录

> 一个用来存放配置，一个用来存储上传文件的目录
>
> 启动前需要先创建Minio外部挂载的配置文件（ /home/minio/config）,和存储上传文件的目录（ /home/minio/data）

```bash
mkdir -p /usr/local/docker/minio/config
mkdir -p /usr/local/docker/minio/data
```

###  4、创建Minio容器并运行

> 多行模式 

```haskell
docker run -p 9000:9000 -p 9090:9090 \
     --net=host \
     --name minio \
     -d --restart=always \
     -e "MINIO_ACCESS_KEY=minioadmin" \
     -e "MINIO_SECRET_KEY=minioadmin" \
     -v /usr/local/dk/docker/minio/data:/data \
     -v /usr/local/dk/docker/minio/config:/root/.minio \
     minio/minio server \
     /data --console-address ":9090" -address ":9000"
```

> 9090端口指的是minio的客户端端口
>
> MINIO_ACCESS_KEY ：账号
>
> MINIO_SECRET_KEY ：密码（账号长度必须大于等于5，密码长度必须大于等于8位）

###  5、访问操作

访问：http://106.53.124.182:9090/login 用户名：密码 minioadmin：minioadmin

![img](E:\Development\Typora\images\7347b1b75055440796d13a9396af56d2.png) 创建用户

![img](E:\Development\Typora\images\85d71387666f4c8eae7cf5d1377030e0.png)

![img](E:\Development\Typora\images\51df36c96dc243f486fee70a28cb2d0f.png) ![img](E:\Development\Typora\images\9e53f48a825e45b9aadb877d46fe7b30.png)

创建组

![img](E:\Development\Typora\images\8db2fcd2c58d4b818724b7fdedb801a1.png)![img](E:\Development\Typora\images\f3630549378843a0862835e3ac56b995.png)

创建accessKey和secretKey 

![img](E:\Development\Typora\images\7e45174f035d478eb2403ebc727cadf4.png)

![img](E:\Development\Typora\images\e3beea9516374cd9a282e5381a2a5339.png) 点击下载

![img](E:\Development\Typora\images\7a23b700366344daa14679d920591d12.png)

 文件内容如下，保存文件，SDK操作文件的[API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020)需要用到

```json
{"url":"http://192.168.124.132:9000","accessKey":"XO1JDovW2FTmGaBb","secretKey":"uG6wMfylUnOVH5WzwxqnldOWw2dMshNX","api":"s3v4","path":"auto"}
```

创建Bucket

![img](E:\Development\Typora\images\1a434159f1c54d9fbaf6cef41d8d83e4.png)

![img](E:\Development\Typora\images\4c90c360157740019aabac2cef1e649d.png)![img](E:\Development\Typora\images\76e1bd28a0fd42bfa10621bd4ebf7010.png)

上传文件

![img](E:\Development\Typora\images\58986f3a0d4142d18eee98b3cbf37459.png)

![img](E:\Development\Typora\images\6dd2bf16650a48b9b287dc40f2d047bc.png) ![img](E:\Development\Typora\images\c429a4ebb28346558f03a900d9a93aeb.png)

###  6、SDK操作

官方文档：https://docs.min.io/docs/

![img](E:\Development\Typora\images\c63ec4bb5a0d42e69995eaf05f38c900.png)

 javaSDK：https://docs.min.io/docs/java-client-quickstart-guide.html

![img](E:\Development\Typora\images\fbb290153ad440c685a0b71c7beb73d2.png)

 maven依赖

```xml
        #低版本的okhttp会报错提示
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>4.9.0</version>
        </dependency>
        <dependency>
            <groupId>io.minio</groupId>
            <artifactId>minio</artifactId>
            <version>8.4.2</version>
            <exclusions>
                <exclusion>
                    <artifactId>okhttp</artifactId>
                    <groupId>com.squareup.okhttp3</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```

![img](E:\Development\Typora\images\b1c21dff2b0544f6b0f5dfd820cfe324.png)

测试文件上传 

```java

import io.minio.BucketExistsArgs;
import io.minio.MakeBucketArgs;
import io.minio.MinioClient;
import io.minio.UploadObjectArgs;
import io.minio.errors.MinioException;
 
import java.io.IOException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
 
public class FileUploader {
 
    public static void main(String[] args) throws IOException, NoSuchAlgorithmException, InvalidKeyException {
        try {
            // Create a minioClient with the MinIO server playground, its access key and secret key.
            MinioClient minioClient =
                    MinioClient.builder()
                            .endpoint("http://192.168.124.132:9000")
                            .credentials("XO1JDovW2FTmGaBb", "uG6wMfylUnOVH5WzwxqnldOWw2dMshNX")
                            .build();
 
            // Make 'asiatrip' bucket if not exist.
            boolean found = minioClient.bucketExists(BucketExistsArgs.builder().bucket("public").build());
            if (!found) {
                // Make a new bucket called 'asiatrip'.
                minioClient.makeBucket(MakeBucketArgs.builder().bucket("public").build());
            } else {
                System.out.println("Bucket 'public' already exists.");
            }
 
            // Upload '/home/user/Photos/asiaphotos.zip' as object name 'asiaphotos-2015.zip' to bucket
            // 'asiatrip'.
            minioClient.uploadObject(
                    UploadObjectArgs.builder()
                            .bucket("public")
                            .object("credentials.json")
                            .filename("C:/Users/lai.huanxiong/Downloads/credentials.json")
                            .build());
            System.out.println("'C:/Users/lai.huanxiong/Downloads/credentials.json' is successfully uploaded as " + "object 'credentials.json' to bucket 'public'.");
        } catch (MinioException e) {
            System.out.println("Error occurred: " + e);
            System.out.println("HTTP trace: " + e.httpTrace());
        }
    }
}
```

![img](E:\Development\Typora\images\3e55cf31a46d4c4aab03ebdb0ddc542d.png)

 文件上传成功展示

![img](E:\Development\Typora\images\6e5a403aad544764a442ac0da3a94cb4.png)

至此，[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)搭建Minio服务器和简单操作完成！！！
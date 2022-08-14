# Docker安装redis

## 1.下载Redis镜像

| 命令                  | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| docker pull redis     | 下载最新版Redis镜像 (其实此命令就等同于 : docker pull redis:latest ) |
| docker pull redis:xxx | 下载指定版本的Redis镜像 (xxx指具体版本号)                    |

## 2.创建Redis配置文件

> 启动前需要先创建Redis外部挂载的配置文件 （ /home/redis/conf/redis.conf ）
> 之所以要先创建 , 是因为Redis本身容器只存在 /etc/redis 目录 , 本身就不创建 redis.conf 文件
> 当服务器和容器都不存在 redis.conf 文件时, 执行启动命令的时候 docker 会将 redis.conf 作为目录创建 , 这并不是我们想要的结果 。

![image-20220813012524177](E:\Development\Typora\images\image-20220813012524177.png)

其中redis.conf可以在官网下载

## 3.创建Redis容器并启动

```bash
docker run \
-d \
--name redis \
-p 6379:6379 \
--restart unless-stopped \
-v /usr/local/dk/docker/redis/data:/data \
-v /usr/local/dk/docker/redis/redis.conf:/etc/redis/redis.conf \
redis /etc/redis/redis.conf \
```

在这里我们可以不指定密码和appendonly，再创建完后修改redis.conf文件即可

| --appendonly yes | 在Redis容器启动redis-server服务器并打开Redis持久化配置 |
| ---------------- | ------------------------------------------------------ |
|                  |                                                        |

## 4.查看Redis是否运行

```bash
### 查看Docker运行中的容器
docker ps 
docker ps | grep redis
docker logs redis
```

>  报错：chown: changing ownership of '.': Permission denied，权限不足！[请跳转！！！](https://baocl.blog.csdn.net/article/details/115701152)



5.进入Redis容器

```bash
### 通过 Docker 命令进入 Redis 容器内部
docker exec -it redis /bin/bash
docker exec -it redis bash
### 进入 Redis 控制台
redis-cli
### 添加一个变量为 key 为 name , value 为 bella 的内容
> set name bella
### 查看 key 为 name 的 value 值
> get name
 
### 或者也可以直接通过Docker Redis 命令进入Redis控制台 (上面两个命令的结合)
docker exec -it redis redis-cli
```



## 6.Redis 配置文件修改

修改挂载的redis.conf

| 命令              | 功能                                                         |
| :---------------- | :----------------------------------------------------------- |
| appendonly yes    | 启动Redis持久化功能 (默认 no , 所有信息都存储在内存 [重启丢失] 。 设置为 yes , 将存储在硬盘 [重启还在]) |
| protected-mode no | 关闭protected-mode模式，此时外部网络可以直接访问 (docker貌似自动开启了) |
| bind 0.0.0.0      | 设置所有IP都可以访问 (docker貌似自动开启了)                  |
| requirepass 密码  | 设置密码                                                     |

在本地连接服务器redis的时候，发现连接失败，这是因为服务器上的redis开启保护模式运行，该模式下是无法进行远程连接的。只需要修改redis目录下的redis.conf文件，找到 **protected-mode yes** ，将yes 改为no 就可以成功连接了。

![img](E:\Development\Typora\images\2325401-20210320111112971-1083114324.jpg)

同时：
1、将 bind 127.0.0.1 注释掉，就运行外界进行连接
2、将 **daemonize** 设置 yes的时候，则开启redis在后台运行

还有一个就是slave-read-only no 从服务器没有写的权限

## 7.进入有密码的Redis控制台

> 如果你设置了密码,需要通过如下命令进入Redis控制台 

```python
## 进入Redis容器
docker exec -it redis /bin/bash
## 通过密码进入Redis控制台
redis-cli -h 127.0.0.1 -p 6379 -a 123456
```

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAVG91Y2gm,size_20,color_FFFFFF,t_70,g_se,x_16.png)
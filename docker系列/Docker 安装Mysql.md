# Docker 安装Mysql

## 1、寻找Mysql镜像

> 在Docker镜像仓库寻找Mysql镜像

![img](https://img-blog.csdnimg.cn/51b7cfc1a548458d9296d21860d79208.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAVG91Y2gm,size_20,color_FFFFFF,t_70,g_se,x_16)

>  Docker 下载Mysql镜像的命令

![img](https://img-blog.csdnimg.cn/f3b151176956496eb0d0e373e2ad1a56.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAVG91Y2gm,size_20,color_FFFFFF,t_70,g_se,x_16)

![img](https://img-blog.csdnimg.cn/5867ad2f54d2404b879ce0fefe6be401.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAVG91Y2gm,size_20,color_FFFFFF,t_70,g_se,x_16)

##  2、下载Mysql镜像

| 命令                  | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| docker pull mysql     | 下载最新版Mysql镜像 (其实此命令就等同于 : docker pull mysql:latest ) |
| docker pull mysql:xxx | 下载指定版本的Mysql镜像 (xxx指具体版本号)                    |

![img](https://img-blog.csdnimg.cn/5a5325f5291341eebe51355bef110f93.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAVG91Y2gm,size_20,color_FFFFFF,t_70,g_se,x_16)

>  检查当前所有Docker下载的镜像

```undefined
docker images
```

##  3、创建Mysql容器并运行

> Docker 创建Mysql容器	



### mysql5.7：

```diff
docker run \
-p 3306:3306 \
--name mysql5 \
--restart unless-stopped \
-v /usr/local/dk/docker/mysql5/log:/var/log/mysql \
-v /usr/local/dk/docker/mysql5/data:/var/lib/mysql \
-v /usr/local/dk/docker/mysql5/conf:/etc/mysql.d \
-e MYSQL_ROOT_PASSWORD=root \
-d  mysql:5.7

mysql.d
```

### mysql8.0：

```diff
 docker run \
> -p 3306:3306 \
> --name mysql \
> --restart unless-stopped \
> -v /mydata/mysql/data:/var/lib/mysql \
> -v /mydata/mysql/conf.d:/etc/mysql/conf.d \
> -e MYSQL_ROOT_PASSWORD=root \
> -d  mysql:8.0 

```

注意映射位置

| 命令                                 | 描述                                                         |
| :----------------------------------- | :----------------------------------------------------------- |
| docker run                           | 创建一个新的容器 , 同时运行这个容器                          |
| –name mysql                          | 启动容器的名字                                               |
| -d                                   | 后台运行                                                     |
| -p 3306:3306                         | 将容器的 3306 (后面那个) 端口映射到主机的 3306 (前面那个) 端口 |
| –restart unless-stopped              | 容器重启策略                                                 |
| -v /mydata/mysql/log:/var/log/mysql  | 将日志文件夹挂载到主机                                       |
| -v /mydata/mysql/data:/var/lib/mysql | 将mysql储存文件夹挂载到主机                                  |
| -v /mydata/mysql/conf:/etc/mysql     | 将配置文件夹挂载到主机                                       |
| -e MYSQL_ROOT_PASSWORD=root          | 设置 root 用户的密码                                         |
| mysql:5.7                            | 启动哪个版本的 mysql (本地镜像的版本)                        |
| \                                    | shell 命令换行符                                             |

>  注意 : 命令中所有 冒号 前面的是主机配置 , 冒号 后面的是mysql容器配置 。
> –restart unless-stopped : 在docker重启时重启当前容器。但不包含docker重启时已停止的容器。

## 4、查看Mysql是否运行

```r
### 查看Docker运行中的容器
docker ps  
```

## ![img](https://img-blog.csdnimg.cn/c8f3965da2d3417cbddf3ee5906ce5e1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAVG91Y2gm,size_20,color_FFFFFF,t_70,g_se,x_16)

```python
## 通过Docker命令进入Mysql容器内部
docker exec -it mysql /bin/bash
## 或者
docker exec -it mysql bash

```



## Linux登录Mysql

### 1.登录本地mysql:

```bash
mysql -u 用户名 -p
# 例如
mysql -u root -p  #先输入，回车

# 也可不用空格
mysql -u用户名 -p
```

然后提示输入密码，回车即可；

### 2.登录远程mysql：

有主机名和端口号，有时也没有端口号

```bash
mysql -h 主机 -P 端口 -u 用户名 -p

#也可不用空格
mysql -h主机 -P端口 -u用户名 -p
```

然后提示输入密码，回车。

\##

注意输入密码的时候不显示哦！！！



docker cp 9a3a62b39e96:/etc/my.cnf  /

docker cp /my.cnf  99987ef916d1:/etc/
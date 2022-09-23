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



## 3. 创建挂载的目录文件

   一个myql的配置目录 在容器：`/etc/mysql` ，这里可以从其他容器中拷贝过来

```
docker cp mysql5:/etc/mysql     /usr/local/dk/docker/mysql5
```



```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
binlog-format=ROW 
server_id=1 
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

#log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
[client]
socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/

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
-v /usr/local/dk/docker/mysql5/conf/:/etc/mysql/conf.d   \
-e MYSQL_ROOT_PASSWORD=root \
-d  mysql:5.7
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

![img](https://img-blog.csdnimg.cn/c8f3965da2d3417cbddf3ee5906ce5e1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAVG91Y2gm,size_20,color_FFFFFF,t_70,g_se,x_16)

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



## MySQL开启BinLog

1.启动好后，用mysql客户端工具边接，在未设置之前，先查看一下，mysql5.7是否默认开启，查看脚本如下：

```sql
show variables like '%log_bin%'
```

结果如下：

![img](E:\Development\Typora\images\70.png)

看得出mysql5.7默认是未开启的，下面就开始设置

2.找到刚挂载到本地的mysql设置目录 /etc/mysql

```html
vim /usr/local/dk/docker/mysql5/mysql/mysql.conf.d/mysqld.cnf
```

![img](E:\Development\Typora\images\70-16638636551011.png)

在以上修改的文件下方，添加上红框中的两条

这一个参数的作用是mysql会根据这个配置自动设置log_bin为on状态，自动设置log_bin_index文件为你指定的文件名后跟.index

第二个参数 ，用的如果是5.7及以上版本的话，重启mysql服务会报错，这个时候我们必须还要指定这样一个参数，随机指定一个不能和其他集群中机器重名的字符串，如果只有一台机器，那就可以随便指定了

7.设置完后重启mysql容器

```html
docker restart mysql
```

再次查询就会看到已开启mysql的binlog日志，如下图：

![img](E:\Development\Typora\images\70-16638636551022.png)

这个时候，在数据库中创建一个数据库、表，插入一些数据，就会在/var/lib/mysql容器中看到以下，或者是看挂载出来对应的目录上,

![img](E:\Development\Typora\images\70-16638636551023.png)

![img](E:\Development\Typora\images\70-16638636551024.png)



在数据库中查询日志，如下

```html
show binlog events in 'mysql-bin.000001';
show binlog events in 'mysql-bin.000002';
show binlog events in 'mysql-bin.000003';
#FLUSH LOGS
```



![img](E:\Development\Typora\images\70-16638636551025.png)



就可以通过以上数据进行数据恢复

也可以直接操作容器如下:

```html
docker exec mysql bash -c "echo 'log-bin=/var/lib/mysql/mysql-bin' >> /etc/mysql/mysql.conf.d/mysqld.cnf"

docker exec mysql bash -c "echo 'server-id=123454' >> /etc/mysql/mysql.conf.d/mysqld.cnf"

docker restart mysql
 SHOW  GLOBAL VARIABLES LIKE '%log%';
```

##  





docker cp 188f53eba9aa:/etc/my.cnf  /

docker cp /my.cnf  41e8f3cb58d6:/etc/my.cnf
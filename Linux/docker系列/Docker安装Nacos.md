# Docker安装Nacos

# 单机版

## 1、拉取镜像

在https://hub.[docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020).com/r/[nacos](https://so.csdn.net/so/search?q=nacos&spm=1001.2101.3001.7020)/nacos-server/tags
中查看所需版本，并下载

```bash
docker pull nacos/nacos-server
```

## 2、创建[挂载](https://so.csdn.net/so/search?q=挂载&spm=1001.2101.3001.7020)目录

```bash
# 创建 nacos配置存放目录
mkdir -p /usr/local/dk/docker/nacos/conf  
# 创建 nacos日志存放目录
mkdir -p /usr/local/dk/docker/nacos/logs  
# 创建 nacos数据存放目录
mkdir -p /usr/local/dk/docker/nacos/data  
# 第三步： 创建目录做映射， 主要映射配置文件和日志
mkdir -p /home/nacos/logs
mkdir -p /home/nacos/init.d
vi /home/nacos/init.d/custom.properties
```

将如下内容编辑进新文件：

```properties
server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848
 
spring.datasource.platform=mysql
 
db.num=1
db.url.0=jdbc:mysql://192.168.217.13:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=root
nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false
 
management.metrics.export.elastic.enabled=false
 
management.metrics.export.influx.enabled=false
 
 
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i
 
nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
nacos.naming.distro.taskDispatchThreadCount=1
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
nacos.naming.expireInstance=true
```

## 3、初始化数据库

在 `conf` 目录下，提供了 MySQL 数据库初始化脚本 [`nacos-mysql.sql`](https://github.com/alibaba/nacos/blob/develop/distribution/conf/nacos-mysql.sql)。

我们可以根据自己的需要，创建一个库

![image-20220815200903427](E:\Development\Typora\images\image-20220815200903427.png)



## 4、启动容器

```bash
docker run -d -p 8848:8848 \
--privileged=true \
--restart=always \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-v /usr/local/dk/docker/nacos/logs:/home/nacos/logs \
-v /usr/local/dk/docker/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties --name nacos nacos/nacos-server
```



然后访问http://192.168.56.10:8848/nacos/  使用默认的用户「nacos/nacos」进行登录。

> 友情提示：生产环境下，一定要记得修改默认的用户的密码噢。
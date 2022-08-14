# Docker安装Nacos

## 1、拉取镜像

在https://hub.[docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020).com/r/[nacos](https://so.csdn.net/so/search?q=nacos&spm=1001.2101.3001.7020)/nacos-server/tags
中查看所需版本，并下载

```bash
docker pull nacos/nacos-server:v2.0.4
```

## 2、创建[挂载](https://so.csdn.net/so/search?q=挂载&spm=1001.2101.3001.7020)目录

```bash
# 创建 nacos配置存放目录
mkdir -p /home/docker/nacos/conf  
# 创建 nacos日志存放目录
mkdir -p /home/docker/nacos/logs  
# 创建 nacos数据存放目录
mkdir -p /home/docker/nacos/data  
```



## 3、启动容器

```bash
docker run --env MODE=standalone --name nacos -d -p 8848:8848 nacos/nacos-server
```

然后访问[http://192.168.56.10:8848(自己ip)/nacos/index.htm](http://192.168.56.10:8848/nacos/index.htm) 
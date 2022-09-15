

# Docker安装Rabbitmq

## 1.使用[docker](https://so.csdn.net/so/search?q=docker&spm=1001.2101.3001.7020)查询rabbitmq的镜像

`docker search rabbitmq`
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1NTAyMzM2,size_16,color_FFFFFF,t_70.png)

## 2.安装镜像

安装name为rabbitmq的这里是直接安装最新的，如果需要安装其他版本在rabbitmq后面跟上版本号即可
`docker pull rabbitmq`
![在这里插入图片描述](E:\Development\Typora\images\20210713142423729.png)

## 3.运行mq：

### 简单版

```bash
docker run -d --hostname my-rabbit --name rabbit-mqtt -p 15672:15672  -p  5672:5672 rabbitmq
```

![在这里插入图片描述](E:\Development\Typora\images\20210713142432574.png)

> 默认账户密码为guest

### 复杂版

（设置账户密码，hostname）

```javascript
docker run -d -p 15672:15672  -p  5672:5672   -e RABBITMQ_DEFAULT_USER=root -e RABBITMQ_DEFAULT_PASS=root --name rabbitmq --hostname=rabbitmqhostone  rabbitmq
```



### mqtt版本

#### 运行

```
docker run -d --hostname my-rabbit --name rabbit-mqtt -p 15672:15672  -p  5672:5672 -p 1883:1883 -p 15675:15675 rabbitmq
```

RabbitMQ 默认关闭MQTT 协议，需用命令手动扩展，RabbitMQ 的MQTT 协议分为两种。

-  rabbitmq_mqtt 提供与后端服务交互使用，端口1883
-  rabbitmq_web_mqtt 提供与前端交互使用，端口15675

#### 开启插件

然后进入[rabbitmq](https://so.csdn.net/so/search?q=rabbitmq&spm=1001.2101.3001.7020)容器内部，开启插件

```bash
docker exec -it rabbitmq bin/bash
rabbitmq-plugins enable rabbitmq_mqtt
rabbitmq-plugins enable rabbitmq_web_mqtt
```

#### 重启

```
docker restart rabbitmq
```

#### 使用MQTT.fx工具测试



![图片](E:\Development\Typora\images\640.png)协议对应端口号

使用`MQTT` 协议默认的交换机 `Exchange` 为 `amp.topic`，而我们订阅的主题会在 `Queues` 注册一个客户端队列，路由 `Routing key` 就是我们设置的主题。

![图片](E:\Development\Typora\images\640-16631750957043.png)

## 4.注意

### 1.Web-UI界面无法访问 

通过`docker ps -a`查看部署的mq容器id，在通过 `docker exec -it rabbitmq /bin/bash` 进入容器内部在
运行：`rabbitmq-plugins enable rabbitmq_management`

### 2.Stats in management UI are disabled on this node

```bash
#进入rabbitmq容器
docker exec -it {rabbitmq容器名称或者id} /bin/bash

#进入容器后，cd到以下路径
cd /etc/rabbitmq/conf.d/

#修改 management_agent.disable_metrics_collector = false
echo management_agent.disable_metrics_collector = false > management_agent.disable_metrics_collector.conf

#退出容器
exit

#重启rabbitmq容器
docker retart {rabbitmq容器id}
```



![在这里插入图片描述](E:\Development\Typora\images\20210713142438754.png)

现在可以通过访问http://linuxip:15672，访问web界面，这里的用户名和密码默认都是guest
输入命令：exit退出容器目录.



### 3.添加用户权限

![image-20220813005602109](E:\Development\Typora\images\image-20220813005602109.png)

点进去然后授予对应权限

![image-20220813005620340](E:\Development\Typora\images\image-20220813005620340.png)

### 4.exchange名

/bookcollection和bookcollection是特么两个exchange!!!

我特么人傻了！！！

一定要先创建exchange在运行java程序！！！


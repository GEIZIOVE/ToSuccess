# MQTT.fx使用

# 使用说明

mqtt.fx打开后的主页面如下：
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70.png)
点击齿轮进行连接设置
![在这里插入图片描述](E:\Development\Typora\images\20200722091312247.png)
本地连接设置：
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-16631749403081.png)
用户信息设置：
![在这里插入图片描述](E:\Development\Typora\images\20200722091836758.png)
SSL安全证书设置：
![在这里插入图片描述](E:\Development\Typora\images\20200722091906976.png)
网络代理设置：
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-16631749403092.png)
遗嘱设置：
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-16631749403093.png)

# 连接测试

## 1、启动mosquitto

我是直接使用虚拟机在本机上进行测试的，虚拟机中的系统是Ubuntu18.04
首先启动mosquitto

```bash
mosquitto -v
1
```

如图：
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-16631749403094.png)
查看虚拟机的IP地址，下一步配置使用
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-16631749403095.png)

## 2、在主机中打开MQTT.FX软件

设置连接信息
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-16631749403096.png)
IP为mosquitto所在的IP，端口号默认为1883。
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-16631749403107.png)
点击进行连接
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-16631749403108.png)
连接成功以后可以进行发布订阅。
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-16631749403109.png)

## 3、发布测试

先在虚拟机中订阅主题：
不了解的参考此篇文章：[参考连接](https://blog.csdn.net/qq_33406883/article/details/107429946)
![在这里插入图片描述](E:\Development\Typora\images\20200805170419298.png)
在MQTT.fx端进行发布：
消息主题为：nihao
消息内容：hello MQTT
![在这里插入图片描述](E:\Development\Typora\images\20200805170159397.png)
订阅端收到消息：
![在这里插入图片描述](E:\Development\Typora\images\20200805170625958.png)

## 4、订阅测试

在MQTT.fx中订阅主题 nihao
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-166317494031110.png)
在虚拟机中发布一个主题nihao,消息内容为helloworld
![在这里插入图片描述](E:\Development\Typora\images\2020080517113423.png)
MQTT.fx收到订阅的消息：
![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNDA2ODgz,size_16,color_FFFFFF,t_70-166317494031111.png)
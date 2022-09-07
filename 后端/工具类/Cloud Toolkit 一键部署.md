# Cloud Toolkit: 一键部署

# 部署

## 一.部署到Linux服务器

### 1.添加服务器

1. 在IntelliJ IDEA顶部菜单栏中选择***\*Tools\** > \**Alibaba Cloud\** > \**Alibaba Cloud View\** > \**Host\****。

2. 在弹出的**Host**页签中单击**Add Host**。![add host](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/6127662061/p111868.png)

3. 在**Add Host**对话框中设置**Host List**、**Username**、**Password**和**Tag**等参数，完成后单击**Add**。

   ![add host](E:\Development\Typora\images\p111879.png)





### 2.部署应用

1. 在IntelliJ IDEA顶部菜单中选择***\*Alibaba Cloud\** > \**Deploy to Host...\****。

2. 在**Deploy to Host**对话框设置部署参数，然后单击**Run**。

   ![image-20220725010117310](E:\Development\Typora\images\image-20220725010117310.png)

   部署参数说明如下表所示：

   | 参数                 | 描述                                                         |
   | :------------------- | :----------------------------------------------------------- |
   | **File**             | **Maven Build**：若当前工程采用Maven构建，可以直接构建并部署。**Upload File**：若当前工程不是采用Maven构建，或在本地已存在打包好的部署文件，可以选择并上传本地的部署文件。**Gradle Build**：若当前工程采用Gradle构建，选择此项来构建并部署。 |
   | **Target Host**      | 在下拉列表中选择Tag，然后在该Tag中选择要部署的服务器。       |
   | **Target Directory** | 输入在服务器上的部署路径，如`/root/tomcat/webapps`。         |
   | **Command**          | 输入应用启动命令，如`sh /root/restart.sh`。                  |

> 1.Name 随便填，只是一个配置发布名称
>
> 2.选中你要发布到哪台服务器，需要在 Alibaba Cloud Explorer 中配置了这里才会出现
>
> 3.Target Directory 你要把这个Jar包上传到哪个目录，要填正确
> 我这边上传的目录是：/data/server/project
> 注意两点：
> 1.需要存在这个目录，不会自动创建，
> 2.配置的用户要有写入权限
>
> 4.After deploy 部署后 需要执行的文件 这里需要配置一个脚本用来重启项目。
> 可以单独使用命令，也可以使用已经上传的shell脚本。



Spring Boot应用的Command命令

若将Linux系统的/root/springbootdemo目录作为Spring Boot应用运行的基目录，则需将Spring Boot应用的JAR包部署到/root/springbootdemo目录下。

对应的Command配置为：

```bash
sh /root/sh/restart-springboot.sh
```

restart-springboot.sh脚本为：

```bash
source /etc/profile
netstat -anp|grep 端口号|awk '{printf $7}'|cut -d/ -f1 |xargs kill -9 || true
nohup java -jar /root/springbootdemo/springbootdemo-0.0.1-SNAPSHOT.jar > nohup.log 2>&1 &


source /etc/profile
killall java
nohup java -jar /usr/local/dk/myProject/BookCollection-0.0.1-SNAPSHOT.jar > nohup.log 2>&1 &
```

## 二.微服务部署

### 1.创建机器（Host）

1. 在IntelliJ IDEA中打开您的工程。

2. 在IntelliJ IDEA顶部菜单栏中选择***\*Tools\** > \**Alibaba Cloud\** > \**Alibaba Cloud View\** > \**Host\****。![image-20220725010545060](E:\Development\Typora\images\image-20220725010545060.png)

3. 单击右侧**Add Host**，在**Add Host**页面新增机器。![add host](E:\Development\Typora\images\p107980.png)

   配置参数说明如下。

   | 参数            | 描述                                                         |
   | :-------------- | :----------------------------------------------------------- |
   | **Host List**   | 机器IP地址，若有多台机器，换行输入每个IP地址。               |
   | **Port**        | 机器端口                                                     |
   | **SSH Profile** | SSH密钥**Create new profile**：创建新的私钥，您需要设置**Profile Name**、**Method**、**Username**和**Password**。**Method**为选择登录方式，可选择**Password**或**Select a Private Key**的方式登录。**Use exit profile**：使用已存在的私钥。关于SSH密钥详情请参见[用户数据隐私说明](https://help.aliyun.com/document_detail/100343.htm#concept-100343-zh)。 |

   **说明** Host参数配置完成后，建议单击**Test Connection**测试机器是否连接成功。若有多台机器IP，只测试第一台机器的连接状态。

4. 确认配置参数，单击**Add**。

### 2.部署方式

1. 在IntelliJ IDEA界面右上方选择框中单击**Edit Configuration...**。![image-20220725010908203](E:\Development\Typora\images\image-20220725010908203.png)

2. 在**Run/Debug Configuration**页面单击左上角**+**，选择**Deploy to Host**。![Deploy to Host](E:\Development\Typora\images\p107960.png)

   部署参数说明如下表所示：

   | 参数                 | 描述                                                         |
   | :------------------- | :----------------------------------------------------------- |
   | **Name**             | 部署名，建议以英文命名。                                     |
   | **File**             | **Maven Build**：若当前工程采用Maven构建，可以直接构建并部署。**Upload File**：若当前工程不是采用Maven构建，或在本地已存在打包好的部署文件，可以选择并上传本地的部署文件。**Gradle Build**：若当前工程采用Gradle构建，可以直接构建并部署。 |
   | **Target Host**      | 目标主机。                                                   |
   | **Target Directory** | 目标部署微服务路径。                                         |
   | **After deploy**     | 设置部署上传后执行的命令。                                   |
   | **Before launch**    | 打包部署工程。                                               |

3. 确认部署参数，先单击**Apply**，然后单击**OK**。

## 三.部署多模块工程

Cloud Toolkit可以用于部署多模块工程中的某个子模块的场景。本文档将以在IntelliJ IDEA中部署Meetup多模块工程中的Consumer子模块到SAE为例介绍部署方法。

### 背景信息

若您有一个Meetup多模块工程，结构为：

- Consumer
- Provider
- Provider-api

其中Consumer模块和Provider模块均为Meetup工程的子模块，且都依赖于Provider-api模块。

### 部署多模块工程中的子模块

1. 在IntelliJ IDEA界面左侧的Project中右键单击Meetup工程，在快捷菜单中选择***\*Alibaba Cloud\** > \**Deploy to SAE...\****。

2. 在**Deploy to SAE**对话框中设置部署参数。

   ![deploy to sae](E:\Development\Typora\images\p111270.png)

   **说明** 若您尚未未在SAE上创建应用，可在对话框右上角单击**Create Serverless Application on SAE Console**，跳转到SAE控制台创建应用。

   部署参数说明如下。

   | 参数                    | 参数                                                         | 描述                                                         |
   | :---------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
   | 应用信息（Application） | **Region**                                                   | 应用所在地域。                                               |
   | **Namespace**           | 应用所在命名空间。                                           |                                                              |
   | **Application**         | 应用名称。                                                   |                                                              |
   | 部署方式（Deploy File） | **Maven Build**                                              | 选择Maven Build方式来构建应用时，系统会默认添加一个Maven任务来构建部署包。如果您需要部署多模块工程中的一个子模块，请参见[部署多模块工程中的子模块](https://help.aliyun.com/document_detail/100310.html)。 |
   | **Upload File**         | 选择Upload File方式来构建应用时，选择上传您的WAR包或者JAR包，然后进行部署。 |                                                              |
   | **Image**               | 选择Image方式来构建应用时，需要填入一个镜像地址，然后进行部署。 |                                                              |
   | **Gradle Build**        | 选择Gradle Build方式来构建应用时，可以直接构建并部署。       |                                                              |

   **说明** 若您已使用 Jar/War 包部署应用，使用 Cloud Toolkit 部署应用时只能选择 Maven Build 或 Upload File 两种部署方式；若您已使用镜像部署应用，使用 Cloud Toolkit 部署应用时只能选择 Image 部署方式。

3. 对Meetup父工程执行`mvn clean install`命令（默认执行）。![父工程](E:\Development\Typora\images\p111620.png)

4. 对Consumer子工程执行`mvn clean package`命令。

   1. 在**Deploy to EDAS**对话框的**Before launch**区域单击**+**。
   2. 在**Add New Configuration**菜单中选择**Run Maven Goal**。![run maven goal](E:\Development\Typora\images\p111607.png)
   3. 在**Select Maven Goal**对话框中单击文件夹图标选择Consumer子模块，在**Command line**栏输入clean package，然后单击**OK**。![select maven goal](E:\Development\Typora\images\p111609.png)

5. 先单击**Apply**，然后单击**Run**。

### 结果验证

部署开始后，IntelliJ IDEA的**Console**区域会打印部署日志。您可以根据日志信息检查部署结果。

您还可以登录SAE控制台，在部署应用的基本信息页面查看部署结果。



## 四. 多目录上传Multirun Deployment

上述的**Deploy To Host上传的原文件只能是单一目录或者文件**，如果我部署时候需要上传多个目录，则需要用到 Multirun Deployment。

比如我希望先上传脚本，给权限，然后上传jar包，启动，需要通过Multirum Deployment实现。

![img](E:\Development\Typora\images\watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAY3NkbmNqaA==,size_12,color_FFFFFF,t_70,g_se,x_16.png)

 ![img](E:\Development\Typora\images\watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAY3NkbmNqaA==,size_20,color_FFFFFF,t_70,g_se,x_16.png)

**先配置好单个Deploy To Host操作，然后将它们在Multrum Deployment中组合**

### 4.1 配置好单个Deploy To Host操作

deploy_sh的Deploy To Host

![img](E:\Development\Typora\images\watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAY3NkbmNqaA==,size_20,color_FFFFFF,t_70,g_se,x_16-16586833995479.png)

deploy-demo4的Deploy To Host

![img](E:\Development\Typora\images\watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAY3NkbmNqaA==,size_20,color_FFFFFF,t_70,g_se,x_16-165868339954710.png)

### 4.2 配置**Multrum Deployment操作**

demo4-ullt的Multrum Deployment

![img](E:\Development\Typora\images\watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAY3NkbmNqaA==,size_20,color_FFFFFF,t_70,g_se,x_16-165868339954711.png)

### 4.3 保存配置

点击Apply保存部署配置



### 4.4 启动部署

在run Configurations处可以找到和修改部署配置。

![img](E:\Development\Typora\images\6aec29c2f1e449bcb43cbcd1cf6c4183.png)



# 最佳实践

## 1.使用Cloud Toolkit查看远程服务器按日滚动的日志文件

日志是检测系统运行正常的重要手段，以及收集程序运行状态数据的重要来源。在开发过程中，通常会根据日期来格式化日志文件名，便于查看和检索数据。但由于每天的操作日志不同，导致需要经常手动修改命令。本文为您介绍如何使用Cloud Toolkit完成一次性配置，提高部署效率。

## 操作步骤

本文以日志文件格式`/root/xxx/log/eureka-2020-05-21.log`为例进行阐述。

1. 在IntelliJ IDEA中打开您的工程。
2. 在IntelliJ IDEA顶部菜单栏中选择***\*Tools\** > \**Alibaba Cloud\** > \**Deploy to Host...\****。
3. 在**Deploy to Host**页面选择**Advanced**。![advanced](E:\Development\Typora\images\p110000.png)
4. 在**Command**命令框中输入`tail -f /root/xxx/log/eureka-$(date +%Y-%m-%d).log`。
5. 确认执行，单击**Apply**，然后单击**Run**即可。
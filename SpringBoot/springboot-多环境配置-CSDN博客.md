本文链接：https://blog.csdn.net/weixin_52986315/article/details/125596202

### 

- [Spring Boot Profiles](https://blog.csdn.net/weixin_52986315/article/details/125596202?ops_request_misc=&request_id=&biz_id=102&utm_term=SpringBoot系列之profiles配置&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduweb~default-1-125596202.185^v2^tag_show&spm=1018.2226.3001.4450#Spring_Boot_Profiles_1)
- - [profile配置方式](https://blog.csdn.net/weixin_52986315/article/details/125596202?ops_request_misc=&request_id=&biz_id=102&utm_term=SpringBoot系列之profiles配置&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduweb~default-1-125596202.185^v2^tag_show&spm=1018.2226.3001.4450#profile_14)
  - [profile激活方式](https://blog.csdn.net/weixin_52986315/article/details/125596202?ops_request_misc=&request_id=&biz_id=102&utm_term=SpringBoot系列之profiles配置&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduweb~default-1-125596202.185^v2^tag_show&spm=1018.2226.3001.4450#profile_57)



# Spring Boot Profiles

Profile的是配置文件的意思，我们在开发Spring Boot应用时，通常同一个项目会被安装到不同的环境,而不同的环境又需要不同的配置。比如：

- 开发环境，应用需要连接一个可供调试的数据库单机进程
- 生产环境，应用需要使用正式发布的数据库，通常是高可用的集群
- 测试环境，应用只需要使用内存式的模拟数据库

其中数据库地址、服务器端口等等配置都不同，如果每次打包时，都要修改配置文件，那么就会非常麻烦。

Spring[框架](https://so.csdn.net/so/search?q=框架&spm=1001.2101.3001.7020)提供了profile的管理功能，我们可以使用profile功能来区分不同环境的配置。然后可以通过激活、指定参数等方式快速动态的切换环境。

## profile配置方式

**1) 多文件方式**
新建多个配置文件，命名格式：`application-环境名.yml`
![在这里插入图片描述](E:\Development\Typora\images\f34f24ffa22c42a18898b6dc6682dc10.png)

在`application.ym`l中 配置想要上线的环境（当选择的文件和application.yml文件存在相同的配置时，application.yml中的配置会被覆盖掉）

```java
spring:
 profiles:
   active: dev #需要使用的配置文件的后缀
```

`application-dev.yml` 开发环境

```yml
server:
  port: 8081
```

` application-pro.yml` 生产环境

```yml
server:
  port: 8082
```

 `application-test.yml `测试环境

```yml
server:
  port: 8083
12
```

**2) yml多文档方式**
该方式只需要一个`application.yml`配置文件即可，在配置文件中使用 — （三个横杠）来分隔不同的环境配置

```yml
---
server:
  port: 8081
spring:
  profiles: dev
---
server:
  port: 8082
spring:
  profiles: pro
---
server:
  port: 8083
spring:
  profiles: test
---
```

## profile激活方式

**1) 配置文件**
在yml配置文件中配置：

```java
spring:
  profiles:
    active: test
```

**2) 虚拟机参数**
在VM options指定：`-Dspring.profiles.active -dev`![在这里插入图片描述](E:\Development\Typora\images\8a90e6458e974252919289538822cf93.png)
紧接着启动该项目，你会在控制台看到此时的环境就是你刚才设置的开发环境，覆盖了我们在配置文件中的激活配置
![在这里插入图片描述](E:\Development\Typora\images\8169b9fb2be94ffc8d627356de0977c6.png)

通过命令行连续的两个中划线 `--`，后面接 `配置项=配置值` 的方式，修改配置文件中对应的**配置项**为对应的**配置值**。例如说，`--配置项=配置值`。如果希望修改多个配置项，则使用多组 `--` 即可。例如说，`--配置项1=配置值1 --配置项2=配置值2`。

**3) 命令行参数**
第一种：`--spring.profiles.active=pro`
![在这里插入图片描述](E:\Development\Typora\images\e5fbcb4e41324a1da27de8aaa76a8275.png)
第二种：`java -jar xxx.jar --spring.profiles.active =test`
使用maven打包项目，打开该jar包所在目录，接着启动该项目，不会可以参考这个>>>[Spring Boot的启动方式](https://blog.csdn.net/weixin_52986315/article/details/125542132?spm=1001.2014.3001.5501)
![在这里插入图片描述](E:\Development\Typora\images\ec4974f5c35949c18e299452b9b840ba.png)



## 配置文件加载顺序

### 内部配置文件加载顺序

> Spring Boot程序启动时，会从以下位置加载配置文件：
>
> 1. 项目根目录：当前项目下的/config目录下
> 2. 项目根目录：当前项目的根目录下
> 3. classpath：classpath的/config目录下
> 4. classpath：classpath的根目录下
>
> 加载顺序为上面的排列顺序，高[优先级](https://so.csdn.net/so/search?q=优先级&spm=1001.2101.3001.7020)配置文件的属性会生效

```
注意：优先级高的配置文件只覆盖优先级低的配置文件中的重复项。低级配置文件的独有项仍然有效。
```

目录结构如下：
![在这里插入图片描述](E:\Development\Typora\images\0aa3cde74eeb47278f6860f44fac8e56.png)



### 外部配置文件加载顺序

> 通过指定配置spring.config.location来改变默认配置，一般在项目已经打包后，我们可以通过指令来加载外部文件的配置：
> `java -jar xxx.jar --spring.config.location=e://Java/application.yml`
> ![在这里插入图片描述](E:\Development\Typora\images\1195f148eeed480aadb184be5d501cd7.png)
>
> 

​			当然如果你觉得在命令行指定外部配置文件的位置太麻烦，那么我再告诉你种方法，那就是在你想启动的项目jar包所在的文件夹下新建一个`application.yml`配置文件，或者在该文件夹下新建一个config的文件夹并在config文件夹下新建一个`application.yml`配置文件。

![在这里插入图片描述](E:\Development\Typora\images\32fb4e901b9f41feb88bb107335e1d30-16579716273514.png)

这时候该项目就会自动读取该配置文件，如果两个同时存在，他们也是有优先级的，config文件下的yml文件是优先于与jar包同级的yml文件。

更详细的介绍可以查看[Spring Boot 中文文档](https://www.springcloud.cc/spring-boot.html)
# 第二章 Maven的安装与配置

## 2.1安装目录分析

- bin：该目录包含了mvn的运行脚本
  - mvn：基于UNIX平台的shell脚本
  - mvn.bat:基于Windows平台的bat脚本
    - 在命令行输入任何一条mvn命令时， 实际上就是在 调用这些脚本。
  - mvnDebug 和mvnDebug. bat：前者是UNIX平台的shell脚本， 后者是Windows平台的bat脚本，两者基本一样，mvnDebug多了一条MAVEN_DEBUG_OPTS配置， 其作用就是在运行 Maven 时开启 debug， 以便调试Maven 本身
  - m2. conf：classworlds的配置文件
- boot：该目录只包含一个文件， 以 maven 3. 0为例， 该文件为 plexus-classworlds-2.2.3jar
  - plexus-classworlds 是一个类加载器框架， 相对于默认的 java 类加载器， 它提供了更丰富的语法以方便配置， Maven 使用该框架加载自己的类库。**一般人不考虑**
- conf:
  - settings. xml:我们更偏向于复制该文件至～/.m2/目录下（～表示用户目录）， 然后修改该文件， 在用户范围定制 Maven 的行为。
- lib：该目录包含了所有 Maven 运行时需要的 Java 类库



# 第三章 Maven入门

## 3.1 POM

POM(Porject Object Model,项目对象模型)



```xml
<?xml version = "1.0 " encoding = "UTF-8"?>
<project xmlns = "htp : //maven. apache.org /POM 4.0.0"
    xmlns:xsi = "http ://www . w3.org /2001/8MLSchema-instance"xsi 		:schemaLocat ion = "http :// maven.apache.org/ POM/4. 0.C
http : //maven. apache.org / maven-v4_0._0 . xsd" >
	<modelversion >4.0.0 </ modelversion >
	<groupId >com.juvenxu.mvnbook < /groupId >
	<artifactId> hello-world < /artifactId >
	<version> 1.0-SNAPSHOT < /version >	
    < name > Maven Hello wor1d Project < /name >
</project >
```



project是所有pom. xml的根元素，

​	**modelVersion**指定了当前POM模型的版本，对于Maven 2及
Maven 3来说，它只能是4.0.0。( so important)

​	**groupId**定义了项目属于哪个组

​	**artifactId**定义了当前Maven项目在组中唯一的ID





## 3.2 编写主代码

Java代码约定：

​	1.项目主代码放到src/main/java目录下

​	2.项目中Java类的包都应该基于项目的 groupId和artifactld



​	默认情况下，Maven构建的所有输出都在target/目录中，项目主代码编译至target/classes目录。



## 3.4打包和运行

​	POM默认打包类型为Jar，我们通过命令mvn clean package进行打包得到jar包后，如果有需要的话， 就可以复制这个jar文件到其他项目 的 Classpath中从而使用HelloWorld类。但是， 如何才能让其他的Maven项目直接引用这个 jar呢？还需要一个安装的步骤， 执行mvn clean **install**:，该命令将输出的jar包安装到Maven的本地仓库。



​	**默认打包生成的 jar 是不能够直接运行的**， 因为带有 main 方法的类信息不会添加到manifest 中（打开 jar 文件中的 META-INF/MANIFEST. MF 文件， 将无法看到 Main-Class 行）。



























































































# 第五章 坐标和依赖

## 5.1 何为Maven坐标

Maven的世界中拥有数量非常巨大的构件，也就是平时用的一些jar、war等文件

Maven的坐标元素包括 groupId 、artifactId、version、packaging、classifier

## 5.2坐标详解

- **groupId**：定义当前Maven项目隶属的实际项目。 【必须】
  1. Maven项目与实际的项目不一定是一一对应的关系
  2. groupId不应该对应项目隶属的组织或者公司。因为组织或者公司会有很多实际项目
  3. groupId的表示方式与Java包名的表示方式类似，通常与域名反向一一对应
- **artifactId：**定义实际项目中的一个Maven项目（模块）。 【必须】
  1. ​		<u>推荐使用实际项目的名称作为artifactId的前缀。</u>eg：account-email
  2. ​        默认情况下Maven生成的构件会以artifactId作为开头
- **version：**定义Maven项目当前所处的版本。 【必须】
- **packaging：**定义Maven项目的打包方式。【可选】
  - 打包方式会影响生命周期，比如命令不同
  - 默认打包方式为**jar**
- **classifier：**帮助定义构建输出的一些附属构件。 【不可定义】
  - 不能直接定义项目的classifier，因为附属构件不是项目直接默认生成的，而是由附加的插件帮助生成。



## 5.4 依赖的配置

其实一个依赖声明可以包含如下的一些元素：

```xml
<project >
    <dependencies >
        <dependency >
            <groupId >.. < grouprd >
            <artifactid > ... < /artifactid >
            <version > ...< /version >
            <type > ... </type >  //依赖的类型对应于项目坐标定义的packaging,默认为jar
            <scope > ... < /scope > //标记依赖是否可选
            <optional > </ optional >
            <exclusions >  //用来排除传递性依赖
            	<exclusion >< /exclusion>
             </exclusions >
        </ dependency >				
    </dependencies >
</ project >
…

```





## 5.5 依赖范围

前置知识

> ​	首先需要知道， Maven在编译项目主代码的时候需要使用一 套classpath。 在上例中， 编译项目主代码的时候需要用到 spring-core，该文件以依赖的方式被引人到classpath中。 
>
> ​	其次，Maven在编译和执行测试的时候会使用另外一套classpath。 上例中的 JUnit就是一个很好的例子， 该文件也以依赖的方式引入到测试使用的classpath中， 不同的是这里的依赖范 围是test
>
> ​	 最后， 实际运行Maven项目的时候， 又会使用一 套classpath， 上例中的spring­core需要在该classpath中， 而JUnit则不需要。

![image-20220629175831737](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220629175831737.png)

![image-20220629175955544](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220629175955544.png)

![image-20220629180014452](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220629180014452.png)





## 5.6传递性依赖

### 5.6.2 传递性依赖和依赖范围

​		假设A依赖于B,B依赖于C，我们说A对于B是第一直接依赖，B对于C是第二直接依赖，A对于C是传递性依赖。第**一直接依赖的范围和第二直接依赖的范围决定了传递性依赖的范围**，如表5-2所示，最左边一行表示第一直接依赖范围，最上面一行表示第二直接依赖范围，中间的交叉单元格则表示传递性依赖范围。

![image-20220629180816081](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220629180816081.png)

​		仔细观察一下表5-2，可以发现这样的规律:	

​		1.当第二直接依赖的范围是**compile**的时候，传递性依赖的范围与第一直接依赖的范围一致;

​		2.当第二直接依赖的范围是test 的时候，依赖不会得以传递;

​		3.当第二直接依赖的范围是**provided**的时候，只传递第一直接依赖范围也为 provided 的依赖，且传递性依赖的范围同样为provided;

​		4.当第二直接依赖的范围是**runt-ime**的时候，传递性依赖的范围与第一直接依赖的范围一致，但compile例外，此时传递性依赖的范围为runtime。



## 5.7 依赖调节

​	原则一：路径最近者优先;

​			A- >B->C- >X(1.0) ,A- > D-> X(2.0)   则X（2.0）被引用

​	原则二：第一声明者优先



## 5.8 可选依赖

<optional>true</optional>

可选依赖不会传递，第一直接依赖需要自己显示声明该依赖

​	不建议使用



## 5.9排除依赖

<exclusions>

一个<exclusions>可以包含一个或者多个exclusion子元素，声明exclusion的时候只需要groupld和artifactId

## 5.10 归类依赖

![image-20220629185111363](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220629185111363.png)



# 第六章 仓库

​	自己重新看书



# 第七章 生命周期和插件





















































































# 第九章 Nexus

妈的 运行命令**nexus.exe/run** 老子服了

  <mirror>

​    <!--该镜像的唯一标识符。id用来区分不同的mirror元素。 -->

​    <id>maven-public</id>

​    <!--镜像名称 -->

​    <name>maven-public</name>

​    <!--*指的是访问任何仓库都使用我们的私服-->

​    <mirrorOf>*</mirrorOf>

​    <!--该镜像的URL。构建系统会优先考虑使用该URL，而非使用默认的服务器URL。 -->

​    <url>http://127.0.0.1:8081/repository/maven-public/</url>   

  </mirror>





  <server>

   <id>SNAPSHOT</id>

   <username>admin</username>

   <password>root</password>

  </server>

  <server>

   <id>Releases</id>

   <username>admin</username>

   <password>root</password>

  </server>





[nexus搭建maven私服_一个运维小青年的博客-CSDN博客](https://blog.csdn.net/tianmingqing0806/article/details/125334844#:~:text=启动方式（2种）： 2.1 start命令启动（后台进程形式） 在,%2Fnexus%2Fnexus-3.38.1-01%2Fbin 目录下， 执行脚本命令，以后台进程的形式（不占用当前命令终端窗口），启动 Nexus 服务：)

[使用Nexus搭建Maven私服教程（附：配置并使用私服教程） (hangge.com)](https://www.hangge.com/blog/cache/detail_2844.html)

[Nexus私服搭建、配置、上传_⊙ω⊙ 在学习的路上越走越远～～～的博客-CSDN博客_nexus upload](https://blog.csdn.net/qq_43070052/article/details/123436959)

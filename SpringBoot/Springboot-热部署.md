## **1、开启IDEA的自动编译（静态）**

**具体步骤：打开顶部工具栏 File -> Settings -> Build,Execution,Deployment -> Compiler 然后勾选 Build project automatically 。**

**![img](E:\Development\Typora\images\2919314-20220702115833449-2089172900.png)**

 

## 2、 开启IDEA的自动编译（动态）

**具体步骤：打开顶部工具栏 File -> Settings -> \**Advanced Settings\** -> Compiler -> 然后勾选** **Allow auto-make to start even if developed application is currently running。**

 

![img](E:\Development\Typora\images\2919314-20220702120030735-1418715619.png)

 

##  3、开启IDEA的热部署策略（非常重要）

**具体步骤：点击Edit COnfigurations...进入**

![img](E:\Development\Typora\images\2919314-20220702120323919-621148278.png)

 

**选择Modif options -> On 'Updata' actrion -> Update classes and resources**

 

 ![img](E:\Development\Typora\images\2919314-20220702120853858-1195310799.png)

## 4、在pom.xml文件中导入热部署插件

## 1.添加devtools jar包到工程

[![复制代码](E:\Development\Typora\images\copycode.gif)](javascript:void(0);)

```
<!--devtools 热部署依赖 -->
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
<!--optional 表示依赖是否向下传递 true表示不向下传递  默认值是false向下传递 -->
            <optional>true</optional>
</dependency>
```

[![复制代码](E:\Development\Typora\images\copycode.gif)](javascript:void(0);)

### 2.添加plugin到pom.xml

[![复制代码](E:\Development\Typora\images\copycode.gif)](javascript:void(0);)

```
<build>
    <plugins>
        <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <fork>true</fork>
          <addResources>true</addResources>
        </configuration>
      </plugin>
    </plugins>
 </build>
```

[![复制代码](E:\Development\Typora\images\copycode.gif)](javascript:void(0);)

## 5、测试

通过快捷键：Ctrl+shift+F9启动热部署。

为了测试配置的热部署是否有效，接下来，在不关闭当前项目的情况下，将 DemoController 类中的请求处理方法 info() 的返回值从”Spring Boot“修改为 “Spring Boot-Suzbuing” 并保存，

通过快捷键，查看控制台信息会发现项目能够自动构***建和编译，说明项目热部署生效。***

```
@RequestMapping("/info")
    public String info() {
        return "Spring Boot";
    }
```

***![img](E:\Development\Typora\images\2919314-20220702121633246-356150417.png)***

 
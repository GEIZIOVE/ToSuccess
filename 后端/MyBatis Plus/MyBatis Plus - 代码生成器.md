# MyBatis Plus - 代码生成器

[代码生成器配置 | MyBatis-Plus](https://www.mybatis-plus.com/config/generator-config.html)

## 1.添加maven依赖

```xml
<!-- mybatis-plus代码生成器的依赖-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>

<!-- velocity 模板引擎, 默认 -->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.2</version>
</dependency>

<!-- freemarker 模板引擎 -->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.28</version>
</dependency>

<!--Mysql驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

```



## 2.代码生成器配置

### 2.1数据源 `dataSourceConfig` 配置

##### 基础配置

| 属性     | 说明       | 示例                                     |
| -------- | ---------- | ---------------------------------------- |
| url      | jdbc路径   | jdbc:mysql://127.0.0.1:3306/mybatis-plus |
| username | 数据库账号 | root                                     |
| password | 数据库密码 | 123456                                   |

##### 可选配置

| 方法                              | 说明                         | 示例                       |
| --------------------------------- | ---------------------------- | :------------------------- |
| dbQuery(IDbQuery)                 | 数据库查询                   | new MySqlQuery()           |
| schema(String)                    | 数据库schema(部分数据库适用) | mybatis-plus               |
| typeConvert(ITypeConvert)         | 数据库类型转换器             | new MySqlTypeConvert()     |
| keyWordsHandler(IKeyWordsHandler) | 数据库关键字处理器           | new MySqlKeyWordsHandler() |

```java
//数据源配置
DataSourceConfig dsc = new DataSourceConfig();
dsc.setUrl("jdbc:mysql://106.53.124.182:3306/Book_Collection_System?useUnicode=true&useSSL=false&characterEncoding=utf8");
dsc.setUsername("root");
dsc.setPassword("root");
dsc.setDbType(DbType.MYSQL);// 数据库类型 默认为 MYSQL
dsc.setDriverName("com.mysql.cj.jdbc.Driver"); // 设置驱动名称
//dsc.setSchemaName("public"); 
//dsc.setTypeConvert(new MySqlTypeConvert()); // 自定义数据库类型转换

```



### 2.2数据库表配置

```java
// 数据库表配置
StrategyConfig strategy = new StrategyConfig();
strategy.setNaming(NamingStrategy.underline_to_camel); // 表名生成策略
strategy.setColumnNaming(NamingStrategy.underline_to_camel); // 字段名生成策略
strategy.setExclude(new String[]{"appeal", "book","user"}); // 排除生成的表
strategy.setEntityLombokModel(true); // 【实体】是否为lombok模型（默认 false）
strategy.setRestControllerStyle(true); //【实体】是否为链式模型（默认 false）
strategy.setVersionFieldName("version"); // 乐观锁属性名称
strategy.setLogicDeleteFieldName("is_delete"); // 逻辑删除属性名称
strategy.setRestControllerStyle(true); //生成 @RestController 控制器
strategy.setSuperEntityClass("com.hong.dk.entity.BaseEntity"); //设置继承的实体类 没有就不用设置
strategy.setSuperEntityColumns("create_time,update_time,version,is_delete"); // 需要被继承的字段名称
mpg.setStrategy(strategy);
```



### 2.3包名配置

```java
// 包名配置
PackageConfig pc = new PackageConfig();
pc.setModuleName("bookcollect");
pc.setParent("com.hong.dk");
//        pc.setEntity("entity");
//        pc.setMapper("mapper");
//        pc.setXml("mapper");
//        pc.setService("service");
//        pc.setServiceImpl("service.impl");
//        pc.setController("controller");
mpg.setPackageInfo(pc);
```



### 2.4全局策略 `globalConfig` 配置

```java
// 全局配置
//所有字段如果没有特别说明都默认为false
GlobalConfig gc = new GlobalConfig();
String projectPath = System.getProperty("user.dir"); // 获取当前项目路径
gc.setOutputDir(projectPath + "/src/main/java"); // 设置输出目录
gc.setFileOverride(true); // 是否覆盖文件
gc.setOpen(false); // 是否打开生成的文件目录，默认为true
gc.setEnableCache(false); // XML 二级缓存 默认为false
gc.setAuthor("wqh"); // 设置开发人员
gc.setKotlin(false); // 开启 Kotlin 模式 默认为false
gc.setSwagger2(true);  //实体属性 Swagger2 注解
gc.setActiveRecord(false);// 不需要ActiveRecord特性的请改为false
gc.setBaseResultMap(true);// XML是否生成ResultMap映射
gc.setBaseColumnList(false);// XML是否生成ColumnList列表
gc.setEntityName("%s");//设置实体名，占位符是%s，会将实体名替换进去
gc.setMapperName("%sMapper");//设置Mapper名，占位符是%s，会将实体名替换进去
gc.setXmlName("%sMapper");//设置Mapper XML名，占位符是%s，会将实体名替换进去
gc.setServiceName("%sService");//设置Service名，占位符是%s，会将实体名替换进去
gc.setServiceImplName("%sServiceImpl");//设置Service实现名，占位符是%s，会将实体名替换进去
gc.setControllerName("%sController");//设置Controller名，占位符是%s，会将实体名替换进去
gc.setIdType(IdType.AUTO);//主键策略 AUTO->自增，INPUT->用户输入，NONE->不需要主键,ASSIGN_UUID->UUID，ASSIGN_ID->自增
mpg.setGlobalConfig(gc);
```



具体参见[代码生成器配置 | MyBatis-Plus](https://www.mybatis-plus.com/config/generator-config.html)



### 2.5自定义配置

```java
// 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // 如果模板引擎是 freemarker
        //String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
         String templatePath = "/templates/mapper.xml.vm";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/src/main/resources/mapper/" + pc.getModuleName()
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });

        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
                // 判断自定义文件夹是否需要创建
                checkDir("调用默认方法创建的目录，自定义目录用");
                if (fileType == FileType.MAPPER) {
                    // 已经生成 mapper 文件判断存在，不想重新生成返回 false
                    return !new File(filePath).exists();
                }
                // 允许生成模板文件
                return true;
            }
        });


        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);


    /**
     * 输出模板，如果按照官方原生的可以不配置，也可以配置自定义的模板
     */
//        TemplateConfig templateConfig = new TemplateConfig();
//        // 配置自定义输出模板
//        //指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别，默认vm，xml不输出
//        templateConfig.setEntity("template/entity.java");
//        templateConfig.setService("myTemplates/service.java");
//        templateConfig.setServiceImpl("myTemplates/serviceImpl.java");
//        templateConfig.setController("myTemplates/controller.java");
//        templateConfig.setMapper("myTemplates/mapper.java");
//        templateConfig.setXml(null);
//        mpg.setTemplate(templateConfig);


```





## 3.自定义输出模板

​		`mybatis-plus-generator`的jar包里面有原生的模板文件，在jar包的`templates`目录下：

btl对应的是beetl模板引擎

ftl对应freemarker模板引擎，

vm对应的是velocity模板引擎，默认是vm，如果想更改使用的引擎可以通过setTemplateEngine方法设置，模板中的全局变量可以参考下面方法里面的变量`com.baomidou.mybatisplus.generator.engine.AbstractTemplateEngine#getObjectMap`

![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdobWowMQ==,size_16,color_FFFFFF,t_70.png)

 		如果不想使用原生的模板，可以编写自己的模板，基于原生模板修改，新的模板要放在resources下面，注意写的时候不要加上模板文件的后缀，会根据引擎自动识别，如果不想输出对应的文件，需要设置为null，否则会拿jar包下面原生的模板生成。



```java
//指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别，默认vm，xml不输出
templateConfig.setEntity("myTemplates/entity.java");
templateConfig.setXml(null);
```

![img](E:\Development\Typora\images\2019073010225485.png)



## 4.关于日志输出

 在代码生成器中实际上是有生成过程的日志输出的，如果配置不正确也会有对应的提示，但是在很多的教程里面都没有提到这一点，控制台都提示log4j异常了。

![img](E:\Development\Typora\images\20190730103650533.png)

 关于日志，实际上在模板引擎中就有log4j的api包引入了，但是log4j还需要有对应的实现才能输出日志，所以会提示上图的异常，因此在上面我的依赖中还加入了slf4j-log4j12的实现依赖

![img](E:\Development\Typora\images\20190730103753254.png) ![img](E:\Development\Typora\images\20190730103730455.png)

这时候我们再来运行，发现还会报错，这是因为缺少log4j的properties文件，下面我们建立一个log4j.properties文件，注意文件名不要改，就叫这个名字，配置内容参考下面的，也可以用你常用的，具体配置的属性自行查找吧

 

![img](E:\Development\Typora\images\2019073010403094.png)

![img](E:\Development\Typora\images\2019073010412018.png)

```properties
log4j.rootLogger=DEBUG, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

上面该补的补完之后我们就可以看到代码生成器的控制台日志了

![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdobWowMQ==,size_16,color_FFFFFF,t_70-16585155516873.png)

# SpringBoot-jasypt 配置文件加密

JDK版本-1.8

# 1.低版本2.x

先记录低版本的加密方式-----PBEWithMD5AndDES

## 1）引入jar包

```XML
<dependency>



            <groupId>com.github.ulisesbocchio</groupId>



            <artifactId>jasypt-spring-boot-starter</artifactId>



            <version>2.1.0</version>



        </dependency>
```

如上一个启动类，在联网的情况下就能自动下载其余加密需要的jar包。如果是在内网的情况下，可以手动在项目的lib目录引入所需jar包，如下：

```XML
<dependency>



    <groupId>org.jasypt</groupId>



    <artifactId>jasypt</artifactId>



    <version>1.9.2</version>



</dependency>



<dependency>



    <groupId>com.github.ulisesbocchio</groupId>



    <artifactId>jasypt-spring-boot</artifactId>



    <version>2.1.0</version>



</dependency>
```

## 2）生成密码

```java
@Test



    public void testEncrypt() {



        StandardPBEStringEncryptor standardPBEStringEncryptor = new StandardPBEStringEncryptor();



        EnvironmentPBEConfig config = new EnvironmentPBEConfig();



 



        config.setAlgorithm("PBEWithMD5AndDES");          // 加密的算法，这个算法是默认的



        config.setPassword("Angel");                        // 加密的密钥，随便自己填写，很重要千万不要告诉别人



        standardPBEStringEncryptor.setConfig(config);



        String plainText = "123456";         //自己的密码



        String encryptedText = standardPBEStringEncryptor.encrypt(plainText);



        System.out.println(encryptedText);



    }
```

如图，数据库密码为123456，密钥为Angel，加密算法也如上；

运行如图：4o+eS8OaWQ7HcjVgrkoX0A==

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX-W5u-WMluaIkOmjjl8=,size_20,color_FFFFFF,t_70,g_se,x_16.png)

##  3）测下解密

```java
@Test



    public void testDe() {



        StandardPBEStringEncryptor standardPBEStringEncryptor = new StandardPBEStringEncryptor();



        EnvironmentPBEConfig config = new EnvironmentPBEConfig();



 



        config.setAlgorithm("PBEWithMD5AndDES");



        config.setPassword("Angel");



        standardPBEStringEncryptor.setConfig(config);



        String encryptedText = "4o+eS8OaWQ7HcjVgrkoX0A==";   //加密后的密码



        String plainText = standardPBEStringEncryptor.decrypt(encryptedText);



        System.out.println(plainText);



    }
```

运行如图：

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX-W5u-WMluaIkOmjjl8=,size_20,color_FFFFFF,t_70,g_se,x_16-16601687854881.png)

## 4）yml配置

记好加密好的密码，在yml文件里将数据源那里用**ENC套上**，如下：

```XML
spring:



 datasource:



  username: root



  password: ENC(4o+eS8OaWQ7HcjVgrkoX0A==)



  url: jdbc:mysql://localhost:3306/test?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC



  driver-class-name: com.mysql.cj.jdbc.Driver



 



jasypt:



 encryptor:



  password: Angel



  algorithm: PBEWithMD5AndDES
```

## 5）测测登录

题外话，启动类上**无需**开启加密注解（@EnableEncryptableProperties）

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX-W5u-WMluaIkOmjjl8=,size_20,color_FFFFFF,t_70,g_se,x_16-16601687854892.png)

# 2.高版本 3.x

## 1)引入jar包

```XML
<dependency>



            <groupId>com.github.ulisesbocchio</groupId>



            <artifactId>jasypt-spring-boot-starter</artifactId>



            <version>3.0.3</version>



        </dependency>
```

与前面 一致，没有外网的条件，引入如下jar包

```xml
<!-- https://mvnrepository.com/artifact/org.jasypt/jasypt -->



<dependency>



    <groupId>org.jasypt</groupId>



    <artifactId>jasypt</artifactId>



    <version>1.9.3</version>



</dependency>



<!-- https://mvnrepository.com/artifact/com.github.ulisesbocchio/jasypt-spring-boot -->



<dependency>



    <groupId>com.github.ulisesbocchio</groupId>



    <artifactId>jasypt-spring-boot</artifactId>



    <version>3.0.3</version>



</dependency>
```

## 2）生成密码

这里用的是一个工具类，CV大法。

```java
package com.example.demo.utils;



 



/**



 * @author 杨帅帅



 * @time 2021/12/5 - 1:00



 */



import org.jasypt.encryption.pbe.PooledPBEStringEncryptor;



import org.jasypt.encryption.pbe.config.SimpleStringPBEConfig;



 



public class JasypUtil {



 



    private static final String PBEWITHHMACSHA512ANDAES_256 = "PBEWITHHMACSHA512ANDAES_256";



 



    /**



     * @Description: Jasyp 加密（PBEWITHHMACSHA512ANDAES_256）



     * @Author:      Rambo



     * @CreateDate:  2020/7/25 14:34



     * @UpdateUser:  Rambo



     * @UpdateDate:  2020/7/25 14:34



     * @param		 plainText  待加密的原文



     * @param		 factor     加密秘钥



     * @return       java.lang.String



     * @Version:     1.0.0



     */



    public static String encryptWithSHA512(String plainText, String factor) {



        // 1. 创建加解密工具实例



        PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();



        // 2. 加解密配置



        SimpleStringPBEConfig config = new SimpleStringPBEConfig();



        config.setPassword(factor);



        config.setAlgorithm(PBEWITHHMACSHA512ANDAES_256);



        // 为减少配置文件的书写，以下都是 Jasyp 3.x 版本，配置文件默认配置



        config.setKeyObtentionIterations( "1000");



        config.setPoolSize("1");



        config.setProviderName("SunJCE");



        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");



        config.setIvGeneratorClassName("org.jasypt.iv.RandomIvGenerator");



        config.setStringOutputType("base64");



        encryptor.setConfig(config);



        // 3. 加密



        return encryptor.encrypt(plainText);



    }



 



    /**



     * @Description: Jaspy解密（PBEWITHHMACSHA512ANDAES_256）



     * @Author:      Rambo



     * @CreateDate:  2020/7/25 14:40



     * @UpdateUser:  Rambo



     * @UpdateDate:  2020/7/25 14:40



     * @param		 encryptedText  待解密密文



     * @param		 factor         解密秘钥



     * @return       java.lang.String



     * @Version:     1.0.0



     */



    public static String decryptWithSHA512(String encryptedText, String factor) {



        // 1. 创建加解密工具实例



        PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();



        // 2. 加解密配置



        SimpleStringPBEConfig config = new SimpleStringPBEConfig();



        config.setPassword(factor);



        config.setAlgorithm(PBEWITHHMACSHA512ANDAES_256);



        // 为减少配置文件的书写，以下都是 Jasyp 3.x 版本，配置文件默认配置



        config.setKeyObtentionIterations( "1000");



        config.setPoolSize("1");



        config.setProviderName("SunJCE");



        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");



        config.setIvGeneratorClassName("org.jasypt.iv.RandomIvGenerator");



        config.setStringOutputType("base64");



        encryptor.setConfig(config);



        // 3. 解密



        return encryptor.decrypt(encryptedText);



    }



    



    public static void main(String[] args) {



        String factor = "Angel";



        String plainText = "123456";



 



        String encryptWithSHA512Str = encryptWithSHA512(plainText, factor);



        String decryptWithSHA512Str = decryptWithSHA512(encryptWithSHA512Str, factor);



        System.out.println("采用AES256加密前原文密文：" + encryptWithSHA512Str);



        System.out.println("采用AES256解密后密文原文:" + decryptWithSHA512Str);



    }



}
```

执行这个main方法，来，见证奇迹的时刻

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX-W5u-WMluaIkOmjjl8=,size_20,color_FFFFFF,t_70,g_se,x_16-16601687854893.png)

```css
Exception in thread "main" org.jasypt.exceptions.EncryptionOperationNotPossibleException: Encryption raised an exception. A possible cause is you are using strong encryption algorithms and you have not installed the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files in this Java Virtual Machine



	at org.jasypt.encryption.pbe.StandardPBEByteEncryptor.handleInvalidKeyException(StandardPBEByteEncryptor.java:1207)



	at org.jasypt.encryption.pbe.StandardPBEByteEncryptor.encrypt(StandardPBEByteEncryptor.java:996)



	at org.jasypt.encryption.pbe.StandardPBEStringEncryptor.encrypt(StandardPBEStringEncryptor.java:655)



	at org.jasypt.encryption.pbe.PooledPBEStringEncryptor.encrypt(PooledPBEStringEncryptor.java:465)



	at com.example.demo.utils.JasypUtil.encryptWithSHA512(JasypUtil.java:41)



	at com.example.demo.utils.JasypUtil.main(JasypUtil.java:78)
```

 这个错，很有意思，在加密的时候，手动配置了JCE的方法来支持加密的程序，然而jasypt3.x版本的对于jdk8自带的jce的包不兼容，需要升级一下，所以到网上下载jdk8对应的JCE包jce_policy-8。

下载后解压有两个jar包，如图：

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX-W5u-WMluaIkOmjjl8=,size_19,color_FFFFFF,t_70,g_se,x_16.png)

到该目录下替换即可：C:\Program Files\Java\jdk1.8.0_25\jre\lib\security

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX-W5u-WMluaIkOmjjl8=,size_20,color_FFFFFF,t_70,g_se,x_16-16601687854894.png) 然后运行main方法：8jLUdq0Fr7UhJGNwK/Nc6i6/WV4+UBpvtfBLDh4e3jZMJZAhPqfZdGlpFEUk24UZ

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX-W5u-WMluaIkOmjjl8=,size_20,color_FFFFFF,t_70,g_se,x_16-16601687854895.png)

 成功。

## 3）yml配置

```XML
spring:



 datasource:



  username: root



  password: ENC(8jLUdq0Fr7UhJGNwK/Nc6i6/WV4+UBpvtfBLDh4e3jZMJZAhPqfZdGlpFEUk24UZ)



  url: jdbc:mysql://localhost:3306/test?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC



  driver-class-name: com.mysql.cj.jdbc.Driver



 



jasypt:



 encryptor:



  password: Angel



  algorithm: PBEWITHHMACSHA512ANDAES_256
```

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX-W5u-WMluaIkOmjjl8=,size_20,color_FFFFFF,t_70,g_se,x_16-16601687854896.png)

 改个登录密码：

![img](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAX-W5u-WMluaIkOmjjl8=,size_20,color_FFFFFF,t_70,g_se,x_16-16601687854897.png)

很顺利，不妨再来验证加密算法，如图我如果改为

```java
jasypt:



 encryptor:



  password: Angel



  algorithm: PBEWithMD5AndDES
```

 运行项目则报错，图略。

yml文件里面需要配置密钥以及算法，才能正确解密访问数据库，那你看这个Angel密钥是用password来修饰的，是否会被外行的人认作密码呢？答案是肯定的，所以呀需要使用写小手段，把密钥也写成**ENC()**这种格式的就OK了。
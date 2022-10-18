# AJ-Captcha

[功能概述 | AJ-Captcha (beliefteam.cn)](https://ajcaptcha.beliefteam.cn/captcha-doc/)

# 简介

 		`AJ-Captcha`行为验证码，包含滑动拼图、文字点选两种方式，UI支持弹出和嵌入两种方式。后端提供Java实现，前端提供了`php、angular、html、vue、uni-app、flutter、android、ios`等代码示例。

#### 功能描述

| 滑动拼图                                                     | 文字点选                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![滑动拼图](https://gitee.com/anji-plus/captcha/raw/master/images/%E6%BB%91%E5%8A%A8%E6%8B%BC%E5%9B%BE.gif) | ![点选文字](https://gitee.com/anji-plus/captcha/raw/master/images/%E7%82%B9%E9%80%89%E6%96%87%E5%AD%97.gif) |
| 图1-1                                                        | 图1-2                                                        |

#### 概念术语描述

| 术语       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 验证码类型 | 1）滑动拼图 blockPuzzle 2）文字点选 clickWord                |
| 验证       | 用户拖动/点击一次验证码拼图即视为一次“验证”，不论拼图/点击是否正确 |
| 二次校验   | 验证数据随表单提交到后台后，后台需要调用`captchaService.verification`做二次校验。目的是核实验证数据的有效性。 |



#### 交互流程

① 用户访问应用页面，请求显示行为验证码
② 用户按照提示要求完成验证码拼图/点击
③ 用户提交表单，前端将第二步的输出一同提交到后台
④ 验证数据随表单提交到后台后，后台需要调用captchaService.verification做二次校验。
⑤ 第4步返回校验通过/失败到产品应用后端，再返回到前端。如下图所示。 ![时序图](E:/Development/Typora/images/shixu.png)



#### 源码包目录结构

```c
├─core
│ ├─captcha    java核心源码
│ └─captcha-spring-boot-starter    springboot快速启动
├─images       效果图
├─service
│ ├─go    后端为go项目示例
│ ├─php    后端为php项目示例
│ ├─springboot    后端为springboot项目示例
│ └─springmvc    后端为springmvc非springboot项目示例
└─view       多语言客户端示例
 ├─android    原生android实现示例
 ├─angular    angular实现示例
 ├─flutter    flutter实现示例
 ├─html    原生html实现示例
 ├─ios    原生ios实现示例
 ├─php    php实现示例
 ├─react    react实现示例
 ├─uni-app    uni-app实现示例
 ├─wx-applet    微信小程序实现示例
 └─vue    vue实现示例
```



#### 本地启动

 第一步，启动后端，导入Eclipse或者Intellij,启动service/springboot的StartApplication。[社区底图库](https://gitee.com/anji-plus/AJ-Captcha-Images)
  第二步，启动前端，使用visual code打开文件夹view/vue，npm install后npm run dev，浏览器登录

```
npm install
npm run dev

DONE  Compiled successfully in 29587ms                       12:06:38
I  Your application is running here: http://localhost:8081
```

 详细的前后端接入文档，后端示例代码service目录下，前端示例代码view目录下。



# Java快速开始

## SpringBoot项目

参考示例：service/springboot

## 1.引入依赖

```xml
<dependency>
   <groupId>com.anji-plus</groupId>
   <artifactId>spring-boot-starter-captcha</artifactId>
   <version>1.3.0</version>
</dependency>
```

## 2.修改application

```properties
# 滑动验证，底图路径，不配置将使用默认图片
# 支持全路径
# 支持项目路径,以classpath:开头,取resource目录下路径,例：classpath:images/jigsaw
aj.captcha.jigsaw=classpath:images/jigsaw
    #滑动验证，底图路径，不配置将使用默认图片
    ##支持全路径
# 支持项目路径,以classpath:开头,取resource目录下路径,例：classpath:images/pic-click
aj.captcha.pic-click=classpath:images/pic-click

# 对于分布式部署的应用，我们建议应用自己实现CaptchaCacheService，比如用Redis或者memcache，
# 参考CaptchaCacheServiceRedisImpl.java
# 如果应用是单点的，也没有使用redis，那默认使用内存。
# 内存缓存只适合单节点部署的应用，否则验证码生产与验证在节点之间信息不同步，导致失败。
# ！！！ 注意啦，如果应用有使用spring-boot-starter-data-redis，
# 请打开CaptchaCacheServiceRedisImpl.java注释。
# redis ----->  SPI： 在resources目录新建META-INF.services文件夹(两层)，参考当前服务resources。
# 缓存local/redis...
aj.captcha.cache-type=local
# local缓存的阈值,达到这个值，清除缓存
#aj.captcha.cache-number=1000
# local定时清除过期缓存(单位秒),设置为0代表不执行
#aj.captcha.timing-clear=180
#spring.redis.host=10.108.11.46
#spring.redis.port=6379
#spring.redis.password=
#spring.redis.database=2
#spring.redis.timeout=6000

# 验证码类型default两种都实例化。
aj.captcha.type=default
# 汉字统一使用Unicode,保证程序通过@value读取到是中文，可通过这个在线转换;yml格式不需要转换
# https://tool.chinaz.com/tools/unicode.aspx 中文转Unicode
# 右下角水印文字(我的水印)
aj.captcha.water-mark=\u6211\u7684\u6c34\u5370
# 右下角水印字体(不配置时，默认使用文泉驿正黑)
# 由于宋体等涉及到版权，我们jar中内置了开源字体【文泉驿正黑】
# 方式一：直接配置OS层的现有的字体名称，比如：宋体
# 方式二：自定义特定字体，请将字体放到工程resources下fonts文件夹，支持ttf\ttc\otf字体
# aj.captcha.water-font=WenQuanZhengHei.ttf
# 点选文字验证码的文字字体(文泉驿正黑)
# aj.captcha.font-type=WenQuanZhengHei.ttf
# 校验滑动拼图允许误差偏移量(默认5像素)
aj.captcha.slip-offset=5
# aes加密坐标开启或者禁用(true|false)
aj.captcha.aes-status=true
# 滑动干扰项(0/1/2)
aj.captcha.interference-options=2

#点选字体样式 默认Font.BOLD
aj.captcha.font-style=1
#点选字体字体大小
aj.captcha.font-size=25
#点选文字个数,存在问题，暂不支持修改
#aj.captcha.click-word-count=4

aj.captcha.history-data-clear-enable=false

# 接口请求次数一分钟限制是否开启 true|false
aj.captcha.req-frequency-limit-enable=false
# 验证失败5次，get接口锁定
aj.captcha.req-get-lock-limit=5
# 验证失败后，锁定时间间隔,s
aj.captcha.req-get-lock-seconds=360
# get接口一分钟内请求数限制
aj.captcha.req-get-minute-limit=30
# check接口一分钟内请求数限制
aj.captcha.req-check-minute-limit=60
# verify接口一分钟内请求数限制
aj.captcha.req-verify-minute-limit=60
```

## 3.*自定义实现CaptchaCacheService

`非常重要`。对于分布式多实例部署的应用，应用必须自己实现`CaptchaCacheService`，比如用`Redis`或者`memcache`，参考`service/springboot/src/.../CaptchaCacheServiceRedisImpl.java`

```java
package com.anji.captcha.demo.service;

import com.anji.captcha.service.CaptchaCacheService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;

import java.util.concurrent.TimeUnit;

/**
 * 对于分布式部署的应用，我们建议应用自己实现CaptchaCacheService，比如用Redis，参考service/spring-boot代码示例。
 * 如果应用是单点的，也没有使用redis，那默认使用内存。
 * 内存缓存只适合单节点部署的应用，否则验证码生产与验证在节点之间信息不同步，导致失败。
 *
 * ☆☆☆ SPI： 在resources目录新建META-INF.services文件夹(两层)，参考当前服务resources。
 * @Title: 使用redis缓存
 * @author lide1202@hotmail.com
 * @date 2020-05-12
 */
public class CaptchaCacheServiceRedisImpl implements CaptchaCacheService {

    @Override
    public String type() {
        return "redis";
    }

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public void set(String key, String value, long expiresInSeconds) {
        stringRedisTemplate.opsForValue().set(key, value, expiresInSeconds, TimeUnit.SECONDS);
    }

    @Override
    public boolean exists(String key) {
        return stringRedisTemplate.hasKey(key);
    }

    @Override
    public void delete(String key) {
        stringRedisTemplate.delete(key);
    }

    @Override
    public String get(String key) {
        return stringRedisTemplate.opsForValue().get(key);
    }

   @Override
   public Long increment(String key, long val) {
      return stringRedisTemplate.opsForValue().increment(key,val);
   }
}
```



## 4.新建META-INF.services文件夹

**在resources目录新建META-INF.services文件夹，参考resource/META-INF/services中的写法**

![image-20220925192216998](E:/Development/Typora/images/image-20220925192216998.png)



```
com.anji.captcha.demo.service.CaptchaCacheServiceRedisImpl
```

![image-20220925192307009](E:/Development/Typora/images/image-20220925192307009.png)



## 5.后端二次校验接口

**二次校验参数请查看前端接入文档,例：vue,html接入文档等**

以登录为例，用户在提交表单到后台，会携带一个验证码相关的参数。后端登录接口login，首先调用`CaptchaService.verification`做二次校验，

```java
@Autowired
private CaptchaService captchaService;

@PostMapping("/login")
public ResponseModel get(@RequestBody CaptchaVO captchaVO) {
    //必传参数：captchaVO.captchaVerification
    ResponseModel response = captchaService.verification(captchaVO);
    if(response.isSuccess() == false){
        //验证码校验失败，返回信息告诉前端
        //repCode  0000  无异常，代表成功
        //repCode  9999  服务器内部异常
        //repCode  0011  参数不能为空
        //repCode  6110  验证码已失效，请重新获取
        //repCode  6111  验证失败
        //repCode  6112  获取验证码失败,请联系管理员
        //repCode  6113  底图未初始化成功，请检查路径
        //repCode  6201  get接口请求次数超限，请稍后再试!
        //repCode  6206  无效请求，请重新获取验证码
        //repCode  6202  接口验证失败数过多，请稍后再试
        //repCode  6204  check接口请求次数超限，请稍后再试!
    }
    return response;
}
```



## 后端接口

#### 获取验证码接口：http://*:*/captcha/get

##### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/java.html#请求参数)请求参数：

```json
{
	"captchaType": "blockPuzzle",  //验证码类型 clickWord
	"clientUid": "唯一标识"  //客户端UI组件id,组件初始化时设置一次，UUID（非必传参数）
}
```

##### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/java.html#响应参数)响应参数：

```json
{
    "repCode": "0000",
    "repData": {
        "originalImageBase64": "底图base64",
        "point": {    //默认不返回的，校验的就是该坐标信息，允许误差范围
            "x": 205,
            "y": 5
        },
        "jigsawImageBase64": "滑块图base64",
        "token": "71dd26999e314f9abb0c635336976635", //一次校验唯一标识
        "secretKey": "16位随机字符串", //aes秘钥，开关控制，前端根据此值决定是否加密
        "result": false,
        "opAdmin": false
    },
    "success": true,
    "error": false
}
```

#### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/java.html#核对验证码接口接口-http-captcha-check)核对验证码接口接口：http://*:*/captcha/check

##### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/java.html#请求参数-2)请求参数：

```json
{
	 "captchaType": "blockPuzzle",
	 "pointJson": "QxIVdlJoWUi04iM+65hTow==",  //aes加密坐标信息
	 "token": "71dd26999e314f9abb0c635336976635"  //get请求返回的token
}
```

##### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/java.html#响应参数-2)响应参数：

```json
{
    "repCode": "0000",
    "repData": {
        "captchaType": "blockPuzzle",
        "token": "71dd26999e314f9abb0c635336976635",
        "result": true,
        "opAdmin": false
    },
    "success": true,
    "error": false
}
```

### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/java.html#防刷功能-补充中)防刷功能(补充中)

```
a.同一用户，登录错误3次才要求验证码，考虑是登录模块的功能。
b.同一用户，限制请求验证码，5分钟不能超过100次等。
以上功能，我们会在service/springboot示例代码中提供额外的参考代码，不集成在jar
```



# Vue快速开始

## 兼容性

```
IE8+、Chrome、Firefox.(其他未测试)
```

## [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#初始化组件)初始化组件

```
1)复制view/vue/src/components/verifition文件夹,到自己工程对应目录下,在登录页面插入如下代码。

2)安装请求和加密依赖

  npm install axios  crypto-js   -S
```

## [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#基础示例)基础示例

```javascript
<template>
    <Verify
	@success="success" //验证成功的回调函数
	:mode="pop"     //调用的模式
	:captchaType="blockPuzzle"    //调用的类型 点选或者滑动
        :imgSize="{ width: '330px', height: '155px' }" //图片的大小对象
        ref="verify"
    ></Verify>
    //mode="pop"模式
    <button @click="useVerify">调用验证组件</button>
</template>

****注: mode为"pop"时,使用组件需要给组件添加ref值,并且手动调用show方法 例: this.$refs.verify.show()****
****注: mode为"fixed"时,无需添加ref值,无需调用show()方法****

<script>
//引入组件
import Verify from "./../../components/verifition/Verify";
export default {
	name: 'app',
	components: {
		Verify
	}
	methods:{
	   success(params){
	     // params 返回的二次验证参数, 和登录参数一起回传给登录接口，方便后台进行二次验证
           },
           useVerify(){
              this.$refs.verify.show()
           }
	}
}
</script>
```

## [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#回调事件)回调事件

| 参数            | 类型     | 说明                                                         |
| --------------- | -------- | ------------------------------------------------------------ |
| success(params) | funciton | 验证码匹配成功后的回调函数,params为返回需回传服务器的二次验证参数 |
| error           | funciton | 验证码匹配失败后的回调函数                                   |
| ready           | funciton | 验证码初始化成功的回调函数                                   |

## [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#验证码参数)验证码参数

| 参数        | 类型   | 说明                                                         |
| ----------- | ------ | ------------------------------------------------------------ |
| captchaType | String | 1）滑动拼图 blockPuzzle 2）文字点选 clickWord                |
| mode        | String | 验证码的显示方式，弹出式pop，固定fixed，默认：mode : ‘pop’   |
| vSpace      | String | 验证码图片和移动条容器的间隔，默认单位是px。如：间隔为5px，默认:vSpace:5 |
| explain     | String | 滑动条内的提示，不设置默认是：'向右滑动完成验证'             |
| imgSize     | Object | 其中包含了width、height两个参数，分别代表图片的宽度和高度，支持百分比方式设置 如:{width:'400px',height:'200px'} |

## [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#默认接口api地址)默认接口api地址

| 请求URL        | 请求方式 |
| -------------- | -------- |
| /captcha/get   | Post     |
| /captcha/check | Post     |

## [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#获取验证码接口详情)获取验证码接口详情

#### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#接口地址-http-captcha-get)接口地址：http://*:*/captcha/get

```
组件内部默认请求服务器地址: process.env.BASE_API ; 是vue项目打包配置地址,方便分环境打包
```

##### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#请求参数)请求参数：

```json
{
	"captchaType": "blockPuzzle",  //验证码类型 clickWord
  "clientUid": "唯一标识"  //客户端UI组件id,组件初始化时设置一次，UUID（非必传参数）
}
```

##### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#响应参数)响应参数：

```json
{
    "repCode": "0000",
    "repData": {
        "originalImageBase64": "底图base64",
        "point": {    //默认不返回的，校验的就是该坐标信息，允许误差范围
            "x": 205,
            "y": 5
        },
        "jigsawImageBase64": "滑块图base64",
        "token": "71dd26999e314f9abb0c635336976635", //一次校验唯一标识
        "result": false,
        "opAdmin": false
    },
    "success": true,
    "error": false
}
```

## [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#核对验证码接口详情)核对验证码接口详情

#### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#请求接口-http-captcha-check)请求接口：http://*:*/captcha/check

```
组件内部默认请求服务器地址: process.env.BASE_API ; 是vue项目打包配置地址,方便分环境打包
```

##### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#请求参数-2)请求参数：

```json
{
	 "captchaType": "blockPuzzle",
	 "pointJson": "QxIVdlJoWUi04iM+65hTow==",  //aes加密坐标信息
	 "token": "71dd26999e314f9abb0c635336976635"  //get请求返回的token
}
```

##### [#](https://ajcaptcha.beliefteam.cn/captcha-doc/captchaDoc/vue.html#响应参数-2)响应参数：

```json
{
    "repCode": "0000",
    "repData": {
        "captchaType": "blockPuzzle",
        "token": "71dd26999e314f9abb0c635336976635",
        "result": true,
        "opAdmin": false
    },
    "success": true,
    "error": false
}
```






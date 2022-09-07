# Feign远程调用传输文件

#### **依赖**

```xml
<!--SpringCloud openfeign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.2.RELEASE</version>
        </dependency>

```

#### 服务接口端

```java
   //图片上传
    @PostMapping(value = "/upload",consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public Result upload(@RequestPart("imgFile") MultipartFile imgFile){
    .......
    .}
注意加上consumes = MediaType.MULTIPART_FORM_DATA_VALUE)和@RequestPart注解
```

#### [Feign](https://so.csdn.net/so/search?q=Feign&spm=1001.2101.3001.7020)调用端

```java
package com.zhubayi.backend.feign;

import com.zhubayi.backend.config.MultipartSupportConfig;
import com.zhubayi.common.entity.Result;
import com.zhubayi.common.pojo.Setmeal;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

/**
 * @author zhubayi
 */
@FeignClient(name = "health-provider")
public interface SetmealFeignService {
    @PostMapping(value = "/setmeal/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    Result upload(@RequestPart("imgFile") MultipartFile imgFile);
}
```

**`注意加上consumes = MediaType.MULTIPART_FORM_DATA_VALUE)和@RequestPart注解`**

Feign调用端还要向容器注入`SpringFormEncoder`

```java
    @Bean
    public Encoder feignFormEncoder() {
        return new SpringFormEncoder();
    }

```

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzMyMTQ2,size_16,color_FFFFFF,t_70.png)

### 这种方式当传输的`非文件类型`的时候会报错，采用下面的方式可以解决。

### 第二种方式

新建配置在`远程调用feign端`加上类`MultipartSupportConfig`

```java
package com.zhubayi.backend.config;

import feign.codec.Encoder;
import feign.form.spring.SpringFormEncoder;
import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.http.HttpMessageConverters;
import org.springframework.cloud.openfeign.support.SpringEncoder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author zhubayi
 */
@Configuration
public class MultipartSupportConfig {
    @Autowired
    private ObjectFactory<HttpMessageConverters> messageConverters;
    //微服务传输文件用
    @Bean
    public Encoder feignFormEncoder() {
        return new SpringFormEncoder(new SpringEncoder(messageConverters));
    }
}

```

**在远程调用端`@FeignClient`加上**
`configuration = MultipartSupportConfig.class`

```java
package com.zhubayi.backend.feign;

import com.zhubayi.backend.config.MultipartSupportConfig;
import com.zhubayi.common.entity.Result;
import com.zhubayi.common.pojo.Setmeal;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

/**
 * @author zhubayi
 */
@FeignClient(name = "health-provider",configuration = MultipartSupportConfig.class)
public interface SetmealFeignService {
    @PostMapping(value = "/setmeal/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    Result upload(@RequestPart("imgFile") MultipartFile imgFile);
    @PostMapping(value = "/setmeal/add")
    Result add(@RequestBody Setmeal setmeal,@RequestParam("checkgroupIds") Integer[] checkgroupIds);
}

注意加上consumes = MediaType.MULTIPART_FORM_DATA_VALUE)和@RequestPart注解
```

**服务端**

```java
package com.zhubayi.provider.controller;

import com.zhubayi.common.constant.MessageConstant;
import com.zhubayi.common.entity.Result;
import com.zhubayi.common.pojo.Setmeal;
import com.zhubayi.common.utils.QiniuUtils;
import com.zhubayi.provider.service.SetmealService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.util.UUID;

/**
 * @author zhubayi
 * 套餐管理
 */
@RestController
@RequestMapping("/setmeal")
public class SetmealController {
    @Autowired
    private SetmealService setmealService;
    //图片上传
    @PostMapping(value = "/upload",consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public Result upload(@RequestPart("imgFile") MultipartFile imgFile){
        try{
            //获取原始文件名
            String originalFilename = imgFile.getOriginalFilename();
            int lastIndexOf = originalFilename.lastIndexOf(".");
            //获取文件后缀
            String suffix = originalFilename.substring(lastIndexOf - 1);
            //使用UUID随机产生文件名称，防止同名文件覆盖
            String fileName = UUID.randomUUID().toString() + suffix;
            QiniuUtils.upload2Qiniu(imgFile.getBytes(),fileName);
            //图片上传成功
            Result result = new Result(true, MessageConstant.PIC_UPLOAD_SUCCESS);
            result.setData(fileName);
            return result;
        }catch (Exception e){
            e.printStackTrace();
            //图片上传失败
            return new Result(false,MessageConstant.PIC_UPLOAD_FAIL);
        }
    }
    //新增
    @PostMapping("/add")
    public Result add(@RequestBody Setmeal setmeal, Integer[] checkgroupIds){
        System.out.println(checkgroupIds);
        System.out.println(setmeal);
//        try {
//            setmealService.add(setmeal,checkgroupIds);
//        }catch (Exception e){
//            //新增套餐失败
//            return new Result(false,MessageConstant.ADD_SETMEAL_FAIL);
//        }
        //新增套餐成功
        return new Result(true,MessageConstant.ADD_SETMEAL_SUCCESS);
    }
}
注意加上consumes = MediaType.MULTIPART_FORM_DATA_VALUE)和@RequestPart注解
```

特别注意几点：

- `consumes = MediaType.MULTIPART_FORM_DATA_VALUE` 必须存在
- 文件部分注解为`@RequestPart`,要不然出现错误：the request was rejected because no multipart boundary was found
- `attachment`为传过来的文件的name的值。不可以随意更改。要不然出现错误：`Required request part 'file' is not present`,有时候我们需要将文件流转成MultipartFile。比如：我们有时候需要导出EXCEL发邮件，但是发邮件有专门的服务时，就需要将`EXCEL`文件流传到邮件服务发送，需要代码构建`MultipartFile`，`DiskFileItemFactory`，这个时候要注意`FeildName`填写的就是`attachment`。
- `SpringFormEncoder` 不可以少,两种方式都用了它的。
- 请求方式为`post`，`get`会报错
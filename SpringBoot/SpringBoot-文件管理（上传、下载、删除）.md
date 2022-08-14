SpringBoot-文件管理（上传、下载、删除）



# 文件管理

## 存储路径

首先在application.yml中定义一个文件路径

```yaml
fy:
  path: E:/images/
```



## FileService

```java
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.http.HttpServletResponse;

public interface FileService {

    //上传文件到本地服务器
    String uploadLocal(MultipartFile file);
    //下载文件(展示到客户端)
    void download(String name, HttpServletResponse response);
    //删除文件
    Boolean delete(String name);
}
```



## FileServiceImpl

```java
import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;

import com.hong.service_oss.service.FileService;
import com.hong.service_oss.utils.ConstantOssPropertiesUtils;
import org.joda.time.DateTime;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.util.UUID;

@Service
public class FileServiceImpl implements FileService {
    @Value("${fy.path}")
    private String baseUrl;

    @Override
    public String uploadLocal(MultipartFile file) {

        //原始文件名称
        String originalFilename = file.getOriginalFilename(); //xxx.jpg
        String suffix = originalFilename.substring(originalFilename.lastIndexOf("."));// .jpg

        //生成随机唯一值，使用uuid，添加到文件名称里面,防止文件名称重复造成文件覆盖
        String fileName = UUID.randomUUID().toString().replaceAll("-", "")+suffix; //.replaceAll("-","") 将所有的-替换成空字符串
        File dir = new File(baseUrl);      //创建一个目录对象

        if(!dir.exists()){      //判断目录是否存在
            dir.mkdirs();   //如果不存在，创建目录
        }
        try{
            file.transferTo(new File(baseUrl+fileName)); //将文件写入目录
        }catch (IOException e){
            e.printStackTrace();
            return null;
        }
        return baseUrl+fileName; //返回文件路径
    }

    @Override
    public void download(String name, HttpServletResponse response) {
        try{
            //输入流，通过输入流读取文件内容
            FileInputStream fileInputStream = new FileInputStream(new File(baseUrl + name));
            //输出流，通过输出流将文件内容写到客户端，在浏览器中显示
            ServletOutputStream outputStream = response.getOutputStream();
            response.setContentType("image/jpeg");//设置响应类型
            int len= 0;
            byte[] bytes = new byte[1024];
            while ((len = fileInputStream.read(bytes)) != -1) { //读取文件内容
                outputStream.write(bytes, 0, len); //写入客户端
                outputStream.flush(); //刷新缓冲区
            }
            //关闭资源
            outputStream.close();
            fileInputStream.close();
        } catch (FileNotFoundException e) {
            //给客户端返回404错误和错误信息
            response.setStatus(404);
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Boolean delete(String name)  {
        File file = new File(baseUrl + name);
        if(file.exists()){
            file.delete();
            return true;
        }
        return false;
    }
}
```



## FileApiController

```java
import com.hong.fy_common_commomutil.common.result.Result;
import com.hong.service_oss.service.FileService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;

@Api(tags = "文件管理")
@RestController
@RequestMapping("/api/file")
public class FileApiController {

    @Autowired
    private FileService fileService;

    //上传文件到本地服务器
    @ApiOperation(value = "上传文件到本地服务器")
    @PostMapping("/local/fileUpload")
    public Result fileUploadLocal(@RequestParam("file") MultipartFile file) {
        //file是一个临时文件，需要转存到指定位置，否则本次请求完成后临时文件会删除
        //获取上传文件
        if (file == null) {
            return Result.fail("文件为空");
        }
        String url = fileService.uploadLocal(file);
        return Result.ok(url);
    }

    //文件下载
    @ApiOperation(value = "文件下载")
    @GetMapping("/download")
    public void filedownload(String name , HttpServletResponse response) {
       fileService.download(name,response);
    }

    //文件删除
    @ApiOperation(value = "文件删除")
    @DeleteMapping("/delete")
    public Result filedelete(String name)  {
        Boolean flag = fileService.delete(name);
        if (flag) {
            return Result.ok("删除成功");
        } else {
            return Result.fail("删除失败");
        }
    }
}
```
# SpringBoot - MinIO



> 接下来我们将结合SpringBoot来实现一个完整的图片上传与删除操作。

## 上传流程示意图：

![img](E:\Development\Typora\images\minio_use_22.90f2d51e.png)

## 1.添加依赖

- 在pom.xml中添加MinIO的相关依赖：

```xml
 #低版本的okhttp会报错提示
    <dependency>
        <groupId>com.squareup.okhttp3</groupId>
        <artifactId>okhttp</artifactId>
        <version>4.9.0</version>
    </dependency>
    <dependency>
        <groupId>io.minio</groupId>
        <artifactId>minio</artifactId>
        <version>8.4.2</version>
        <exclusions>
            <exclusion>
                <artifactId>okhttp</artifactId>
                <groupId>com.squareup.okhttp3</groupId>
            </exclusion>
        </exclusions>
    </dependency>
```



## 2.开启上传功能

- 在SpringBoot中开启文件上传功能，需要在application.yml添加如下配置：

```yaml
spring:
  servlet:
    multipart:
      enabled: true #开启文件上传
      max-file-size: 10MB #限制文件上传大小为10M
```



## 3.添加配置信息

在application.yml中添加

```yml
minio:
  endpoint: http://106.53.124.182:9000
  bucketName: public
  accessKey: OV5xLXR9kUxpO3Oi
  secretKey: w0UvfFW2NNVkVj7N4JeeVU
```







## 4.配置config

```java
@Data
@Configuration
@ConfigurationProperties(prefix = "minio")
public class MinioConfig {
    private String endpoint; // minio服务器地址
    private String accessKey; // minio服务器访问密钥
    private String secretKey; // minio服务器访问密钥 
    private String bucketName; // minio服务器存储bucket名称

    @Bean
    public MinioClient minioClient() { // 创建 MinioClient 对象
        return MinioClient.builder()
                .endpoint(endpoint) // 设置 Minio 服务器地址
                .credentials(accessKey, secretKey) // 设置 Minio 服务器的访问凭证
                .build(); // 创建 MinioClient 对象
    }
}
```



## 5.添加util类

```java
@Component
@Slf4j
public class MinioUtil {
    @Autowired
    private MinioConfig prop;

    @Resource
    private MinioClient minioClient;

    /**
     * 查看存储bucket是否存在
     * @return boolean
     */
    public Boolean bucketExists(String bucketName) {
        Boolean found;
        try {
            found = minioClient.bucketExists(BucketExistsArgs.builder()
                    .bucket(bucketName)
                    .build()); //BucketExistsArgs是一个构造器类，用来构造BucketExistsArgs对象
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return found;
    }

    /**
     * 创建存储bucket
     * @return Boolean
     */
    public Boolean makeBucket(String bucketName) {
        try {
            minioClient.makeBucket(MakeBucketArgs.builder()
                    .bucket(bucketName)
                    .build());
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }
    /**
     * 删除存储bucket
     * @return Boolean
     */
    public Boolean removeBucket(String bucketName) {
        try {
            minioClient.removeBucket(RemoveBucketArgs.builder()
                    .bucket(bucketName)
                    .build());
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }
    /**
     * 获取全部bucket
     */
    public List<Bucket> getAllBuckets() {
        try {
            List<Bucket> buckets = minioClient.listBuckets();
            return buckets;

        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }


    /**
     * 文件上传
     *
     * @param file 文件
     * @return Boolean
     */
    public String upload(MultipartFile file) {
        String originalFilename = file.getOriginalFilename();
        //判断文件名是否为空
        if (originalFilename == null || originalFilename.equals("")) {
           throw new RuntimeException("文件名不能为空");
        }
        //获取文件后缀名
        String fileName = UUID.randomUUID() + originalFilename.substring(originalFilename.lastIndexOf("."));
        //获取当前日期，格式为yyyyMMdd
        DateFormat dateFormat = new SimpleDateFormat("yyyy/MM/dd");
        Date date = new Date();
        String s = dateFormat.format(date);

        String objectName = s + "/" + fileName;
        try {
            PutObjectArgs objectArgs = PutObjectArgs.builder()
                    .bucket(prop.getBucketName())
                    .object(objectName)
                    .stream(file.getInputStream(), file.getSize(), -1)
                    .contentType(file.getContentType())
                    .build();
            //文件名称相同会覆盖
            minioClient.putObject(objectArgs);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
        return objectName;
    }

    /**
     * 预览图片
     * @param fileName
     * @return
     */
    public String preview(String fileName){
        // 查看文件地址
        GetPresignedObjectUrlArgs build = new GetPresignedObjectUrlArgs().builder()
                .bucket(prop.getBucketName())
                .object(fileName) //obejct方法是获取文件地址的方法
                .method(Method.GET)
                .build();
        try {
            String url = minioClient.getPresignedObjectUrl(build);
            return url;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 文件下载
     * @param fileName 文件名称
     * @param res response
     * @return Boolean
     */
    public void download(String fileName, HttpServletResponse res) {
        GetObjectArgs objectArgs = GetObjectArgs.builder().bucket(prop.getBucketName())
                .object(fileName).build();
        try (GetObjectResponse response = minioClient.getObject(objectArgs)){
            byte[] buf = new byte[1024];
            int len;
            try (FastByteArrayOutputStream os = new FastByteArrayOutputStream()){
                while ((len=response.read(buf))!=-1){
                    os.write(buf,0,len);
                }
                os.flush();
                byte[] bytes = os.toByteArray();
                res.setCharacterEncoding("utf-8");
                // 设置强制下载不打开
                // res.setContentType("application/force-download");
                res.addHeader("Content-Disposition", "attachment;fileName=" + fileName);
                try (ServletOutputStream stream = res.getOutputStream()){
                    stream.write(bytes);
                    stream.flush();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 查看文件对象
     * @return 存储bucket内文件对象信息
     */
    public List<Item> listObjects() {
        Iterable<Result<Item>> results = minioClient.listObjects(
                ListObjectsArgs.builder().bucket(prop.getBucketName()).build());
        List<Item> items = new ArrayList<>();
        try {
            for (Result<Item> result : results) {
                items.add(result.get());
            }
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
        return items;
    }

    /**
     * 删除
     * @param fileName
     * @return
     * @throws Exception
     */
    public boolean remove(String fileName){
        try {
            minioClient.removeObject( RemoveObjectArgs.builder().bucket(prop.getBucketName()).object(fileName).build());
        }catch (Exception e){
            return false;
        }
        return true;
    }

}
```



## 6.添加Controller

- 添加一个`MinioController`控制器用于实现文件的上传和删除操作：

- ```java
  @Slf4j
  @RestController
  @RequestMapping(value = "/")
  public class MinioController {
  
  
      @Autowired
      private MinioUtil minioUtil;
      @Autowired
      private MinioConfig prop;
  
      Map<String,Object> map=new HashMap<>();
  
  
      //查看存储bucket是否存在
      @GetMapping("/bucketExists")
      public Result bucketExists(@RequestParam("bucketName") String bucketName) {
          map.put("bucketName",minioUtil.bucketExists(bucketName));
          return Result.ok(map);
      }
  
  
      //创建存储bucket
      @GetMapping("/makeBucket")
      public Result makeBucket(String bucketName) {
          return Result.ok(map.put("bucketName",minioUtil.makeBucket(bucketName)));
      }
  
      //删除存储bucket
      @GetMapping("/removeBucket")
      public Result removeBucket(String bucketName) {
  
          return Result.ok(map.put("bucketName",minioUtil.removeBucket(bucketName)));
      }
  
      //获取全部bucket
      @GetMapping("/getAllBuckets")
      public Result getAllBuckets() {
          List<Bucket> allBuckets = minioUtil.getAllBuckets();
          return Result.ok(map.put("allBuckets",allBuckets));
      }
  
      //文件上传
      @PostMapping("/upload")
      public Result upload(@RequestParam("file") MultipartFile file) {
          String objectName = minioUtil.upload(file);
          if (null != objectName) {
              return Result.ok(map.put("url",(prop.getEndpoint() + "/" + prop.getBucketName() + "/" + objectName)));
          }
          return Result.fail();
      }
  
      //预览
      @GetMapping("/preview")
      public Result preview(@RequestParam("fileName") String fileName) {
          return Result.ok(map.put("filleName",minioUtil.preview(fileName)));
      }
  
      //下载
      @GetMapping("/download")
      public Result download(@RequestParam("fileName") String fileName, HttpServletResponse res) {
          minioUtil.download(fileName,res);
          return Result.ok();
      }
  
  
      @PostMapping("/delete")
      public Result remove(String url) {
          String objName = url.substring(url.lastIndexOf(prop.getBucketName()+"/") + prop.getBucketName().length()+1);
          minioUtil.remove(objName);
          return Result.ok(map.put("objName",objName));
      }
  }
  ```





## 7.其他工具类

### Result

```java
/**
 * 全局统一返回结果类
 */
@Data
public class Result<T> {

    private Integer code;
    private String message;
    private T data;
    public Result(){}
    protected static <T> Result<T> build(T data) {
        Result<T> result = new Result<T>();
        if (data != null)
            result.setData(data);
        return result;
    }

    public static <T> Result<T> build(T body, ResultCodeEnum resultCodeEnum) { //第一个标志这个方法是泛型方法，第二个是List<T>是返回值。泛型方法返回值前必须带一个<T>，这是一种约定，表示该方法是泛型方法，否则报错。
        Result<T> result = build(body);
        result.setCode(resultCodeEnum.getCode());
        result.setMessage(resultCodeEnum.getMessage());
        return result;
    }

    public static <T> Result<T> build(Integer code, String message) {
        Result<T> result = build(null);
        result.setCode(code);
        result.setMessage(message);
        return result;
    }

    public static<T> Result<T> ok(){
        return Result.ok(null);
    }

    /**
     * 操作成功
     * @param data
     * @param <T>
     * @return
     */
    public static<T> Result<T> ok(T data){
        Result<T> result = build(data);
        return build(data, ResultCodeEnum.SUCCESS);
    }

    public static<T> Result<T> fail(){
        return Result.fail(null);
    }

    /**
     * 操作失败
     * @param data
     * @param <T>
     * @return
     */
    public static<T> Result<T> fail(T data){
        Result<T> result = build(data);
        return build(data, ResultCodeEnum.FAIL);
    }

    public Result<T> message(String msg){
        this.setMessage(msg);
        return this;
    }

    public Result<T> code(Integer code){
        this.setCode(code);
        return this;
    }

    public boolean isOk() {
        if(this.getCode().intValue() == ResultCodeEnum.SUCCESS.getCode().intValue()) {
            return true;
        }
        return false;
    }
}
```



### ResultCodeEnum

```java
/**
 * 统一返回结果状态信息类
 */
@Getter
public enum ResultCodeEnum {

    SUCCESS(200,"成功"),
    FAIL(201, "失败"),
    ;

    private Integer code;
    private String message;

    private ResultCodeEnum(Integer code, String message) {
        this.code = code;
        this.message = message;
    }
}
```





## 8.官方API

[MinIO | Java Client API Reference](https://docs.min.io/docs/java-client-api-reference.html)
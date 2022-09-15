# SpringBoot入门 - 定制自己的Banner

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/spring/springboot/springboot-x-hello-banner.html

## 什么是Banner

我们在启动Spring Boot程序时，有如下Banner信息：

![img](E:\Development\Typora\images\springboot-hello-banner-1.png)

那么如何自定义成自己项目的名称呢？

## [¶](#如何更改banner) 如何更改Banner

> 更改Banner有如下几种方式：

### **banner.txt配置**（最常用）

在application.yml中添加配置

```yml
spring:
  banner:
    charset: UTF-8
    location: classpath:banner.txt  
```

在resource下创建banner.txt，内容自定义：

```yml
----welcome----
https://pdai.tech
---------------  
```



修改后，重启的app的效果

![img](E:\Development\Typora\images\springboot-hello-banner-3.png)

### **SpringApplication启动时设置参数**

```java
SpringApplication application = new SpringApplication(App.class);
/**
* Banner.Mode.OFF:关闭;
* Banner.Mode.CONSOLE:控制台输出，默认方式;
* Banner.Mode.LOG:日志输出方式;
*/
application.setBannerMode(Banner.Mode.OFF); // here
application.run(args);   
```

SpringApplication还可以设置自定义的Banner的接口类

![img](E:\Development\Typora\images\springboot-hello-banner-6.png)



### 注意

Maven打包的时候需要放行.txt文件

![image-20220912182004631](E:\Development\Typora\images\image-20220912182004631.png)

## [¶](#文字banner的设计) 文字Banner的设计

> 如何设计上面的文字呢？

### [¶](#一些设计banner的网站) 一些设计Banner的网站

可以通过这个网站进行设计：[patorjk Banner  (opens new window)](http://patorjk.com/software/taag)

比如：

![img](E:\Development\Typora\images\springboot-hello-banner-2.png)

我们修改banner.txt, 运行的效果如下

![img](E:\Development\Typora\images\springboot-hello-banner-4.png)

### [¶](#idea中banner的插件) IDEA中Banner的插件

IDEA中也有插件，不过没有预览功能

![img](E:\Development\Typora\images\springboot-hello-banner-5.png)

### [¶](#其它工具) 其它工具

http://www.network-science.de/ascii/

http://www.degraeve.com/img2txt.php

http://www.bootschool.net/ascii

## [¶](#banner中其它配置信息) Banner中其它配置信息

> 除了文件信息，还有哪些信息可以配置呢？比如Spring默认还带有SpringBoot当前的版本号信息。

在banner.txt中，还可以进行一些设置：

```bash
# springboot的版本号 
${spring-boot.version}             
 
# springboot的版本号前面加v后上括号 
${spring-boot.formatted-version}

# MANIFEST.MF文件中的版本号 
${application.version}              
 
# MANIFEST.MF文件的版本号前面加v后上括号 
${application.formatted-version}

# MANIFEST.MF文件中的程序名称
${application.title}

# ANSI样色/样式等
${Ansi.NAME} (or ${AnsiColor.NAME}, ${AnsiBackground.NAME}, ${AnsiStyle.NAME})
  

```

简单的测试如下（**注意必须打包出Jar, 才会生成resources/META-INF/MANIFEST.MF**）：

![img](E:\Development\Typora\images\springboot-hello-banner-7.png)

## [¶](#动画banner的设计) 动画Banner的设计

> 那我能不能设置动态的Banner呢？比如一个图片？

SpringBoot2是支持图片形式的Banner，

```yml
spring:
  main:
    banner-mode: console
    show-banner: true
  banner:
    charset: UTF-8
    image:
      margin: 0
      height: 10
      invert: false
      location: classpath:pdai.png
   
```

效果如下（需要选择合适的照片，不然效果不好, 所以这种方式很少使用），

![img](E:\Development\Typora\images\springboot-hello-banner-8.png)

注意： 格式不能太大，不然会报错

```bash
org.springframework.boot.ImageBanner     : Image banner not printable: class path resource [banner.gif] (class java.lang.ArrayIndexOutOfBoundsException: '4096')
  
        @pdai: 代码已经复制到剪贴板
    
```

## [¶](#进一步思考) 进一步思考

### [¶](#图片banner是如何起作用的) 图片Banner是如何起作用的？

> 发现 Springboot 可以把图片转换成 ASCII 图案，那么它是怎么做的呢？以此为例，我们看下Spring 的Banner是如何生成的呢？

- 获取Banner
  - 优先级是环境变量中的Image优先,格式在IMAGE_EXTENSION中
  - 然后才是banner.txt
  - 没有的话就用SpringBootBanner
- 如果是图片
  - 获取图片Banner（属性配置等）
  - 转换成ascii

**获取banner**

```java
class SpringApplicationBannerPrinter {
    static final String BANNER_LOCATION_PROPERTY = "spring.banner.location";
    static final String BANNER_IMAGE_LOCATION_PROPERTY = "spring.banner.image.location";
    static final String DEFAULT_BANNER_LOCATION = "banner.txt";
    static final String[] IMAGE_EXTENSION = new String[]{"gif", "jpg", "png"};
    private static final Banner DEFAULT_BANNER = new SpringBootBanner(); // 默认的Spring Banner
    private final ResourceLoader resourceLoader;
    private final Banner fallbackBanner;

    // 获取Banner，优先级是环境变量中的Image优先,格式在IMAGE_EXTENSION中，然后才是banner.txt
    private Banner getBanner(Environment environment) {
        SpringApplicationBannerPrinter.Banners banners = new SpringApplicationBannerPrinter.Banners();
        banners.addIfNotNull(this.getImageBanner(environment));
        banners.addIfNotNull(this.getTextBanner(environment));
        if (banners.hasAtLeastOneBanner()) {
            return banners;
        } else {
            return this.fallbackBanner != null ? this.fallbackBanner : DEFAULT_BANNER;
        }
    }
  
```



**获取图片Banner**

```java
private Banner getImageBanner(Environment environment) {
    String location = environment.getProperty("spring.banner.image.location");
    if (StringUtils.hasLength(location)) {
        Resource resource = this.resourceLoader.getResource(location);
        return resource.exists() ? new ImageBanner(resource) : null;
    } else {
        String[] var3 = IMAGE_EXTENSION;
        int var4 = var3.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            String ext = var3[var5];
            Resource resource = this.resourceLoader.getResource("banner." + ext);
            if (resource.exists()) {
                return new ImageBanner(resource);
            }
        }

        return null;
    }
}
  
```



**获取图片的配置等**

```java
private void printBanner(Environment environment, PrintStream out) throws IOException {
    int width = (Integer)this.getProperty(environment, "width", Integer.class, 76);
    int height = (Integer)this.getProperty(environment, "height", Integer.class, 0);
    int margin = (Integer)this.getProperty(environment, "margin", Integer.class, 2);
    boolean invert = (Boolean)this.getProperty(environment, "invert", Boolean.class, false); // 图片的属性
    BitDepth bitDepth = this.getBitDepthProperty(environment);
    ImageBanner.PixelMode pixelMode = this.getPixelModeProperty(environment);
    ImageBanner.Frame[] frames = this.readFrames(width, height); // 读取图片的帧

    for(int i = 0; i < frames.length; ++i) {
        if (i > 0) {
            this.resetCursor(frames[i - 1].getImage(), out);
        }

        this.printBanner(frames[i].getImage(), margin, invert, bitDepth, pixelMode, out);
        this.sleep(frames[i].getDelayTime());
    }

}
  
```



**转换成ascii**

```java
private void printBanner(BufferedImage image, int margin, boolean invert, BitDepth bitDepth, ImageBanner.PixelMode pixelMode, PrintStream out) {
    AnsiElement background = invert ? AnsiBackground.BLACK : AnsiBackground.DEFAULT;
    out.print(AnsiOutput.encode(AnsiColor.DEFAULT));
    out.print(AnsiOutput.encode(background));
    out.println();
    out.println();
    AnsiElement lastColor = AnsiColor.DEFAULT;
    AnsiColors colors = new AnsiColors(bitDepth);

    for(int y = 0; y < image.getHeight(); ++y) {
        int x;
        for(x = 0; x < margin; ++x) {
            out.print(" ");
        }

        for(x = 0; x < image.getWidth(); ++x) {
            Color color = new Color(image.getRGB(x, y), false);
            AnsiElement ansiColor = colors.findClosest(color);
            if (ansiColor != lastColor) {
                out.print(AnsiOutput.encode(ansiColor));
                lastColor = ansiColor;
            } 

            out.print(this.getAsciiPixel(color, invert, pixelMode)); // // 像素点转换成字符输出
        }

        out.println();
    }

    out.print(AnsiOutput.encode(AnsiColor.DEFAULT));
    out.print(AnsiOutput.encode(AnsiBackground.DEFAULT));
    out.println();
}
  
```
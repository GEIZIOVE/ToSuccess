# SpringBoot-Email

## 1.引入依赖



```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```



## 2.application配置

**使用qq发送**

```yml
spring:
 mail:
    host: smtp.qq.com #邮件服务器地址
    username: 1601709391@qq.com #邮件服务器用户名
    password: tlxamrtiwmvbiijd #签名
    default-encoding: UTF-8 #邮件编码格式
    port: 587 #邮件服务器端口
    properties: 
      mail:
      smtp:
      auth: true 
      socketFactory:
      class: javax.net.ssl.SSLSocketFactory 

thymeleaf:
  prefix: classpath:/templates  #prefix：指定模板所在的目录
  check-template-location: true  #check-tempate-location: 检查模板路径是否存在
  cache: false  #cache: 是否缓存，开发模式下设置为false，避免改了模板还要重启服务器，线上设置为true，可以提高性能。
  suffix:  .html
  #encoding: UTF-8
  #content-type: text/html
  mode: HTML5
```

> 注意：password不是邮箱登录密码，而是第一步中获取的授权码
>

**使用163发送**

```yml
spring:
  mail:
    host: smtp.163.com #邮件服务器地址
    username: wqh8888882022@163.com #邮件服务器用户名
    password: NPXGLDEKAOOHWULU #签名 163:NPXGLDEKAOOHWULU  qq:waudfrgemnmlidjd
    default-encoding: UTF-8 #邮件编码格式
    protocol: smtp
#    port: 465 #邮件服务器端口
    properties:
      mail:
      smtp:
      auth: true
      socketFactory:
      class: javax.net.ssl.SSLSocketFactory
```

## 3.创建一个实体类EmailDTO 

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class EmailDTO {

    /**
     * 邮箱号
     */
    private String email;

    /**
     * 主题
     */
    private String subject;

    /**
     * 内容
     */
    private String content;
}
```



## 4.使用

要给工具类加上`@Component`注解
用@autowired注入JavaMailSenderImpl后 将整个类交给了Spring管理
因此类上必须加@Component
在调用该工具类的时候也要用`@Autowired`注入

**我这里结合消息队列使用**

```java
import com.alibaba.fastjson.JSON;
import com.hong.dk.bookcollect.entity.pojo.dto.EmailDTO;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Component;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;
import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

import static com.hong.dk.bookcollect.constant.MQPrefixConst.EMAIL_QUEUE;


/**
 * 通知邮箱
 * @author wqh
 **/
@Component
@RabbitListener(queues = EMAIL_QUEUE)
public class EmailConsumer {

    /**
     * 邮箱号
     */
    @Value("${spring.mail.username}")
    private String email;
    @Autowired
    private JavaMailSender javaMailSender;
    @Autowired
    private TemplateEngine templateEngine;

    @RabbitHandler
    public void process(byte[] data) throws MessagingException { //data是消息内容
        EmailDTO emailDTO = JSON.parseObject(new String(data), EmailDTO.class);
//        SimpleMailMessage message = new SimpleMailMessage();
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);

        helper.setFrom(email); //发送者
        helper.setTo(emailDTO.getEmail()); //接收者
        helper.setSubject(emailDTO.getSubject()); //主题

        String code = emailDTO.getContent();
        Map<String, Object> valueMap = new HashMap<>();
        //如果code长度为六位，则使用code_email.html模板
        if (code.length() == 6) {
            //将code转为Arraylist
            ArrayList<String> codeList = new ArrayList<>();
            for (int i = 0; i < code.length(); i++) {
                codeList.add(code.substring(i, i + 1));
            }

            valueMap.put("code", codeList);
            Context context = new Context();
            context.setVariables(valueMap);
            String content = this.templateEngine.process("code_email2", context);
            helper.setText(content, true);
        }else {
            //        valueMap.put("to", emailDTO.getEmail());
            valueMap.put("email", emailDTO.getEmail());
            valueMap.put("content", emailDTO.getContent());
            // 添加正文（使用thymeleaf模板）
            Context context = new Context();
            context.setVariables(valueMap);
            String content = this.templateEngine.process("mail", context);
            helper.setText(content, true);
        }
        javaMailSender.send(message);
    }

}
```



## 5.Thymeleaf模板

> Thymeleaf的默认配置期望所有HTML文件都放在 **resources/templates ** 目录下，以.html扩展名结尾。

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>邮箱验证码</title>
  <style>
    table {
      width: 700px;
      margin: 0 auto;
    }
    #top {
      width: 700px;
      border-bottom: 1px solid #ccc;
      margin: 0 auto 30px;
    }
    #top table {
      font: 12px Tahoma, Arial, 宋体;
      height: 40px;
    }
    #content {
      width: 680px;
      padding: 0 10px;
      margin: 0 auto;
    }
    #content_top {
      line-height: 1.5;
      font-size: 14px;
      margin-bottom: 25px;
      color: #4d4d4d;
    }
    #content_top strong {
      display: block;
      margin-bottom: 15px;
    }
    #content_top strong span {
      color: #f60;
      font-size: 16px;
    }
    #verificationCode {
      color: #f60;
      font-size: 24px;
    }
    #content_bottom {
      margin-bottom: 30px;
    }
    #content_bottom small {
      display: block;
      margin-bottom: 20px;
      font-size: 12px;
      color: #747474;
    }
    #bottom {
      width: 700px;
      margin: 0 auto;
    }
    #bottom div {
      padding: 10px 10px 0;
      border-top: 1px solid #ccc;
      color: #747474;
      margin-bottom: 20px;
      line-height: 1.3em;
      font-size: 12px;
    }
    #content_top strong span {
      font-size: 18px;
      color: #FE4F70;
    }
    #sign {
      text-align: right;
      font-size: 18px;
      color: #FE4F70;
      font-weight: bold;
    }
    #verificationCode {
      height: 100px;
      width: 680px;
      text-align: center;
      margin: 30px 0;
    }
    #verificationCode div {
      height: 100px;
      width: 680px;
    }
    .button {
      color: #FE4F70;
      margin-left: 10px;
      height: 80px;
      width: 80px;
      resize: none;
      font-size: 42px;
      border: none;
      outline: none;
      padding: 10px 15px;
      background: #ededed;
      text-align: center;
      border-radius: 17px;
      box-shadow: 6px 6px 12px #cccccc,
      -6px -6px 12px #ffffff;
    }
    .button:hover {
      box-shadow: inset 6px 6px 4px #d1d1d1,
      inset -6px -6px 4px #ffffff;
    }
  </style>
</head>
<body>
<table>
  <tbody>
  <tr>
    <td>
      <div id="top">
        <table>
          <tbody><tr><td></td></tr></tbody>
        </table>
      </div>
      <div id="content">
        <div id="content_top">
          <strong>尊敬的用户：您好！</strong>
          <strong>
            您正在进行<span>注册账号</span>操作，请在验证码中输入以下验证码完成操作：
          </strong>
          <div id="verificationCode">
            <button class="button" th:each="a:${code}">[[${a}]]</button>
          </div>
        </div>
        <div id="content_bottom">
          <small>
            注意：此操作可能会修改您的密码、登录邮箱或绑定手机。如非本人操作，请及时登录并修改密码以保证帐户安全
            <br>（工作人员不会向你索取此验证码，请勿泄漏！)
          </small>
        </div>
      </div>
      <div id="bottom">
        <div>
          <p>此为系统邮件，请勿回复<br>
            请保管好您的邮箱，避免账号被他人盗用
          </p>
          <p id="sign">————————西南石油大学校园取件系统</p>
        </div>
      </div>
    </td>
  </tr>
  </tbody>
</table>
</body>
```



## 6.可能遇到的问题

### 1.解决Error resolving template template might not exist or might not be accessible问题

很明显也很好理解，就是模板页不存在或者找不到了，其实是存在的。

```properties
spring.thymeleaf.cache=false
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

改为

```properties
spring.thymeleaf.cache=false
spring.thymeleaf.prefix=classpath:/templates
spring.thymeleaf.suffix=.html
```

但有的时候第一种管用…..我也不理解…..



### 2.邮箱的签名password

![image-20220902155354256](E:\Development\Typora\images\image-20220902155354256.png)



每次修改qq密码以后都需要重新申请签名….





## Springboot mail 文本邮件

文本邮件是最简单也是最基础的一种邮件，使用 Spring 封装的 `JavaMailSender` 直接发送就可以了。

创建 `MailService` 类，注入 `JavaMailSender` 用于发送邮件，使用 `@Value("${spring.mail.username}")` 绑定配置文件中的参数用于设置邮件发送的来邮箱。使用 `@Service` 注解把 `MailService` 注入到 Spring 容器，使用 `Lombok` 的 `@Slf4j` 引入日志。

```java
/**
 * <p>
 * 邮件服务
 *
 * @Author niujinpeng
 * @Date 2019/3/10 21:45
 */
@Service
@Slf4j
public class MailService {

    @Value("${spring.mail.username}")
    private String from;

    @Autowired
    private JavaMailSender mailSender;

    /**
     * 发送简单文本邮件
     * 
     * @param to
     * @param subject
     * @param content
     */
    public void sendSimpleTextMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        message.setFrom(from);
        mailSender.send(message);
        log.info("【文本邮件】成功发送！to={}", to);
    }
}
```



创建 Springboot 的单元测试类测试文本邮件，实验中的收信人为了方便，都设置成了自己的邮箱。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MailServiceTest {

    @Autowired
    private MailService mailService;
    @Autowired
    private TemplateEngine templateEngine;

    @Test
    public void sendSimpleTextMailTest() {
        String to = "niumoo@126.com";
        String subject = "Springboot 发送简单文本邮件";
        String content = "<p>第一封 Springboot 简单文本邮件</p>";
        mailService.sendSimpleTextMail(to, subject, content);
    }
}
```



运行单元测试，测试文本邮件的发送。

PS：如果运行报出异常 `AuthenticationFailedException: 535 Error`. 一般都是用户名和密码有误。

```log
Caused by: javax.mail.AuthenticationFailedException: 535 Error: authentication failed

	at com.sun.mail.smtp.SMTPTransport$Authenticator.authenticate(SMTPTransport.java:965)
	at com.sun.mail.smtp.SMTPTransport.authenticate(SMTPTransport.java:876)
	at com.sun.mail.smtp.SMTPTransport.protocolConnect(SMTPTransport.java:780)
	at javax.mail.Service.connect(Service.java:366)
	at org.springframework.mail.javamail.JavaMailSenderImpl.connectTransport(JavaMailSenderImpl.java:517)
	at org.springframework.mail.javamail.JavaMailSenderImpl.doSend(JavaMailSenderImpl.java:436)
	... 34 more
```

正常运行输出成功发送的日志。

```log
2019-03-11 23:35:14.743  INFO 13608 --- [           main] n.codingme.boot.service.MailServiceTest  : Started MailServiceTest in 3.964 seconds (JVM running for 5.749)
2019-03-11 23:35:24.718  INFO 13608 --- [           main] net.codingme.boot.service.MailService    : 【文本邮件】成功发送！to=niumoo@126.com
```

查看邮箱中的收信。

![文本邮件](E:\Development\Typora\images\e391dbce4b614779ae9a024908ef0275.jpg)

文本邮件正常收到，同时可见文本邮件中的 HTML 标签也不会被解析。

## [#](https://www.wdbyte.com/2019/03/springboot/springboot-13-email/#springboot-mail-html-邮件)Springboot mail HTML 邮件

在上面的 `MailService` 类里新加一个方法 `sendHtmlMail`，用于测试 HTML 邮件。

```java
    /**
     * 发送 HTML 邮件
     * 
     * @param to
     * @param subject
     * @param content
     * @throws MessagingException
     */
    public void sendHtmlMail(String to, String subject, String content) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();
        MimeMessageHelper messageHelper = new MimeMessageHelper(message, true);
        messageHelper.setFrom(from);
        messageHelper.setTo(to);
        messageHelper.setSubject(subject);
        // true 为 HTML 邮件
        messageHelper.setText(content, true);
        mailSender.send(message);
        log.info("【HTML 邮件】成功发送！to={}", to);
    }
```



在测试方法中增加 HTML 邮件测试方法。

```java
    @Test
    public void sendHtmlMailTest() throws MessagingException {
        String to = "niumoo@126.com";
        String subject = "Springboot 发送 HTML 邮件";
        String content = "<h2>Hi~</h2><p>第一封 Springboot HTML 邮件</p>";
        mailService.sendHtmlMail(to, subject, content);
    }
```



运行单元测试，查看收信情况。

![HTML 邮件](E:\Development\Typora\images\147b7f68d12e0080b408a581414fb92a.jpg)

HTML 邮件正常收到，HTML 标签也被解析成对应的样式。

## [#](https://www.wdbyte.com/2019/03/springboot/springboot-13-email/#springboot-mail-附件邮件)Springboot mail 附件邮件

在上面的 `MailService` 类里新加一个方法 `sendAttachmentMail`，用于测试 附件邮件。

```java
    /**
     * 发送带附件的邮件
     * 
     * @param to
     * @param subject
     * @param content
     * @param fileArr
     */
    public void sendAttachmentMail(String to, String subject, String content, String... fileArr)
        throws MessagingException {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper messageHelper = new MimeMessageHelper(mimeMessage, true);
        messageHelper.setFrom(from);
        messageHelper.setTo(to);
        messageHelper.setSubject(subject);
        messageHelper.setText(content, true);

        // 添加附件
        for (String filePath : fileArr) {
            FileSystemResource fileResource = new FileSystemResource(new File(filePath));
            if (fileResource.exists()) {
                String filename = fileResource.getFilename();
                messageHelper.addAttachment(filename, fileResource);
            }
        }
        mailSender.send(mimeMessage);
        log.info("【附件邮件】成功发送！to={}", to);
    }
```



在测试方法中增加附件邮件测试方法。

```java
    @Test
    public void sendAttachmentTest() throws MessagingException {
        String to = "niumoo@126.com";
        String subject = "Springboot 发送 HTML 附件邮件";
        String content = "<h2>Hi~</h2><p>第一封 Springboot HTML 附件邮件</p>";
        String filePath = "pom.xml";
        mailService.sendAttachmentMail(to, subject, content, filePath, filePath);
    }
```



运行单元测试，查看收信情况。

![附件邮件](E:\Development\Typora\images\0ab42e65ae35c7970d7954bf5a985a79.jpg)

带附件的邮件正常收到，多个附件的实现方式同理。

## [#](https://www.wdbyte.com/2019/03/springboot/springboot-13-email/#springboot-mail-图片邮件)Springboot mail 图片邮件

图片邮件和其他的邮件方式略有不同，图片邮件需要先在内容中定义好图片的位置并出给一个记录 ID ，然后在把图片加到邮件中的对于的 ID 位置。

在上面的 `MailService` 类里新加一个方法 `sendImgMail`，用于测试 附件邮件。

```java
   /**
     * 发送带图片的邮件
     *
     * @param to
     * @param subject
     * @param content
     * @param imgMap
     */
    public void sendImgMail(String to, String subject, String content, Map<String, String> imgMap)
        throws MessagingException {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper messageHelper = new MimeMessageHelper(mimeMessage, true);
        messageHelper.setFrom(from);
        messageHelper.setTo(to);
        messageHelper.setSubject(subject);
        messageHelper.setText(content, true);
        // 添加图片
        for (Map.Entry<String, String> entry : imgMap.entrySet()) {
            FileSystemResource fileResource = new FileSystemResource(new File(entry.getValue()));
            if (fileResource.exists()) {
                String filename = fileResource.getFilename();
                messageHelper.addInline(entry.getKey(), fileResource);
            }
        }
        mailSender.send(mimeMessage);
        log.info("【图片邮件】成功发送！to={}", to);
    }
```



在测试方法中增加图片邮件测试方法，测试方法中使用的 apple.png 是项目里的一个图片。可以看上面的项目结构。

```java
    @Test
    public void sendImgTest() throws MessagingException {
        String to = "niumoo@126.com";
        String subject = "Springboot 发送 HTML 图片邮件";
        String content =
            "<h2>Hi~</h2><p>第一封 Springboot HTML 图片邮件</p><br/><img src=\"cid:img01\" /><img src=\"cid:img02\" />";
        String imgPath = "apple.png";
        Map<String, String> imgMap = new HashMap<>();
        imgMap.put("img01", imgPath);
        imgMap.put("img02", imgPath);
        mailService.sendImgMail(to, subject, content, imgMap);
    }
```



运行单元测试，查看收信情况。

![图片邮件](E:\Development\Typora\images\a676fadf6b40195c7de746adff0700bc.jpg)

两个图片正常显示在邮件里。

## [#](https://www.wdbyte.com/2019/03/springboot/springboot-13-email/#springboot-mail-模版邮件)Springboot mail 模版邮件

模版邮件的用处很广泛，像经常收到的注册成功邮件或者是操作通知邮件等都是模版邮件，模版邮件往往只需要更改其中的几个变量。Springboot 中的模版邮件首选需要选择一款模版引擎，在引入依赖的时候已经增加了模版引擎 `Thymeleaf`.

模版邮件首先需要一个邮件模版，我们在 `Templates` 下新建一个 `HTML` 文件 `RegisterSuccess.html`. 其中的 username 是给我们自定义的。

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>注册成功通知</title>
</head>
<body>
<p>[[${username}]]，您好!
</p>
<p>
    新的公钥已添加到你的账户:<br/>
    标题: HP-WIN10 <br/>
    如果公钥无法使用，你可以在这里重新添加： SSH Keys
</p>
</body>
</html>
```

在邮件服务 `MailService` 中注入模版引擎，然后编写邮件模版发送代码。

```java
    @Autowired
    private TemplateEngine templateEngine;

    /**
     * 发送模版邮件
     * 
     * @param to
     * @param subject
     * @param paramMap
     * @param template
     * @throws MessagingException
     */
    public void sendTemplateMail(String to, String subject, Map<String, Object> paramMap, String template)
        throws MessagingException {
        Context context = new Context();
        // 设置变量的值
        context.setVariables(paramMap);
        String emailContent = templateEngine.process(template, context);
        sendHtmlMail(to, subject, emailContent);
        log.info("【模版邮件】成功发送！paramsMap={}，template={}", paramMap, template);
    }
```

在单元单元测试中增加模版邮件测试方法，然后发送邮件测试。

```java
    @Test
    public void sendTemplateMailTest() throws MessagingException {
        String to = "niumoo@126.com";
        String subject = "Springboot 发送 模版邮件";
        Map<String, Object> paramMap = new HashMap();
        paramMap.put("username", "Darcy");
        mailService.sendTemplateMail(to, subject, paramMap, "RegisterSuccess");
    }
```

查看收信情况。

![模版邮件](E:\Development\Typora\images\8244d50efe797cfa3ceb14aab7b7b461.jpg)

可以发现模版邮件已经正常发送了。










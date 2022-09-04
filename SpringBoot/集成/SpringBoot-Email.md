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
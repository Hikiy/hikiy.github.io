---
layout: post
title:  "Spring Boot学习记录：发送邮件"
date:   2019-05-22 17:24:00 +0200
categories: SpringBoot
excerpt: 在Spring Boot中发送邮件
tagg: Spring
---

# 发送邮件

### pom依赖
```
<dependencies>
	<dependency> 
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-mail</artifactId>
	</dependency> 
</dependencies>
```
### application.properties配置
```
# 邮箱配置
# 邮箱服务器地址
spring.mail.host=smtp.qq.com
spring.mail.username=xxx@qq.com
# 授权码
spring.mail.password=xxxxx
spring.mail.default-encoding=UTF-8
# 以谁来发送邮件
mail.fromMail.addr=xxx@qq.com
# 超时时间（可选）
# spring.mail.properties.mail.smtp.connectiontimeout=5000
# spring.mail.properties.mail.smtp.timeout=3000
# spring.mail.properties.mail.smtp.writetimeout=5000

# 163
# spring.mail.host=smtp.163.com #邮箱服务器地址
# spring.mail.username=xxx.163.com #用户名
# spring.mail.password=ooo #开启POP3之后设置的客户端授权码
# spring.mail.default-encoding=UTF-8 #编码
```
### 实现类
```
@Component
public class MailUtil {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    @Autowired
    private JavaMailSender javaMailSender;

    @Value("${mail.fromMail.addr}")
    private String from;
    
    //发送普通邮件
    public void sendMail(String to, String subject, String content){
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);

        try {
            javaMailSender.send(message);
            logger.info("邮件已经发送。");
        } catch (Exception e) {
            logger.error("发送邮件时发生异常！", e);
        }
    }

    //发送HTML邮件
    public void sendHtmlMail(String to, String subject, String content) {
        MimeMessage message = javaMailSender.createMimeMessage();

        try {
            //true表示需要创建一个multipart message
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(content, true);

            javaMailSender.send(message);
            logger.info("html邮件发送成功");
        } catch (Exception e) {
            logger.error("发送html邮件时发生异常！", e);
        }
    }
    
    //发送附件邮件
    public void sendAttachmentsMail(String to, String subject, String content, String filePath){
        MimeMessage message = javaMailSender.createMimeMessage();

        try {
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(content, true);

            FileSystemResource file = new FileSystemResource(new File(filePath));
            String fileName = filePath.substring(filePath.lastIndexOf(File.separator));
            helper.addAttachment(fileName, file);

            javaMailSender.send(message);
            logger.info("附件邮件已经发送。");
        } catch (Exception e) {
            logger.error("发送附件邮件时发生异常！", e);
        }
    }
}
```
### 测试
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestMail {
    @Autowired
    private MailUtil mailUtil;

    @Test
    @Ignore
    public void testSendMail(){
        String to = "xxx@qq.com";
        String subject = "来自Hiki的邮件哦！";
        String content = "一路走来累了吧，别放弃";
        mailUtil.sendMail(to,subject,content);
    }

    @Test
    @Ignore
    public void sendHTMLMail() {
        String content="<html>\n" +
                "<body>\n" +
                "    <h3>来自Hiki的HTML邮件哦！</h3>\n" +
                "    <a href=\"https://github.com/Hikiy\">Hiki的Github</a>\n"+
                "    <p>一路走来累了吧，别放弃</p>"+
                "</body>\n" +
                "</html>";
        mailUtil.sendHtmlMail("xxx@qq.com","来自Hiki的HTML邮件哦！",content);
    }
    
    @Test
    @Ignore
    public void sendAttachmentsMail() {
        String filePath="e:\\mail\\test.txt";
        String content="<html>\n" +
                "<body>\n" +
                "    <h3>来自Hiki的HTML邮件哦！</h3>\n" +
                "    <a href=\"https://github.com/Hikiy\">Hiki的Github</a>\n"+
                "    <p>送你个文件</p>"+
                "</body>\n" +
                "</html>";
        mailUtil.sendAttachmentsMail("695954591@qq.com", "来自Hiki的HTML邮件哦！", content, filePath);
    }
}
```

### 邮件模板
这里用thymeleaf  

**pom依赖**
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
**`resorces/templates` 下创建 `email.html` **
```
<!DOCTYPE html>
<html lang="zh" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8"/>
        <title>Title</title>
    </head>
    <body>
        小老弟,这是Hiki的邮件,请点击下面的链接<br/>
        <a href="#" th:href="@{ https://github.com/Hikiy/{id}(id=${id}) }">开启新大陆</a>
    </body>
</html>
```
**测试**
```
    @Test
    public void sendTemplateMail() {
    //目前会报错。。暂时没找到什么原因
        Context context = new Context();
        context.setVariable("id", "SpringBootLearn");
        String emailContent = templateEngine.process("emailT", context);
        mailUtil.sendHtmlMail("695954591@qq.com","来自Hiki的HTML邮件哦！",emailContent);
    }
```

  


<br /><br /><br /><br />
> [项目代码](https://github.com/Hikiy/SpringBootLearn)  
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.05.22  
> 更新日期：2019.05.23

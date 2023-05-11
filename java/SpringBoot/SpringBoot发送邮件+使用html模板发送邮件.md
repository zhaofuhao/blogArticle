## SpringBoot发送邮件+使用html模板发送邮件

这两天在公司做商城系统有一个业务用到了发送邮件功能 springboot 有`spring-boot-starter-mail`

### 前期准备

邮箱需要开启smtp服务 获得smtp密钥

1. 第一步引入pom依赖

```xml
 <dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

2. yml配置

```yaml
spring:
  mail:
    username: ***********@163.com
    password: **** #邮箱提供的smtp密钥
    host: smtp-mail.163.com
    port: 587
    properties:
      mail:
        smtp:
          starttls:
            required: true
```

3. html模板准备

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="description" content="email code">
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<!--邮箱发票模板-->
<body>
<div style="background-color:#ECECEC; padding: 35px;">
    <table cellpadding="0" align="center"
           style="width: 800px;height: 100%; margin: 0px auto; text-align: left; position: relative; border-top-left-radius: 5px; border-top-right-radius: 5px; border-bottom-right-radius: 5px; border-bottom-left-radius: 5px; font-size: 14px; font-family:微软雅黑, 黑体; line-height: 1.5; box-shadow: rgb(153, 153, 153) 0px 0px 5px; border-collapse: collapse; background-position: initial initial; background-repeat: initial initial;background:#fff;">
        <tbody>
        <tr>
            <th valign="middle"
                style="height: 25px; line-height: 25px; padding: 15px 35px; border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: RGB(148,0,211); background-color: RGB(148,0,211); border-top-left-radius: 5px; border-top-right-radius: 5px; border-bottom-right-radius: 0px; border-bottom-left-radius: 0px;">
                <font face="微软雅黑" size="5" style="color: rgb(255, 255, 255); ">投豹</font>
            </th>
        </tr>
        <tr>
            <td style="word-break:break-all">
                <div style="padding:25px 35px 40px; background-color:#fff;opacity:0.8;">

                    <h2 style="margin: 5px 0px; ">
                        <font color="#333333" style="line-height: 20px; ">
                            <font style="line-height: 22px; " size="4">
                                尊敬的用户：</font>
                        </font>
                    </h2>
                    <!-- 中文 -->
                    <p>您好！您申请的发票已经开票</p><br>
                    <!-- 英文 -->
                    <h2 style="margin: 5px 0px; ">
                        <font color="#333333" style="line-height: 20px; ">
                            <font style="line-height: 22px; " size="4">
                               发票下载</font>
                        </font>
                    </h2>
                    <p><font color="#ff8c00">{0}</font></p>
                    <div style="width:100%;margin:0 auto;">
                        <div style="padding:10px 10px 0;border-top:1px solid #ccc;color:#747474;margin-bottom:20px;line-height:1.3em;font-size:12px;">
                         <!--- <p>****团队</p>
                            <p>联系我们：********</p>
                            <br> !-->
                            <p>此为系统邮件，请勿回复<br>
                                Please do not reply to this system email
                            </p>
                            <!--<p>©***</p>-->
                        </div>
                    </div>
                </div>
            </td>
        </tr>
        </tbody>
    </table>
</div>
</body>
</html>
```



### 开发

```java

    @Autowired
    JavaMailSenderImpl mailSender;

/**
     * 发送带附近的邮件信息
     * @param to 收件人
     * @param subject  主题
     * @param content  内容
     * @param filePath 附件
     */
    public static void sendAttachmentsMail(String to, String subject, String content, String filePath) {
           MimeMessage message = javaMailSender.createMimeMessage();
        try {
           
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setSubject(subject);
            helper.setText(buildContent(filePath + ""), true);
            helper.setTo(email);
            helper.setFrom(to);
            javaMailSender.send(message);
        } catch (MessagingException e) {
            throw new CustomerException(messageSourceUtil.getMessage(I18nConstant.SYSTEM_ERROR), "500");
        }

    }


    /**
     * 读取邮件模板
     * 替换模板中的信息
     *
     * @param title 内容
     * @return
     */
    public static String buildContent(String title) {
        //加载邮件html模板
        Resource resource = new ClassPathResource("templates/mailtemplate.ftl");
        InputStream inputStream = null;
        BufferedReader fileReader = null;
        StringBuffer buffer = new StringBuffer();
        String line = "";
        try {
            inputStream = resource.getInputStream();
            fileReader = new BufferedReader(new InputStreamReader(inputStream));
            while ((line = fileReader.readLine()) != null) {
                buffer.append(line);
            }
        } catch (Exception e) {
            log.info("发送邮件读取模板失败{}", e);
        } finally {
            if (fileReader != null) {
                try {
                    fileReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        //替换html模板中的参数
        return MessageFormat.format(buffer.toString(), title);
    }
```


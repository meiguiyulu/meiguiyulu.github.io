# Springboot邮件

*****************

## 1. 依赖

```xml
        <!-- mail 依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>

<!--  
		如果存在空指针异常 可以添加该依赖
		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
        </dependency>-->
```

## 2. `yaml`配置文件

```yaml
spring:
  mail:
    # 邮件服务器地址
    host: smtp.163.com
    # 协议
    protocol: smtp
    # 编码格式
    default-encoding: utf-8
    # 授权码（在邮箱开通服务时获取）
    password: 邮箱的授权码
    # 发送者邮箱地址
    username: 自己的邮箱(例如xxx@163.com)
    # 端口（不同邮箱端口号不同）
    port: 25
```

## 3. 发送邮件

### 3.1 普通邮件

```java
@SpringBootTest
class MailApplicationTests {

    /*
        @Value("${spring.mail.username}")
        private String mailUsername;
        */
    @Autowired
    private MailProperties mailProperties;

    @Autowired
    private JavaMailSender javaMailSender;

    @Test
    void sendSimpleMail() {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setSubject("邮件主题: 简单邮件");
        message.setText("邮件内容: 测试简单邮件");
        message.setFrom(mailProperties.getUsername());
        message.setTo("1609226090@qq.com");
        javaMailSender.send(message);
    }

}
```

### 3.2 带有html标签的邮件

```java
@SpringBootTest
class MailApplicationTests {

    /*
        @Value("${spring.mail.username}")
        private String mailUsername;
        */
    @Autowired
    private MailProperties mailProperties;

    @Autowired
    private JavaMailSender javaMailSender;

    @Test
    void sendMailWithHtml() throws MessagingException {
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message);
        helper.setSubject("邮件主题: 带有Html标签");
        helper.setText("你好, <h1>云杰</h1>", true); // true表示文本中含有html标签
        helper.setFrom(mailProperties.getUsername());
        helper.setTo("1609226090@qq.com");
        javaMailSender.send(message);
    }
}
```

![image-20220425130522089](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220425130522089.png)

### 3.3 带有附件的邮件

​		注意这里在构建邮件对象上和前文有所差异，这里是通过 `javaMailSender` 来获取一个复杂邮件对象，然后再利用 `MimeMessageHelper` 对邮件进行配置，`MimeMessageHelper` 是一个邮件配置的辅助工具类，创建时候的 `true` 表示构建一个 `multipart message` 类型的邮件，有了 `MimeMessageHelper` 之后，我们针对邮件的配置都是由 `MimeMessageHelper` 来代劳。

​		最后通过 `addAttachment` 方法来添加一个附件。

```java
@SpringBootTest
class MailApplicationTests {

    /*
        @Value("${spring.mail.username}")
        private String mailUsername;
        */
    @Autowired
    private MailProperties mailProperties;

    @Autowired
    private JavaMailSender javaMailSender;


    @Test
    void sendMailWithAttach() throws MessagingException {
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setSubject("邮件主题: 带有附件");
        helper.setFrom(mailProperties.getUsername());
        helper.setTo("1609226090@qq.com");
        helper.setText("这是测试附件的邮件正文");
        helper.setSentDate(new Date());
        helper.addAttachment("_20220207183327.jpg", new File("D:\\相册\\_20220207183327.jpg"));
        javaMailSender.send(message);
    }
}

```

![image-20220425185039251](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220425185039251.png)

### 3.4 带有图片的邮件

这里的邮件 `text` 是一个 `HTML` 文本，里边涉及到的图片资源先用**一个占位符(唯一的`id`标识)**占着，`setText` 方法的第二个参数 `true` 表示第一个参数是一个 `HTML` 文本。

`setText` 之后，再通过 `addInline` 方法来添加图片资源。`addInline`方法的第一个参数是 **占位符**，第二个参数是 **图片**。

```java
@SpringBootTest
class MailApplicationTests {

    /*
        @Value("${spring.mail.username}")
        private String mailUsername;
        */
    @Autowired
    private MailProperties mailProperties;

    @Autowired
    private JavaMailSender javaMailSender;

    @Test
    void sendMailWithImg() throws MessagingException {
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setSubject("邮件主题: 带有图片");
        helper.setFrom(mailProperties.getUsername());
        helper.setTo("1609226090@qq.com");
        helper.setSentDate(new Date());

        /* 图片1 */
        String id1 = "test1";
        FileSystemResource resource1 = new FileSystemResource(new File("D:\\相册\\_20220207183327.jpg"));
        /* 图片2 */
        String id2 = "test2";
        FileSystemResource resource2 = new FileSystemResource(new File("D:\\相册\\_20220207183350.jpg"));

        helper.setText("<p>hello 大家好，这是一封测试邮件，这封邮件包含两种图片，分别如下</p>" +
                "<p>第一张图片：</p><img src='cid:" + id1 + "'/>" +
                "<p>第二张图片：</p><img src='cid:" + id2 + "'/>", true);
        helper.addInline(id1, resource1);
        helper.addInline(id2, resource2);

        javaMailSender.send(message);
    }
}
```

![image-20220425185953475](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220425185953475.png)

> 但是在实际开发中，第二种和第三种都不是使用最多的邮件发送方案。因为正常来说，邮件的内容都是比较的丰富的，所以大部分邮件都是通过 **`HTML`** 来呈现的，如果直接拼接 **`HTML`**  字符串，这样以后不好维护，为了解决这个问题，一般邮件发送，都会有相应的邮件模板。最具代表性的两个模板就是 `Freemarker` 模板和 `Thyemeleaf` 模板了。

## 4. 邮件与模板引擎相结合

### 4.1 使用 Freemaker

#### 4.1.1 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

#### 4.1.2 创建 `mail.ftl` 作为邮件发送模板

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p>hello 欢迎加入 xxx 大家庭，您的入职信息如下：</p>
<table border="1">
    <tr>
        <td>姓名</td>
        <td>${username}</td>
    </tr>
    <tr>
        <td>工号</td>
        <td>${num}</td>
    </tr>
    <tr>
        <td>薪水</td>
        <td>${salary}</td>
    </tr>
</table>
<div style="color: #ff1a0e">一起努力创造辉煌</div>
</body>
</html>
```

#### 4.1.3 发送邮件

```java
@Data
public class User {

    private String username;
    private Integer num;
    private Double salary;

}
```



```java
    @Test
    public void sendFreemarkerMail() throws Exception {
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setSubject("邮件主题: 带有图片");
        helper.setFrom(mailProperties.getUsername());
        helper.setTo("1609226090@qq.com");
        helper.setSentDate(new Date());
        /* 构建Freemarker的基本配置 */
        Configuration configuration = new Configuration(Configuration.VERSION_2_3_0);
        /* 配置模板位置 */
        ClassLoader loader = MailApplication.class.getClassLoader();
        configuration.setClassLoaderForTemplateLoading(loader, "templates");
        //加载模板
        Template template = configuration.getTemplate("mail.ftl");
        User user = new User();
        user.setUsername("云杰");
        user.setNum(1);
        user.setSalary((double) 99999);
        StringWriter out = new StringWriter();
        //模板渲染，渲染的结果将被保存到 out 中 ，将out 中的 html 字符串发送即可
        template.process(user, out);
        helper.setText(out.toString(), true);
        javaMailSender.send(message);
    }
```

### 4.2 使用Thymeleaf

​		推荐在 Spring Boot 中使用 Thymeleaf 来构建邮件模板。因为 Thymeleaf 的自动化配置提供了一个 TemplateEngine，通过 TemplateEngine 可以方便的将 Thymeleaf 模板渲染为 HTML ，同时，Thymeleaf 的自动化配置在这里是继续有效的 。

#### 4.2.1 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

```

#### 4.2.2 创建 `Thymeleaf` 邮件模板

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p>hello 欢迎加入 xxx 大家庭，您的入职信息如下：</p>
<table border="1">
    <tr>
        <td>姓名</td>
        <td th:text="${username}"></td>
    </tr>
    <tr>
        <td>工号</td>
        <td th:text="${num}"></td>
    </tr>
    <tr>
        <td>薪水</td>
        <td th:text="${salary}"></td>
    </tr>
</table>
<div style="color: #ff1a0e">一起努力创造辉煌</div>
</body>
</html>

```

#### 4.2.3 发送邮件

```java
    @Autowired
    private MailProperties mailProperties;

    @Autowired
    private JavaMailSender javaMailSender;

    @Autowired
    private TemplateEngine templateEngine;

    @Test
    public void sendThymeleafMail() throws MessagingException {
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setSubject("邮件主题: 使用Thymeleaf模板");
        helper.setFrom(mailProperties.getUsername());
        helper.setTo("1609226090@qq.com");
        helper.setSentDate(new Date());

        /* 使用Thymeleaf模板 */
        Context context = new Context();
        context.setVariable("username", "小杰");
        context.setVariable("num", "00001");
        context.setVariable("salary", "99999");
        String process = templateEngine.process("mail.html", context);

        helper.setText(process, true);
        javaMailSender.send(message);
    }
```

![image-20220425195521995](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220425195521995.png)
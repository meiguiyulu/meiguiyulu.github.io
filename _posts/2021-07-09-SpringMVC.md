---
layout: post
title: "SpringMVC"
date: 2021-07-09 09:00:00 +0800 
categories: 研究生
tag: SpringMVC
---
* content
{:toc}
# SpringMVC

## 1、MVC

**MVC是三个单词的首字母缩写，它们是Model（模型）、View（视图）和Controller（控制）。**

![image-20210630131320902](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210630131320902.png)

这个模式认为，程序不论简单或复杂，从结构上看，都可以分成三层。

```xml
1）最上面的一层，是直接面向最终用户的"视图层"（View）。它是提供给用户的操作界面，是程序的外壳。

2）最底下的一层，是核心的"数据层"（Model），也就是程序需要操作的数据或信息。

3）中间的一层，就是"控制层"（Controller），它负责根据用户从"视图层"输入的指令，选取"数据层"中的数据，然后对其进行相应的操作，产生最终结果。
```

这三层是紧密联系在一起的，但又是互相独立的，每一层内部的变化不影响其他层。每一层都对外提供接口（Interface），供上面一层调用。这样一来，软件就可以实现模块化，修改外观或者变更数据都不用修改其他层，大大方便了维护和升级。

## 2、回顾Servlet

- 在 `index.jsp` 新建一个form表单

  - ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
      <head>
        <title>$Title$</title>
      </head>
      <body>
        <form action="/hello" method="post">
          <input name="method">
          <input type="submit">
        </form>
      </body>
    </html>
    ```

- 新建一个 `HelloServlet` 类

  - ```java
    public class HelloServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            // 获取前端参数
            String method = req.getParameter("method");
            if ("add".equals(method)){
                req.getSession().setAttribute("msg", "执行了add方法");
            }
            if ("delete".equals(method)){
                req.getSession().setAttribute("msg", "执行了delete方法");
            }
            // 调用业务层
    
            // 视图转发或者重定向
            req.getRequestDispatcher("/jsp/test.jsp").forward(req, resp);
        }
    
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            doGet(req, resp);
        }
    }
    ```

- 在 `web.xml` 中配置 `HelloServlet`

  - ```xml
        <servlet>
            <servlet-name>HelloServlet</servlet-name>
            <servlet-class>servlet.HelloServlet</servlet-class>
        </servlet>
        <servlet-mapping>
            <servlet-name>HelloServlet</servlet-name>
            <url-pattern>/hello</url-pattern>
        </servlet-mapping>
    ```

- 新建 `test.jsp`

  - ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
      ${msg}
    </body>
    </html>
    ```

## 3、Hello SpringMVC

### 3.1、配置DispatchServlet

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--配置DispatchServlet: 这个事SpringMVC的核心：请求分发器、前端控制器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--DispatcherServlet要绑定Spring的配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <!--启动级别设置1, 即服务器一启动就启动-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!--
    SpringMVC中，/与/*的区别
    /   只匹配所有的请求, 不会去匹配jsp页面;
    /*  匹配所有的请求, 包括jsp页面;
    -->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

### 3.2、编写配置文件springmvc-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--处理器映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!--处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--BeanNameUrlHandlerMapping-->
    <bean id="/hello" class="controller.helloController"/>

</beans>
```

### 3.3、编写控制器类

```java
public class helloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView modelAndView = new ModelAndView();

        // 业务代码
        String result = "Hello SpringMVC";
        modelAndView.addObject("msg", result);

        // 视图跳转
        modelAndView.setViewName("test");

        return modelAndView;
    }
}
```

### 3.4、编写前端页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${msg}
</body>
</html>
```



## 4、注解开发SpringMVC

### 4.1、配置 `web.xml`，注册 `DispatcherServlet`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--配置DispatchServlet: 这个事SpringMVC的核心：请求分发器、前端控制器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--DispatcherServlet要绑定Spring的配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
        <!--启动级别设置1, 即服务器一启动就启动-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!--
    SpringMVC中，/与/*的区别
    /   只匹配所有的请求, 不会去匹配jsp页面;
    /*  匹配所有的请求, 包括jsp页面;
    -->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

### 4.2、编写springmvc配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--自动扫描包, 让指定包下的注解生效, 由IoC容器统一管理-->
    <context:component-scan base-package="controller"/>
    <!--让SpringMVC不处理.css、.js、.html、.mp3、.mp4等静态资源-->
    <mvc:default-servlet-handler/>
    <!--
    支持MVC注解驱动：
        在spring中一般采用@RequestMapping注解来完成映射关系
        要想使@RequestMapping注解生效
        必须向上下文注册DefaultAnnotationHandlerMapping和AnnotationMethodHandlerAdapter实例
        这两个实例分别在类级别和方法级别处理
        而annotation-driven配置可以帮助我们自动完成上述两个实例的注入
    -->
    <mvc:annotation-driven/>

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

### 4.3、创建对应的 controller控制类 

```java
@Controller
//@RequestMapping("/hello")
public class helloController {

    // localhost:8080/h1
    @RequestMapping("/h1")
    public String Hello(Model model){
        model.addAttribute("msg", "Hello SpringMVC");
        return "hello";  // 会被视图解析器处理
    }
}
```

### 4.4、编写前端页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    ${msg}
</body>
</html>
```

![image-20210707133334902](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210707133334902.png)

## 5、Controller配置总结

- 控制器负责解析用户的请求并将其转换为一个模型。
- 在Spring MVC中一个控制器类可以包含多个方法

### 5.1、Controller的配置方式

#### 5.1.1、实现Controller接口

```java
//实现该接口的类获得控制器功能
public interface Controller {
   //处理请求且返回一个模型与视图对象
   ModelAndView handleRequest(HttpServletRequest var1, HttpServletResponse var2) throws Exception;
}
```

**测试**

- 编写一个 `Controller` 类 `ControllerTest1`

  - ```java
    public class controllerTest1 implements Controller {
        @Override
        public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
            ModelAndView modelAndView = new ModelAndView();
            modelAndView.addObject("msg", "controllerTest1");
            modelAndView.setViewName("test");
    
            return modelAndView;
        }
    }
    ```

- 在Spring配置文件中注册请求的bean；name对应请求路径，class对应处理请求的类

  - ```xml
    <bean id="/test1" class="controller.controllerTest1"/>
    ```

- 编写前端test.jsp，注意在对应的目录下编写，对应我们的视图解析器

  - ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
        ${msg}
    </body>
    </html>
    ```

  - ![image-20210707143559015](C:/Users/LYJ/AppData/Roaming/Typora/typora-user-images/image-20210707143559015.png)

- ![image-20210707143621766](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210707143621766.png)

缺点：

- 实现接口Controller定义控制器是较老的办法
- 缺点是：一个控制器中只有一个方法，如果要多个方法则需要定义多个Controller；定义的方式比较麻烦

#### 5.1.2、使用注解@Controller

- 在Spring配置文件中声明组件扫描

  - ```xml
        <!--自动扫描包, 让指定包下的注解生效, 由IoC容器统一管理-->
        <context:component-scan base-package="controller"/>
    ```

- 增加一个 `controllerTest2`类，使用注解实现

  - ```java
    @Controller
    public class controllerTest2 {
        @RequestMapping("/test2")
        public String printHello(Model model){
    
            model.addAttribute("msg", "controllerTest2");
            return "test";
        }
    }
    ```

  - ![image-20210707145023109](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210707145023109.png)

- ![image-20210707145214401](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210707145214401.png)

### 5.2、RequestMapping

- @RequestMapping注解用于映射url到控制器类或一个特定的处理程序方法。可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

- ```java
  @Controller
  @RequestMapping("/c3")
  public class controllerTest3 {
  
      @RequestMapping("test3")
      public String test(Model model){
          model.addAttribute("msg", "Hello, this is controllerTest3");
          return "test";
      }
  }
  ```

  - ![image-20210707145805116](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210707145805116.png)



## 6、RestFul 风格

### 6.1、概念

Restful就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

**传统方式操作资源**  ：通过不同的参数来实现不同的效果！方法单一，post 和 get

​	http://127.0.0.1/item/queryItem.action?id=1 查询,GET

​	http://127.0.0.1/item/saveItem.action 新增,POST

​	http://127.0.0.1/item/updateItem.action 更新,POST

​	http://127.0.0.1/item/deleteItem.action?id=1 删除,GET或POST

**使用RESTful操作资源** ：可以通过不同的请求方式来实现不同的效果！如下：请求地址一样，但是功能可以不同！

​	http://127.0.0.1/item/1 查询,GET

​	http://127.0.0.1/item 新增,POST

​	http://127.0.0.1/item 更新,PUT

​	http://127.0.0.1/item/1 删除,DELETE

### 6.2、测试

#### 6.2.1、传统方法

```java
@Controller
public class controllerTest4 {

    @RequestMapping("/add")
    public String test1(int a, int b, Model model){
        int res = a + b;
        model.addAttribute("msg", "结果是:\t" + res);
        return "test";
    }
}
```

![image-20210707153227051](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210707153227051.png)

#### 6.2.2、RestFul风格

```java
    @RequestMapping("/add/{a}/{b}")
    public String test2(@PathVariable int a, @PathVariable int b, Model model){
        int res = a + b;
        model.addAttribute("msg", "结果是:\t" + res);
        return "test";
    }
```

![image-20210707163857189](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210707163857189.png)

**使用method属性指定请求类型**

用于约束请求的类型，可以收窄请求范围。指定请求谓词的类型如GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE等。

Demo

```java
    @RequestMapping(value = "/add/{a}/{b}", method = RequestMethod.POST)
    public String test2(@PathVariable int a, @PathVariable int b, Model model){
        int res = a + b;
        model.addAttribute("msg", "结果是:\t" + res);
        return "test";
    }
```

![image-20210707164642849](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210707164642849.png)

将请求方法改为 GET 即可

```java
    @RequestMapping(value = "/add/{a}/{b}", method = RequestMethod.GET)
    public String test2(@PathVariable int a, @PathVariable int b, Model model){
        int res = a + b;
        model.addAttribute("msg", "结果是:\t" + res);
        return "test";
    }
```

#### 3.2.3、小结：

Spring MVC 的 @RequestMapping 注解能够处理 HTTP 请求的方法, 比如 GET, PUT, POST, DELETE 以及 PATCH。

**所有的地址栏请求默认都会是 HTTP GET 类型的。**

方法级别的注解变体有如下几个：组合注解

```java
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping
```

`@GetMapping` 是一个组合注解，平时使用的会比较多！

它所扮演的是 `@RequestMapping(method =RequestMethod.GET)` 的一个快捷方式。

## 7、重定向和转发

### 方法一：ServletAPI

通过设置 `ServletAPI` , 不需要视图解析器 .

1、通过 `HttpServletResponse` 进行输出

2、通过 `HttpServletResponse` 实现重定向

3、通过 `HttpServletResponse` 实现转发

```java
@Controller
public class ResultGo {

   @RequestMapping("/result/t1")
   public void test1(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
       rsp.getWriter().println("Hello,Spring BY servlet API");
  }

   @RequestMapping("/result/t2")
   public void test2(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
       rsp.sendRedirect("/index.jsp");
  }

   @RequestMapping("/result/t3")
   public void test3(HttpServletRequest req, HttpServletResponse rsp) throws Exception {
       //转发
       req.setAttribute("msg","/result/t3");
       req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req,rsp);
  }
}
```

### 方法二：SpringMVC

**通过SpringMVC来实现转发和重定向 - 无需视图解析器；**

```java
    @RequestMapping("/m1/t2")
    public String test2(HttpServletRequest request, HttpServletResponse response, Model model){
        HttpSession session = request.getSession();
        System.out.println("Session Id:\t" + session.getId());
        model.addAttribute("msg", "return '/test.jsp'");
        // 转发
        return "/WEB-INF/jsp/test.jsp";
    }

    @RequestMapping("/m1/t3")
    public String test3(HttpServletRequest request, HttpServletResponse response, Model model){
        HttpSession session = request.getSession();
        System.out.println("Session Id:\t" + session.getId());
        model.addAttribute("msg", "return 'forward:/WEB-INF/jsp/test.jsp/test.jsp'");
        // 转发
        return "forward:/WEB-INF/jsp/test.jsp";
    }

    @RequestMapping("/m1/t4")
    public String test4(HttpServletRequest request, HttpServletResponse response, Model model){
        HttpSession session = request.getSession();
        System.out.println("Session Id:\t" + session.getId());
        model.addAttribute("msg", "return 'redirect:/WEB-INF/jsp/test.jsp/test.jsp'");
        // 重定向
        // redirect重定向不能访问WEB-INF目录下的东西
        return "redirect:/index.jsp";
    }
```

**通过SpringMVC来实现转发和重定向 - 有视图解析器；**

重定向 , 不需要视图解析器 , 本质就是重新请求一个新地方嘛 , 所以注意路径问题.

**在有视图解析器的情况下**, 下面两段代码的效果是相同的

- ```java
      @RequestMapping("/m1/t4")
      public String test4(HttpServletRequest request, HttpServletResponse response, Model model){
          HttpSession session = request.getSession();
          System.out.println("Session Id:\t" + session.getId());
          model.addAttribute("msg", "return 'redirect:/WEB-INF/jsp/test.jsp/test.jsp'");
          // 重定向
          return "redirect:/index.jsp";
      }
  ```

- ```java
      @RequestMapping("/m1/t4")
      public String test4(HttpServletRequest request, HttpServletResponse response, Model model){
          HttpSession session = request.getSession();
          System.out.println("Session Id:\t" + session.getId());
          model.addAttribute("msg", "return 'redirect:/WEB-INF/jsp/test.jsp/test.jsp'");
          // 重定向
          return "index";
      }
  ```



## 8、数据处理

### 8.1、处理提交数据

#### 8.1.1、单个参数

**提交的域名称和处理方法的参数名一致**

![image-20210708204231078](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210708204231078.png)

**提交的域名称和处理方法的参数名不一致，需要注解 `@RequestParam` **

![image-20210708202956936](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210708202956936.png)

#### 8.1.2、对象

如果要传递的是一个对象，比如说 `User` 对象：

```java
public class User {
    private int id;
    private int age;
    private String name;
}
```

![image-20210708204503244](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210708204503244.png)

![image-20210708204534560](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210708204534560.png)

### 8.2、数据显示到前端

#### 8.2.1、第一种 : 通过ModelAndView

**这是非注解形式下的**

```java
public class helloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView modelAndView = new ModelAndView();

        // 业务代码
        String result = "Hello SpringMVC";
        modelAndView.addObject("msg", result);

        // 视图跳转
        modelAndView.setViewName("test");

        return modelAndView;
    }
}
```

#### 8.2.2、第二种 : 通过Model

**这是注解形式下的**

```java
@Controller
public class controllerTest2 {
    @RequestMapping("/test2")
    public String printHello(Model model){

        model.addAttribute("msg", "controllerTest2");
        return "test";
    }
}
```

#### 8.2.3、第三种：通过ModelMap

```java
    @GetMapping("/t3")
    public String test3(ModelMap modelMap){
        modelMap.addAttribute("msg", "modelMap");
        return "test";
    }
```

### 8.3、处理乱码

**测试步骤**

- 新建一个 form 表单

  - ```jsp
    <form action="/e/t" method="post">
     <input type="text" name="name">
     <input type="submit">
    </form>
    ```

- 后台编写对应的处理类

  - ```java
    @Controller
    public class Encoding {
       @RequestMapping("/e/t")
       public String test(Model model,String name){
           model.addAttribute("msg",name); //获取表单提交的值
           return "test"; //跳转到test页面显示输入的值
      }
    }
    ```

- 测试结果：

- ![image-20210709124315938](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210709124315938.png)

**解决方法：**

- **方式一：自定义过滤器**

  - ```java
    public class encodingFilter implements Filter {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
    
        }
    
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            request.setCharacterEncoding("utf-8");
            response.setCharacterEncoding("utf-8");
    
            chain.doFilter(request, response);
        }
    
        @Override
        public void destroy() {
    
        }
    }
    ```

  - ```xml
        <!--配置过滤器-->
        <filter>
            <filter-name>encodingFilter</filter-name>
            <filter-class>filter.encodingFilter</filter-class>
        </filter>
        <filter-mapping>
            <filter-name>encodingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>
    ```

- **方式二：SpringMVC自定义过滤器**

  - ```xml
        <!--SpringMVC自带过滤器-->
        <filter>
            <filter-name>encoding</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
            <init-param>
                <param-name>encoding</param-name>
                <param-value>utf-8</param-value>
            </init-param>
        </filter>
        <filter-mapping>
            <filter-name>encoding</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>
    ```

- 方式三：

  - ```java
    /**
    * 解决get和post请求 全部乱码的过滤器
    */
    public class GenericEncodingFilter implements Filter {
    
       @Override
       public void destroy() {
      }
    
       @Override
       public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
           //处理response的字符编码
           HttpServletResponse myResponse=(HttpServletResponse) response;
           myResponse.setContentType("text/html;charset=UTF-8");
    
           // 转型为与协议相关对象
           HttpServletRequest httpServletRequest = (HttpServletRequest) request;
           // 对request包装增强
           HttpServletRequest myrequest = new MyRequest(httpServletRequest);
           chain.doFilter(myrequest, response);
      }
    
       @Override
       public void init(FilterConfig filterConfig) throws ServletException {
      }
    
    }
    
    //自定义request对象，HttpServletRequest的包装类
    class MyRequest extends HttpServletRequestWrapper {
    
       private HttpServletRequest request;
       //是否编码的标记
       private boolean hasEncode;
       //定义一个可以传入HttpServletRequest对象的构造函数，以便对其进行装饰
       public MyRequest(HttpServletRequest request) {
           super(request);// super必须写
           this.request = request;
      }
    
       // 对需要增强方法 进行覆盖
       @Override
       public Map getParameterMap() {
           // 先获得请求方式
           String method = request.getMethod();
           if (method.equalsIgnoreCase("post")) {
               // post请求
               try {
                   // 处理post乱码
                   request.setCharacterEncoding("utf-8");
                   return request.getParameterMap();
              } catch (UnsupportedEncodingException e) {
                   e.printStackTrace();
              }
          } else if (method.equalsIgnoreCase("get")) {
               // get请求
               Map<String, String[]> parameterMap = request.getParameterMap();
               if (!hasEncode) { // 确保get手动编码逻辑只运行一次
                   for (String parameterName : parameterMap.keySet()) {
                       String[] values = parameterMap.get(parameterName);
                       if (values != null) {
                           for (int i = 0; i < values.length; i++) {
                               try {
                                   // 处理get乱码
                                   values[i] = new String(values[i]
                                          .getBytes("ISO-8859-1"), "utf-8");
                              } catch (UnsupportedEncodingException e) {
                                   e.printStackTrace();
                              }
                          }
                      }
                  }
                   hasEncode = true;
              }
               return parameterMap;
          }
           return super.getParameterMap();
      }
    
       //取一个值
       @Override
       public String getParameter(String name) {
           Map<String, String[]> parameterMap = getParameterMap();
           String[] values = parameterMap.get(name);
           if (values == null) {
               return null;
          }
           return values[0]; // 取回参数的第一个值
      }
    
       //取所有值
       @Override
       public String[] getParameterValues(String name) {
           Map<String, String[]> parameterMap = getParameterMap();
           String[] values = parameterMap.get(name);
           return values;
      }
    }
    ```



## 9、Json交互处理

### 9.1、什么是JSON？

- JSON(JavaScript Object Notation, JS 对象标记) 是一种轻量级的数据交换格式，目前使用特别广泛。
- 采用完全独立于编程语言的**文本格式**来存储和表示数据。



**JSON 键值对**是用来保存 JavaScript 对象的一种方式，和 JavaScript 对象的写法也大同小异，键/值对组合中的键名写在前面并用双引号 "" 包裹，使用冒号 : 分隔，然后紧接着值：

```java
{"name": "QinJiang"}
{"age": "3"}
{"sex": "男"}
```



**代码测试**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <script>
        // 编写一个JavaScript对象
        var user = {
            name: "云小杰",
            age: 23,
            sex: "男"
        };
        console.log("JS对象:");
        console.log(user);
        console.log("=======================");
        // 将JS对象转换为JSON对象
        var json = JSON.stringify(user);
        console.log("JSON对象:");
        console.log(json);
        console.log("=======================");
        // 将JSON对象转换为JavaScript对象
        var obj = JSON.parse(json);
        console.log("JS对象:");
        console.log(obj);
    </script>
</body>
</html>
```

![image-20210709133724144](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210709133724144.png)

### 9.2、Controller返回JSON数据

Jackson应该是目前比较好的json解析工具了

当然工具不止这一个，比如还有阿里巴巴的 fastjson 等等。

我们这里使用 `Jackson` ，使用它需要导入它的jar包；

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.4</version>
</dependency>
```

**测试步骤：**

#### 9.2.1、返回一个对象

- 新建一个 `User` 实体类

  - ```java
    public class User {
    
        private String name;
        private int age;
        private String sex;
    }
    ```

- 编写 `userController` 类

  - ```java
    @Controller
    public class userController {
    
        @ResponseBody // 使用该注解, 不会走视图解析器
        @RequestMapping("/j1")
        public String json1() throws JsonProcessingException {
            //创建一个jackson的对象映射器，用来解析数据
            ObjectMapper mapper = new ObjectMapper();
            // 创建一个对象
            User user = new User("云小杰", 23, "男");
            // 将我们的对象解析成JSON格式
            String str = mapper.writeValueAsString(user);
            return str;
        }
    }
    ```

- 出现乱码问题

  - ![image-20210709141338731](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210709141338731.png)

- 解决方法：

  - 通过 `@RequestMaping` 的 `produces` 属性来实现
  - `@RequestMapping(value = "/j1", produces = "application/json;charset=utf-8")`
  - ![image-20210709141607765](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210709141607765.png)

- **代码优化**

  - 在 `springmvc-servlet.xml` 文件中配置以下代码：

  - ```xml
        <!--JSON乱码问题配置-->
        <mvc:annotation-driven>
            <mvc:message-converters register-defaults="true">
                <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                    <constructor-arg value="UTF-8"/>
                </bean>
                <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                    <property name="objectMapper">
                        <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                            <property name="failOnEmptyBeans" value="false"/>
                        </bean>
                    </property>
                </bean>
            </mvc:message-converters>
        </mvc:annotation-driven>
    ```

  - 在类上直接使用 **@RestController** ，这样子，里面所有的方法都只会返回 json 字符串了，不用再每一个都添加`@ResponseBody` ！我们在前后端分离开发中，一般都使用 `@RestController` ，十分便捷！

  - ```java
    @RestController
    public class userController {
    
        @RequestMapping("/j1")
        public String json1() throws JsonProcessingException {
            //创建一个jackson的对象映射器，用来解析数据
            ObjectMapper mapper = new ObjectMapper();
            // 创建一个对象
            User user = new User("云小杰", 23, "男");
            // 将我们的对象解析成JSON格式
            String str = mapper.writeValueAsString(user);
            return str;
        }
    }
    ```

#### 9.2.2、返回一个集合

- ```java
      @RequestMapping("/j2")
      public String json2() throws JsonProcessingException {
          ObjectMapper mapper = new ObjectMapper();
          List<User> list = new ArrayList<>();
          User user1 = new User("云小杰1号", 23, "男");
          User user2 = new User("云小杰1号", 23, "男");
          User user3 = new User("云小杰1号", 23, "男");
          User user4 = new User("云小杰1号", 23, "男");
  
          list.add(user1);
          list.add(user2);
          list.add(user3);
          list.add(user4);
  
          String str = mapper.writeValueAsString(list);
          return str;
      }
  ```

- ![image-20210709144200381](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210709144200381.png)

#### 9.2.3、返回时间

```java
    @RequestMapping("/j3")
    public String json3() throws JsonProcessingException {
        //创建一个jackson的对象映射器，用来解析数据
        ObjectMapper mapper = new ObjectMapper();

        Date date = new Date();
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        return mapper.writeValueAsString(dateFormat.format(date));
    }


```

![image-20210709144740819](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210709144740819.png)



### 9.3、Fastjson

fastjson.jar是阿里开发的一款专门用于Java开发的包，可以方便的实现json对象与JavaBean对象的转换，实现JavaBean对象与json字符串的转换，实现json对象与json字符串的转换。实现json的转换方法很多，最后的实现结果都是一样的。

fastjson 的 pom依赖！

```xml
        <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.76</version>
        </dependency>
```

**举例：**

```java
    @RequestMapping("/j4")
    public String json4(){
        List<User> list = new ArrayList<>();
        User user1 = new User("云小杰1号", 23, "男");
        User user2 = new User("云小杰1号", 23, "男");
        User user3 = new User("云小杰1号", 23, "男");
        User user4 = new User("云小杰1号", 23, "男");

        list.add(user1);
        list.add(user2);
        list.add(user3);
        list.add(user4);

        String str = JSON.toJSONString(list);
        return str;
    }
```

![image-20210709150951174](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210709150951174.png)

```java
    @RequestMapping("/j4")
    public String json4(){
        List<User> list = new ArrayList<>();
        User user1 = new User("云小杰1号", 23, "男");
        User user2 = new User("云小杰1号", 23, "男");
        User user3 = new User("云小杰1号", 23, "男");
        User user4 = new User("云小杰1号", 23, "男");

        list.add(user1);
        list.add(user2);
        list.add(user3);
        list.add(user4);

        System.out.println("*******Java对象 转 JSON字符串*******");
        String str1 = JSON.toJSONString(list);
        System.out.println("JSON.toJSONString(list)==>"+str1);
        String str2 = JSON.toJSONString(user1);
        System.out.println("JSON.toJSONString(user1)==>"+str2);

        System.out.println("\n****** JSON字符串 转 Java对象*******");
        User jp_user1=JSON.parseObject(str2,User.class);
        System.out.println("JSON.parseObject(str2,User.class)==>"+jp_user1);

        System.out.println("\n****** Java对象 转 JSON对象 ******");
        JSONObject jsonObject1 = (JSONObject) JSON.toJSON(user2);
        System.out.println("(JSONObject) JSON.toJSON(user2)==>"+jsonObject1.getString("name"));

        System.out.println("\n****** JSON对象 转 Java对象 ******");
        User to_java_user = JSON.toJavaObject(jsonObject1, User.class);
        System.out.println("JSON.toJavaObject(jsonObject1, User.class)==>"+to_java_user);

        String str = JSON.toJSONString(list);
        return str;
    }
```

## 10、Ajax研究

### 10.1、简介

- **AJAX = Asynchronous JavaScript and XML（异步的 JavaScript 和 XML）**
- **Ajax 不是一种新的编程语言，而是一种用于创建更好更快以及交互性更强的Web应用程序的技术。**
- 传统的网页(即不用ajax技术的网页)，想要更新内容或者提交一个表单，都需要重新加载整个网页。
- 使用ajax技术的网页，通过在后台服务器进行少量的数据交换，就可以实现异步局部更新。
- 使用Ajax，用户可以创建接近本地桌面应用的直接、高可用、更丰富、更动态的Web用户界面。



### 10.2、伪造Ajax

我们可以使用前端的一个标签来伪造一个 Ajax 的样子。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Iframe测试体验页面无刷新</title>

  <script>
    function go() {
      var url = document.getElementById("url").value;
      document.getElementById("frame1").src = url;
    }
  </script>

</head>
<body>

<div>
  <p>请输入地址: </p>
  <p>
    <input type="text" id="url" value="https://www.baidu.com/">
    <input type="button" value="提交" onclick="go()">
  </p>
</div>

  <div>
    <iframe id="frame1" style="width: 100%;height: 500px;"></iframe>
  </div>
</body>
</html>
```

![image-20210710190512220](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210710190512220.png)

### 10.3、jQuery.ajax

jQuery 是一个高效、精简并且功能丰富的 JavaScript 工具库。它提供的 API 易于使用且兼容众多浏览器，这让诸如 HTML 文档遍历和操作、事件处理、动画和 Ajax 操作更加简单。

Ajax的核心是XMLHttpRequest对象(XHR)。XHR为向服务器发送请求和解析服务器响应提供了接口。能够以异步方式从服务器获取新数据。

```jsp
<%--
  Created by IntelliJ IDEA.
  User: LYJ
  Date: 2021/7/10
  Time: 16:35
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
<%--    <script src="${pageContext.request.contextPath}/statics/js/jquery-3.6.0.js"></script>--%>
    <link rel="stylesheet" href="statics/js/jquery-3.6.0.js">

    <script>
      function test(){
        $.post({
          url: "${pageContext.request.contextPath}/test2",
          data: {"name": $("#username").val()},
          success: function (data){
            alert(data);
          }
        })
      }
    </script>

  </head>
  <body>
    <%--失去焦点的时候 发起一个请求到后台--%>
  用户名：<input type="text" id="username" onclick="test()">
  </body>
</html>
```



```java
    @RequestMapping("/t2")
    public void test2(String name, HttpServletResponse response) throws IOException {
        System.out.println("test--name==>" + name);
        if ("yunxiaojie".equals(name)){
            response.getWriter().print("true");
        } else {
            response.getWriter().print("false");
        }
    }
```

## 11、Ajax实例

### 11.1、Ajax加载数据

- 新建一个方法：

  - ```java
    @RequestMapping("/test3")
    public List<User> test3(){
        List<User> list = new ArrayList<>();
    
        list.add(new User("云杰1号", 18, "男"));
        list.add(new User("云杰2号", 23, "男"));
        list.add(new User("云杰3号", 19, "男"));
    
        return list;
    }
    ```

- 新建前端页面 `test2.jsp`，测试Ajax

  - ```jsp
    <%--
      Created by IntelliJ IDEA.
      User: LYJ
      Date: 2021/7/10
      Time: 19:52
      To change this template use File | Settings | File Templates.
    --%>
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>Title</title>
    
        <script src="${pageContext.request.contextPath}/statics/js/jquery-3.6.0.js"></script>
        <script>
            $(function (){
                $("#btn").click(function (){
                    $.post("${pageContext.request.contextPath}/test3", function (data){
                        console.log(data);
                        var html = "";
                        for (let i = 0; i < data.length; i++) {
                            html += "<tr>" +
                            "<td>" + data[i].name + "</td>" +
                            "<td>" + data[i].age + "</td>" +
                            "<td>" + data[i].sex + "</td>" +
                                "</tr>";
                        }
                        $("#content").html(html);
                    })
                })
            });
        </script>
    
    </head>
    <body>
        <input type="button" value="加载数据" id="btn">
        <table>
            <tr>
                <td>姓名</td>
                <td>年龄</td>
                <td>性别</td>
            </tr>
            <tbody id="content">
                <%--数据：后台--%>
            </tbody>
        </table>
    </body>
    </html>
    ```

- 测试结果：

  - ![image-20210710201343019](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210710201343019.png)

### 11.2、Ajax注册提示效果

- 新建一个方法来判断输入的用户名和密码

  - ```java
        @RequestMapping("/test4")
        public String test4(String name, String pwd){
            String msg = "";
            // 以下只是例子，真实数据应该从数据库中获取
            if (name != null){
                if ("admin".equals(name)){
                    msg = "OK";
                } else {
                    msg = "用户名有误";
                }
            }
            if (pwd != null) {
                if ("123456".equals(pwd)) {
                    msg = "OK";
                } else {
                    msg = "密码有误";
                }
            }
            return msg;
        }
    ```

- 编写前端的登陆页面 `login.jsp`

  - ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>Title</title>
        <script src="${pageContext.request.contextPath}/statics/js/jquery-3.6.0.js"></script>
    
        <script>
            function a1(){
                $.post({
                    url: "${pageContext.request.contextPath}/test4",
                    data: {"name": $("#name").val()},
                    success: function (data){
                        if (data.toString() === 'OK'){
                            $("#userInfo").css("color", "green");
                        } else {
                            $("#userInfo").css("color", "red");
                        }
                        $("#userInfo").html(data);
                    }
                })
            }
            function a2(){
                $.post({
                    url: "${pageContext.request.contextPath}/test4",
                    data: {"name": $("#pwd").val()},
                    success: function (data){
                        if (data.toString() === 'OK'){
                            $("#pwdInfo").css("color", "green");
                        } else {
                            $("#pwdInfo").css("color", "red");
                        }
                        $("#userInfo").html(data);
                    }
                })
            }
        </script>
    
    </head>
    <body>
        <p>
            用户名：<input type="text" id="name" onblur="a1()">
            <span id="userInfo"></span>
        </p>
        <p>
            密码：<input type="password" id="pwd" onblur="a2()">
            <span id="pwdInfo"></span>
        </p>
    </body>
    </html>
    ```
  
- ![image-20210710212529956](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210710212529956.png)

## 12、拦截器

### 12.1、概述

SpringMVC的处理器拦截器类似于Servlet开发中的过滤器Filter,用于对处理器进行预处理和后处理。开发者可以自己定义一些拦截器来实现特定的功能。

**过滤器与拦截器的区别：**拦截器是AOP思想的具体应用。

**过滤器**

- servlet规范中的一部分，任何java web工程都可以使用
- 在url-pattern中配置了/*之后，可以对所有要访问的资源进行拦截

**拦截器** 

- 拦截器是SpringMVC框架自己的，只有使用了SpringMVC框架的工程才能使用
- 拦截器只会拦截访问的**控制器**方法， 如果访问的是jsp/html/css/image/js是不会进行拦截的(**自带静态资源过滤**)



### 12.2、自定义拦截器

想要自定义拦截器，必须实现 `HandlerInterceptor` 接口。

- 自定义拦截器：

  - ```java
    public class myInterceptor implements HandlerInterceptor {
        
        // true: 放行; false: 不放行
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            System.out.println("=====================preHandle===============================");
            return true;
        }
    
        // 拦截日志
        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            System.out.println("=====================postHandle===============================");
        }
    
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            System.out.println("=====================清理===============================");
        }
    }
    
    ```

- 在核心配置文件 `applicationContext.xml` 中注册：

  - ```xml
        <!--拦截器配置-->
        <mvc:interceptors>
            <mvc:interceptor>
                <!--/**代表这个请求下面的所有请求, 比如/admin/**代表/admin下面的所有请求, /admin/xxxx-->
                <mvc:mapping path="/**"/>
                <bean class="config.myInterceptor"/>
            </mvc:interceptor>
        </mvc:interceptors>
    ```

- 编写测试的控制类：

  - ```java
    @RestController
    public class TestController {
    
        @RequestMapping("/test1")
        public String test1(){
            System.out.println("TestController");
            return "Test";
        }
    }
    ```

- ![image-20210710215020831](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210710215020831.png)

### 12.3、拦截器实例：登陆判断验证

要实现的功能如下：

- 首先是进入 `index.jsp` 页面，有两个选项：1、主页；2、登陆页面
- 只有登录才能进行主页；
- 若未登录直接点击进入主页会被拦截，跳转到登陆页面



- `index.jsp`

  - ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
      <head>
        <title>$Title$</title>
      </head>
      <body>
      <h1><a href="${pageContext.request.contextPath}/user/toLogin">去登陆页面</a></h1>
      <h1><a href="${pageContext.request.contextPath}/user/toMain">去首页</a></h1>
      </body>
    </html>
    ```

- 对应的请求处理方法

  - ```java
    @Controller
    @RequestMapping("/user")
    public class loginController {
    
        @RequestMapping("/toMain")
        public String toMain(){
            return "main";
        }
    
        @RequestMapping("/toLogin")
        public String toLogin(){
            return "login";
        }
    
        @RequestMapping("/login")
        public String Login(String username, String password, HttpSession session){
            /**
             * 登陆成功, 跳转到成功页面, 并且把用户的信息存在session中
             */
            session.setAttribute("userLoginInfo", username);
            return "main";
        }
    
        @RequestMapping("/logout")
        public String loginOut(String username, String password, HttpSession session){
            /**
             * 注销
             */
            session.removeAttribute("userInfo");
            return "login";
        }
    }
    ```

- 编写页面：

  - `login.jsp`

    - ```jsp
      <%@ page contentType="text/html;charset=UTF-8" language="java" %>
      <html>
      <head>
          <title>登陆页面</title>
      </head>
      <body>
      <%--WEB-INF下的所有页面或者资源, 只有通过controller, 或者Servlet进行访问--%>
          <h1>登陆页面</h1>
      <form action="${pageContext.request.contextPath}/user/login" method="post">
          用户名: <input type="text" name="username">
          密码: <input type="password" name="password">
          <input type="submit" value="登录">
      </form>
      </body>
      </html>
      ```

  - `main.jsp`

    - ```jsp
      <%@ page contentType="text/html;charset=UTF-8" language="java" %>
      <html>
      <head>
          <title>首页</title>
          <!-- 新 Bootstrap 核心 CSS 文件 -->
          <link href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
      
          <!-- jQuery文件。务必在bootstrap.min.js 之前引入 -->
          <script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
      
          <!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
          <script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
      </head>
      <body>
          <div class="alert alert-success" role="alert">${userLoginInfo}好，这是主页</div>
          <a href="${pageContext.request.contextPath}/user/logout" class="btn btn-danger">注销</a>
      </body>
      </html>
      ```

- 编写拦截器：

  - ```java
    public class loginInterceptor implements HandlerInterceptor {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    
            // 登录可以放行, 此时session中有对象
            HttpSession session = request.getSession();
            if (session.getAttribute("userInfo")!=null){
                return true;
            }
            // 或者是直接进入登陆页面, 可以放行
            if (request.getRequestURI().contains("login")){
                return true;
            }
    
            // 用户没有登陆跳转到登陆页面
            request.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(request, response);
            return false;
        }
    }
    ```

- 配置拦截器：

  - ```xml
        <!--拦截器配置-->
        <mvc:interceptors>
            <mvc:interceptor>
                <!--/**代表这个请求下面的所有请求, 比如/admin/**代表/admin下面的所有请求, /admin/xxxx-->
                <mvc:mapping path="/**"/>
                <bean class="config.myInterceptor"/>
            </mvc:interceptor>
    
            <mvc:interceptor>
                <mvc:mapping path="/user/**"/>
                <bean class="config.loginInterceptor"/>
            </mvc:interceptor>
        </mvc:interceptors>
    ```

- 效果：

  - ![image-20210711110405480](C:/Users/LYJ/AppData/Roaming/Typora/typora-user-images/image-20210711110405480.png)![image-20210711110414486](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210711110414486.png)
  - ![image-20210711110508500](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210711110508500.png)





## 13、文件上传和下载

文件上传是项目开发中最常见的功能之一 ,springMVC 可以很好的支持文件上传，但是SpringMVC上下文中默认没有装配`MultipartResolver`，因此默认情况下其不能处理文件上传工作。如果想使用Spring的文件上传功能，则需要在上下文中配置`MultipartResolve`。

**前端表单要求：**为了能上传文件，必须将表单的 method设置为POST，并将 `enctype` 设置为 `multipart/form-data` 。只有在这样的情况下，浏览器才会把用户选择的文件以二进制数据发送给服务器；



**对表单中的 enctype 属性做个详细的说明：**

- `application/x-www=form-urlencoded`：默认方式，只处理表单域中的 value 属性值，采用这种编码方式的表单会将表单域中的值处理成 URL 编码方式。
- `multipart/form-data`：这种编码方式会以二进制流的方式来处理表单数据，这种编码方式会把文件域指定文件的内容也封装到请求参数中，不会对字符编码。
- `text/plain`：除了把空格转换为 "+" 号外，其他字符都不做编码处理，这种方式适用直接通过表单发送邮件



### 13.1、文件上传

1、导入文件上传的jar包，`commons-fileupload` , Maven会自动帮我们导入他的依赖包 commons-io 包；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>LearnSpringMVC</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>SpringMVC-09-File</artifactId>

    <properties>
        <maven.compiler.source>16</maven.compiler.source>
        <maven.compiler.target>16</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.4</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

</project>
```

2、配置bean：`multipartResolver`

==**这个bena的id必须为：multipartResolver ， 否则上传文件会报400的错误！**==

```xml
<!--文件上传配置-->
<bean id="multipartResolver"  class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
   <!-- 请求的编码格式，必须和jSP的pageEncoding属性一致，以便正确读取表单的内容，默认为ISO-8859-1 -->
   <property name="defaultEncoding" value="utf-8"/>
   <!-- 上传文件大小上限，单位为字节（10485760=10M） -->
   <property name="maxUploadSize" value="10485760"/>
   <property name="maxInMemorySize" value="40960"/>
</bean>
```

`CommonsMultipartFile` 的 常用方法：

- **String getOriginalFilename()：获取上传文件的原名**
- **InputStream getInputStream()：获取文件流**
- **void transferTo(File dest)：将上传文件保存到一个目录文件中**

3、编写前端页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
    <form action="/upload" enctype="multipart/form-data" method="post">
      <input type="file" name="file"/>
      <input type="submit" value="upload">
    </form>
  </body>
</html>
```

4、编写上传的方法

方法一：

```java
//@RequestParam("file") 将name=file控件得到的文件封装成CommonsMultipartFile 对象
    //批量上传CommonsMultipartFile则为数组即可
    @RequestMapping("/upload")
    public String fileUpload(@RequestParam("file") CommonsMultipartFile file , HttpServletRequest request) throws IOException {

        //获取文件名 : file.getOriginalFilename();
        String uploadFileName = file.getOriginalFilename();

        //如果文件名为空，直接回到首页！
        if ("".equals(uploadFileName)){
            return "redirect:/index.jsp";
        }
        System.out.println("上传文件名 : "+uploadFileName);

        //上传路径保存设置
        String path = request.getServletContext().getRealPath("/upload");
        //如果路径不存在，创建一个
        File realPath = new File(path);
        if (!realPath.exists()){
            realPath.mkdir();
        }
        System.out.println("上传文件保存地址："+realPath);

        InputStream is = file.getInputStream(); //文件输入流
        OutputStream os = new FileOutputStream(new File(realPath,uploadFileName)); //文件输出流

        //读取写出
        int len=0;
        byte[] buffer = new byte[1024];
        while ((len=is.read(buffer))!=-1){
            os.write(buffer,0,len);
            os.flush();
        }
        os.close();
        is.close();
        return "redirect:/index.jsp";
    }
```

方法二：

```java
@RequestMapping("/upload2")
    public String  fileUpload2(@RequestParam("file") CommonsMultipartFile file, HttpServletRequest request) throws IOException {

        //上传路径保存设置
        String path = request.getServletContext().getRealPath("/upload");
        File realPath = new File(path);
        if (!realPath.exists()){
            realPath.mkdir();
        }
        //上传文件地址
        System.out.println("上传文件保存地址："+realPath);

        //通过CommonsMultipartFile的方法直接写文件（注意这个时候）
        file.transferTo(new File(realPath +"/"+ file.getOriginalFilename()));

        return "redirect:/index.jsp";
    }
```

4、下载

```java
@RequestMapping(value="/download")
    public String downloads(HttpServletResponse response , HttpServletRequest request) throws Exception{
        //要下载的图片地址
        String  path = request.getServletContext().getRealPath("/upload");
        String  fileName = "基础语法.jpg";

        //1、设置response 响应头
        response.reset(); //设置页面不缓存,清空buffer
        response.setCharacterEncoding("UTF-8"); //字符编码
        response.setContentType("multipart/form-data"); //二进制传输数据
        //设置响应头
        response.setHeader("Content-Disposition",
                "attachment;fileName="+ URLEncoder.encode(fileName, "UTF-8"));

        File file = new File(path,fileName);
        //2、 读取文件--输入流
        InputStream input=new FileInputStream(file);
        //3、 写出文件--输出流
        OutputStream out = response.getOutputStream();

        byte[] buff =new byte[1024];
        int index=0;
        //4、执行 写出操作
        while((index= input.read(buff))!= -1){
            out.write(buff, 0, index);
            out.flush();
        }
        out.close();
        input.close();
        return null;
    }
```


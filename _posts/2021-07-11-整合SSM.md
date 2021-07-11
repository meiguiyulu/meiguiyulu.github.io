# 整合SSM

新建数据库

```mysql
 CREATE DATABASE `ssmbuild`;
 
 USE `ssmbuild`;
 
 CREATE TABLE `books`(
	`bookID` INT(10) NOT NULL auto_increment COMMENT '书id',
	`bookName` VARCHAR(10) NOT NULL COMMENT '书名',
	`bookCounts` INT(11) NOT NULL COMMENT '数量',
	`detail` VARCHAR(200) NOT NULL COMMENT '描述',
	KEY `bookID`(`bookID`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `books`(`bookID`, `bookName`, `bookCounts`, `detail`)
VALUES
(1, 'Java', 1, '从入门到放弃'),
(2, 'MySQL', 10, '从删库到跑路'),
(3, 'Linux', 5, '从进门到进牢')
```



添加需要的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>ssmbuild</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>16</maven.compiler.source>
        <maven.compiler.target>16</maven.compiler.target>
    </properties>

    <!--依赖-->
    <dependencies>
        <!--Junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <!--数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.25</version>
        </dependency>
        <!-- 数据库连接池 c3p0-->
        <!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.5</version>
        </dependency>

        <!--Servlet - JSP -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <!--Mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.6</version>
        </dependency>

        <!--Spring-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.8</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.8</version>
        </dependency>
    </dependencies>

    <!--静态资源导出问题-->
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>

</project>
```



## 1、MyBatis层

### 1.1、新建数据库配置文件 `db.properties`

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
jdbc.username=root
jdbc.password=7012+2
```

### 1.2、新建MyBatis配置文件 `MyBatis-Config.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration核心配置文件-->
<configuration>
    <!--配置数据源，交给Spring了-->
    
    <typeAliases>
        <package name="pojo"/>
    </typeAliases>
    <mappers>
        <mapper resource="dao/BookMapper.xml"/>
    </mappers>
    
</configuration>
```

### 1.3、编写实体类 `Books`

**注意：尽量与数据库中的属性名字相等**

```java
public class Books {

    private int bookID;
    private String bookName;
    private int bookCounts;
    private String detail;
}
```

### 1.4、在 dao 文件夹下新建接口 `BookMapper`

```java
public interface BookMapper {

    // 增加一本书
    int addBook(Books books);
    // 删除一本书
    int deleteBookById(@Param("bookId") int id);

    // 更新一本书
    int updateBook(Books books);

    // 查询一本书
    Books queryBookById(@Param("bookId")int id);

    // 查询全部书
    List<Books> queryAllBooks();

}
```

### 1.5、新建接口 `BookMapper` 的配置文件 `BookMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">


<mapper namespace="dao.BookMapper">

    <insert id="addBook" parameterType="Books">
        insert into ssmbuild.books(bookName, bookCounts, detail)
        values (#{bookName}, #{bookCounts}, #{detail})
    </insert>

    <delete id="deleteBookById" parameterType="int">
        delete
        from ssmbuild.books
        where bookID = #{bookId};
    </delete>

    <update id="updateBook" parameterType="Books">
        update ssmbuild.books
        set bookName=#{bookName}, bookCounts=#{bookName}, detail=#{bookName}
        where bookID = #{bookID};
    </update>

    <select id="queryBookById" resultType="Books">
        select *
        from ssmbuild.books
        where bookID = #{bookId};
    </select>

    <select id="queryAllBooks" resultType="Books">
        select * from ssmbuild.books;
    </select>

</mapper>
```

### 1.6、在MyBatis配置文件中配置 `BookMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--configuration核心配置文件-->
<configuration>
    <!--配置数据源，交给Spring了-->
    
    <typeAliases>
        <package name="pojo"/>
    </typeAliases>
    
</configuration>
```

### 1.7、在 Service 文件夹下新建接口 `BookService` 和相应的实现类

```java
public interface BookService {
    // 增加一本书
    int addBook(Books books);
    // 删除一本书
    int deleteBookById(int id);
    // 更新一本书
    int updateBook(Books books);
    // 查询一本书
    Books queryBookById(int id);
    // 查询全部书
    List<Books> queryAllBooks();
}

public class BookServiceImpl implements BookService{

    // Service层要调dao层
    private BookMapper bookMapper;

    public void setBookMapper(BookMapper bookMapper) {
        this.bookMapper = bookMapper;
    }

    @Override
    public int addBook(Books books) {
        return bookMapper.addBook(books);
    }

    @Override
    public int deleteBookById(int id) {
        return bookMapper.deleteBookById(id);
    }

    @Override
    public int updateBook(Books books) {
        return bookMapper.updateBook(books);
    }

    @Override
    public Books queryBookById(int id) {
        return bookMapper.queryBookById(id);
    }

    @Override
    public List<Books> queryAllBooks() {
        return bookMapper.queryAllBooks();
    }
}
```



## 2、Spring层

### 2.1、整合MyBatis

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!--1、关联数据库配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>

    <!--2、数据库连接池-->
    <!--种类：
    1、dbcp: 半自动化操作, 不能自动连接
    2、c3p0: 自动化操作(自动化地加载配置文件, 并且可以自动设置到对象中!)
    3、druid
    -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="#{jdbc.driver}"/>
        <property name="jdbcUrl" value="#{jdbc.url}"/>
        <property name="user" value="#{jdbc.username}"/>
        <property name="password" value="#{jdbc.password}"/>

        <!-- c3p0连接池的私有属性 -->
        <property name="initialPoolSize" value="10"/>
        <property name="maxPoolSize" value="30"/>
        <property name="minPoolSize" value="10"/>
        <!-- 关闭连接后不自动commit -->
        <property name="autoCommitOnClose" value="false"/>
        <!-- 获取连接超时时间 -->
        <property name="checkoutTimeout" value="10000"/>
        <!-- 当获取连接失败重试次数 -->
        <property name="acquireRetryAttempts" value="2"/>
    </bean>


    <!-- 3.配置SqlSessionFactory对象 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 配置MyBatis全局配置文件:mybatis-config.xml -->
        <property name="configLocation" value="classpath:MyBatis-Config.xml"/>
    </bean>

    <!-- 4.配置扫描Dao接口包，动态实现Dao接口注入到spring容器中 -->
    <!--解释 ：https://www.cnblogs.com/jpfss/p/7799806.html-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 注入sqlSessionFactory -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!-- 给出需要扫描Dao接口包 -->
        <property name="basePackage" value="dao"/>
    </bean>

</beans>
```



### 2.2、整合Service层

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!--1、扫描Service下的包-->
    <context:component-scan base-package="service"/>

    <!--2、将我们所有的业务类注入到Spring，可以通过配置或者注解实现-->
    <bean id="BookServiceImpl" class="service.BookServiceImpl">
        <property name="bookMapper" value="bookMapper"/>
    </bean>

    <!--3、事务配置-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据源-->
        <property name="dataSource" value="dataSource"/>
    </bean>

    <!--4、AOP事务支持-->

</beans>
```

## 3、SpringMVC层

### 3.1、配置 `web.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--DispatcherServlet-->
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!--一定要注意:我们这里加载的是总的配置文件，之前被这里坑了！-->
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--encodingFilter-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>
            org.springframework.web.filter.CharacterEncodingFilter
        </filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--Session过期时间-->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>
    
</web-app>
```

### 3.2、新建 `spring-mvc.xml` 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context.xsd
   http://www.springframework.org/schema/mvc
   https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 配置SpringMVC -->
    <!-- 1.开启SpringMVC注解驱动 -->
    <mvc:annotation-driven />
    <!-- 2.静态资源默认servlet配置-->
    <mvc:default-servlet-handler/>

    <!-- 3.配置jsp 显示ViewResolver视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>

    <!-- 4.扫描web相关的bean -->
    <context:component-scan base-package="controller" />

</beans>
```

## 4、功能一：查询全部书籍

### 4.1、编写一个控制器类 `BookController `

```java
@Controller
@RequestMapping("/book")
public class bookController {

    // Controller层调Service层
    @Autowired
    @Qualifier("BookServiceImpl")
    private BookService bookService;

    @RequestMapping("/allBook")
    public String QueryAllBooks(Model model){
        List<Books> books = bookService.queryAllBooks();
        model.addAttribute("books", books);
        return "allBooks";
    }
}
```

### 4.2、编写一个前端视图 `allBooks.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>书籍列表</title>
    <!-- 引入 Bootstrap -->
    <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container">
        <div class="row clearfix">
            <div class="col-md-12 column">
                <div class="page-header">
                    <h1>
                        <small>书籍列表----所有书籍</small>
                    </h1>
                </div>
            </div>
        </div>
        <div class="row clearfix">
            <div class="col-md-12 column">
                <table class="table table-hover table-striped">
                    <thead>
                        <tr>
                            <th>书籍编号</th>
                            <th>书籍名称</th>
                            <th>书籍数目</th>
                            <th>书籍详情</th>
                        </tr>
                    </thead>
                    <tbody>
                        <c:forEach var="book" items="#{requestScope.get('books')}">
                            <tr>
                                <td>${book.getBookID()}</td>
                                <td>${book.getBookName()}</td>
                                <td>${book.getBookCounts()}</td>
                                <td>${book.getDetail()}</td>
                            </tr>
                        </c:forEach>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</body>
</html>
```

### 4.3、测试画面

![image-20210710125206472](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210710125206472.png)

## 5、功能二：添加书籍

### 5.1、在 `allBooks.jsp`页面增加添加书籍的按钮

```jsp
        <div class="row">
            <div class="col-md-4 column">
                <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/toAddBook">新增书籍</a>
            </div>
        </div>
```

### 5.2、新建跳转到添加书籍页面的方法

```java
    // 跳转到添加书籍页面
    @RequestMapping("/toAddBook")
    public String toAddBook(){
        return "addBook";
    }
```

### 5.3、新建添加书籍的前端页面 `addBook.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>添加书籍</title>
    <!-- 引入 Bootstrap -->
    <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container">
        <div class="row clearfix">
            <div class="col-md-12 column">
                <div class="page-header">
                    <h1>
                        <small>添加书籍</small>
                    </h1>
                </div>
            </div>
        </div>

        <div class="col-md-4 column">
            <form action="${pageContext.request.contextPath}/book/addBook" method="post">
                <div class="form-group">
                    <label for="bookName">书籍名称</label>
                    <input type="text" class="form-control" name="bookName" id="bookName" placeholder="书籍名称" required>
                </div>
                <div class="form-group">
                    <label for="bookNum">书籍数目</label>
                    <input type="text" class="form-control" name="bookCounts" id="bookNum" placeholder="书籍数目" required>
                </div>
                <div class="form-group">
                    <label for="detail">描述</label>
                    <input type="text" class="form-control" name="detail" id="detail" placeholder="描述" required>
                </div>
                <input type="submit" class="btn btn-primary" value="添加">
            </form>
        </div>
    </div>
</body>
</html>
```

### 5.4、新建添加书籍的方法

```java
    // 处理添加书籍的请求
    @RequestMapping("/addBook")
    public String addBook(Books books){
        System.out.println("addBook===>" + books);
        bookService.addBook(books);
        return "redirect:/book/allBook"; // 重定向
    }
```

### 5.5、测试画面

![image-20210710133954772](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210710133954772.png)

![image-20210710134149454](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210710134149454.png)

![image-20210710134200224](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210710134200224.png)

## 6、功能三：修改、删除书籍

**修改页面，在增加修改、删除两个按钮**

````jsp
        <div class="row clearfix">
            <div class="col-md-12 column">
                <table class="table table-hover table-striped">
                    <thead>
                        <tr>
                            <th>书籍编号</th>
                            <th>书籍名称</th>
                            <th>书籍数目</th>
                            <th>书籍详情</th>
                            <th>操作</th>
                        </tr>
                    </thead>
                    <tbody>
                        <c:forEach var="book" items="#{requestScope.get('books')}">
                            <tr>
                                <td>${book.getBookID()}</td>
                                <td>${book.getBookName()}</td>
                                <td>${book.getBookCounts()}</td>
                                <td>${book.getDetail()}</td>
                                <td>
                                    <a href="${pageContext.request.contextPath}/book/toUpdateBook?id=${book.getBookID()}" class="btn btn-warning">修改</a>
                                    &nbsp;|&nbsp;
                                    <a href="${pageContext.request.contextPath}/book/deleteBook?id=${book.getBookID()}" class="btn btn-danger">删除</a>
                                </td>
                            </tr>
                        </c:forEach>
                    </tbody>
                </table>
            </div>
        </div>
````

### 6.1、修改书籍

#### 6.1.1、新建方法跳转到修改书籍页面

```java
    // 跳转到修改书籍页面
    @RequestMapping("/toUpdateBook")
    public String toUpdateBook(int id, Model model){
        Books book = bookService.queryBookById(id);
        model.addAttribute("QueriedBook", book);
        return "updateBook";
    }
```

#### 6.1.2、新建修改书籍的页面 `updateBook.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>修改书籍</title>
    <!-- 引入 Bootstrap -->
    <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container">
    <div class="row clearfix">
        <div class="col-md-12 column">
            <div class="page-header">
                <h1>
                    <small>修改书籍</small>
                </h1>
            </div>
        </div>
    </div>

    <div class="col-md-4 column">
        <form action="${pageContext.request.contextPath}/book/updateBook" method="post">
            <input type="text" hidden name="bookID" value="${QueriedBook.bookID}">
            <div class="form-group">
                <label for="bookName">书籍名称</label>
                <input type="text" class="form-control" name="bookName" id="bookName" placeholder="书籍名称" value="${QueriedBook.bookName}" required>
            </div>
            <div class="form-group">
                <label for="bookCounts">书籍数目</label>
                <input type="text" class="form-control" name="bookCounts" id="bookCounts" placeholder="书籍数目" value="${QueriedBook.bookCounts}" required>
            </div>
            <div class="form-group">
                <label for="detail">描述</label>
                <input type="text" class="form-control" name="detail" id="detail" placeholder="描述" value="${QueriedBook.detail}" required>
            </div>
            <input type="submit" class="btn btn-primary" value="修改">
        </form>
    </div>
</div>
</body>
</html>

```

#### 6.1.3、实现修改书籍的方法

```java
    // 修改书籍
    @RequestMapping("/updateBook")
    public String UpdateBook(Books books){
        bookService.updateBook(books);
        return "redirect:/book/allBook";
    }
```

### 6.2、删除书籍

#### 6.2.1、实现删除书籍的方法

```java
    // 删除书籍
    @RequestMapping("/deleteBook")
    public String DeleteBookById(int id){
        bookService.deleteBookById(id);
        return "redirect:/book/allBook";
    }
```



### 6.3、测试画面

![image-20210710144053106](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20210710144053106.png)

## 7、方法四：搜索功能

### 7.1、首先在主页面中新建搜索框

```jsp
        <div class="row">
            <div class="col-md-4 column">
                <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/toAddBook">新增书籍</a>
            </div>
            <div class="col-md-4 column">
                <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/allBook">显示全部书籍</a>
            </div>
            <div class="col-md-4 column">
                <form class="form-inline" method="post" action="${pageContext.request.contextPath}/book/queryBookByName">
                    <span style="color: red; font-weight: bold">${error}</span>
                    <div class="form-group">
                        <input type="text" class="form-control" name="queryBookName" placeholder="请输入要查询的书籍名称">
                    </div>
                    <button type="submit" class="btn btn-primary">搜索</button>
                </form>
            </div>
        </div>
```

### 7.2、新建一个方法实现按照书籍名称查询

**按照自底向上的顺序：dao层-->service层-->controller层**

```java
    // 根据书名搜索树书
    List<Books> queryBookByName(@Param("bookName") String queryBookByName);
```



```xml
    <select id="queryBookByName" resultType="Books">
        select * from ssmbuild.books
        where bookName like concat('%', #{bookName},'%');
    </select>
```



```java
    // 根据书名搜索书
    List<Books> queryBookByName(String queryBookByName);
```



```java
    @Override
    public List<Books> queryBookByName(String queryBookByName) {
        return bookMapper.queryBookByName(queryBookByName);
    }
```



```java
    // 搜索书籍
    @RequestMapping("/queryBookByName")
    public String queryBookByName(String queryBookName, Model model){
        List<Books> books = bookService.queryBookByName(queryBookName);
        System.out.println("queryBookByName==>" + books);
        if (books == null){
            model.addAttribute("error", "未查到");
        }else{
            model.addAttribute("books", books);
        }
        return "allBooks";
    }
```


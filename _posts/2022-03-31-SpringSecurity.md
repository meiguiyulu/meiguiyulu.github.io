# SpringSecurity

## 1. 权限管理

### 1.1 定义

​	基本上涉及到用户参与的系统都要进行权限管理，权限管理属于系统安全的范畴，权限管理实现 **`对用户访问系统的控制`**，按照 **`安全规则`** 或者 **`安全策略`**控制用户 **`可以访问而且只能访问自己被授权的资源。`**

​		权限管理保护用户 **身份认证** 和 **授权** 两部分，简称 **认证授权**。对于需要访问控制的资源，用户首先经过身份认证，认证通过后用户具有该资源的访问权限，方可访问。

### 1.2 认证

​		**`身份认证`**，就是判断一个用户是否为合法用户的处理过程。最常见的简单身份认证方式是系统通过核对用户输入的用户名和口令，看其是否与系统中的存储的该用户的用户名和口令一致，来判断用户身份是否正确。对于采用 **指纹** 等系统，则出示指纹；对于硬件 key 等刷卡系统，则需要刷卡。

### 1.3 授权 

​		**`授权`**，即访问控制，控制谁能访问哪些资源。主体进行身份认证后需要分配权限方可访问系统的资源，对于某些资源没有权限是无法访问的。

### 1.4 解决方案

和其他领域不同，在 Java 企业级开发中，安全管理框架非常少，目前比较常见的就是：

- Shiro
  - Shiro 本身是一个老牌的安全管理框架，有着众多的优点，例如轻量、简单、易于集成、可以在JavaSE环境中使用等。不过，在微服务时代，Shiro 就显得力不从心了，在微服务面前和扩展方面，无法充分展示自己的优势。
- 开发者自定义
  - 也有很多公司选择自定义权限，即自己开发权限管理。但是一个系统的安全，不仅仅是登录和权限控制这么简单，我们还要考虑种各样可能存在的网络政击以及防彻策略，从这个角度来说，开发者白己实现安全管理也并非是一件容易的事情，只有大公司才有足够的人力物力去支持这件事情。
- Spring Security
  - Spring Security,作为spring 家族的一员，在和 Spring 家族的其他成员如 Spring Boot Spring Clond等进行整合时，具有其他框架无可比拟的优势，同时对 OAuth2 有着良好的支持，再加上Spring Cloud对 Spring Security的不断加持（如推出 Spring Cloud Security )，让 Spring Securiy 不知不觉中成为微服务项目的首选安全管理方案。

## 2. 简介、历史、架构

### 2.1 官方定义

- 官网：https://spring.io/projects/spring-security。

`Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.`

`Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirements`

> `Spring Security` 是一个功能强大、可高度定制的 **`身份验证`** 和 **`访问控制`** 的框架，或者说用来实现系统中权限管理的框架。

==注意：== `Spring Security`只能用于 Spring 的应用程序中。

### 2.2 历史

Spring Security 的前身是 Acegi Security ，是 Spring 项目组中用来提供安全认证服务的框架。

Spring Security 为基于J2EE企业应用软件提供了全面安全服务。特别是使用领先的J2EE解决方案-Spring框架开发的企业软件项目。

### 2.3 整体架构

在 `Spring Security` 的架构设计中，**`认证Authentication`**和**`授权Authorization`** 是分开的，无论使用什么样的认证方式。都不会影响授权，这是**两个独立**的存在，这种独立带来的好处之一，就是可以非常方便地整合一些外部的解决方案。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220110112541559.png)

#### 2.3.1 认证

##### AuthenticationManager

在 `Spring Security` 中认证是由`AuthenticationManager`接口来负责的，接口定义为：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220110104531129.png)

```java
public interface AuthenticationManager { 
    Authentication authenticate(Authentication authentication) 
                                                          throws AuthenticationException;
}
```

- 返回 `Authentication` 表示认证成功
- 返回 `AuthenticationException` 异常，表示认证失败。

AuthenticationManager 主要实现类为 ProviderManager，在 ProviderManager 中管理了众多 AuthenticationProvider 实例。在一次完整的认证流程中，Spring Security 允许存在多个 AuthenticationProvider ，用来实现多种认证方式，这些 AuthenticationProvider 都是由 ProviderManager 进行统一管理的。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220110103518334.png)

##### Authentication

认证以及认证成功的信息主要是由 Authentication 的实现类进行保存的，其接口定义为：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220110104815645.png)

```java
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    Object getCredentials();
    Object getDetails();
    Object getPrincipal();
    boolean isAuthenticated();
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

- `getAuthorities` 	 获取用户权限信息
- `getCredentials `      获取用户凭证信息，一般指密码
- `getDetails`              获取用户详细信息
- `getPrincipal   `          获取用户身份信息，用户名、用户对象等
- `isAuthenticated`    用户是否认证成功

##### SecurityContextHolder

​		SecurityContextHolder 用来获取登录之后用户信息。Spring Security 会将登录用户数据保存在 Session 中。但是，为了使用方便,Spring Security在此基础上还做了一些改进，其中最主要的一个变化就是**线程绑定**。**当用户登录成功后,Spring Security 会将登录成功的用户信息保存到 SecurityContextHolder 中**。SecurityContextHolder 中的数据保存默认是通过ThreadLocal 来实现的，使用 ThreadLocal 创建的变量只能被当前线程访问，不能被其他线程访问和修改，也就是用户数据和请求线程绑定在一起。当登录请求处理完毕后，Spring Security 会将 SecurityContextHolder 中的数据拿出来保存到 Session 中，同时将 SecurityContexHolder 中的数据清空。以后每当有请求到来时，Spring Security 就会先从 Session 中取出用户登录数据，保存到 SecurityContextHolder 中，方便在该请求的后续处理过程中使用，同时在请求结束时将 SecurityContextHolder 中的数据拿出来保存到 Session 中，然后将 Security SecurityContextHolder 中的数据清空。这一策略非常方便用户在 Controller、Service 层以及任何代码中获取当前登录用户数据。

#### 2.3.2 授权

当完成认证后，接下米就是授权了。在 Spring Security 的授权体系中，有两个关键接口，

##### AccessDecisionManager

> AccessDecisionManager (访问决策管理器)，用来决定此次访问是否被允许。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220110110946267.png)

##### AccessDecisionVoter

> AccessDecisionVoter (访问决定投票器)，投票器会检查用户是否具备应有的角色，进而投出赞成、反对或者弃权票。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220110111011018.png)

AccessDecisionVoter 和 AccessDecisionManager 都有众多的实现类，在 AccessDecisionManager 中会挨个遍历 AccessDecisionVoter，进而决定是否允许用户访问，因而 AccessDecisionVoter 和 AccessDecisionManager 两者的关系类似于 AuthenticationProvider 和 ProviderManager 的关系。

##### ConfigAttribute

> ConfigAttribute，用来保存授权时的角色信息

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220110111037603.png)

在 Spring Security 中，用户请求一个资源(通常是一个接口或者一个 Java 方法)需要的角色会被封装成一个 ConfigAttribute 对象，在 ConfigAttribute 中只有一个 getAttribute方法，该方法返回一个 String 字符串，就是角色的名称。一般来说，角色名称都带有一个 `ROLE_` 前缀，投票器 AccessDecisionVoter 所做的事情，其实就是比较用户所具各的角色和请求某个资源所需的 ConfigAtuibute 之间的关系。

## 3. 环境搭建

### 3.1 新建Springboot项目

### 3.2 导入Spring Security依赖

```xml
        <!--引入Spring Security依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

![image-20220409204655286](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220409204655286.png)

- 默认用户名：`user`
- 默认密码：控制台打印出的 `uuid`



### 3.3 实现原理

https://docs.spring.io/spring-security/site/docs/5.5.4/reference/html5/#servlet-architecture

 在 `Spring Security` 中，`认证、授权`等功能都是通过 ==过滤器==实现的。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/securityfilterchain.png)

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/multi-securityfilterchain.png)

​		值得注意的是，过滤器并不是直接直接放在 Web 项目的原生过滤器中，而是通过 FilterChainProxy 统一管理。Spring Security 中的过滤器链通过 FilterChainProxy 嵌入到 Web 项目的原生过滤器链中。FilterChainProxy 作为一个顶层的管理者，将统一管理 Security Filter。FilterChainProxy 本身是通过 Spring 框架提供的 DelegatingFilterProxy 整合到原生的过滤器链中。

### 3.4 源码分析

| 过滤器                                          | 过滤器作用                                               | 默认是否加载 |
| ----------------------------------------------- | -------------------------------------------------------- | ------------ |
| ChannelProcessingFilter                         | 过滤请求协议 HTTP 、HTTPS                                | NO           |
| `WebAsyncManagerIntegrationFilter`              | 将 WebAsyncManger 与 SpringSecurity 上下文进行集成       | YES          |
| `SecurityContextPersistenceFilter`              | 在处理请求之前,将安全信息加载到 SecurityContextHolder 中 | YES          |
| `HeaderWriterFilter`                            | 处理头信息加入响应中                                     | YES          |
| CorsFilter                                      | 处理跨域问题                                             | NO           |
| `CsrfFilter`                                    | 处理 CSRF 攻击                                           | YES          |
| `LogoutFilter`                                  | 处理注销登录                                             | YES          |
| OAuth2AuthorizationRequestRedirectFilter        | 处理 OAuth2 认证重定向                                   | NO           |
| Saml2WebSsoAuthenticationRequestFilter          | 处理 SAML 认证                                           | NO           |
| X509AuthenticationFilter                        | 处理 X509 认证                                           | NO           |
| AbstractPreAuthenticatedProcessingFilter        | 处理预认证问题                                           | NO           |
| CasAuthenticationFilter                         | 处理 CAS 单点登录                                        | NO           |
| OAuth2LoginAuthenticationFilter                 | 处理 OAuth2 认证                                         | NO           |
| Saml2WebSsoAuthenticationFilter                 | 处理 SAML 认证                                           | NO           |
| `UsernamePasswordAuthenticationFilter`          | 处理表单登录                                             | YES          |
| OpenIDAuthenticationFilter                      | 处理 OpenID 认证                                         | NO           |
| `DefaultLoginPageGeneratingFilter`              | 配置默认登录页面                                         | YES          |
| `DefaultLogoutPageGeneratingFilter`             | 配置默认注销页面                                         | YES          |
| ConcurrentSessionFilter                         | 处理 Session 有效期                                      | NO           |
| DigestAuthenticationFilter                      | 处理 HTTP 摘要认证                                       | NO           |
| BearerTokenAuthenticationFilter                 | 处理 OAuth2 认证的 Access Token                          | NO           |
| `BasicAuthenticationFilter`                     | 处理 HttpBasic 登录                                      | YES          |
| `RequestCacheAwareFilter`                       | 处理请求缓存                                             | YES          |
| `SecurityContextHolder<br />AwareRequestFilter` | 包装原始请求                                             | YES          |
| JaasApiIntegrationFilter                        | 处理 JAAS 认证                                           | NO           |
| RememberMeAuthenticationFilter                  | 处理 RememberMe 登录                                     | NO           |
| `AnonymousAuthenticationFilter`                 | 配置匿名认证                                             | YES          |
| OAuth2AuthorizationCodeGrantFilter              | 处理OAuth2认证中授权码                                   | NO           |
| `SessionManagementFilter`                       | 处理 session 并发问题                                    | YES          |
| `ExceptionTranslationFilter`                    | 处理认证/授权中的异常                                    | YES          |
| `FilterSecurityInterceptor`                     | 处理授权相关                                             | YES          |
| SwitchUserFilter                                | 处理账户切换                                             | NO           |

Spring Security 提供了 30 多个过滤器，默认情况下 SpringBoot 在对 Spring Security 进入自动化配置的时候，会创建一个名为 SpringSecurityFilterChain 的过滤器，并注入到 Spring 容器中，这个过滤器将负责所有的安全管理，包括用户认证、授权、重定向到登陆页面等。具体可参考 WebSecurityConfiguration 源码：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220111211538604.png)

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220111211436764.png)

### 3.5 自动配置分析

SpringBootWebSecurityConfiguration 类是 spring boot 自动配置类，通过这个源码得知，默认情况下对所有请求进行权限控制:

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnDefaultWebSecurity
@ConditionalOnWebApplication(type = Type.SERVLET)
class SpringBootWebSecurityConfiguration {
    @Bean
    @Order(SecurityProperties.BASIC_AUTH_ORDER)
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) 
    throws Exception {
    //要求所有请求都要认证
    http.authorizeRequests().anyRequest().authenticated()
    .and().formLogin().and().httpBasic();
        return http.build();
    }
}
```

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220112095052138.png)

**这就是为什么在引入 Spring Security 中没有任何配置情况下，请求会被拦截的原因！**

通过上面对自动配置分析，我们也能看出默认生效条件为：

```java
class DefaultWebSecurityCondition extends AllNestedConditions {

    DefaultWebSecurityCondition() {
        super(ConfigurationPhase.REGISTER_BEAN);
    }

    @ConditionalOnClass({ SecurityFilterChain.class, HttpSecurity.class })
    static class Classes {

    }

    @ConditionalOnMissingBean({ WebSecurityConfigurerAdapter.class, SecurityFilterChain.class })
    static class Beans {

    }
}
```

- 条件一： classpath中存在 SecurityFilterChain.class, HttpSecurity.class
- 条件二 ：没有自定义 WebSecurityConfigurerAdapter.class, SecurityFilterChain.class

**结论：**只要我们不自定义配置类(比如WebSecurityConfigurerAdapter)，那么以上条件都是满足的，也就加载默认的配置。否则如果要进行自定义配置，就要继承 WebSecurityConfigurerAdapter类，通过覆盖类中方法达到修改默认配置的目的。

**WebSecurityConfigurerAdapter** 这个类极其重要，Spring Security 核心配置都在这个类中：

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220112095638356.png)



### 3.6 流程分析（生成登陆页面）

![image-20220409215505816](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220409215505816.png)

1. 请求 /hello 接口，在引入 Spring Security 以后会经过一系列过滤器
2. 在请求到达 FilterSecurityInterceptor 时，发现请求并未认证。请求被拦截，并抛出 AccessDeniedException 异常。
3. 抛出的 AccessDeniedException 异常会被ExceptionTranslationFilter 捕获，该Filter 会调用 LoginUrlAuthenticationEntryPoint#commence 方法给客户端返回 302，要求客户端进行重定向到 /login 页面。
4. 客户端发出 /login 请求。
5. /login 请求会再次被拦截器中 DefaultLoginPageGeneratingFilter 拦截到，并在拦截器中返回生成登陆页面。

**就是通过以上流程，Spring Security默认过滤器中生成了登陆页面，并返回！**

### 3.7 默认用户生成

 

## 4. 自定义认证

```java
@Component
public class SecurityConfig extends WebSecurityConfigurerAdapter{
    //登录成功处理器
    @Autowired
    private AuthenticationSuccessHandler authenticationSuccessHandler;
    //登录失败处理器
    @Autowired
    AuthenticationFailureHandler authenticationFailureHandler;
    //退出成功处理器
    @Autowired
    private LogoutSuccessHandler logoutSuccessHandler;

    //刚被注入时就会调这个方法进行预设
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        http.authorizeRequests()
                //antMatchers()参数可以有多个url且均为ant风格,denyAll()表示拒绝所有请求.
                .antMatchers("/hello").denyAll()
                //任何请求都要认证,anyRequest()必须在最后.
                .anyRequest().authenticated()
                //返回HttpSecurity
                .and()
                //采用表单认证
                .formLogin()
                //以下三项设置均是可选的
                    //确认登录的url
                    .loginProcessingUrl("/login")
                    //自定义的usernameParameter
                    .usernameParameter("username")
                    //自定义的passwordParameter
                    .passwordParameter("password")
                //认证成功后的处理器，相关介绍在下面。
                .successHandler(authenticationSuccessHandler)
                //认证成功后的处理器，相关介绍在下面。
                .failureHandler(authenticationFailureHandler)
                .and()
                .logout()
                //以下这项设置是可选的
                    //自定义的退出url以及请求方式
                    .logoutRequestMatcher(new OrRequestMatcher(new AntPathRequestMatcher("/logout1","GET"),new AntPathRequestMatcher("/logout2","POST")))
                //成功后的处理器，相关介绍在下面。
                .logoutSuccessHandler(logoutSuccessHandler)
                .and()
                //禁用csrf
                .csrf().disable();
    }
}
```





### 4.1 自定义资源权限规则

想实现如下功能：

- `/index` 公共资源，所有人都可以访问
- `/hello` 受保护资源 需要进行权限管理

在项目中添加如下配置即可实现对资源权限规则的设置：

**自定义配置类继承 `WebSecurityConfigurerAdapter`并重写 `configure` 方法。 **

![image-20220410101945554](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220410101945554.png)

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        /**
         * 注意：
         *  要放行的请求必须放在认证之前
         * */

        http.authorizeRequests() // 开启请求的权限管理
                .mvcMatchers("/index").permitAll() // 放行 /index 请求
//                .mvcMatchers("/hello").authenticated() // 认证 /hello 请求
                .anyRequest().authenticated() // 对所有请求进行认证 
                .and()
                .formLogin();
    }
}
```

**说明：**

- `mvcMatchers("/xxx").permitAll()` 代表放行资源。该资源为公共资源，无需认证和授权即可访问。
- `anyRequest().authenticated()` 代表所有资源必须认证之后才能访问。
- `formLogin()` 代表开启表单认证

**注意：放行的资源必须放在所有认证请求之前。**

###  4.2 自定义登录界面

- 定义登录 controller

  - ```java
    @Controller
    public class LoginController {
    
        @RequestMapping("/login.html")
        public String login() {
            return "login";
        }
    }
    ```

- 定义登录页面

  - ```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>登录</title>
    </head>
    <body>
    
    <h1>用户登录</h1>
    <form method="post" th:action="@{/doLogin}">
        用户名: <input name="username" type="text"> <br>
        密码: <input name="password" type="password"> <br>
        <button>登录</button>
    </form>
    
    </body>
    </html>
    ```

  - **需要注意的是：**

    - 登录表单必须是 `post` 登录，请求路径此处为`/doLogin`，必须在Spring Security 配置类中进行配置
    - 用户名的 name 默认是 `username`，若修改必须在Spring Security 配置类中进行配置
    - 密码的 name 默认是 `password`，若修改必须在Spring Security 配置类中进行配置

- 配置 Spring Security 配置类

  - ```java
    @Configuration
    public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
    
            /**
             * 注意：
             *  要放行的请求必须放在认证之前
             * */
    
            http.authorizeRequests() // 开启请求的权限管理
                    .mvcMatchers("/login.html").permitAll()
                    .mvcMatchers("/index").permitAll() // 放行 /index 请求
    //                .mvcMatchers("/hello").authenticated() // 认证 /hello 请求
                    .anyRequest().authenticated() // 对所有请求进行认证
                    .and()
                    .formLogin()
                    .loginPage("/login.html") /** 自定义登录界面 注意：一定自定义登陆页面，就必须指定登录的url **/
                    .loginProcessingUrl("/doLogin")  /** 指定处理登录请求的 url **/
    /**
     *               修改form表单处 username 与 password 的参数名 默认是username与password
     *               .usernameParameter("uname")
     *               .passwordParameter("passwd")
     **/
                   // .successForwardUrl("/hello") // 认证成功以后的跳转路径 使用的是forward跳转，跳转以后地址不变；会根据上一次请求的url进行跳转
                   // .defaultSuccessUrl("/hello") //认证成功跳转  redirect的重定向跳转 地址会变；会根据上一次请求的url进行跳转
                    .and()
                    .csrf().disable(); // 禁止csrf 跨站请求保护
        }
    }
    
    ```

![image-20220410113219548](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220410113219548.png)

> `successForwardUrl` 与 `defaultSuccessUrl` 方法只适用于前后端不分离的情况.

### 4.3 自定义登陆成功处理(前后端分离)

有时候页面跳转并不能满足我们，特别是在前后端分离开发中就不需要成功之后跳转页面。只需要给前端返回JSON 对象通知成功与否。这个时候可以通过自定义 `AuthenticationSuccessHandle` 实现。

```java
public interface AuthenticationSuccessHandler {

	/**
	 * Called when a user has been successfully authenticated.
	 * @param request the request which caused the successful authentication
	 * @param response the response
	 * @param chain the {@link FilterChain} which can be used to proceed other filters in
	 * the chain
	 * @param authentication the <tt>Authentication</tt> object which was created during
	 * the authentication process.
	 * @since 5.2.0
	 */
	default void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
			Authentication authentication) throws IOException, ServletException {
		onAuthenticationSuccess(request, response, authentication);
		chain.doFilter(request, response);
	}
}
```

**根据接口的描述信息，可以得知登录成功会自动回调这个方法，进一步查看它的默认实现，发现 `successForwardUrl、defaultSuccessUrl`也是由它的子类实现的。**

- 自定义`AuthenticationSuccessHandle` 

```java
/**
 * 自定义认证成功之后处理
 */
public class LoginSuccessHandler implements AuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        Map<String, Object> map = new HashMap<>();
        map.put("msg", "登陆成功");
        map.put("status", 200);
        map.put("authentication", authentication);
        response.setContentType("application/json;charset=UTF-8");
        String string = new ObjectMapper().writeValueAsString(map);
        response.getWriter().println(string);
    }
}
```

- `Spring Security` 配置类

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        /**
         * 注意：
         *  要放行的请求必须放在认证之前
         * */

        http.authorizeRequests() // 开启请求的权限管理
                .mvcMatchers("/login.html").permitAll()
                .mvcMatchers("/index").permitAll() // 放行 /index 请求
//                .mvcMatchers("/hello").authenticated() // 认证 /hello 请求
                .anyRequest().authenticated() // 对所有请求进行认证
                .and()
                .formLogin()
                .loginPage("/login.html") /** 自定义登录界面 注意：一定自定义登陆页面，就必须指定登录的url **/
                .loginProcessingUrl("/doLogin")  /** 指定处理登录请求的 url **/
/**
 *               修改form表单处 username 与 password 的参数名 默认是username与password
 *               .usernameParameter("uname")
 *               .passwordParameter("passwd")
 **/
               // .successForwardUrl("/hello") // 认证成功以后的跳转路径 使用的是forward跳转，跳转以后地址不变；会根据上一次请求的url进行跳转
//                .defaultSuccessUrl("/hello") //认证成功跳转  redirect的重定向跳转 地址会变；会根据上一次请求的url进行跳转
                .successHandler(new LoginSuccessHandler()) /*前后端分录处理登陆成功请求*/
                .and()
                .csrf().disable(); // 禁止csrf 跨站请求保护
    }
}

```

![image-20220410150712736](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220410150712736.png)

### 4.4 显示登录失败信息

Spring Security 会在登陆失败之后将异常信息存储到 `request、Session` 作用域中 key 为 `SPRING_SECURITY_LAST_EXCEPTION` 常量中，源码可以参考`SimpleUrlAuthenticationFailureHandler`。

![image-20220410154905907](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220410154905907.png)

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        /**
         * 注意：
         *  要放行的请求必须放在认证之前
         * */

        http.authorizeRequests() // 开启请求的权限管理
            // ...
                .and()
            // ...
                .failureForwardUrl("/login.html") 
            // 认证失败以后的跳转路径 使用的是forward跳转，跳转以后地址不变
			// .failureUrl("/login.html") //认证失败跳转  redirect的重定向跳转 地址会变
                .and()
                .csrf().disable(); // 禁止csrf 跨站请求保护
    }
}
```

- `failureForwardUrl` 是将失败信息存储带 `request`作用域中
- `failureUrl` 是存储到 `Session` 作用域中。

### 4.5 自定义登录失败处理(前后端分离)

​		自定义的失败处理可以通过自定义 `AuthenticationFaliureHandle` 实现。

- 自定义`AuthenticationFaliureHandle` 

```java
/**
 * 自定义认证失败处理方法
 */
public class LoginFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        Map<String, Object> map = new HashMap<>();
        map.put("msg", "登录失败");
        map.put("status", 500);
        response.setContentType("application/json;charset=UTF-8");
        String string = new ObjectMapper().writeValueAsString(map);
        response.getWriter().println(string);
    }
}
```

- `Spring Security` 配置类

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        /**
         * 注意：
         *  要放行的请求必须放在认证之前
         * */

        http.authorizeRequests() // 开启请求的权限管理
            // ...
                .and()
            // ...
                .failureHandler(new LoginFailureHandler()) /*前后端分录自定义登陆失败处理方案*/
                .and()
                .csrf().disable(); // 禁止csrf 跨站请求保护
    }
}
```

### 4.6 注销登录

Spring Security 中也提供了默认的注销登录配置，在开发时也可以按照自己需求对注销进行个性化定制

- 开启注销登录 `默认是开启的`

  - ```java
    @Configuration
    public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests() // 开启请求的权限管理
                // ...
                    .and()
                // ...
                    .and()
                    .logout()
                    .logoutUrl("/logout") // 指定注销登录的url
                    .invalidateHttpSession(true) // 默认 会话失效
                    .clearAuthentication(true) // 默认 清除认证标记
                    .logoutSuccessUrl("/login.html") // 注销登录成功之后跳转画面
                    .and()
                    .csrf().disable(); // 禁止csrf 跨站请求保护
        }
    }
    
    ```

  - 通过 `/logout` 方法开启注销配置

  - `/logoutUrl` 指定退出登录的请求地址，默认是 `GET` 请求，路径是 `/logout`

  - `invalidateHttpSession`表示退出时 `Session` 是否失效，默认为 `true`

  -  `clearAuthentication`表示退出时是否清除认证信息，默认为 `true`

  - `logoutSuccessUrl` 表示退出登录时的跳转路径

- 配置多个注销登录请求

  - 如果项目中有需求，开发者可以配置多个注销登录的请求，同时还可以指定请求的方法：

  - ```java
    @Configuration
    public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests() // 开启请求的权限管理
                // ...
                    .and()
                // ...
                    .and()
                    .logout()
    /*                .logoutUrl("/logout") // 指定注销登录的url*/
                    .logoutRequestMatcher(new OrRequestMatcher((
                            new AntPathRequestMatcher("/logout1", "POST"),
                            new AntPathRequestMatcher("/logout", "GET")
                    ))
                    .invalidateHttpSession(true) // 默认 会话失效
                    .clearAuthentication(true) // 默认 清除认证标记
                    .logoutSuccessUrl("/login.html") // 注销登录成功之后跳转画面
                    .and()
                    .csrf().disable(); // 禁止csrf 跨站请求保护
        }
    }
    ```

- 前后端分离注销登录配置

  - 前后端分离开发，注册成功之后不需要页面跳转，只需要注销成功的信息返回前端即可，此时可以通过自定义 `LogoutSuccessHandler`实现向前端返回注销的信息

  - ```java
    /**
     * 自定义注销成功的处理
     */
    public class MyLogoutSuccessHandler implements LogoutSuccessHandler {
        @Override
        public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
            HashMap<String, Object> res = new HashMap<>();
            res.put("msg", "退出成功");
            res.put("exitingUser", authentication);
            res.put("status", "200");
            response.setContentType("application/json;charset=UTF-8");
            String string = new ObjectMapper().writeValueAsString(res);
            response.getWriter().print(string);
        }
    }
    ```

  - `Spring Security`配置类

  - ```java
    @Configuration
    public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests() // 开启请求的权限管理
                // ...
                    .and()
                // ...
                    .and()
                    .logout()
    /*                .logoutUrl("/logout") // 指定注销登录的url*/
                    .logoutRequestMatcher(new OrRequestMatcher((
                            new AntPathRequestMatcher("/logout1", "POST"),
                            new AntPathRequestMatcher("/logout", "GET")
                    ))
                    .invalidateHttpSession(true) // 默认 会话失效
                    .clearAuthentication(true) // 默认 清除认证标记
    //                .logoutSuccessUrl("/login.html") // 注销登录成功之后跳转画面
                    .logoutSuccessHandler(new MyLogoutSuccessHandler()) /*前后端分离 注销登录成功的处理*/
                    .and()
                    .csrf().disable(); // 禁止csrf 跨站请求保护
        }
    }
    ```



### 4.7 获取登录用户数据

#### SecurityContextHolder

​		Spring Security 会将登录的用户数据保存在 Session中。但是，为了使用方便，Spring Security在此基础上做了一些修改，其中最主要的一个变化是线程绑定。当用户登陆成功以后，Spring Security会将登录成功的用户信息保存到 SecurityContextHolder。

​		SecurityContextHolder 中的数据保存默认是通过 ThreadLocal 实现的，使用 ThreadLocal 创建的变量只能当前线程访问，不能被其他线程访问和修改，也就是用户数据和请求线程绑定在一起。当登录请求处理完毕后，Spring Security 会将 SecurityContextHolder 中的数据拿出来保存到 Session 中，同时将 SecurityContextHolder 中的数据清空，以后每当有请求到来时，Spring Security 就会先从 Session 中取出用户登录数据，保存到 SecurityContextHolder 中，同时在该请求的后续处理过程中使用，同时在请求结束时将 SexurityContextHolder 中的数据保存到 Session 中，然后将 SecurityContextHolder 中的数据清空。

​	实际上SecurityContextHolder 中存储的是 SecurityContext，在SecurityContext 中存储的是 Authentication。

![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/securitycontextholder.png)

`SecurityContextHolder`使用了策略模式：

```java
public class SecurityContextHolder {

	public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";

	public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";

	public static final String MODE_GLOBAL = "MODE_GLOBAL";

	private static final String MODE_PRE_INITIALIZED = "MODE_PRE_INITIALIZED";

	public static final String SYSTEM_PROPERTY = "spring.security.strategy";

	private static String strategyName = System.getProperty(SYSTEM_PROPERTY);

	private static SecurityContextHolderStrategy strategy;
    //...
    	private static void initializeStrategy() {
		if (MODE_PRE_INITIALIZED.equals(strategyName)) {
			Assert.state(strategy != null, "When using " + MODE_PRE_INITIALIZED
					+ ", setContextHolderStrategy must be called with the fully constructed strategy");
			return;
		}
		if (!StringUtils.hasText(strategyName)) {
			// Set default
			strategyName = MODE_THREADLOCAL;
		}
		if (strategyName.equals(MODE_THREADLOCAL)) {
			strategy = new ThreadLocalSecurityContextHolderStrategy();
			return;
		}
		if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
			strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
			return;
		}
		if (strategyName.equals(MODE_GLOBAL)) {
			strategy = new GlobalSecurityContextHolderStrategy();
			return;
		}
		// Try to load a custom strategy
		try {
			Class<?> clazz = Class.forName(strategyName);
			Constructor<?> customStrategy = clazz.getConstructor();
			strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
		}
		catch (Exception ex) {
			ReflectionUtils.handleReflectionException(ex);
		}
	}
    //...
}
```

#### 获取认证之后的用户数据

##### 单线程模式

```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        System.out.println("Hello Spring Security");
        // 1. 获取认证信息
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        System.out.println("身份信息: " + authentication.getPrincipal());
        System.out.println("权限信息: " + authentication.getAuthorities());
        return "Hello Spring Security";
    }
}
```

#### 父子线程模式：MODE_INHERITABLETHREADLOCAL（数据可以在本线程和后代线程中获取)

![image-20220410194210547](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220410194210547.png)

```java
要传系统参数
-Dspring.security.strategy=MODE_INHERITABLETHREADLOCAL
    
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
      new Thread(()->{
        Authentication authentication = SecurityContextHolder
          .getContext().getAuthentication();
        System.out.println("认证信息: "+authentication);
      }).start();
      return "hello security";
    }
}
```

##### 页面中获取用户信息

- 引入依赖

  - ```xml
    <dependency>
        <groupId>org.thymeleaf.extras</groupId>
        <artifactId>thymeleaf-extras-springsecurity5</artifactId>
    </dependency>
    ```

- 引入命名空间

  - ```html
    xmlns:sec="http://www.thymeleaf.org/extras/spring-security"
    ```

- 页面中使用

  - ```html
    <ul>
        <li sec:authentication="principal.username"></li>
        <li sec:authentication="principal.authorities"></li>
        <li sec:authentication="principal.accountNonExpired"></li>
        <li sec:authentication="principal.accountNonLocked"></li>
    </ul>
    ```

### 4.8 自定义数据源

#### 4.8.1 认证流程分析

<img style="zoom:70%" src="https://gitee.com/yun-xiaojie/blog-image/raw/master/img/abstractauthenticationprocessingfilter.png">

- 发起认证请求，请求中携带用户名、密码，该请求会被 `UsernamePasswordAuthenticationFilter` 拦截
- 在`UsernamePasswordAuthenticationFilter` 的 `attemptAuthentication` 方法中将请求中的用户名、密码封装为 `Authentication` 对象
  - 认证成功，将认证信息存储到 `SecurityContextHolder` 中，并调用 记住我等，回调`AuthenticationSuccessHandler`处理。
  - 认证失败，清除 `SecurityContextHolder` 以及记住我中的信息，回调`AuthenticationFailureHandler`处理。

#### 4.8.2 三者关系

从上面分析中得知，`AuthenticationManager` 是认证的核心类，但实际上在底层真正认证时离不开 `ProviderManager` 以及`AuthenticationProvider`。

- `AuthenticationManager` 是一个认证管理器，它定义了 Spring Security 过滤器要执行的认证操作。
- `ProviderManager` 是`AuthenticationManager`的实现类。Spring Security认证时候默认使用的就是 `ProviderManager` 。
- `AuthenticationProvider`是针对不同身份类型执行的具体的身份认证。

`ProviderManager` 是`AuthenticationManager`的唯一实现，也是Spring Security默认使用的实现。



![img](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/providermanager.png)

**关于ProviderManager**

在 Spring Security 中，允许系统同时支持多种不同的认证方式，例如同时支持用户名/密码认证、ReremberMe 认证、手机号码动态认证等，而不同的认证方式对应了不同的 AuthenticationProvider，所以一个完整的认证流程可能由多个 AuthenticationProvider 来提供。

多个 AuthenticationProvider 将组成一个列表，这个列表将由 ProviderManager 代理。换句话说，在ProviderManager 中存在一个 AuthenticationProvider 列表，在Provider Manager 中遍历列表中的每一个 AuthenticationProvider 去执行身份认证，最终得到认证结果。

ProviderManager 本身也可以再配置一个 AuthenticationManager 作为 parent，这样当ProviderManager 认证失败之后，就可以进入到 parent 中再次进行认证。理论上来说，ProviderManager 的 parent 可以是任意类型的 AuthenticationManager，但是通常都是由ProviderManager 来扮演 parent 的角色。

ProviderManager 本身也可以有多个，多个ProviderManager 共用同一个 parent。有时，一个应用程序有受保护资源的逻辑组（例如，所有符合路径模式的网络资源，如/api/**），每个组可以有自己的专用 AuthenticationManager。通常，每个组都是一个ProviderManager，它们共享一个父级。然后，父级是一种 `全局`资源，作为所有提供者的后备资源。

根据上面的介绍，我们绘出新的ProvideManager 和 AuthentictionProvider 关系：

![img](http://shrink.fun/note/spring-security/SpringSecurity.assets/image-20220118061343516.png)

**总结：AuthenticationManager 是认证管理器，在Spring Security 中有全局AuthenticationManger，也可以有局部 AuthentioncationManager。全局的AuthenticationManager用来对全局认证进行处理，局部的AuthenticationManager用来对某些特殊资源认证处理。无论是全局认证管理器还是局部认证管理器都是由ProviderManager进行实现的。每一个ProviderManager中都代理一个AuthenticationProvider的列表，列表中每一个实现代表一种身份认证方式。认证时底层数据源需要调用UserDetailService实现。**

#### 4.8.3 配置全局AuthenticationManager

- 默认

  ```java
      /*springboot对security的默认配置中，会在工厂中默认创建AuthenticationManager*/
      @Autowired
      public void initialize(AuthenticationManagerBuilder builder) throws Exception {
          System.out.println("SpringBoot 默认配置: " + builder);
          InMemoryUserDetailsManager userDetails = new InMemoryUserDetailsManager();
          userDetails.createUser(User.withUsername("aaa").password("{noop}123").roles("admin").build()); // {noop}表示明文
          builder.userDetailsService(userDetails);
      }
  ```

- 优化：容器中注入 `UserDetailsService` 即可

- ```java
      @Bean
      public UserDetailsService userDetailsService() {
          InMemoryUserDetailsManager userDetails = new InMemoryUserDetailsManager();
          userDetails.createUser(User.withUsername("aaa").password("{noop}123").roles("admin").build()); // {noop}表示明文
          return userDetails;
      }
  
      /*springboot对security的默认配置中，会在工厂中默认创建AuthenticationManager*/
  /*    @Autowired
      public void initialize(AuthenticationManagerBuilder builder) throws Exception {
          System.out.println("SpringBoot 默认配置: " + builder);
  
          builder.userDetailsService(userDetails);
      }*/
  ```

**总结：**

1. 默认 `AuthenticationManager` 会默认找当前项目中是否存在自定义的 `UserDetailsService` 实例，如果存在则**自动**将该实例设置为数据源。
2. 默认 `AuthenticationManager` 在工厂中使用的时候直接在代码中注入即可。

- 自定义

  ```java
      @Bean
      public UserDetailsService userDetailsService() {
          InMemoryUserDetailsManager userDetails = new InMemoryUserDetailsManager();
          userDetails.createUser(User.withUsername("aaa").password("{noop}123").roles("admin").build()); // {noop}表示明文
          return userDetails;
      }
  
      @Override
      public void configure(AuthenticationManagerBuilder builder) throws Exception {
          System.out.println("自定义AuthenticationManager: " + builder);
          builder.userDetailsService(userDetailsService());
      }
  ```

**总结：**

1. 通过 `configure(AuthenticationManagerBuilder builder)` 自定义 `AuthenticationManager` 以后，会将工厂中自动配置的  `AuthenticationManager`进行覆盖。

2. 通过 `configure(AuthenticationManagerBuilder builder)` 自定义 `AuthenticationManager` 必须指定认证的数据源  `UserDetailsService` 实例。

3. 通过该方法生成的`AuthenticationManager` 是本地的，不能在代码的其他位置进行注入。

   - 解决方法：

   - ```java
         @Override
         public void configure(AuthenticationManagerBuilder builder) throws Exception {
             System.out.println("自定义AuthenticationManager: " + builder);
             builder.userDetailsService(userDetailsService());
         }
     
         /**
          * 作用：将自定义的AuthenticationManager暴露在工厂中，这样就可以在代码的任何地方进行注入
          * @return
          * @throws Exception
          */
         @Override
         @Bean
         public AuthenticationManager authenticationManagerBean() throws Exception {
             return super.authenticationManagerBean();
         }
     ```

#### 4.8.4 定义内存数据源

- 创建数据库表

  ```mysql
  -- 用户表
  CREATE TABLE `user`
  (
      `id`                    int(11) NOT NULL AUTO_INCREMENT,
      `username`              varchar(32)  DEFAULT NULL,
      `password`              varchar(255) DEFAULT NULL,
      `enabled`               tinyint(1) DEFAULT NULL,
      `accountNonExpired`     tinyint(1) DEFAULT NULL,
      `accountNonLocked`      tinyint(1) DEFAULT NULL,
      `credentialsNonExpired` tinyint(1) DEFAULT NULL,
      PRIMARY KEY (`id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
  -- 角色表
  CREATE TABLE `role`
  (
      `id`      int(11) NOT NULL AUTO_INCREMENT,
      `name`    varchar(32) DEFAULT NULL,
      `name_zh` varchar(32) DEFAULT NULL,
      PRIMARY KEY (`id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
  -- 用户角色关系表
  CREATE TABLE `user_role`
  (
      `id`  int(11) NOT NULL AUTO_INCREMENT,
      `uid` int(11) DEFAULT NULL,
      `rid` int(11) DEFAULT NULL,
      PRIMARY KEY (`id`),
      KEY   `uid` (`uid`),
      KEY   `rid` (`rid`)
  ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;
  ```

- 插入测试数据

  ```mysql
  -- 插入用户数据
  BEGIN;
    INSERT INTO `user`
    VALUES (1, 'root', '{noop}123', 1, 1, 1, 1);
    INSERT INTO `user`
    VALUES (2, 'admin', '{noop}123', 1, 1, 1, 1);
    INSERT INTO `user`
    VALUES (3, 'song', '{noop}123', 1, 1, 1, 1);
  COMMIT;
  -- 插入角色数据
  BEGIN;
    INSERT INTO `role`
    VALUES (1, 'ROLE_product', '商品管理员');
    INSERT INTO `role`
    VALUES (2, 'ROLE_admin', '系统管理员');
    INSERT INTO `role`
    VALUES (3, 'ROLE_user', '用户管理员');
  COMMIT;
  -- 插入用户角色数据
  BEGIN;
    INSERT INTO `user_role`
    VALUES (1, 1, 1);
    INSERT INTO `user_role`
    VALUES (2, 1, 2);
    INSERT INTO `user_role`
    VALUES (3, 2, 4);
    INSERT INTO `user_role`
    VALUES (4, 3, 3);
  COMMIT;
  ```

- 导入依赖

  - ```xml
            <!--druid-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.2.6</version>
            </dependency>
            <!--mysql-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
            </dependency>
            <!--mybatis-->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>2.2.0</version>
            </dependency>
    ```

- 添加配置

  - ```properties
    spring.thymeleaf.cache=false
    
    spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.username=root
    spring.datasource.password=7012+2
    spring.datasource.url=jdbc:mysql://localhost:3306/springsecurity?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    
    #mybatis配置
    mybatis.mapper-location=classpath:com/lyj/mapper/*.xml
    mybatis.type-aliases-package=com.lyj.demo02.entity
    
    # 日志
    logging.level.com.lyj.demo02=debug
    ```

#### 4.8.5 数据库数据源

- 自定义 `UserDetailsService`

  - ```java
    @Component
    public class MyUserDetailService implements UserDetailsService {
    
        @Autowired
        private UserDao userDao;
    
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            // 1.查询用户
            User user = userDao.loadUserByUsername(username);
            if (user == null) {
                throw new UsernameNotFoundException("用户名不正确");
            }
            // 2. 权限查询
            List<Role> roles = userDao.getRolesByUid(user.getId());
            user.setRoles(roles);
            return user;
        }
    }
    
    @Data
    public class User implements UserDetails {
        private Integer id;
        private String username;
        private String password;
        private Boolean enabled; // 账户是否激活
        private Boolean accountNonExpired; // 账户是否过期
        private Boolean accountNonLocked; // 账户是否锁定
        private Boolean credentialsNonExpired; // 密码是否过期
        private List<Role> roles = new ArrayList<>(); // 存储当前用户的角色信息
    
        // 返回权限信息
        @Override
        public Collection<? extends GrantedAuthority> getAuthorities() {
            Set<SimpleGrantedAuthority> authorities = new HashSet<>();
            roles.forEach(role -> {
                SimpleGrantedAuthority simpleGrantedAuthority = new SimpleGrantedAuthority(role.getName());
                authorities.add(simpleGrantedAuthority);
            });
            return authorities;
        }
    
        @Override
        public boolean isAccountNonExpired() {
            return accountNonExpired;
        }
    
        @Override
        public boolean isAccountNonLocked() {
            return accountNonLocked;
        }
    
        @Override
        public boolean isCredentialsNonExpired() {
            return credentialsNonExpired;
        }
    
        @Override
        public boolean isEnabled() {
            return enabled;
        }
    }
    ```

- `Spring Security`配置类中注入 自定义的`UserDetailsService`

  - ```java
    @Configuration
    public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    
        @Autowired
        MyUserDetailService myUserDetailService;
    
        @Override
        public void configure(AuthenticationManagerBuilder builder) throws Exception {
            System.out.println("自定义AuthenticationManager: " + builder);
            builder.userDetailsService(myUserDetailService);
        }
    }
    ```



### ==4.9 传统web开发认真认证案例==

**准备实现功能：一个登录页面用于登录以及显示登陆失败的信息，一个主页显示登陆成功以后的用户信息。**

- 开发页面：

  - ```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>登录</title>
    </head>
    <body>
    
    <h1>用户登录</h1>
    <form method="post" th:action="@{/doLogin}">
        用户名: <input name="uname" type="text"> <br>
        密码: <input name="pwd" type="password"> <br>
        <button>登录</button>
    </form>
    <hr>
    <div th:text="${session.SPRING_SECURITY_LAST_EXCEPTION}"></div>
    </body>
    </html>
    ```

  - ```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org"
          xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
    <head>
        <meta charset="UTF-8">
        <title>系统主页</title>
    </head>
    <body>
    <h1>
        欢迎
        <span sec:authentication="principal.username"></span>
        进入我的系统
    </h1>
    <h3>用户的详细信息:</h3>
    <ul>
        <li sec:authentication="principal.username"></li>
        <li sec:authentication="principal.authorities"></li>
        <li sec:authentication="principal.accountNonExpired"></li>
        <li sec:authentication="principal.accountNonLocked"></li>
        <li sec:authentication="principal.credentialsNonExpired"></li>
    </ul>
    <hr>
    <a th:href="@{/logout}">退出登录</a>
    </body>
    </html>
    ```

- 自定义`userDetailService`

  - ```java
    @Component
    public class MyUserDetailService implements UserDetailsService {
    
        @Autowired
        private UserDao userDao;
    
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            // 1.查询用户
            User user = userDao.loadUserByUsername(username);
            if (user == null) {
                throw new UsernameNotFoundException("用户名不正确");
            }
            // 2. 权限查询
            List<Role> roles = userDao.getRolesByUid(user.getId());
            user.setRoles(roles);
            return user;
        }
    }
    ```

- `Spring Security`配置类

  - ```java
    /**
     * 自定义 spring security相关配置
     */
    @Configuration
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
        @Autowired
        private MyUserDetailService myUserDetailService;
    
        @Override
        protected void configure(AuthenticationManagerBuilder builder) throws Exception {
            builder.userDetailsService(myUserDetailService);
        }
    
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests()
                    .mvcMatchers("/login.html").permitAll()
                    .anyRequest().authenticated()
                    .and()
                    .formLogin()
                    .loginPage("/login.html")
                    .loginProcessingUrl("/doLogin")
                    .passwordParameter("pwd")
                    .usernameParameter("uname")
                    .defaultSuccessUrl("/index.html", true)
                    .failureForwardUrl("/login.html") // 登录失败重定向到登陆页面
                    .and()
                    .logout()
                    .logoutUrl("/logout")
                    .logoutSuccessUrl("/login.html")
                    .and()
                    .csrf().disable();
    
        }
    }
    ```

### ==4.10 前后端分离开发认证总结案例==

- 自定义 `UsernamePasswordAuthenticationFilter` 用于读取 `JSON` 数据完成登录功能

  ```java
  /**
   * 自定义前后端分离 Filter
   */
  
  public class LoginFilter extends UsernamePasswordAuthenticationFilter {
      @Override
      public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
          /**
           * 1. 判断是否是 post 请求
           * 2. 判断是否是 json 格式请求类型
           * 3. 从json数据中获取用户输入的用户名和密码进行验证
           */
          // 判断是否是 post 请求
          if (!request.getMethod().equals("post")) {
              throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
          }
  
          // 2. 判断是否是 json 格式请求类型
          if (request.getContentType().equalsIgnoreCase(MediaType.APPLICATION_JSON_VALUE)) {
              // 3. 从json数据中获取用户输入的用户名和密码进行验证 {"":xxx, "":xxx}
              try {
                  Map<String, String> userInfo = new ObjectMapper().readValue(request.getInputStream(), Map.class);
                  String username = userInfo.get(getUsernameParameter());
                  String password = userInfo.get(getPasswordParameter());
                  System.out.println("用户名: " + username + " 密码: " + password);
                  UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(username, password);
                  setDetails(request, authenticationToken);
                  return this.getAuthenticationManager().authenticate(authenticationToken);
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
          return super.attemptAuthentication(request, response);
      }
  }
  ```

- `Spring Security` 配置类

  ```java
  @Configuration
  public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
      @Autowired
      private MyUserDetailService myUserDetailService;
  
      @Override
      protected void configure(AuthenticationManagerBuilder builder) throws Exception {
          builder.userDetailsService(myUserDetailService);
      }
  
      @Override
      public AuthenticationManager authenticationManagerBean() throws Exception {
          return super.authenticationManagerBean();
      }
  
      // 自定义 Filter 交给工厂
      @Bean
      public LoginFilter loginFilter() throws Exception {
          LoginFilter loginFilter = new LoginFilter();
          loginFilter.setFilterProcessesUrl("/doLogin");
          loginFilter.setUsernameParameter("uname");  // 指定请求 json 的用户名 key
          loginFilter.setPasswordParameter("pwd");    // 指定请求 json 的密码  key
          loginFilter.setAuthenticationManager(authenticationManagerBean());
          loginFilter.setAuthenticationSuccessHandler(new LoginSuccessHandler());
          loginFilter.setAuthenticationFailureHandler(new LoginFailureHandler());
          return loginFilter;
      }
  
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          http.authorizeRequests()
                  .anyRequest().authenticated()
                  .and()
                  .formLogin() // 前后端分离系统需要重写formLogin的登陆实现 自定义UsernamePasswordAuthenticationFilter
                  .and()
                  .logout()
                  .logoutUrl("/logout")
                  .logoutSuccessHandler(new MyLogoutSuccessHandler())
                  .and()
                  .csrf().disable();
  
  /**
  *         http.addFilterAt();       将自定义的 filter 替换过滤器链中的某个 filter
  *         http.addFilterBefore();   将自定义的 filter 替换过滤器链中某个 filter 之前
  *         http.addFilterAfter();    将自定义的 filter 替换过滤器链中某个 filter 之后
  */
          http.addFilterAt(loginFilter(), UsernamePasswordAuthenticationFilter.class);
  
      }
  }
  ```

### 4.10 验证码功能



## 5. 密码加密



## 6.  Remember Me



## 7.  会话管理



## 8. CSRF



## 9. 跨域处理



## 10. 异常处理



## 11. 授权&授权模型



## 12. OAuth2 & Jwt



## 13. 前后端分离从零实战


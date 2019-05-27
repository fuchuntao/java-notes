### Spring Boot

#### 1.日志使用：

#### Spring Boot： slf4j + logback

```java
/**
 * 日志使用：
 *  Spring Boot： slf4j + logback
 *
 * 1、常见一个日志记录器
 * 2、哪些地方需要记录日志？
 *  业务接口中的方法
 *
 *  日志级别：
 *    优先级从小到大：trace<debug<info<warn<error
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot03LoggingApplicationTests {

    //创建一个日志记录器
    Logger logger = LoggerFactory.getLogger(SpringBoot03LoggingApplicationTests.class);

    @Test
    public void contextLoads() {
//        System.out.println();
        logger.trace("这是trace日志...");
        logger.debug("这是debug日志...");
        //Spring Boot 默认的日志根 level 是 info （root）
        logger.info("这是info日志...");
        logger.warn("这是warn日志...");
        logger.error("这是error日志...");
    }

}
```

##### 1.1、可以在全局配置文件application.properties,或者在resources文件夹下的config文件下的application.yml中修改日志root的默认级别：info

```properties
#修改日志root默认级别 : info
logging.level.root=debug
#日志输出路径
# 默认在项目所属磁盘的根目录创建 logs（根文件夹） 和 spring（子文件夹）
# spring boot 默认的日志文件名是 spring.log
logging.path=/logs/spring

# 日志文件名
# 可以指定文件的绝对路径和文件名
logging.file=D:/logs/spring-boot.log

#日志输出格式
# %d 日期时间
# %5level(%l) : 日志级别，
#   5： 表示5个字符宽度,
#        -5表示：左对齐
#       5表示：右对齐
# %thread(%t): 线程名字
# %logger: logger名字(创建logger对象的参数的全类名)
# %msg(%m): 日志消息
# %n: 换行
# 控制台
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} %5level ===> [%15t] %logger{40} : %m%n
# 文件
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} %5level ===> [%15t] %logger{40} : %m%n
```

### 2.web开发

#### 1、静态资源的两种映射

##### 1.1、webjars：以jar包的方式引入静态资源,在pom.xml文件中引入

```xml
<!--jquery-->
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.3.1-1</version>
        </dependency>
<!--bootstrap-->
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>bootstrap</artifactId>
            <version>4.1.3</version>
        </dependency>
```

```java

```



##### 1.2、"/**" 访问当前项目的任何资源，都去（静态资源的文件夹）找映射

```
"classpath:/META-INF/resources/",
"classpath:/resources/",
"classpath:/static/",
"classpath:/public/"
"/"：当前项目的根路径
```

### 3.Thymeleaf模板引擎

#### 1.引入Thymeleaf的依赖到pom.xml文件中

```xml
 <!--thymeleaf-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

#### 2.Thymeleaf使用

只要我们把HTML页面放在classpath:/templates/，thymeleaf就能自动渲染；

```html
<html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
```

### 4、国际化

#### 1、实现LocaleResolver

```java

/**
 * 自定义语言选择的 LocaleResolver
 */
public class MyLocaleResolver implements LocaleResolver {

    private Logger logger = LoggerFactory.getLogger(MyLocaleResolver.class);

    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        //获取请求参数 locale 的值
        String value = request.getParameter("locale");
        Locale locale = Locale.getDefault();
        if(!StringUtils.isEmpty(value)){
            String[] values = value.split("_");
            locale = new Locale(values[0], values[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }
}


```

```java
/**
 * web 自定义配置
 */
@Configuration
public class MyConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //登录页面
        registry.addViewController("/index.html").setViewName("login");
        registry.addViewController("/").setViewName("login");
        registry.addViewController("/main.html").setViewName("dashboard");
    }

    /**
     * 注册自定义拦截器
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //登录验证拦截器
        registry.addInterceptor(new LoginInterceptor())
                //拦截的uri
                .addPathPatterns("/**")
                //排除拦截的uri
                .excludePathPatterns("/", "/index.html", "/login", "/logout", "/asserts/**", "/webjars/**");
    }

    /**
     * 自己写的 LocaleResolver 组件加入到 Spring IOC 容器中
     * @Bean
     *   作用：将一个 组件(Java对象)纳入到 Spring IOC 容器中管理
     *      默认方法名即为 bean 在 IOC 容器中 的名字
     *      通过 name 属性可以更改 bean 的名字
     * @return
     */
    @Bean
    public LocaleResolver localeResolver(){
        return new MyLocaleResolver();
    }
}
```

2.前端页面显示a标签

```html
<a class="btn btn-sm" th:href="@{/index.html(locale='zh_CN')}">中文</a>
			<a class="btn btn-sm" th:href="@{/index.html(locale='en_US')}">English</a>
```

### 5.CRUD中页面的片段插入方法

```html
<!-- topbar -->
<nav class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0" th:fragment="topbar">
    <a class="navbar-brand col-sm-3 col-md-2 mr-0" href="#" th:text="${session.loginUser}">Company name</a>
    <input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search">
    <ul class="navbar-nav px-3">
        <li class="nav-item text-nowrap">
            <a class="nav-link" href="#" th:href="@{/logout}">Sign out</a>
        </li>
    </ul>
</nav>

<!--插入的是名字-->
<!-- topbar -->
<div th:replace="commons/bar::topbar"></div>
<!-- sidebar -->
<nav class="col-md-2 d-none d-md-block bg-light sidebar" id="sidebar">
    <div class="sidebar-sticky">
        <ul class="nav flex-column">
            <li class="nav-item">
                <a class="nav-link active" href="#" th:href="@{/main.html}"
                   th:class="${activeUri=='main'?'nav-link active':'nav-link'}">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-home">
                        <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                        <polyline points="9 22 9 12 15 12 15 22"></polyline>
                    </svg>
                    Dashboard
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="#" th:href="@{/emps}"
                   th:class="${activeUri=='emps'?'nav-link active':'nav-link'}">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-users">
                        <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
                        <circle cx="9" cy="7" r="4"></circle>
                        <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
                        <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
                    </svg>
                    Employees
                </a>
            </li>
        </ul>
    </div>
</nav>
<!--插入的是id-->
<!-- sidebar -->
<div th:replace="commons/bar::#sidebar(activeUri='main')"></div>
```

### 6.页面的时间格式转换为yyyy-MM-dd

#### 1.在页面做处理格式转换

```html
${#dates.format(emp.birth, 'yyyy-MM-dd')}

```

#### 2.在application.yml文件中的全局配置格式转换

```yml

# 静态html页面不缓存，生成环境设置 true
spring:
  thymeleaf:
    cache: false
  # 配置全局日期格式
  mvc:
    date-format: yyyy-MM-dd
# web 根路径
server:
  servlet:
    context-path: /oa
```

### 7、如何定制错误页面

#### 1.有模板引擎的情况下；error/状态码; 

【将错误页面命名为 错误状态码.html 放在模板引擎文件夹里面的error文件夹下】，发生此状态码的错误就会来到 对应的页面；
我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态
码.html）；

#### 2.定制错误信息



### 8、用sring boot支持jsp(使用外部容器，必须创建一个war项目)

#### 1.依赖相关包到pom.xml文件中

```xml 
<!--支持jsp-->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
  </dependency>

```

#### 2.在application.properties中加上jsp配置

```properties
# jsp
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

#### 3.注册Servlet三大组件【Servlet、Filter、Listener】

```java
@Configuration
public class WebConfig {

    /**
     * 注册 Servlet
     * @return
     */
    @Bean
    public ServletRegistrationBean<MyServlet> myServlet(){
        ServletRegistrationBean<MyServlet> servletServletRegistrationBean =
                new ServletRegistrationBean<>(new MyServlet(), "/myServlet");
        return  servletServletRegistrationBean;
    }

    /**
     * 注册 Filter
     * @return
     */
    @Bean
        public FilterRegistrationBean<MyFilter> myFilter(){
            FilterRegistrationBean<MyFilter> filterFilterRegistrationBean =
                    new FilterRegistrationBean<>(new MyFilter(), myServlet());

        return  filterFilterRegistrationBean;
    }

    /**
     * 注册 Servlet Listener
     * ServletListenerRegistrationBean
     */

}

```



### 7、spring boot和mybatis的整合

#### 1.导入相关的starter到pom.xml文件中(用druid数据源)

```xml
 		<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>
 <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.12</version>
        </dependency>
```

##### 1.1、导入Druid数据源

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return new DruidDataSource();
    }

    //配置 Druid 监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();

        //druid 监控管理后台的账号和密码
        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
//        initParams.put("deny","192.168.15.21");

        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return  bean;
    }

}

```



#### 2.在全局配置中配置数据源属性 

```yml
# 配置数据源属性:
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis
    username: root
    password: 123456
# mybatis 的配置文件
mybatis:
  #主配置文件位置
  config-location: classpath:mybatis/mybatis-config.xml
  #mapper映射文件位置
  mapper-locations: classpath:mybatis/mapper/*Mapper.xml
```

#### 3.编写Mapp接口，使用@Mapper注解加到类上

```java
@Mapper
public interface EmployeeMapper {

//    @Select("select * from tb_employee where id = #{id}")
    public Employee getEmployeeById(Integer id);

}
```

##### 3.1、mybatis-config.xml主配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    
    <settings>
        <!-- 开启驼峰命名规则 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

##### 3.2、*Mapper.xml映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.xinyan.springboot.mapper.EmployeeMapper">

   <select id="getEmployeeById" resultType="com.xinyan.springboot.domain.Employee">
       select *
       from tb_employee
       where id = #{id}
   </select>
</mapper>
```


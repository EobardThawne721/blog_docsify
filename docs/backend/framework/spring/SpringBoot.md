# Spring Boot

## 一. Spring Boot相关配置

### 1.1 创建Spring Boot项目

>  1.首先选择idea项目，创建新的模块，选择 **Spring Initializr**，然后下一步	

==注意：1.如果要创建Eclipse项目需要下载STS的插件，选择"帮助=>EclipseMarketPlace" 搜索sts即可==

==2.阿里云创建springboot项目路径：== https://start.aliyun.com/  



> 2. 填写相关的值，Packaging可以选择jar，然后下一步
>
>    注意：如果打包成war包，会多出一个ServletInitializer类，不用管它



> 3. 选择Web下的Spring Web，版本选择**2.5.6(SNAPSHOT)**，然后一直下一步即可



> 4. 删除其它无用的文件，如.gitignore，mvnw .cmd .mvn等



> 5.创建一个SpringMVC控制器

```JAVA
@RestController
public class UserController {

    @RequestMapping("/index")
    public String index(){
        return "hello world";
    }
}
```



> 6. 运行项目下的Spring Boot启动器(项目名Application实体类)即可



> 7. 访问路径localhost:8080/index



==注意：1.Spring Boot启动器存放的位置可以放在controller同一个包下或controller包的上一级，**但是不能放在controller的子包和平级(如service包或dao包)下**==

​			==2.无论是整合了什么，都是从Spring Boot启动类开始运行==



### 1.2 修改端口及访问路径

​		在resources文件夹下的application.properties配置即可



#### 1.2.1 修改端口

​			当tomcat的配置端口不一致时，可以通过下面配置来修改tomcat的端口

```properties
#修改访问端口号
server.port=8080
```



#### 1.2.2 修改项目访问路径(了解)

```properties
#修改项目访问路径,默认为 /
server.servlet.context-path=/springboot
```

> ​	修改之前访问：localhost:8080/index
>
> ​	修改之后访问:   localshot:8080/springboot/index



### 1.3 访问静态资源

​	在resources/static文件夹下创建静态资源，如html，css，js，img等，在浏览器直接访问**localhost:8080/index.jsp** 即可

```properties
#配置静态资源访问路径,低版本的springboot需要手动配置路径,eg：2.3.6版本
spring.mvc.static-path-pattern=/static/**
```

==注意：resources/templates文件夹是模板文件，默认视图支持是Thymeleaf，Spring Boot不能单独支持jsp文件需要单独配置==



### 1.4 热部署

​		不关闭服务器的情况下更新代码会自动更新到服务器中，不需要手动停掉服务器更新，速度比手动关闭更快。**建议在开发中使用，项目上线后关闭！会存在缓存等问题**

> ​	第一步：导入依赖

```xml-dtd
   <!--Spring 官方提供的热部署插件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
```

> ​	第二步：选择setting->build->compile->Build Project Automatically

> ​	第三步：shift+ctrl+alt +/ 选择第一个registry,找到compiler.automake.allow.when.app.running选项勾选

==**注意：在Idea中ctrl+F9可以快速热部署**==



### 1.5 打包成jar包部署

> ​	第一步：导入插件

```xml-dtd
     <!--打包jar包插件-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
```

> ​	第二步：找到右边的maven按钮 -> 找到当前项目的Lifecycle ->  双击package

> ​	第三步：找到当前项目位置，运行cmd 执行命令

```
java -jar 打包的jar名
```

`如果想把打包的项目更改配置文件的值,如(更改端口号),则使用下列方式`

```
java -jar 打包的jar名 --server.port=8081 --spring.profiles.active=test 
```

> 注意：`--`后面的`server.port=8081`是按照开发中application.properties里面的配置来对照设置的



### 1.6 yaml配置文件的使用

​		如果配置文件不适用application.properties 可以换成application.yml文件

```yaml
server:
  port: 8081
  servlet:
    context-path: /springboot
```

> ​	使用注意：
>
> ​						1.在yml中，如果 **: 后面**有值得话，必须加一个空格
>
> ​						2.以空格的缩进来控制层级关系，最左边的是顶层，空格对齐就表示属于同一层的元素

==注意：1.在Spring Boot中，**优先读取application.yml的配置文件**，最后才是application.properties配置文件==

​			==2.**如果在yml文件中有特殊符号如：% &，用单引号包起来即可**==



### 1.7 多环境切换

#### 1.7.1 切换配置环境

`1.在resources目录下另外创建两个配置文件：一个用于开发环境，一个用于生产环境`

```
开发环境：
		application-dev.properties
生产环境
		application-prod.properties
```

==注意：如果要用不同的环境切换，配置文件只能命名为 **application-自定义.properties或application-自定义.yml** 这种形式==



`2.在application.properties或application.yml文件中指定环境`

```yaml
spring:
  profiles:
    active: dev
```

或者

```properties
spring.profiles.active=dev
```



#### 1.7.2 获取项目运行参数

```java
@Configuration
public class ProfileConfig {

    @Resource
    private ApplicationContext context;
    
    @Autowired
    private ServletWebServerApplicationContext webServerAppCtxt;

    //获取环境
    public String getActiveProfile() {
        return context.getEnvironment().getActiveProfiles()[0];	
    }
    
    //获取端口号
    public void getPort(){
        int port = webServerAppCtxt.getWebServer().getPort();
        System.err.println("端口号为："+port+" ,进来服务！");       
    }
}
```



### 1.8 配置文件自定义属性注入（重要）

#### 1.8.1 自定义yml文件注入

`1.在application.yml或application.properties配置值`

```yaml
role:
    roleId: 1
    role-name: guanliyyuan
```



`2.在类中注入`

```JAVA
@Component
@Data
@ConfigurationProperties(prefix = "role")	//表示前缀为role,注入的时候会自动加上role.属性名
public class Role {

    private Integer roleId;

    private String roleName;
}
```



**可选：**

> ​	pom文件中可以选择是否加入yml的提示依赖：加入了依赖可以在yml文件中有提示
>
> ```xml-dtd
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-configuration-processor</artifactId>
>     <optional>true</optional>
> </dependency>
> ```



#### 1.8.2 自定义properties文件注入

`1.创建role.properties属性文件`

```properties
role.roleId=2
role.roleName=xioawangb
```



`2.在类中注入`

```JAVA
@Component
@Data
@ConfigurationProperties(prefix = "role")		//指定配置文件的前缀
@PropertySource("classpath:role.properties")	//指定自定义的配置文件
public class Role {
 
    private Integer roleId;

    private String roleName;
}
```



### 1.9 配置文件的注意

​			**1. 若resources文件夹下的配置文件如果存在很多**，可以在**resources文件夹下建一个config文件夹**，然后把所有的**配置文件放入config文件中**，Spring Boot也是**可以读取到的**，并且读取配置文件的**优先级还比直接放在resources文件夹下高**



​			**2. 如果是properties配置文件中有数组形式的参数，则通过数组方式配；如果是yml配置文件中有数组形式的参数，则通过-方式配**

```
//properties形式
spring.cloud.nacos.config.ext-config[0].data-id=datasource.yml
spring.cloud.nacos.config.ext-config[1].data-id=mybatisplus.yml
spring.cloud.nacos.config.ext-config[2].data-id=others.yml

//yml形式
spring:
  cloud:
    gateway:
      routes:
        - id: test1
          uri: http://www.baidu.com
          predicates:
            - Query=url,baidu
        - id: test2
          uri: http://www.youku.com
          predicates:
            - Query=url,youku
```





### 1.10 使用外部tomcat容器

​				若想要让Spring Boot不用内置的tomcat容器运行，用本机的tomcat容器运行



`步骤1:打包为war包`

```xml-dtd
 <packaging>war</packaging>
```



`步骤2:排除打包时候引入Spring Boot内置的tomcat容器`

```xml-dtd
<!--让tomcat不参与打包部署-->	
	<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
```



`步骤3:让外部容器运行时关联运行Spring Boot启动器`

```JAVA
public class TomcatStartUp extends SpringBootServletInitializer{
    
      @Override
      protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
          //传入Spring Boot启动器类class
          return builder.sources(SpringBootApplication.class);
    }
}
```



`步骤4:idea/eclipse 配置外部tomcat，并将项目从tomcat运行即可`



### 1.11 自定义配置类

​				使用@Configuration注解标志当前类为配置类，通过@Bean注解配置其它类，等同于之前在applicationContext.xml文件中配置其它Bean。

`注意:@Bean标识的方法名就为该类的id名`



### 1.12 单元测试

```JAVA
@SpringBootTest
class SpringbootApplicationTests {

    @Test
    void contextLoads() {
        
    }
```

> 注意：@Test注解导包应该为`import org.junit.jupiter.api.Test;`否则单元测试会出现NullPointerException





### 1.13 排除项目自动装配类

> **在微服务开发中，对于ES搜索服务模块不需要数据库的自动装配，需要取消数据库连接的自动装配**

```java
//排除数据库的自动装配
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableFeignClients 
public class WeGouServiceSearchApplication {
    public static void main(String[] args) {
        SpringApplication.run(WeGouServiceSearchApplication.class,args);
    }
}
```







---

## 二. 整合其它

### 2.1 整合原生Servlet

#### 2.1.1 方式1：通过注解(常用)

```JAVA
//创建Servlet
@WebServlet("/indexServlet")
public class IndexServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("调用原生的servlet");
    }

}
```

```JAVA
//Spring Boot启动器扫描
@SpringBootApplication
@ServletComponentScan	//扫描原生Servlet相关的
public class SpringbootStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootStudyApplication.class, args);
    }

}
```



#### 2.1.2 方式2：通过编写方法(了解)

```JAVA
//创建Servlet
@WebServlet("/indexServlet")
public class IndexServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("调用原生的servlet");
    }

}
```

```JAVA
//Spring Boot启动器扫描
@SpringBootApplication
public class SpringbootStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootStudyApplication.class, args);
    }

    @Bean
    public ServletRegistrationBean geServletRegistrationBean(){
        ServletRegistrationBean registrationBean=new ServletRegistrationBean(new IndexServlet());
        //注册自定义的servlet路径
        registrationBean.addUrlMappings("/indexServlet");
        return registrationBean;
    }
}
```



### 2.2 整合过滤器

#### 2.2.1 执行时机

> **请求 => 过滤器 => 拦截器 => controller接口**



#### 2.2.2 使用原生过滤器（了解）

```JAVA
@WebFilter(filterName = "myFilter",urlPatterns = "/indexServlet")
//@WebFilter(filterName = "myFilter",urlPatterns = {"/index",".action"}) 过滤多个请求路径
public class IndexFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("过滤器前执行");
        filterChain.doFilter(servletRequest,servletResponse);
        System.out.println("过滤器放行");
    }
    
}
```

```JAVA
@SpringBootApplication
@ServletComponentScan	//扫描原生Servlet相关的
public class SpringbootStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootStudyApplication.class, args);
    }

}
```

> ​	执行：localhost:8080/indexServlet
>
> ​			过滤器前执行
>
> ​			调用原生的servlet
>
> ​			过滤器放行



#### 2.2.3 全局过滤器（重要）

> **全局过滤器一般用于全局日志处理、统一请求头处理、跨域CORS、鉴权、黑名单等**
>
> * **OncePerRequestFilter：一次请求，只过滤一遍，绝不重复过滤**
>
> * **重复过滤情况：转发、异步请求、错误页面跳转**

```java
@Slf4j
@Component
public class GlobalRequestFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {

        log.info("全局过滤器：请求URL = {}", request.getRequestURI());
        
        // 如果是接口文档、图标、监控,直接放行
         if (requestUrl.startsWith("/doc") || requestUrl.startsWith("/swagger") || requestUrl.startsWith("/favicon") || requestUrl.startsWith("/v3/api-docs") || requestUrl.startsWith("/actuator")) {
            // 继续过滤链，让请求体被读取
            chain.doFilter(request, response);
            return;
        }
        
        
        try {
            // 放行，让请求继续走到 拦截器 → Controller 的逻辑
            filterChain.doFilter(request, response);
        } finally {
 //当try中走完之后，最终可以返回到finally块执行日志打印：请求方法、url、请求参数、所有的header参数
            log.info("全局过滤器：请求处理完成");
        }
    }
}
```







### 2.3 整合原生监听器

```JAVA
@WebListener
public class IndexListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("监听器初始化");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("监听器销毁");
    }
}
```

```JAVA
@SpringBootApplication
@ServletComponentScan	//扫描原生Servlet相关的
public class SpringbootStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootStudyApplication.class, args);
    }

}
```

> ​	执行：localhost:8080/indexServlet
>
> ​					监听器初始化
>
> ​					过滤器前执行
>
> ​					调用原生的servlet	
>
> ​					过滤器放行
>
> ​					监听器销毁(销毁容器的时候触发)



### 2.4 整合JSP

> 前期环境准备：
>
> ​		1.项目结构创建webapp文件，Idea默认不会创建webapp文件夹
>
> ​					a.点击项目结构=>模块=>选择web=>选择Development右边的 +，路径一定要变成类似的 ......\src\main\webapp\WEB-INF\web.xml
>
> ​					b.选择下面web resource右边的+，路径一定要变成类似的 .....\src\main\webapp，然后应用确定即可
>
> 
>
> ​		2.修改运行环境配置
>
> ​				**点击idea上方的运行/配置，选择Environment=>工作目录=>选择MODULE_WORKING_DIR应用即可**



```xml-dtd
<!--pom文件必须引用依赖-->
        <!--jstl-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
        </dependency>
        <!-- jasper的支持-->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
        </dependency>
```

```properties
#application.properties设置
#设置前缀
spring.mvc.view.prefix=/WEB-INF/jsp/
#设置后缀
spring.mvc.view.suffix=.jsp
```

```JAVA
//控制器
@Controller
public class ListController {

    @RequestMapping("/getList")
    public String userList(Model model){
        List userList=new ArrayList();
        userList.add(new User(1,"zs1","man"));
        userList.add(new User(2,"zs2","man"));
        userList.add(new User(3,"zs3","man"));
        userList.add(new User(4,"zs4","man"));
        model.addAttribute("userList",userList);
        return "userList";
    }
}
```

```JSP
<!--userList.jsp页面-->
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<c:forEach items="${userList}" var="user">
        <c:out value="${user.id}"/> <br>
        <c:out value="${user.name}"/> <br>
        <c:out value="${user.sex}"/>
        <hr>
    </c:forEach>
```



### 2.5 整合freemarker

​			Spring Boot整合freemarker要求必须将模板放在src/main/resources/templates下，后缀名为 .ftl

```xml-dtd
<!--pom.xml导入依赖-->
      <!--导入freemarker依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>
```

```JAVA
//控制器
@Controller
public class ListController {

    @RequestMapping("/getListByFreemarker")
    public String getListByFreemarker(Model model){
        List userList=new ArrayList();
        userList.add(new User(1,"zs1","man"));
        userList.add(new User(2,"zs2","man"));
        userList.add(new User(3,"zs3","man"));
        userList.add(new User(4,"zs4","man"));
        model.addAttribute("userList",userList);
        return "list";
    }
}
```

```properties
#application.properties配置

#设置freemarker的加载路径
spring.freemarker.template-loader-path=classpath:/templates
#Spring Boot 2.0版本以上后缀名默认为.ftlh,需要改
#获取直接把后缀改为.html也可以，最好改成.html
spring.freemarker.suffix=.ftl
```

```
<!--在src/main/resources/templates下创建list.ftl文件-->
<html>
<head>
    <title>Title</title>
</head>
<body>
<#list userList as user>
    ${user.id} <br>
    ${user.name} <br>
    ${user.sex}
    <hr>
</#list>
</body>
</html>
```

==注意：1.创建模板文件可以先创建html类型，然后去掉头文件，然后修改后缀为.ftl==

​			==2.里面遍历request作用域的集合用 **<#list 作用域键名 as 自定义变量>  ${自定义变量.属性名}  </#list>**==



### 2.6 整合thymeleaf(重点掌握)

#### 2.6.1 简单入手

```xml-dtd
<!--pom.xml导入依赖-->
	 <!--导入thymeleaf的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

```properties
#application.properties配置

#设置thymeleaf的前缀
spring.thymeleaf.prefix=classpath:/templates/
#设置thymeleaf的后缀
spring.thymeleaf.suffix=.html
#设置thymeleaf的模式
spring.thymeleaf.mode=HTML5
#设置thymeleaf的编码格式
spring.thymeleaf.encoding=UTF-8
```

```java
//控制器
@Controller
public class ListController {

    @RequestMapping("/thymeleaf")
    public String index(Model model){
        model.addAttribute("msg","welcome to thymeleaf");
        return "thymeleaf";
    }
}
```

```HTML
<!--在src/main/resources/templates下创建thymeleaf.html文件-->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"> <!--加上引用-->
<head>
    <meta charset="UTF-8">
    <title>Show User</title>
</head>
<body>
    <span th:text="${msg}" /> 	<!--th:text="${el表达式}" 输出作用域的值-->
</body>
</html>
```

==注意：必须将模板放在src/main/resources/templates下，该目录下是受保护的，不能直接访问==



#### 2.6.2 #strings字符串工具

> ```html
> 输出作用域：		<span th:text="${msg}" /> <br>
> 自定义属性输出： 	<input type="text" th:value="'嘻嘻嘻'"> 
> 
> 判断作用域的对象是否为空：<span th:text="${#strings.isEmpty(msg)}" /> 
> 
> 判断作用域是否包含字符(不区分大小写)：
> <span th:text="${#strings.containsIgnoreCase(msg,'LEAF')}"/>
> 
> 截取字符串:从0-8截取
> ${#strings.abbreviate(special.skuName,8)}
> ```

==<font color="bright">注意：#strings有许多API可以调用，它拥有String的一系列API，不能省略strings前面的#</font>==



#### 2.6.3 #dates 日期工具

> ```html
> 输出日期(按照浏览器的日期输出格式):<span th:text="${#dates.format(date)}" /> 
> 
> 输出日期自定义:<span th:text="${#dates.format(date,'yyyy年MM月dd日 HH:mm:ss')}" /> 
> 
> 输出年：<span th:text="${#dates.year(date)}" /> 
> 输出月：<span th:text="${#dates.month(date)}" />
> 输出日：<span th:text="${#dates.day(date)}" />
> ```

==<font color="bright">注意：#dates有许多API可以调用，它拥有Date的一系列API，不能省略dates前面的#</font>==



#### 2.6.4 条件判断与swtich

> ```html
> 判断: 
>    <span th:if="${sex}==0">男</span>
>       <span th:if="${sex}==1">女</span>
> ```



> ```html
>   swtich:
> <select th:switch="${function}">
>     	<option th:case="1">管理员</option>
>     	<option th:case="2">其它</option>
> </select>
> ```



#### 2.6.5 数字格式化

> ```
> ${ #numbers.formatDecimal(价格变量,0,'COMMA',2,'POINT')}
> ```



#### 2.6.6 遍历集合对象

##### 2.6.6.1 遍历List集合

> ```html
> 	遍历集合
> <table th:each="user :${userList}">
>     <tr>
>         <td th:text="${user.id}" />
>         <td th:text="${user.name}" />
>         <td th:text="${user.sex}" />
>     </tr>
> </table>
> ```

==注意：遍历集合类似于foreach的形式==



##### 2.6.6.2 遍历Map集合

```HTML
    遍历map
    <table th:each="map : ${mapList}">
        <tr>
            <td th:each="entry:${map}" th:text="${entry.value.id}" />
            <td th:each="entry:${map}" th:text="${entry.value.name}" />
            <td th:each="entry:${map}" th:text="${entry.value.sex}" />
        </tr>
    </table>
```

==注意：输出map的时候只能写为${entry.value.属性名}==



#### 2.6.7输出作用域

| 语法         | 含义                                   |
| ------------ | -------------------------------------- |
| th:text      | 显示文本，会转义字符                   |
| th:value     | 在input标签中输出值                    |
| th:utext     | 将字符串标签显示为真正标签             |
| [[el表达式]] | 直接输出EL表达式的值，不借助于任何标签 |

```HTML
  <!--直接输出requestValue的值,不借助标签,不会输出两个[]-->
  [[${requestValue}]]	
  <!--借助标签输出-->
  request：		<input th:value="${requestValue}" />
  session:		<span th:text="${session.sessionValue}" /> <br>
  application:	<span th:text="${application.applicationValue}" />

  <!--输出标签-->
	<!--tags作用域的值为:<a href="#">测试</a>-->
 <div th:utext="${tags}">
</div>
```



#### 2.6.8 url路径

​						循环遍历集合的时候(如table遍历request作用域的集合)，`传统的html表达式中不能拿到EL表达式的值`，但是通过Thymeleaf的标签就可以拿到EL表达式的值

```html
	访问路径	<a th:href="@{/user/index}">访问</a>
	带参数路径  <a th:href="@{/Update(id=${user.id})}">修改</a>
	<!--上面的路径等同于:  /Update?id=EL表达式的id值  -->

	Rest风格: <a th:href="@{'/product/show/'+${product.id}}">产品</a>
	<!--上面的路径等同于:  product/show/12"  -->
```



#### 2.6.9 公共页面包含(重要)

​			网站中许多地方都是公共的，例如网站的头部和底部以及左侧菜单导航栏内容，将这些公共的代码放在一个页面中进行引用。

| 语法        | 含义                     |
| ----------- | ------------------------ |
| th:fragment | 定义代码片段模板         |
| th:replace  | 以替换的方式引用代码片段 |
| th:insert   | 以插入的方式引用代码片段 |



`第一步:定义公共页面`

```HTML
	<!--th:fragment方式-->   
	<h1 th:fragment="head">
            这是网页的头部
        </h1>
	<!--id方式--> 
   <footer id="footer">
            这是尾部
   </footer>
```



`第二步:引用公共部分`

```HTML
  		<!--通过th:fragment方式-->
    <div th:replace="public/top.html :: head"></div>

        <!--通过id的方式(建议使用)-->
    <div th:replace="public/top.html :: #footer"></div>

	<div th:insert="public/top.html :: #footer"></div>
```





### 2.7 整合Mybatis

> ​	第一步：引入依赖

```xml-dtd
<!--pom.xml文件-->

<!--导入thymeleaf的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
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
            <version>2.0.1</version>
        </dependency>
        <!--druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.5</version>
        </dependency>

<!--省略其它-->

 <!--解决maven项目中根路径编译不到src/main文件下的xml文件-->
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>
    </build>
```

> ​	第二步：配置整合属性

```properties
#application.properties

#整合mybatis驱动包
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#整合mybatis的url
spring.datasource.url=jdbc:mysql://localhost:3306/gxp
#整合mybatis账号
spring.datasource.username=root
#整合mybatis密码
spring.datasource.password=123456
#整合mybatis的别名
mybatis.type-aliases-package=com.eobard.entity
#整合mybatis数据连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
#打印sql语句
logging.level.com.eobard.dao=debug
```

> ​	第三步：编写entity、dao、service、controller相应代码

> ​	第四步：Spring Boot启动器扫描Mapper.xml文件

```JAVA
@SpringBootApplication
@MapperScan("com.eobard.dao")
public class SpringbootStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootStudyApplication.class, args);
    }
}
```

==注意：1.整合Mybatis的时候如果把映射文件放在src/main/dao下，**一定要在pom.xml文件的build设置，否则会出现not found xxx.xxx.xxx.xxx() statement的异常**，或者在全局配置文件去扫描resource文件夹下的 .xml 映射文件==

```java
//扫描com.wegou包下面所有的mapper包
@MapperScan("com.wegou.*.mapper")
```



> **当配置文件在resource文件夹下的时候，加载配置文件所在的位置**
> mybatis.mapper-locations=classpath:mapper/*.xml



#### 2.7.1 SQL查询扩展

1. DB中日期类型字段查询结果自定义输出格式

```sql
select DATE_FORMAT(日期类型列名,'%Y年%m月%d日 %H:%m:%s') as '时间' from 表 
```

> DATE_FORMAT(日期类型列名,'自定义输出格式')



2. 博客上一篇、下一篇的实现

```SQL
#上一篇
select * from 表名 where id< 20  order by id desc limit 1
#下一篇
select * from 表名 where id> 20  order by id asc limit 1
```

> 注意：在mapper映射文件中不能直接写`>和<`，要写成`&lt和&gt;`;



### 2.8 整合Spring Data JPA

#### 2.8.1 简单入手

> ​	第一步：Pom.xml导入依赖

```xml-dtd
  <!--jpa-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <!--druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.5</version>
        </dependency>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>	
```



> 第二步：配置application.properties属性

```properties
#整合驱动包
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#整合url
spring.datasource.url=jdbc:mysql://localhost:3306/gxp
#整合账号
spring.datasource.username=root
#整合密码
spring.datasource.password=123456
#整合数据连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

#JPA正向工程
spring.jpa.hibernate.ddl-auto=update
#JPA显示sql语句
spring.jpa.show-sql=true
```



> 第三步：创建实体类

```JAVA
@Entity
@Table(name = "user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private String sex;
	
    //省略getter、setter
}
```

==注意：主键和类名必须映射；若其它属性与数据库列名一致，可以省略@Column映射；若不一样需要手动映射；若不想要映射某个字段，则需要使用@Transient==



> 第四步：创建UserDao接口

```java
//必须实现接口
//泛型参数1:操作的实体类, 泛型参数2:操作实体类主键的类型
public interface UserDao extends JpaRepository<User,Integer> {

}
```



> ​	第五步：测试，使用内置API

```JAVA
 	@Resource
    private UserDao userDao;

    @Test
    void addUser() {
        userDao.save(new User(7,"小刘","女"));
    }
```



#### 2.8.2 原生SQL、HQL

```JAVA
//必须实现接口
//泛型参数1:操作的实体类, 泛型参数2:主键的类型
public interface UserRepository extends JpaRepository<User,Integer> {

    //使用hql语句
    @Query("from User where name=:name")
    List<User> findUserByHQL(String name);

    //原生sql
    @Query(value = "select * from user where name like %:name%",nativeQuery=true)
    List<User> findUserBySQL(String name);

    //增删改需要加上@Modifying注解
    //原生SQL
    @Modifying
    @Query(value = "delete from user where  id=:id",nativeQuery = true)
    int deleteById(Integer id);
    
    @Modifying
    @Query(value = "update user set name=:uname where id=:uid",nativeQuery = true)
    int update(String uname,Integer uid);
}
```



> 测试

```JAVA
 @Autowired
 private UserRepository userRepository;  

	@Test
    void find() {
      userRepository.findUserByHQL("张三").forEach(user -> System.out.println(user.getName()));
    }

    @Test
    void find2() {
        userRepository.findUserBySQL("张").forEach(user -> System.out.println(user.getName()));
    }

    @Test
    @Transactional
    @Rollback(false)
    void update() {
        userRepository.update("张七",6);
    }

```

==注意：**增删改的时候要手动加上事务并且还要把回滚设置为false**==



#### 2.8.3 一对一

​			`1.数据库表设计`

> identity表
>
> ​					identity_id  主键
>
> ​					identity_num
>
>  user表：
>
> ​						id 主键
>
> ​						name
>
> ​						sex
>
> ​						identity_id  	外键

​			`2.实体类`

```JAVA
@Entity
@Table(name = "user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private String sex;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name="identityId")				//关联外键
    private Identity identity;

    //省略getter,setter
}
```

```JAVA
@Entity
@Table
public class Identity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer identityId;
    private String identityNum;

     //省略getter,setter
}
```

​			`3.User接口`

```JAVA
public interface UserDao extends JpaRepository<User,Integer> {

}
```

​			`4.测试`

```JAVA
    @Resource
	private UserDao userDao;

	//增加
	@Test
    void addOne2One(){
        User u=new User(null,"gxp","male");
        
        Identity identity=new Identity();
        identity.setIdentityNum("123456789");
        
        u.setIdentity(identity);	//关联一对一
        userDao.save(u);			//保存
    }

	//查询
    @Test
    void queryOne2One(){
        Optional<User> user = userCrudRepository.findById(5);
        System.out.println(user+" "+user.get().getIdentity());
    }
```





### 2.9 整合Swagger 2

#### 2.9.1 简单入手

​	`1.pom.xml导入依赖`

```xml-dtd
  <!--swagger2-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!--swagger2 ui-->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
```



`2. 配置Swagger2`

```JAVA
@Configuration
@EnableSwagger2
public class Swagger2Config {


    @Bean
    public Docket docket(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                //是否使用swagger2：项目上线要关闭,开发打开
                .enable(true)
            	//设置右上角下拉框的组名
            	.groupName("Eobard-Thwane")
                .select()
                //指定扫描接口的包
                .apis(RequestHandlerSelectors.basePackage("com.eobard.controller"))
                //可以根据url路径设置哪些请求加入文档，忽略哪些请求
                //PathSelectors.ant("/user/**")：表示把请求开头为 /user/  的任意方法加入文档
                .paths(PathSelectors.any())
                .build();
    }

    //配置swagger2的ApiInfo信息
    private ApiInfo apiInfo(){
        //作者信息
        Contact contact=new Contact("Eobard-Thawne","","2209473452@qq.com");
        return new ApiInfo(
                "Eobard-Thwane's Rest API document",
                "this website is about to show the Rest api",
                "v1.0",
                "",
                contact,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList()
        );
    }
}
```



`3.编写控制器`

```JAVA
@RestController
public class IndexController {

    @GetMapping("/user")
    public User user(){ 
        //只要我们的接口中,返回值中存在实体类,它就会被扫描到swagger中
        return  new User(1,"ZJL","男");
    }
}
```



`4.访问页面`

```
可以看到所有的接口信息和实体类信息

			http://localhost:8080/swagger-ui.html 
```



#### 2.9.2 常用注解

> ​	@ApiModel("xxx实体类") 																						//用于实体类的类上面
>
> ​	@ApiModelProperty("xxx属性")																			//用于实体类的属性上面
>
> ​	
>
> ​	@Api(tags = "XXXController",description = "xxx的控制器")								//用于控制器的实体类上面
>
> ​	@ApiOperation(value = "查询用户方法",notes = "根据id查询一个用户")			//用于控制器的方法上面
>
> ​	
>
> ​	@ApiIgnore							//忽略类，接口，方法，参数生成swagger文档，同2.9.3章节自定义注解					



==注意：**1.如果在实体类上加入了Swagger2的注解，如果控制器没有返回该实体类的方法，则swagger-ui.html页面中不会显示该实体类的信息**==

​			==2.不加入注解，swagger-ui.html页面会显示英文信息==





#### 2.9.3 自定义注解排除接口(了解)

`1.创建自定义注解`

```JAVA
/**
 * @Target 常用取值：
 *                      ElementType.FIELD       用于字段上
 *                      ElementType.METHOD      用于方法上
 *                      ElementType.PARAMETER   用于方法形参上
 *                      ElementType.TYPE        用于类型上:类或接口
 *
 * @Retention 常用取值：
 *                      RetentionPolicy.CLASS   字节码时生效
 *                      RetentionPolicy.RUNTIME  运行时生效
 */

//自定义注解排除接口：排除不生成swagger2文档的接口
@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface ExcludeFile {

    String value() default "";
}
```



`2.配置自定义注解`

```JAVA
@Configuration
@EnableSwagger2
public class Swagger2Config {


    @Bean
    public Docket docket(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .groupName("Eobard-Thwane")
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.eobard.controller"))
                //排除自定义注解不生成swagger2文档的接口
                .apis(Predicates.not(RequestHandlerSelectors.withMethodAnnotation(ExcludeFile.class)))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo(){
        Contact contact=new Contact("Eobard-Thawne","","2209473452@qq.com");
        return new ApiInfo(
                "Eobard-Thwane's Rest API document",
                "this website is about to show the Rest api",
                "v1.0",
                "",
                contact,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList()
        );
    }
}
```



`3.使用`

```JAVA
@RestController
public class IndexController {

    @GetMapping("/user")
    public User user(){ 
        return  new User(1,"hlw","男");
    }

    @ExcludeFile			//该接口将不会出现在swagger-ui.html中
    @GetMapping("/role")
    public Role role(Integer id){    
        Role role = new Role();
        role.setRoleId(id);
        return role ;
    }
}
```







#### 2.9.4 生成离线文档

`1.pom.xml导入插件`

```xml-dtd
 			<plugin>
                <groupId>io.github.swagger2markup</groupId>
                <artifactId>swagger2markup-maven-plugin</artifactId>
                <version>1.3.1</version>
                <configuration>
                    <!-- api-docs访问url -->
                    <swaggerInput>http://localhost:8080/v2/api-docs</swaggerInput>
                    <!-- 生成为单个文档，输出路径 -->
                    <outputFile>src/api/md/swagger2-api</outputFile>
                    <config>
                        <!--word文档-->
                        <swagger2markup.markupLanguage>ASCIIDOC</swagger2markup.markupLanguage>
                        <!-- markdown格式文档 -->
<!--                        <swagger2markup.markupLanguage>MARKDOWN</swagger2markup.markupLanguage>-->
                        <!-- 设置输出语言为中文 -->
                        <swagger2markup.outputLanguage>ZH</swagger2markup.outputLanguage>
                        <swagger2markup.pathsGroupedBy>TAGS</swagger2markup.pathsGroupedBy>
                    </config>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.asciidoctor</groupId>
                <artifactId>asciidoctor-maven-plugin</artifactId>
                <version>1.5.6</version>
                <configuration>
                    <!-- asciidoc文档输入路径 -->
                    <sourceDirectory>src/api/md</sourceDirectory>
                    <!-- html文档输出路径 -->
                    <outputDirectory>src/api/html</outputDirectory>
                    <backend>html</backend>
                    <sourceHighlighter>coderay</sourceHighlighter>
                    <!-- html文档格式参数 -->
                    <attributes>
                        <doctype>book</doctype>
                        <toc>left</toc>
                        <toclevels>3</toclevels>
                        <numbered></numbered>
                        <hardbreaks></hardbreaks>
                        <sectlinks></sectlinks>
                        <sectanchors></sectanchors>
                    </attributes>
                </configuration>
            </plugin>
```



`2.在idea的右边maven按钮点击Plugins`

> ​	点击Swagger2markup菜单下的swagger2markup：convertSwagger2markup
>
> ​     再次点击asciidoctor菜单下的asciidoctor：process-asciidoc

==注意：在导出离线文档的时候，首先一定要让Spring Boot启动器运行起来，否则会读不到swagger-ui.html的数据==





### 2.10 整合日志框架		

> #### Spring Boot默认使用的是logback的日志框架	
>
> ##### 			1.**日志的五个级别 TRACE < DEBUG <  INFO <  WARN  <  ERROR**只输出当前级别及其以上
>
> ##### 			2.后期开发可以通过AOP实现记录日志的功能保存在文件和数据库中



#### 2.10.1 输出在控制台

`1.声明日志记录器`

```JAVA
 private static Logger logger = LoggerFactory.getLogger(UserService.class);
```

`2.输出在控制台`

```JAVA
	logger.debug("debug级别");
    logger.info("info级别");
    logger.error("error级别");
```

==注意：我们应该用日志的方式来替换传统使用System.out.println的方式==



#### 2.10.2 输出在文件

> 在application.yml文件或application.properties文件配置

```yaml
logging:
  level:
    com:
      eobard: debug #表示只输出com.eobard包下面debug级别及其以上的
  file:
    path: logs       #表示在当前项目的logs文件夹下生成日志文件或直接写路径
#   name: logs.log   #表示在当前项目的根路径下创建logs.log日志文件
```

==注意：推荐使用path的方式，这样可以让日志单独存放在项目的logs文件夹下==



#### 2.10.3 日志的归并

> ​	在application.yml文件或application.properties文件配置

```yaml
logging:
  level:
    com:
      eobard: debug		 #表示只输出com.eobard包下面debug级别及其以上的
  file:
    path: logs       	 #表示在当前项目的logs文件夹下生成日志文件
  logback:
    rollingpolicy:
      max-file-size: 10MB  # 最大日志文件大小,当日志总大小超过这个限度的时候自动将前面的日志放在压缩包中
      max-history: 7       #日志文件保留天数
```



#### 2.10.4 自定义某个包的日志输出级别

```properties
logging.level.com.eobard.dao=debug
```



### 2.11 整合缓存

> **在没有缓存的情况下，虽然数据库的数据没有发生变化，但是每一次查询操作都会执行一次SQL语句访问数据库。随着时间的积累，用户量不断增加，缓存的使用可以避免服务器宕机。**
>
> **==顺序：本地缓存  =>  redis  =>  DB，本地缓存的过期时间要比redis短==**



#### 2.11.1 常用注解

* **@Cacheable**（查询）：先查缓存，如果缓存不存在则执行业务方法，并把对应返回结果缓存起来
* **@CachePut**（更新）：直接执行业务方法，并把返回结果覆盖缓存
* **@CacheEvict**（删除）：删除一条或整个缓存（`allEntries = true`）
* **@Caching**（组合缓存）：同时执行多个缓存操作（查、删、改）





#### 2.11.2 整合本地缓存

> **本地内存缓存，通过设置统一的缓存盒子、过期时间来放置缓存数据**

`1.SpringBoot启动器开启默认缓存支持`

```java
@SpringBootApplication
@MapperScan("com.eobard.dao")
@EnableCaching 					//开启SpringBoot默认的缓存支持
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```



`2.在查询数据库的Service层方法上加上缓存`

```JAVA
@Service
public class UserServiceImpl implements UserService {
    @Resource
    private UserMapper userMapper;

    @Override
    @Cacheable(cacheNames = "findUserById",key = "#id")
    //保存在内存中,缓存盒子为findUserById，键为这里的id值(类似于redis的K)，使用SpEL表达式获取形参值,一旦tomcat停掉了,缓存就没有了
    public User findUserById(int id) {
        return userMapper.findUserById(id);
    }
}
```



#### 2.11.3 高级用法（过期时间）

1. **自定义Cache实现类**

```java
public class GuavaCache implements org.springframework.cache.Cache {

    private final String name;
    private final Cache<Object, Object> cache;

    public GuavaCache(String name, Cache<Object, Object> cache) {
        this.name = name;
        this.cache = cache;
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public Object getNativeCache() {
        return cache;
    }

    @Override
    @Nullable
    public ValueWrapper get(Object key) {
        Object value = cache.getIfPresent(key);
        return (value != null ? new SimpleValueWrapper(value) : null);
    }

    @Override
    @Nullable
    public <T> T get(Object key, @Nullable Class<T> type) {
        Object value = cache.getIfPresent(key);
        if (value == null) return null;
        if (type != null && !type.isInstance(value)) {
            throw new IllegalStateException("Cached value is not of required type [" + type.getName() + "]");
        }
        return (T) value;
    }

    @Override
    @SuppressWarnings("unchecked")
    public <T> T get(Object key, Callable<T> valueLoader) {
        try {
            Object value = cache.get(key, valueLoader);
            return (T) value;
        } catch (Exception e) {
            throw new ValueRetrievalException(key, valueLoader, e);
        }
    }

    @Override
    public void put(Object key, @Nullable Object value) {
        cache.put(key, value);
    }

    @Override
    @Nullable
    public ValueWrapper putIfAbsent(Object key, @Nullable Object value) {
        Object existing = cache.getIfPresent(key);
        if (existing == null) {
            cache.put(key, value);
            return null;
        }
        return new SimpleValueWrapper(existing);
    }

    @Override
    public void evict(Object key) {
        cache.invalidate(key);
    }

    @Override
    public void clear() {
        cache.invalidateAll();
    }
}
```



2. **自定义 CacheManager 实现，设置过期时间**

```java
public class GuavaCacheManager extends AbstractCacheManager {

    private final ConcurrentMap<String, GuavaCache> cacheMap = new ConcurrentHashMap<>();

    @Override
    protected Collection<? extends Cache> loadCaches() {
        return cacheMap.values();
    }

    @Override
    protected Cache getMissingCache(String name) {
        com.google.common.cache.Cache<Object, Object> nativeCache = CacheBuilder.newBuilder()
                //设置统一的过期时间5分钟
                .expireAfterWrite(5, TimeUnit.MINUTES)
                //设置最大1000个缓存容量
                .maximumSize(1000)
                .build();

        GuavaCache cache = new GuavaCache(name, nativeCache);
        cacheMap.put(name, cache);
        return cache;
    }
}
```



3. **配置类**

```java
@Configuration
@EnableCaching
public class GuavaCacheConfig {

    @Bean
    public CacheManager cacheManager() {
        return new GuavaCacheManager();
    }
}
```





#### 2.11.2 整合Redis缓存

`1.概念`

​					在Spring Data Redis中操作Redis缓存，需要开启Redis缓存，可以借助Spring Data Redis提供的API操作；可以使用==StringRedisTemplate和RedisTemplate操作Redis==。



**StringRedisTemplate 与 RedisTemplate 的区别**

* StringRedisTemplate 继承了 RedisTemplate。

* StringRedisTemplate 只能对 key=String，value=String 的键值对进行操作，RedisTemplate 可以对任何类型的 key-value 键值对操作。
* 两者的数据是不共通的，StringRedisTemplate 只能管理 StringRedisTemplate 里面的数据，RedisTemplate 只能管理 RedisTemplate中 的数据



`2.使用`

> 使用Spring Data Redis的时候需要导入依赖

```xml-dtd
  <!--导入Spring Data Redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

| 方法                                              | 含义             |
| ------------------------------------------------- | ---------------- |
| opsForValue()                                     | 操作String类型   |
| opsForList()                                      | 操作List类型     |
| opsForSet()                                       | 操作Set类型      |
| opsForZSet()                                      | 操作ZSet类型     |
| opsForHash()                                      | 操作Hash类型     |
| expire(String key,Long timeout,TimeUnit timeUnit) | 指定键的过期时间 |

==注意：opsForXXX方法可以链式编程，有许多封装好的操作Redis的方法==



#### 2.11.3 StringRedisTemplate概念

| 语法                                             | 含义                                      |
| ------------------------------------------------ | ----------------------------------------- |
| delete(String key)                               | 删除键                                    |
| hasKey(String key)                               | 判断是否存在键                            |
| set(K key, V value)                              | 通过opsForValue()方法链式设置值           |
| set(K key, V value, long timeout, TimeUnit unit) | 通过opsForValue()方法链式设置值和过期时间 |
| multiSet(Map<? extends K, ? extends V> map)      | 通过opsForValue()方法链式设置多个值       |

```JAVA
    @Resource	//首先注入StringRedisTemplate
    StringRedisTemplate stringRedisTemplate;

    @Test
    public void contextLoads() {
        //设置值
        stringRedisTemplate.opsForValue().set("name","zs");
        //设置有效期的值
        stringRedisTemplate.opsForValue().set("code","123",60, TimeUnit.SECONDS);
    }
```

==注意：省略其它，可以通过opsForXXX方法调用相应的API==



#### 2.11.4 StringRedisTemplate实战

​																				**简单类型、单个数据的缓存**

`1.导入依赖`

```xml-dtd
    <!--导入Spring Data Redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--json工具-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.71</version>
        </dependency>
```



`2.在SpringBoot+Mybatis的环境下更改Service层`

```JAVA
@Service
public class UserServiceImpl implements UserService {
    @Resource
    private UserMapper userMapper;

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public String findUserById(int id) {
        //从缓存中读取
        String user = stringRedisTemplate.opsForValue().get("user_" + id);
        if(StringUtils.isEmpty(user)){
            //缓存中没有则从数据库中读取:这里可以将集合转为JSON数据,便于网页解析
            user= JSON.toJSONString(userMapper.findUserById(id));
            //将数据缓存一天
            stringRedisTemplate.opsForValue().set("user_"+id,user,1, TimeUnit.DAYS);
        }

        return user;
    }

    @Override
    public void deleteById(int id) {
        //删除数据先要将缓存中的数据清除
        stringRedisTemplate.delete("user_"+id);
        userMapper.deleteById(id);
    }
    
    @Override
    public void updateById(int id) {
        //修改数据先要将缓存中的数据清除
        stringRedisTemplate.delete("user_"+id);
        userMapper.updateById(id);
    }
    

}
```

==注意：1. **StringRedisTemplate适合于经常查询且简单的数据**，如单个信息的查询，不频繁修改的数据等==

​				==**2.并且在Service层方法返回的数据都要为String类型**==



#### 2.11.5 RedisTemplate概念

`1.自定义序列规则`

```JAVA
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate();
        //设置redis连接
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer jsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jsonRedisSerializer.setObjectMapper(objectMapper);

        //设置序列化规则
        redisTemplate.setHashKeySerializer(jsonRedisSerializer);
        redisTemplate.setKeySerializer(jsonRedisSerializer);

        //将序列方式更改为StringRedisSerializer
        
       // key采用String的序列化方式
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        // hash的key也采用String的序列化方式
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        // value序列化方式采用jackson
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        // hash的value序列化方式采用jackson
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
}
```

==注意：必须加上上面的配置类，否则会出现增加的key会带有二进制字节码==



`2.使用相应API`

```JAVA
	@Resource
    RedisTemplate<String,String> redisTemplate;

    @Test
    public void test(){
        redisTemplate.opsForValue().set("zs","zs");
        System.out.println( redisTemplate.opsForValue().get("zs"));
    }
}

```



#### 2.11.6 RedisTemplate实战

​																		**复杂类型、多个用户列表的缓存**

`1.导入依赖`

```xml-dtd
    <!--导入Spring Data Redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--json工具-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.71</version>
        </dependency>
```



`2.在SpringBoot+Mybatis的环境下更改Service层`

```JAVA
@Service
@Transactional
public class UserBizImpl implements UserBiz {

    @Resource
    private UserMapper userMapper;
    @Resource
    private RedisTemplate redisTemplate;

    @Override
    public User findUserById(int id) {
        //从缓存中读取
        Object user = redisTemplate.opsForHash().get("userList", "user_" + id);
        //若不存在则从数据库中查询
        if (ObjectUtils.isEmpty(user)) {
            user = userMapper.findUserById(id);
            //将数据保存在缓存中
            redisTemplate.opsForHash().put("userList", "user_" + id, user);
            redisTemplate.expire("userList", 1, TimeUnit.DAYS);
        }
        return (User) user;
    }

    @Override
    public void deleteById(int id) {
        userMapper.deleteById(id);
        //清空缓存中的数据
        redisTemplate.opsForHash().delete("userList", "user_" + id);
    }

    @Override
    public List<User> findUserList() {
        //从缓存中读取用户列表的个数
        Long size = redisTemplate.opsForHash().size("userList");
        List<User> userList=null;
        if (size<=0) {
            //个数小于0:说明缓存没数据,需要读DB
            userList= userMapper.findUserList();
            for (User user :userList ) {
                redisTemplate.opsForHash().put("userList","user_"+user.getId(),user);
                redisTemplate.expire("userList",1,TimeUnit.DAYS);
            }
        }else{
            //从缓存中读取数据
            userList = redisTemplate.opsForHash().values("userList");
        }
        return userList;
    }
}

```

==注意：1.本实例**使用Hash类型来存储用户列表，对一系列存储的数据进行分组，方便管理，适用于经常添加或修改的数据**==

​				==2.在Service层可以返回任何类型==



#### 2.11.7 总结

​					项目在整合了Redis缓存后，缓存的时间不宜过长、不宜过短，可以配合使用定时任务来更新缓存，这样可以防止读取的数据为旧数据



### 2.12 整合Spring Security

> 安全框架使用：
>
> ​			SSM + Shiro
>
> ​			Spring Boot/Spring Cloud + Spring Security

#### 2.12.1 快速入门

`1.导入依赖`

```xml-dtd
   		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```



`2.启动项目`

​			默认会进入登录页面，Spring Security提供的账号默认为`user`，密码为每次启动SpringBoot项目时随机生成在控制台下打印的日志中。

​			eg： Using generated security password: fc00c2c3-d7d8-4634-8961-8b83d4c6e4b6



> 若不想每次都用随机的密码登录，可以在全局配置文件自定义账号、密码、角色，如下图

```properties
#自定义登录的账号
spring.security.user.name=user
#自定义登录的密码
spring.security.user.password=123456
#自定义的角色名
spring.security.user.roles=admin
```



#### 2.12.2  基于内存认证用户

`1.创建配置类`

```JAVA
package com.eobard.config;

@Configuration
public class SecurityConfig  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
       
        auth.inMemoryAuthentication()
                .withUser("root")			//定义新的用户名1:root
                .password("{noop}123456")	//定义新的密码,不加密
            //角色列表为ADMIN和USER,这里不能加前缀ROLE_,因为SpringBoot会自动添加,否则会报错
                .roles("ADMIN","USER")  
            
                .and()    					//定义多个用and()连接
                .withUser("lucy")			//定义新的用户名2:lucy
                .password("{noop}123456")	//定义新的密码,不加密
                .roles("USER"); 	        //角色列表为USER
    }
}
```

> **若想使用加密的密码登录，则使用以下配置**

```JAVA
@Configuration
public class SecurityConfig  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //实例化加密类
        BCryptPasswordEncoder encoder =new BCryptPasswordEncoder();

        auth.inMemoryAuthentication()
                .passwordEncoder(encoder)   //使用加密
                .withUser("root")			//定义新的用户名1:root
                .password(encoder.encode("123456"))	//定义新的密码,加密
            //角色列表为ADMIN和USER,这里不能加前缀ROLE_,因为SpringBoot会自动添加,否则会报错
                .roles("ADMIN","USER")  	
            
                .and()    					//定义多个用and()连接
                .withUser("lucy")			//定义新的用户名2:lucy
                .password(encoder.encode("123456"))//定义新的密码,加密
                .roles("USER");         	//角色列表为USER
    }
}
```

==**特别注意：设置角色列表的时候一定不能加`ROLE_`前缀，SpringBoot为我们自动添加了前缀**==



`2.启动项目`

​			启动项目，用配置类的账号和密码登录即可



#### 2.12.3  自定义登录页面

`1.添加Thymeleaf依赖`

```xml-dtd
   <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```



`2.修改配置类`

```JAVA
package com.eobard.config;

@Configuration
@EnableWebSecurity
public class SecurityConfig  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //实例化加密类
        BCryptPasswordEncoder encoder =new BCryptPasswordEncoder();

        auth.inMemoryAuthentication()
                .passwordEncoder(encoder)   
                .withUser("root")
                .password(encoder.encode("123456"))
                .roles("ADMIN","USER","ROLE")   //角色列表为ADMIN,USER,ROLE
                .and()  
                .withUser("lucy")
                .password(encoder.encode("123456"))
                .roles("USER")         //角色列表为USER
                .and()
                .withUser("lily")
                .password(encoder.encode("123456"))
                .roles("ROLE"); //角色列表为ROLE
    }


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                //过滤静态资源路径、登录路径、错误页面路径
              .antMatchers("/resources/static/**","/login.html","/failure.html").permitAll()
                
            	//必须要有ADMIN角色才可以访问/admin/**
                .antMatchers("/admin/**").access("hasAnyRole('ADMIN')")
            
                //必须要有ADMIN或者USER权限才可以访问/user/**
                .antMatchers("/user/**").access("hasAnyRole('ADMIN','USER')")
            
                //必须要有ADMIN或者ROLE权限才可以访问/role/**
                .antMatchers("/role/**").access("hasAnyRole('ADMIN','ROLE')")
            
                //其它任何请求都要登录才可以访问
                .antMatchers("/**").authenticated()
                .anyRequest()
                .authenticated();

        //自定义认证页面
        http.formLogin()
                .loginPage("/login.html")           //登录的页面路径
                .loginProcessingUrl("/login")       //交给SpringSecurity自带的请求处理登录
                .usernameParameter("userName")      //前台账号name值
                .passwordParameter("password")     //前台密码name值
                .defaultSuccessUrl("/index.html")   //成功页面路径
                .failureForwardUrl("/failure.html") //失败页面路径
                .permitAll();
    }
}
```



`3.创建登录、首页、失败页面`

```HTML
<!--login.html-->
<!DOCTYPE html>
<html lang="en" xmlns="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="/login" method="post">
        <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
        账号：<input type="text" name="userName"> <br>	<!--对应上方.usernameParameter方法-->
        密码：<input type="text" name="password"> <br><!--对应上方.passwordParameter方法-->
        <input type="submit" value="login">
    </form>
</body>
</html>
```

> **这里要携带token令牌提交，否则会403**，详细原因见`Srping Security章节三`

```HTML
<!--index.html-->
<!DOCTYPE html>
<html lang="en" xmlns="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    登录成功
</body>
</html>
```

```HTML
<!--failure.html-->
<!DOCTYPE html>
<html lang="en" xmlns="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    登录失败
</body>
</html>
```



`4.编写控制器跳转到对应页面`

```JAVA
@Controller
public class HelloController {

    @GetMapping("/index.html")
    public String toIndex(){
        return "index";
    }

    @GetMapping("/login.html")
    public String toLogin(){
        return "login";
    }

    @GetMapping("/failure.html")
    public String toFailure(){
        return "failure";
    }

}
```



`5.编写不同用户权限页面控制器`

```JAVA
@RestController
public class IndexController {

    @GetMapping("/admin/{id}")
    public String admin(@PathVariable Integer id){
        return "管理员界面"+id;
    }

    @GetMapping("/user/{id}")
    public String user(@PathVariable Integer id){
        return "user界面"+id;
    }

    @GetMapping("/role/{id}")
    public String role(@PathVariable Integer id){
        return "role界面"+id;
    }
    
     @GetMapping("/order/{id}")
    public  String order(@PathVariable Integer id){
        return "order"+id;
    }

    @GetMapping("/product/{id}")
    public  String product(@PathVariable Integer id){
        return "product"+id;
    }

}
```



`6.测试`

​				分别用自定义的三个用户来登录并访问不同权限页面的路径



#### 2.12.4 基于数据库认证

`1.导入SSM相关的依赖、pom.xml设置编译dao层下的xml文件、主启动类扫描dao层代码`



`2.DB使用脚本、设置全局文件`

```SQL
CREATE DATABASE security_authority;
use security_authority;

drop table if exists `sys_role`;
create table `sys_role` (
 `id` int(11) not null auto_increment comment '编号',
 `role_name` varchar(30) default null comment '角色名称',
 `role_desc` varchar(60) default null comment '角色描述',
 primary key (`id`) using btree,
 key `id` (`id`)
) engine=innodb default charset=utf8;
insert into `sys_role`(`id`,`role_name`,`role_desc`) values
(1,'ROLE_USER','普通用户'),(2,'ROLE_ADMIN','管理员'),
(3,'ROLE_PRODUCT','产品管理员'),(4,'ROLE_ORDER','订单管理员');
drop table if exists `sys_user`;
create table `sys_user` (
 `id` int(11) not null auto_increment,
 `username` varchar(32) not null comment '用户名称',
 `password` varchar(120) not null comment '密码',
 `status` int(1) default '1' comment '1开启0关闭',
 primary key (`id`)
) engine=innodb default charset=utf8;
insert into `sys_user`(`id`,`username`,`password`,`status`) values
(1,'tom','$2a$10$nDbBLveNnpBSIELMIHFRKOJ8wGtAKYOcPsAgZ6kXVlvtddlDNdQMy',1),
(2,'admin','$2a$10$nDbBLveNnpBSIELMIHFRKOJ8wGtAKYOcPsAgZ6kXVlvtddlDNdQMy',1
),
(3,'jerry','$2a$10$nDbBLveNnpBSIELMIHFRKOJ8wGtAKYOcPsAgZ6kXVlvtddlDNdQMy',1
);
drop table if exists `sys_user_role`;
create table `sys_user_role` (
 `uid` int(11) not null comment '用户编号',
 `rid` int(11) not null comment '角色编号',
 primary key (`uid`,`rid`) using btree,
 key `FK_Reference_10` (`rid`),
 constraint `FK_Reference_10` foreign key (`RID`) references `sys_role`
(`ID`),
 constraint `FK_Reference_9` foreign key (`UID`) references `sys_user`
(`id`)
) engine=innodb default charset=utf8;
insert into `sys_user_role`(`uid`,`rid`) values (1,1),(2,1),(3,1),(2,2),
(1,3),(2,3),(2,4),(3,4);
```

> 初始登录密码都是123456



```properties
#整合mybatis驱动包
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#整合mybatis的url
spring.datasource.url=jdbc:mysql://localhost:3306/security_authority?serverTimezone=UTC
#整合mybatis账号
spring.datasource.username=root
#整合mybatis密码
spring.datasource.password=123456
#整合mybatis的别名
mybatis.type-aliases-package=com.eobard.entity
#整合mybatis数据连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
#打印sql语句
logging.level.com.eobard.dao.SysRoleMapper=debug
```





`3.Entity层代码`

```JAVA
//SysUser实体类
public class SysUser {
    private Integer id;
    private  String userName;
    private String password;
    private  Integer status;
    private List<SysRole> roleList;
}

//SysRole实体类
public class SysRole {
    private Integer id;
    private String roleName;
    private String roleDesc;
}
```



`4.Dao层代码`

```JAVA
//SysRoleMapper接口
public interface SysRoleMapper {
    List<SysRole> findRoleListByUserId(Integer userId);
}

//SysUserMapper接口
public interface SysUserMapper {
        SysUser findUserByUserName(String userName);
}
```

```xml-dtd
<!--SysRoleMapper.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.eobard.dao.SysRoleMapper">

    <resultMap id="baseResultMap" type="sysRole">
        <id property="id" column="id"/>
        <result property="roleDesc" column="role_desc" />
        <result property="roleName" column="role_name" />
    </resultMap>

    <select id="findRoleListByUserId" resultMap="baseResultMap">
            SELECT * FROM `sys_role` where id in (select rid from sys_user_role where uid=#{userId})
    </select>
</mapper>
```

==**注意：这里只需要查询SysUser和SysRole之间的中间表和SysRole表既可，不需要三表查询**==

```xml-dtd
<!--SysUserMapper.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.eobard.dao.SysUserMapper">

    <resultMap id="baseResultMap" type="SysUser">
        <id property="id" column="id" />
        <result property="userName" column="userName" />
        <result property="password" column="password" />
        <result property="status" column="status" />
        <collection property="roleList"  ofType="sysUser" column="id" select="com.eobard.dao.SysRoleMapper.findRoleListByUserId"/>
    </resultMap>


    <select id="findUserByUserName" resultMap="baseResultMap" >
        select * from sys_user where userName=#{userName}
    </select>
</mapper>
```

==**注意：这里通过懒加载，可以避免直接三表查询从而提高性能。**==



`5.Service层`

```JAVA
//接口: 需要继承Spring security的认证类,来实现自定义认证
public interface SysUserService  extends UserDetailsService {
}

//实现类
@Service
@Transactional
public class SysUserServiceImpl implements SysUserService {

    @Resource
    private SysUserMapper sysUserMapper;

    @Override
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {
        List<SimpleGrantedAuthority> authorities = new ArrayList<SimpleGrantedAuthority>();
        //调用根据用户名查询用户信息的方法
        SysUser sysUser = sysUserMapper.findUserByUserName(userName);
        //循环当前用户的角色列表并添加到权限中
        sysUser.getRoleList()
                .forEach( role ->
                        authorities.add(
                                new SimpleGrantedAuthority(role.getRoleName())
                        )
                );

        //创建认证用户对象
        //参数1：用户名
        //参数2：密码，其中{noop}表示不进行密码加密处理
        //参数3：角色列表
        User user = new User(sysUser.getUserName(),sysUser.getPassword(),authorities);
        //返回认证用户对象
        return user;
    }
}
```

> 后期不需要在控制器创建登录方法了，SpringSecurity会帮我们实现



`6.修改SecurityConfig配置文件`

```JAVA
package com.eobard.config;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Resource
    //注入登录的业务层
    private SysUserService sysUserService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //实例化加密类
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        //使用数据库认证登录
        auth.userDetailsService(sysUserService).passwordEncoder(encoder);
    }


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                //过滤静态资源路径、登录路径、错误页面路径
                .antMatchers("/resources/static/**", "/login.html", "/failure.html").permitAll()

                //必须要有ADMIN角色才可以访问/admin/**
                .antMatchers("/admin/**").access("hasAnyRole('ADMIN')")

                //必须要有ADMIN或者USER权限才可以访问/user/**
                .antMatchers("/user/**").access("hasAnyRole('ADMIN','USER')")

                //必须要有ADMIN或者ORDER权限才可以访问/order/**
                .antMatchers("/order/**").access("hasAnyRole('ADMIN','ORDER')")
            
              	//必须要有ADMIN或者PRODUCT权限才可以访问/product/**
                .antMatchers("/product/**").access("hasAnyRole('ADMIN','PRODUCT')")

                //其它任何请求都要登录才可以访问
                .antMatchers("/**").authenticated()
                .anyRequest()
                .authenticated();

        //自定义认证页面
        http.formLogin()
                .loginPage("/login.html")           //登录的页面路径
                .loginProcessingUrl("/login")       //交给SpringSecurity自带的请求处理登录
                .usernameParameter("userName")      //前台账号name值
                .passwordParameter("password")     //前台密码name值
                .defaultSuccessUrl("/index.html")   //成功页面路径
                .failureForwardUrl("/failure.html") //失败页面路径
                .permitAll();

    }
}
```



`7.登录,并测试数据库中权限`





#### 2.12.5 注销

`1.配置注销`

​			在SecurityConfig类中更改代码

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  //省略其它代码

    //注销
    http.logout()
            .invalidateHttpSession(true)	//注销时清除session
            .clearAuthentication(true)		//清除认证状态
            .logoutUrl("/logout")			//注销路径
            .logoutSuccessUrl("/login.html");//注销成功后跳转页面
}
```

`2.前台代码`

```HTML
<!--index.html-->
<!DOCTYPE html>
<html lang="en" xmlns="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
登录成功

<!--使用spring security自带的注销-->
<form action="/logout" method="post">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    <label><input type="submit" value="login">注销</label>
</form>
</body>
</html>
```

> 使用自带的注销功能两点注意：
>
> 1. 需要携带token信息
> 2. 需要post提交





#### 2.12.6 显示用户名

`1.导入Thymeleaf整合security依赖`

```xml-dtd
		<dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity5</artifactId>
        </dependency>
```



`2.页面引入头`

```html
<!--login.html-->
<!DOCTYPE html>
<html lang="en" xmlns="http://www.thymeleaf.org" 
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
</head>
<body>
    
<span sec:authentication="principal.username"></span>登录成功 <br>
<span sec:authentication="name"></span>登录成功

<!--注销-->
<form action="/logout" method="post">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    <label><input type="submit" value="login">注销</label>
</form>

</body>
</html>

```

> 其中`principal.username`和`name`为固定写法，获取当前登录的用户名



#### 2.12.7 动态显示菜单

`页面引入头`

```HTML
<!--login.html-->
<!DOCTYPE html>
<html lang="en" xmlns="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
</head>
<body>
<span sec:authentication="principal.username"></span>登录成功 <br>
<span sec:authentication="name"></span>登录成功

<!--注销-->
<form action="/logout" method="post">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    <label><input type="submit" value="login">注销</label>
</form>

<ul>
    <li sec:authorize="hasAnyRole('ADMIN')">
        <a href="/admin/1">admin管理</a>
    </li>
    <li sec:authorize="hasAnyRole('ADMIN','USER')">
        <a href="/user/1">user管理</a>
    </li>
    <li sec:authorize="hasAnyRole('ADMIN','ROLE')">
        <a href="/role/1">role管理</a>
    </li>
    <li sec:authorize="hasAnyRole('ADMIN','ORDER')">
        <a href="/order/1">order管理</a>
    </li>
    <li sec:authorize="hasAnyRole('ADMIN','PRODUCT')">
        <a href="/product/1">product管理</a>
    </li>
</ul>


</body>
</html>
```



#### 2.12.8 基于注解授权

​			若不在`SecurityConfig`类中设置过滤的页面访问路径，要控制权限，就可以采用注解授权来控制

`1.开启注解授权`

```JAVA
package com.eobard.config;

@Configuration
@EnableWebSecurity
//开启注解支持
@EnableGlobalMethodSecurity(jsr250Enabled = true,prePostEnabled = true,securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

   //省略其它代码
    
    
    //这里没有拦截/user/**,/admin/**,/role/** ....路径,需要注解拦截
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                //过滤静态资源路径、登录路径、错误页面路径
                .antMatchers("/resources/static/**", "/login.html", "/failure.html").permitAll()
                .antMatchers("/**").authenticated()
                .anyRequest()
                .authenticated();

        //自定义认证页面
        http.formLogin()
                .loginPage("/login.html")           //登录的页面路径
                .loginProcessingUrl("/login")       //交给SpringSecurity自带的请求处理登录
                .usernameParameter("userName")      //前台账号name值
                .passwordParameter("password")     //前台密码name值
                .defaultSuccessUrl("/index.html")   //成功页面路径
                .failureForwardUrl("/failure.html") //失败页面路径
                .permitAll();

        //注销
        http.logout()
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login.html");
    }
}
```



`2.控制器类中控制权限`

```JAVA
@RestController
public class IndexController {

    //使用PreAuthorize注解控制
    
    @GetMapping("/admin/{id}")
    @PreAuthorize("hasAnyRole('ADMIN')")
    public String admin(@PathVariable Integer id) {
        return "管理员界面" + id;
    }

    @GetMapping("/user/{id}")
    @PreAuthorize("hasAnyRole('USER','ADMIN')")
    public String user(@PathVariable Integer id) {
        return "user界面" + id;
    }

    @GetMapping("/role/{id}")
    @PreAuthorize("hasAnyRole('ADMIN','ROLE')")
    public String role(@PathVariable Integer id) {
        return "role界面" + id;
    }

    @GetMapping("/order/{id}")
    @PreAuthorize("hasAnyRole('ADMIN','ORDER')")
    public  String order(@PathVariable Integer id){
        return "order"+id;
    }

    @GetMapping("/product/{id}")
    @PreAuthorize("hasAnyRole('ADMIN','PRODUCT')")
    public  String product(@PathVariable Integer id){
        return "product"+id;
    }
}
```

> 详细用法见`Spring Security 7.3节`



#### 2.12.9 自定义错误页面

`1.创建异常处理类`

```JAVA
package com.eobard.exception;

@ControllerAdvice
public class ControllerException {

    @ExceptionHandler(RuntimeException.class)
    public String handlerException(RuntimeException e){
        if(e instanceof AccessDeniedException){
            //将该页面放入static文件夹当作静态资源放行
            return "redirect:/403.html";
        }
        //也可以放入static文件夹当作静态资源放行
        return "redirect:/500.html";
    }
}
```



`2.resources/static下创建403.html、500.html`

> 开发的时候不要使用自定义错误页面！



#### 2.12.10 记住我功能

##### 2.12.10.1 基于Cookie

`1.自定义登录页面加上记住我选项`

```HTML
<form action="/login" method="post">
    <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
    账号：<input type="text" name="userName"> <br>
    密码：<input type="text" name="password"> <br>
    <input name="remember-me" type="checkbox" value="true" />记住我
    <input type="submit" value="login">
</form>
```

* name属性值必须是remember-me
* value属性值必须是true，on，yes，1其中一个



`2.修改SecurityConfig类`

```JAVA
 	@Resource
    private SysUserService sysUserService;

	@Override
    protected void configure(HttpSecurity http) throws Exception {
 		//省略其它代码
        
        http.rememberMe()
                .tokenValiditySeconds(20)			//设置20秒过期
                .userDetailsService(sysUserService);//使用继承Spring security的认证类
    }
```



`3.使用IE浏览器测试`

​			使用IE浏览器测试可以直观的看出效果，如果使用其它浏览器(如Edge)会出现Cookie过期了还是可以登录成功(需要在系统中加入注销功能来使Cookie失效或者删除JSessionId)



##### 2.12.10.2 持久化token信息(推荐使用)

`1.创建表`

表的名称和字段不能变动！

```sql
use security_authority;

CREATE TABLE `persistent_logins` (
`username` varchar(64) NOT NULL,
`series` varchar(64) NOT NULL,
`token` varchar(64) NOT NULL,
`last_used` timestamp NOT NULL,
PRIMARY KEY (`series`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



`2.修改SecurityConfig类`

```JAVA
    @Resource
    private DataSource dataSource;

    @Bean
	//将token保存在DB中
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);//注入数据源
        jdbcTokenRepository.setCreateTableOnStartup(false);//是否自动创建表
        return jdbcTokenRepository;
    }

   	@Override
    protected void configure(HttpSecurity http) throws Exception {
		//省略其它
        http.rememberMe()
                .tokenValiditySeconds(20)
                .tokenRepository(persistentTokenRepository())//引用token保存DB类
                .userDetailsService(sysUserService);
    }
```



`3.测试`

​			使用IE浏览器登录后发现，`persistent_logins`表多了一行数据

> 该方式实现步骤：**在客户端的 cookie中，仅保存一个无意义的加密串（与用户名、密码等敏感数据无关）**，然后在数据库中保存该加密串-用户信息的对应关系，自动登录时，用cookie中的加密串，到数据库中验证，如果通过，自动登录才算通过。







### 2.13 整合JWT

1.引入依赖

```xml
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
        </dependency>
```



2.工具类

```java
public class JwtUtils {

    //过期时间为:60分钟
    private static long tokenExpiration = 60*60*1000;
    //加密秘钥
    private static String tokenSignKey = "wegou";

    //私有部分数据K
    private static final String USER_ID="userId";
    private static final String USERNAME="userName";

    /**
     * 根据userId和username 生成token
     * @param userId    用户id
     * @param userName  用户名称
     * @return
     */
    public static String createToken(Long userId, String userName) {
        String token = Jwts.builder()
                .setSubject("WEGOU-USER")   //自定义设置一个主题
                .setExpiration(new Date(System.currentTimeMillis() + tokenExpiration))  //当前时间+过期时间=jwt过期
                .claim(USER_ID, userId)        //设置jwt私有部分数据
                .claim(USERNAME, userName)    //设置jwt私有部分数据
                .signWith(SignatureAlgorithm.HS512, tokenSignKey)   //根据秘钥进行加密
                .compressWith(CompressionCodecs.GZIP)   //对字符串进行压缩,生成一行
                .compact();
        return token;
    }


    public static Long getUserId(String token) {
        if(StringUtils.isEmpty(token)) return null;
        Jws<Claims> claimsJws = Jwts.parser().setSigningKey(tokenSignKey).parseClaimsJws(token);
        Claims claims = claimsJws.getBody();
        Integer userId = (Integer)claims.get(USER_ID);
        return userId.longValue();
    }

    public static String getUserName(String token) {
        if(StringUtils.isEmpty(token)) return "";

        Jws<Claims> claimsJws = Jwts.parser().setSigningKey(tokenSignKey).parseClaimsJws(token);
        Claims claims = claimsJws.getBody();
        return (String)claims.get(USERNAME);
    }

    public static void removeToken(String token) {
        //jwttoken无需删除，客户端扔掉即可。
    }

    public static void main(String[] args) {
        String token = JwtUtils.createToken(7L, "admin");
        System.out.println(token);
        System.out.println(JwtUtils.getUserId(token));
        System.out.println(JwtUtils.getUserName(token));
    }
}
```



3.结果

```
eyJhbGciOiJIUzUxMiIsInppcCI6IkdaSVAifQ.H4sIAAAAAAAAAKtWKi5NUrJSCnd19w_VDQ12DVLSUUqtKFCyMjSzsDQyNQZiHaXS4tQizxQlK3MI0y8xNxWoJzElNzNPqRYADnphrUMAAAA.cUFl_jQ8E1YC-xGkMeY53igfPj635_EiQEW1ztZW0AZjQCHZ-EUJmx4NP3yv7im3pmP-8gkN7d6ZPq1HtTcQoA
7
admin
```







### 2.14 整合knife4j

#### 2.14.1 导入依赖

```xml
  <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-spring-boot-starter</artifactId>
            <version>2.0.8</version>
       		<!--<version>3.0.3</version>-->
 </dependency>
```

> 如果是SpringBoot 2.6+版本需要配置全局配置文件`spring.mvc.pathmatch.matching-strategy=ant_path_matcher`





#### 2.14.2 编写配置类

```java
@Configuration
@EnableSwagger2WebMvc
public class Swagger2Config implements ApplicationRunner {

    //配置普通用户角色的接口配置
    @Bean
    public Docket webApiConfig(){
        List<Parameter> pars = new ArrayList<>();
        
        //可选项：设置token信息,比如使用sa-token这样的权限框架,有些接口需要在header中带上你的token信息来证明你是以登录过的，这里就是使用swagger自带的网页发送请求时默认带上的token信息
        ParameterBuilder tokenPar = new ParameterBuilder();
        tokenPar.name("userId")//这是header中的name
                .description("用户token")
                //.defaultValue(JwtHelper.createToken(1L, "admin"))
                .defaultValue("1")//这是对应的value
                .modelRef(new ModelRef("string"))
                .parameterType("header")
                .required(false)
                .build();
        pars.add(tokenPar.build());

        Docket webApi = new Docket(DocumentationType.SWAGGER_2)
                //设置右上角下拉框的组名
                .groupName("前端API")
                .apiInfo(webApiInfo())
                .select()
                //指定扫描接口的包
                .apis(RequestHandlerSelectors.basePackage("com.eobard.api"))
                //根据url路径设置哪些请求加入文档,其中PathSelectors.any()表示全部都放进去
                .paths(PathSelectors.regex("/api/.*"))
                .build()
                .globalOperationParameters(pars);
        return webApi;
    }


    //配置普通用户角色的接口信息
    private ApiInfo webApiInfo(){
        //作者信息
        Contact contact = new Contact("Eobard Thawne", "", "2209473452@qq.com");
        return new ApiInfoBuilder()
                .title("前端-API文档")
                .description("该文档描述了前端的普通接口API")
                .version("1.0")
                .contact(contact)
                .build();
    }



    //配置管理员角色的接口信息
    private ApiInfo adminApiInfo(){
        //作者信息
        Contact contact = new Contact("Eobard Thawne", "", "2209473452@qq.com");
        return new ApiInfoBuilder()
                .title("后台管理系统-API文档")
                .description("该文档描述了后端的管理员接口API")
                .version("1.0")
                .contact(contact)
                .build();
    }

    //配置管理员角色的接口配置
    @Bean
    public Docket adminApiConfig(){
        List<Parameter> pars = new ArrayList<>();
        
        //可选项：同上
        ParameterBuilder tokenPar = new ParameterBuilder();
        tokenPar.name("adminId")
                .description("用户token")
                .defaultValue("1")
                .modelRef(new ModelRef("string"))
                .parameterType("header")
                .required(false)
                .build();
        pars.add(tokenPar.build());

        Docket adminApi = new Docket(DocumentationType.SWAGGER_2)
                .groupName("后端API")
                .apiInfo(adminApiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.eobard.controller"))
                //只显示admin路径下的页面
                .paths(PathSelectors.regex("/admin/.*"))
                .build()
                .globalOperationParameters(pars);
        return adminApi;
    }
    
    /**
     * 打印 API 文档地址
     */
    @Override
    public void run(ApplicationArguments args) {
        System.out.println("========================================");
        System.out.println("API 文档地址：http://localhost:8080/doc.html");
        System.out.println("========================================");
    }
}
```



#### 2.14.3 编写测试接口

```java
package com.eobard.api;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Api(tags = "前台用户的API")
@RestController
@RequestMapping("/api/index")
public class IndexApi {

    @ApiOperation("获取前台用户的首页数据")
    @GetMapping("/data")
    public String data(){
        return "获取用户首页数据";
    }
}
```

```java
package com.eobard.controller;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Api(tags = "后台管理员的API")
@RestController
@RequestMapping("admin/index")
public class IndexController {

    @ApiOperation("获取后台管理员的首页数据")
    @GetMapping("/data")
    public String data(){
        return "获取管理员首页数据";
    }
}
```



#### 2.14.4 测试

> **输入`ip地址:端口号/doc.html`，常用注解见`2.9.2小节`**







### 2.15 整合MQTT

1. 导入依赖

```xml-dtd
   <!--MQTT相关依赖-->
        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-mqtt</artifactId>
        </dependency>
        <dependency>
            <groupId>org.eclipse.paho</groupId>
            <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
            <version>1.2.5</version> <!-- 指定版本 -->
        </dependency>
```



2. 配置连接信息

```yaml
mqtt:
  brokerUrl: tcp://127.0.0.1:1883
  username: admin
  password: public
```

```java
@Configuration
public class MqttConfig {
    @Value("${mqtt.brokerUrl}")		//tcp://127.0.0.1:1883
    private String brokerUrl;

    @Value("${mqtt.username}")		//默认admin
    private String username;

    @Value("${mqtt.password}")		//默认public
    private String password;

    //注入自定义的消息处理器
    @Autowired
    @Lazy 		//使用@lazy解决循环依赖问题
    private MqttMessageProcessor mqttMessageProcessor;



    // 创建 MqttConnectOptions连接配置类
    @Bean
    public MqttConnectOptions mqttConnectOptions() {
        MqttConnectOptions options = new MqttConnectOptions();
        options.setAutomaticReconnect(true);
        options.setCleanSession(true);
        options.setServerURIs(new String[]{brokerUrl});
        options.setUserName(username);
        options.setPassword(password.toCharArray());
        return options;
    }

    // 创建 MqttPahoClientFactory工厂类
    @Bean
    public MqttPahoClientFactory mqttPahoClientFactory(MqttConnectOptions mqttConnectOptions) {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        factory.setConnectionOptions(mqttConnectOptions);           // 设置连接选项
        return factory;
    }

    //创建输入通道
    @Bean
    public MessageChannel mqttInputChannel() {
        return new DirectChannel();
    }

    // 创建 MqttPahoMessageDrivenChannelAdapter 来监听多个主题
    @Bean
    public MqttPahoMessageDrivenChannelAdapter mqttInbound(MqttPahoClientFactory mqttPahoClientFactory) {
        String[] topics = MqttTopicKeys.getTopics();
        //TODO:Java应用程序的clientID不能和GUI的软件一致,不然会出现断开的问题
        String clientId = UUID.randomUUID().toString();
        MqttPahoMessageDrivenChannelAdapter adapter = new MqttPahoMessageDrivenChannelAdapter(clientId, mqttPahoClientFactory, topics);
        adapter.setOutputChannel(mqttInputChannel());
        return adapter;
    }


    // 设置处理收到消息的方法
    @ServiceActivator(inputChannel = "mqttInputChannel")
    public void handleMqttMessage(Message<String> message) {
        String topic = message.getHeaders().get(MqttHeaders.RECEIVED_TOPIC, String.class);
        //用反射的形式执行消息处理
        mqttMessageProcessor.handleMessage(topic, message);
    }
    
    // 创建 MqttPahoMessageHandler 用于发送消息
    @Bean
    public MqttPahoMessageHandler mqttPahoMessageHandler(MqttPahoClientFactory mqttPahoClientFactory) {
        MqttPahoMessageHandler messageHandler = new MqttPahoMessageHandler(UUID.randomUUID().toString(), mqttPahoClientFactory);
        messageHandler.setAsync(true);  // 设置异步发送
        return messageHandler;
    }
```



3. 多主题消息监听的反射配置类

```java
@Component
public class MqttMessageProcessor {

    @Autowired
    private ApplicationContext applicationContext;

    // 将主题和方法进行映射,K是监听的topic主题名,V是对应的方法名字,方便后面用反射去处理他
    //eg：K=get/status,V=com.xxx.handler.PlatformHandler.getStatus
    private Map<String, Method> topicReceiveMethodMap = new HashMap<>();

    @PostConstruct
    public void init() {
        //只要是在IOC容器中,存在自定义的接受MQTT的注解类
        Map<String, Object> beansWithAnnotation = applicationContext.getBeansWithAnnotation(MqttReceiveHandler.class);
        // 遍历所有这些自定义的消息注解类
        for (Object bean : beansWithAnnotation.values()) {
            Method[] methods = bean.getClass().getDeclaredMethods();
            for (Method method : methods) {
                //如果这个方法有自定义监听的注解,则加入到map中
                if (method.isAnnotationPresent(MqttTopicListener.class)) {
                    MqttTopicListener annotation = method.getAnnotation(MqttTopicListener.class);
                    String topic = annotation.topic();
                    topicReceiveMethodMap.put(topic, method);
                }
            }
        }
    }

    // 通过反射调用方法来处理消息
    public void handleMessage(String topic, Message<String> message) {
        //通过刚刚上面的K获取对应的V,然后去执行反射
        Method method = topicReceiveMethodMap.get(topic);
        if (method != null) {
            try {
                Object targetBean = applicationContext.getBean(method.getDeclaringClass());
                method.invoke(targetBean, message);  // 调用对应的方法
            } catch (Exception e) {
                e.printStackTrace();
            }
        } else {
            System.out.println("没有找到对应：" + topic + "的处理方法");
        }
    }
}
```



4. 自定义注解类

```java
//自定义注解：作用于类上,将该类标记为一个专门用于接收消息的类
@Retention(RetentionPolicy.RUNTIME)
public @interface MqttReceiveHandler {
}

==============================================================================
//自定义的注解：只能作用于方法上
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MqttTopicListener {
    String topic();  //订阅主题的名称
}
```





5. 消息发送类

```java
@Component
public class MqttMessageSender {

    private final MqttPahoMessageHandler mqttPahoMessageHandler;

    //使用构造注入,解决循环依赖问题
    @Autowired
    public MqttMessageSender(MqttPahoMessageHandler mqttPahoMessageHandler) {
        this.mqttPahoMessageHandler = mqttPahoMessageHandler;
    }

    // 发送其他类型的消息
    public void sendMessage(String topic, String payload) {
        // 构建消息
        Message<String> message = MessageBuilder.withPayload(payload)
                .setHeader(MqttHeaders.TOPIC, topic)
                .build();
        // 发送消息
        mqttPahoMessageHandler.handleMessage(message);
    }
}
```



6. 主题处理业务类

```java
@Component
@MqttReceiveHandler
public class PlatformCallbackHandler {

    @Autowired
    private MqttMessageSender mqttMessageSender;

    //平台数据校验监听方法
    @MqttTopicListener(topic = MqttTopicKeys.DEVICE_DATA_VALIDATE)
    public void validate(Message<String> message) throws JsonProcessingException {
        //获取消息json数据
        String payload = message.getPayload();
        ObjectMapper objectMapper = new ObjectMapper();
        //将其转换为DataValidateResult类
        DataValidateResult result = objectMapper.readValue(payload, DataValidateResult.class);
        //创建返回结果
        Map<String, Object> jsonObject = new HashMap();
        jsonObject.put("type", result.getType().getTypeValue());

        switch (result.getType()) {
            //NFC的处理逻辑
            case NFC:
                System.out.println(result);
                break;
            //人脸的处理逻辑
            case FACE:
                System.out.println(result);
                break;
            //二维码的处理逻辑
            case QR_CODE:
                System.out.println(result);
                jsonObject.put("data", "检验通过");
                //发送消息
                mqttMessageSender.sendMessage(MqttTopicKeys.DEVICE_DATA_VALIDATE_RETURN + result.getDeviceId(), JSONUtil.toJsonStr(jsonObject));
                break;
        }
    }
}
```

![image-20250418102936779](./SpringBoot_images/image-20250418102936779-17781166357732.png)

![image-20250418114516656](./SpringBoot_images/image-20250418114516656.png)





---

## 三. 应用

### 3.1 文件上传与下载

#### 3.1.1 上传

`a.导入依赖`

```xml-dtd
   <!-- commons-io -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.7</version>
        </dependency>
        <!-- commons-fileupload -->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.4</version>
        </dependency>
```



`b.修改全局配置文件`

```properties
#application.properties属性文件配置

#设置单个上传文件的大小
spring.servlet.multipart.max-file-size=10MB
#设置一次请求上传的最大文件的大小
spring.servlet.multipart.max-request-size=10MB
```



`c.创建页面和控制器`

```HTML
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="file"><br>
    <input type="submit" value="submit">
</form>
```

```java
@Controller
public class UploadController {

    @RequestMapping("/upload")
    public String upload(MultipartFile file, Model model){
        //判断是否选中文件
        if (!file.isEmpty()) {
            String path = "D://";
            //获取源文件名称
            String oldFileName = file.getOriginalFilename();
            //获取源文件后缀
            String suffix = oldFileName.substring(oldFileName.lastIndexOf('.'), oldFileName.length());
            //重命名文件
            String newFileName = UUID.randomUUID().toString().replace("-","")+suffix;
            //为了解决同一个文件夹文件过多的问题，使用日期作为文件夹管理
            String datePath = new SimpleDateFormat("yyyyMMdd").format(new Date());
            //组装文件名
            String finalName = datePath +"/"+ newFileName;
            //创建文件对象
            //参数1：文件上传的地址 参数2：文件名称
            File dest = new File(path, finalName);
            //判断文件夹是否存在，不存在则创建
            if (!dest.getParentFile().exists()) {
                dest.getParentFile().mkdirs();//创建文件夹
            }
            try {
                //将文件保存到磁盘中
                file.transferTo(dest);
            } catch (IOException e) {
                e.printStackTrace();
            }

            //将上传文件的路径保存在request中
            if(file.getContentType().contains("image")){
                model.addAttribute("url",newFileName);
            }
            return "show";
        }
           return "index";
    }
}

```

```HTML
 <img th:src="@{/img/} + ${url}" />
```



`d.配置虚拟路径回显图片`

```JAVA
@Configuration
public class GlobalWebConfig  implements WebMvcConfigurer{

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
 
        //配置上传文件的静态虚拟路径
        registry.addResourceHandler("/img/**")
                .addResourceLocations("file:D:\\20211114\\");
    }
}
```

==注意：这里必须要配置虚拟访问路径，不然访问不到绝对路径上的图片==



#### 3.1.2 下载

`a.控制器代码`

```JAVA
 @GetMapping("down")
    public ResponseEntity<byte[]> download(HttpServletRequest request,String fileName) throws Exception {
        //上传文件的路径
        String dir="C:\\Users\\Eobard_Thawne\\Desktop";
        //指定下载的文件
        File file=new File(dir+File.separator+fileName);
        //设置响应头
        HttpHeaders headers=new HttpHeaders();
        fileName=getFileName(request,fileName);
        headers.setContentDispositionFormData("attachment",fileName);
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
        try {
            return new ResponseEntity<>(FileUtils.readFileToByteArray(file),headers, HttpStatus.OK);
        } catch (IOException e) {
            e.printStackTrace();
            return new ResponseEntity<byte[]>(e.getMessage().getBytes(),HttpStatus.EXPECTATION_FAILED);
        }
    }

    private String getFileName(HttpServletRequest request, String fileName) throws  Exception {
        String[] blowers={"MSIE","Trident","Edge"};
        String userAgent = request.getHeader("User-Agent");
        for (String keyWord:blowers){
            if(userAgent.contains(keyWord)){
                //IE浏览器
                return URLEncoder.encode(fileName,"UTF-8").replace("+","");
            }
        }
        //火狐等其它浏览器
        return new String(fileName.getBytes("UTF-8"),"ISO-8859-1");
    }
```



`b.前台页面`

```HTML
<a th:href="@{/down(fileName='环境图.png')}" >下载</a>
```

==注意：后期需要从数据库读取上传文件的文件名来下载==



### 3.2 数据校验

#### 3.2.1基本校验

1. 导入pom文件

`````xml
 <!--validation-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
`````



2. 配置实体类注解

* `@NotNull`：不能为 null，**数字用**
* `@NotBlank`：**字符串用**，不能 null、不能空、不能全空格
* `@NotEmpty`：**集合 / 数组用**，不能 null 且长度 > 0
* `@Length(min,max)`：字符串长度范围
* `@Email`：邮箱格式
* `@Pattern(regexp="")`：正则校验（手机号、身份证等）
* `@Min`：数值 ≥ 指定值
* `@Max`：数值 ≤ 指定值
* `@DecimalMin`：小数 / 字符串最小值
* `@DecimalMax`：小数 / 字符串最大值
* `@Positive`：正数
* `@PositiveOrZero`：正数或 0
* `@Negative`：负数
* `@NegativeOrZero`：负数或 0
* `@Past`：必须是过去时间
* `@PastOrPresent`：过去或当前
* `@Future`：必须是未来时间
* `@FutureOrPresent`：未来或当前



3. 控制器上校验，配合全局异常处理器

```java
//控制器  
public Result<String> saveTeacher(@Validated @RequestBody TeacherVO teacherVO) {
        Teacher teacher = new Teacher();
        BeanUtils.copyProperties(teacherVO, teacher);
        boolean flag = teacherService.save(teacher);
        return flag ? Result.success("保存成功") : Result.error("保存失败");
}

//全局异常处理器
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 处理参数校验异常
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Result<String> handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        log.error("参数校验异常: {}", e.getMessage());
        FieldError fieldError = e.getBindingResult().getFieldError();
        if (fieldError != null) {
            return Result.error(fieldError.getField() + ": " + fieldError.getDefaultMessage());
        }
        return Result.error("参数校验失败");
    }
}
```



#### 3.2.2分组校验

1. 定义分组接口，区分新增还是删除生效

`````java
// 两个分组接口，分别在新增和修改时校验数据
 public interface AddGroup extends Default{}
 public interface UpdateGroup extends Default{}
`````



2. 编辑实体类，校验的组

```java
@Data
public class CourseVO {
    @NotNull(groups = UpdateGroup.class)	//修改时会要求不能为空
    private Long id;

    @NotBlank(groups = AddGroup.class)		//新增时会要求不能为空
    private String name;
}
```



3. 控制器设置分组校验

`````java
// 新增
@PostMapping
public Result add(@Validated(AddGroup.class) @RequestBody CourseVO vo)

// 更新
@PutMapping
public Result update(@Validated(UpdateGroup.class) @RequestBody CourseVO vo)
`````





#### 3.2.3其它

​		Spring Boot的数据校验与Spring MVC一模一样，使用详见Spring MVC，**但在前端thymeleaf页面显示数据校验错误需要更改写法**

```HTML
<font color="red" th:errors="${校验实体类类名(首字母小写).属性名}"></font>
```

==注意：光这样写了在数据验证的时候会有异常，应该同Spring MVC一样的，在去往数据校验的页面时，要传个对象回去 (属性名应该是实体类的类名，首字母小写)==

```JAVA
@RequestMapping(value = "/addUser")
public String addUserCheck(User user){
    return "register";
}
```



### 3.3 验证码使用

​							在线网址：	[EasyCaptcha: Java图形验证码，支持gif、中文、算术等类型，可用于Java Web、JavaSE等项目。 (gitee.com)](https://gitee.com/ele-admin/EasyCaptcha)

#### 3.3.1 使用

`1.导入依赖`

```xml-dtd
    <!--easy-captcha验证码-->  
<dependency>
      <groupId>com.github.whvcse</groupId>
      <artifactId>easy-captcha</artifactId>
      <version>1.6.2</version>
   </dependency>
```



`2.验证码类型`

```JAVA
  //输出普通验证码
    @RequestMapping("/code")
    public void captcha(HttpServletRequest request, HttpServletResponse response) throws Exception {
        CaptchaUtil.out(request, response);
    }
```

```java
 	// 使用gif验证码
    @RequestMapping("/code2")
    public void captcha2(HttpServletRequest request, HttpServletResponse response) throws Exception {
        GifCaptcha gifCaptcha = new GifCaptcha();
        //设置长度为4位
        gifCaptcha.setLen(4);
        CaptchaUtil.out(gifCaptcha, request, response);
    }
```

```JAVA
 // 中文gif类型
    @RequestMapping("/code3")
    public void captcha3(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ChineseGifCaptcha captcha = new ChineseGifCaptcha();
        //设置长度为4位
        captcha.setLen(6);
        CaptchaUtil.out(captcha, request, response);
    }
```

```JAVA
  //数学运算验证码
    @RequestMapping("/code4")
    public void captcha4(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ArithmeticCaptcha captcha = new ArithmeticCaptcha(130, 48);
        captcha.setLen(3);  // 几位数运算，默认是两位
        captcha.getArithmeticString();  // 获取运算的公式：3+2=?
        captcha.text();  // 获取运算的结果：5
        CaptchaUtil.out(captcha, request, response);
    }
```

==注意：在使用CaptchaUtil.out()方法的时候，会将验证码的数据放入k为captcha的session中==



`3.验证登录`

```JAVA
 @PostMapping("/ver")
    public String login(String verCode, HttpServletRequest request){
        if (!CaptchaUtil.ver(verCode, request)) {
            CaptchaUtil.clear(request);  // 清除session中的验证码
            System.out.println("验证码错误");
            return "/login";
        }
        else{
            System.out.println("登录成功");
            return "redirect:/index";
        }
    }
```

==注意：登录的时候，使用CaptchaUtil.ver()方法会主动对比验证码的数据，不需要自己手动对比==



### 3.4 异常处理

#### 3.4.1 方式1：错误页面

​		直接在src/main/templates下创建一个error.html的模板页面，**Spring Boot默认会有一个处理异常的控制器**

```HTML
错误信息:<span th:text="${exception}" />
```



#### 3.4.2 方式2：全局异常

返回页面

```JAVA
@ControllerAdvice
public class GlobalExceptionHandler {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @ExceptionHandler(Exception.class)
    public String error(HttpServletRequest request,Exception e){
        String msg="你没有权限操作该页面，请重试！";
        logger.error("出现异常{}",e.getMessage());
        request.setAttribute("msg",msg);
        //错误页面名称
        return "error";
    }
}
```



返回JSON数据

```java
@ControllerAdvice
@ResponseBody						//返回json数据，@RestControllerAdvice=上面两个注解
@Slf4j
public class GlobalExceptionHandler {

    //处理Exception异常,统一返回错误信息
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public Result error(Exception e){
        log.error("系统异常: {}", e.getMessage());
        return Result.error();
    }

    //自定义异常处理需要注册到全局异常处理配置中
    @ExceptionHandler(CustomException.class)
    public Result error(CustomException e){
        return Result.build(null,e.getCode(),e.getMessage());
    }
    
   
     //处理参数校验异常
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Result handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        log.error("参数校验异常: {}", e.getMessage());
        FieldError fieldError = e.getBindingResult().getFieldError();
        if (fieldError != null) {
            return Result.error(fieldError.getField() + ": " + fieldError.getDefaultMessage());
        }
        return Result.error("参数校验失败");
    }


}
```

```java
//在对应的方法中throw new CustomException即可被全局异常捕获,然后返回对应的信息
public PaymentInfo savePaymentInfo(String orderNo) {
    ......
    throw new CustomException(ResultCodeEnum.DATA_ERROR);
    ......
        
}
```







### 3.5 拦截器使用

`1.编写拦截器功能`

```JAVA
public class IndexInterceptor implements HandlerInterceptor {
    private Logger logger= LoggerFactory.getLogger(IndexInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        logger.info(LocalTime.now()+",进入拦截器");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info(LocalTime.now()+",退出拦截器");
    }
}
```



`2.注册拦截器`

```JAVA
@Configuration
public class GlobalWebConfig  implements WebMvcConfigurer{

    //添加自定义的拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册一个拦截器
        registry.addInterceptor(new IndexInterceptor())
                .addPathPatterns("/**")				//需要拦截的路径,/**表示需要拦截所有请求
                .excludePathPatterns("/getList","/tt");		//不需要拦截的路径

        
        //如果有多个拦截器,继续添加即可
        /*
            registry.addInterceptor(new XXXInterceptor())
                    .addPathPatterns("/**")
                    .excludePathPatterns("/XXX");
         */

    }
}
```



### 3.6 Cors跨域配置

​		在前后端分离的Spring Boot项目中，可以配置跨域配置实现前端的访问请求

####  3.6.1 全局配置(推荐)

```JAVA
@Configuration
public class GlobalWebConfig  implements WebMvcConfigurer{

    //添加Cors跨域配置
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")                  //配置支持跨域的路径
                .allowedMethods("POST", "GET", "PUT","DELETE")//配置支持跨域请求的方法
                .allowCredentials(true);            //配置是否允许发送Cookie, 用于凭证请求
    }
}

```



#### 3.6.2 局部配置

```JAVA
 	@RequestMapping("/get")
    @ResponseBody
    @CrossOrigin			//只需要在控制器的方法上加入注解即可
    public User user(){
        return new User(1,"zs1","man");
    }
```



### 3.7  全局格式化日期

​					在application.properties文件中设置

```properties
#格式化日期(全局配置：等同于在实体类属性名上加入@DateTimeFormat注解)
spring.mvc.format.date=yyyy-MM-dd
spring.mvc.format.date-time=yyyy-MM-dd HH:mm:ss

#若使用了FastJson，可以用下面配置格式化JSON
#JSON日期格式化
spring.jackson.date-format= yyyy-MM-dd
#JSON日期格式化设置时区为上海
spring.jackson.time-zone=Asia/Shanghai
```



### 3.8 自定义欢迎页面

```JAVA
@Configuration
public class GlobalWebConfig  implements WebMvcConfigurer{

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //自定义访问根路径的时候跳到login.html页面
        registry.addViewController("/").setViewName("login");
        //自定义访问/index路径的时候跳到index.html页面
        registry.addViewController("/index").setViewName("index");
    }
}

```

==注意: 后期项目完成后可以通过这种方式设置用户访问时候的欢迎页面==



### 3.9 自定义starter

`一.创建Spring Boot项目`

  1. **创建springboot项目的时候包名不要写常用的包名**，因为在springboot项目启动的时候会扫描启动器所在的包中的Bean，**要体现自动装配，就最好把自定义的starter放在其它包中**

  2.  **创建springboot的模块名应该为： xxx-spring-boot-starter**

  3. 导入配置文件依赖

     ```xml-dtd
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-configuration-processor</artifactId>
         <version>2.5.4</version>
         <optional>true</optional>
     </dependency>
     ```

     还要依次勾选设置->构建执行部署->编译器->annotation Processors里面的enable annotation Processing



`二.编写自动装配类`

​			创建自动装配的类交给springboot去装配，**配置类命名规则：XXXAutoConfigure**

```JAVA
@Configuration      									//标志为自动配置类
@EnableConfigurationProperties(IndexProperties.class)   //导入全局配置属性中对应的实体类
/**
* 条件注解： 当全局配置文件中(.yaml或者.properties)配置了eobard.index.name=xxx时候
*			该自定义的starter会生效
*/
@ConditionalOnProperty(value = "eobard.index.name")	
public class IndexAutoConfigure {

     //注入全局配置属性中对应的实体类并且赋值注入,然后传给控制器去获取值
    @Autowired
    private IndexProperties indexProperties;

    @Bean
    public IndexController indexController(IndexProperties indexProperties){
        //传入全局配置属性中对应的实体类到构造方法中便于实现功能
        return new IndexController(indexProperties);	
    }
}
```



`三.编写配置类`

​		创建全局配置属性中对应的实体类，**命名规则：XXXProperties**

```JAVA
//将全局properties文件或者yml文件进行绑定
//前缀为eobard.index
@ConfigurationProperties(prefix = "eobard.index")	
public class IndexProperties {
    
    //属性1
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```



`四.创建自定义starter`

​		创建自定义starter的功能：默认添加一个首页

```JAVA
@RestController
public class IndexController {

    @Autowired
    private IndexProperties indexProperties; //注入全局配置属性中对应的实体类传给控制器去获取值

    public IndexController(IndexProperties indexProperties) {
        this.indexProperties=indexProperties;
    }

    @RequestMapping("/")
    public String index(){
        return indexProperties.getName()+" 欢迎使用自定义starter！";
    }
}
```



`五.创建spring.factories文件`

​		在resources文件下创建文件，并将自动配置类放进去

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
cn.starter.IndexAutoConfigure 
```



==注意：K 只能为org.springframework.boot.autoconfigure.EnableAutoConfiguration=\ 多个用逗号隔开==

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
cn.starter.IndexAutoConfigure,\
cn.starter.Index2AutoConfigure 
```



`六.打包成jar`

​		选择右侧maven中的当前项目选择lifecycle中的install即可



`七.使用`

​		在其它springboot项目中导入自定义的starter依赖即可



### 3.10 任务管理

#### 3.10.1 异步任务

##### 3.10.1.1 无返回值异步任务

​						在实际开发中，**对时效性要求不高的功能都可以使用异步任务**，比如项目可能会向新注册用户发送短信验证码，此时可以考虑使用异步任务调用，因为用户对这个时效性要求不是特别高。



`1.开启SpringBoot异步任务支持`

```JAVA
@SpringBootApplication
@EnableAsync//开启异步任务支持
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```



`2.创建发送信息的异步方法`

```JAVA
@Service
public class SmsService {

    @Async		//标志该方法为异步方法
    public void sendSMS(){
        long start = System.currentTimeMillis();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis();
        System.out.println("子线程所用时间："+(end-start));
    }
}
```



`3.调用`

```JAVA
@Controller
public class SMSController {

    @ResponseBody
    @GetMapping("sms")
    public String send() {
        long start = System.currentTimeMillis();
        smsService.sendSMS();
        long end = System.currentTimeMillis();
        System.out.println("主线程用时："+(end-start));
        return "success";
    }
}

```

> 主线程用时：1ms
> 子线程所用时间：2004ms



##### 3.10.1.2 有返回值异步任务

​								在实际开发中，若遇到某些功能对时效性要求不是很高，但是需要获取统计的返回值，就可以使用有返回的异步任务。



`1.开启SpringBoot异步支持`

`2.编写有返回值的异步方法`

```JAVA
@Service
public class SmsService {

    @Async		 	//有返回值的异步任务1
    public Future<String> processA(){
        long start = System.currentTimeMillis();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis();
        System.out.println("子线程所用时间："+(end-start));
        return new AsyncResult<String>("process A");
    }

    @Async			//有返回值的异步任务2
    public Future<String> processB(){
        long start = System.currentTimeMillis();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis();
        System.out.println("子线程所用时间："+(end-start));
        return new AsyncResult<String>("process B");
    }

}
```



`3.调用`

```JAVA
   	@ResponseBody
    @GetMapping("asc2")
    public String asc2() throws Exception {
        long start = System.currentTimeMillis();
        Future<String> processA = smsService.processA();
        Future<String> processB = smsService.processB();
        String reuslt = processA.get() + " " + processB.get();
        long end = System.currentTimeMillis();
        System.out.println("主线程："+(end-start));
        return reuslt;
    }
```

==注意：通过Future的静态方法get( )可以获取异步任务的返回值；**主线程调用异步方法的时候会有略微的阻塞，因为会将异步任务的返回值返回给主线程**==



#### 3.10.2 定时任务

​						在实际开发中，需要在每个固定的时间进行去执行一个任务，例如服务器在每晚定时进行备份，为了实现上述功能需求，可以使用Spring框架提供定时任务来实现。



> Cron表达式:从左到右（用空格隔开）：秒  分 时   日  月   周

| 字段 | 取值范围                         | 特殊字符   |
| ---- | -------------------------------- | ---------- |
| 秒   | 0~59的整数                       | , - * /    |
| 分   | 0~59的整数                       | , - * /    |
| 时   | 0~23的整数                       | , - * /    |
| 日   | 1~31的整数                       | ,- * ? / L |
| 月   | 1~12的整数或者 JAN-DEC           | , - * /    |
| 周   | 1~7的整数或者 SUN-SAT(1是星期天) | ,- * ? / L |

```ABAP
特殊常用字符:
			*  :表示匹配该域的任意值
			?  :只能用在日和周两个域(只能同时出现一个).它也匹配域的任意值
			-  :表示范围:  	  			 		  eg(在分钟域用):5-20表示5到20分钟每分钟执行一次
			,  :表示列出枚举值: 			   		eg(在分钟域用):5,20表示5和20分钟时执行
			L  :表示最后,只能用在日和周两个域:	   eg(在星期域用): 5L表示最后一个星期4执行
```





##### 3.10.2.1 使用定时任务

`1.SpringBoot开启定时任务支持`

```JAVA
@SpringBootApplication
@EnableScheduling       	//开启定时任务支持
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```



`2.编写定时任务`

```JAVA
package com.eobard.task;

@Component
public class BackupsTask {

    //表示每天凌晨0:0:0执行一次
    @Scheduled(cron = "0 0 0 * * ?")	
    public void backUp(){
        System.out.println("正在备份数据库");
    }
}
```

==**注意：有些定时任务可以配合使用无返回值的异步任务来使用来提高性能**==



### 3.11 邮件发送

1. 打开QQ邮箱点击设置->账户 :往下找到协议,**开启IMAP/SMTP协议开启即可**,根据提示步骤会得到授权码**并保存自己的授权码**
2. 导入依赖

```xml-dtd
    <!--邮件-->
     <dependency>
            <groupId>com.sun.mail</groupId>
            <artifactId>javax.mail</artifactId>
            <version>1.6.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```

3. 编写全局配置文件

```properties
#发件人邮箱服务器相关设置
spring.mail.host=smtp.qq.com
spring.mail.port=587

#配置个人QQ邮箱和授权密码
spring.mail.username=自己的邮箱
spring.mail.password=自己的授权码
```

> 本账号的授权码：tgtoiaqfdctfdjhc



#### 3.11.1  发送纯文本邮件

 `1.编写邮件类`

```JAVA
package com.eobard.task;

@Repository
public class SendEmail {

    @Resource
    private JavaMailSenderImpl javaMailSender;

    @Value("${spring.mail.username}")
    private String from;//发件人

    /**
     * 发送纯文本邮件
     * @param to      收件人邮箱
     * @param subject 主题
     * @param text    内容
     * @return
     */
    public boolean sendSimpleEmail(String to, String subject, String text) {
        try {
            SimpleMailMessage message = new SimpleMailMessage();
            message.setFrom(from);
            message.setTo(to);
            message.setSubject(subject);
            message.setText(text);
            //发送邮件
            javaMailSender.send(message);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }
}
```



`2.测试`

```JAVA
 	@Test
    public void test1(){
        sendEmail.sendSimpleEmail("2209473452@qq.com","headtitle","hello");
    }
```



#### 3.11.2 发送附件和图片邮件

 `1.编写邮件类`

```JAVA
@Repository
public class SendEmail {

    @Resource
    private JavaMailSenderImpl javaMailSender;

    @Value("${spring.mail.username}")
    private String from;//发件人


    /**
     * @param to            收件人邮箱
     * @param subject       主题
     * @param text          内容:可使用HTML5的相应标签
     * @param imgPath       图片路径: 可空,表示不发送图片
     * @param filePath      附件路径: 可空,表示不发送附件
     * @return
     */
    public boolean sendComplexMail(String to,String subject, String text,String imgPath,String filePath){
        MimeMessage message=javaMailSender.createMimeMessage();
        //初始化邮件内容为HTML5
        StringBuilder HTML=new StringBuilder("");
        try {
            MimeMessageHelper helper=new MimeMessageHelper(message,true);
            helper.setFrom(from);
            helper.setTo(to);
            helper.setSubject(subject);
            //将内容追加
            HTML.append(text);
            //存在图片则上传图片
            if(!StringUtils.isEmpty(imgPath)){
                //用系统毫秒作为cid：一个唯一标识发送图片
                String imgId= String.valueOf(System.currentTimeMillis());
                //将图片追加
                HTML.append("<img src='cid:"+imgId+"'/><br />");
                //设置邮件静态资源
                FileSystemResource imgSystemResource = new FileSystemResource(new File(imgPath));
                //发送主体内容
                helper.setText(HTML.toString(),true);
                helper.addInline(imgId,imgSystemResource);
            }

            //存在附件则上传附件
            if(!StringUtils.isEmpty(filePath)){
                //设置邮件附件
                FileSystemResource fileSystemResource = new FileSystemResource(new File(filePath));
                String fileName=filePath.substring(filePath.lastIndexOf(File.separator));
                helper.addAttachment(fileName,fileSystemResource);
            }

            //发送邮件
            javaMailSender.send(message);
            return true;
        } catch (MessagingException e) {
            e.printStackTrace();
            return false;
        }
    }

}
```



`2.测试`

```JAVA
 @Test
    public void test2(){
        String imgPath="C:\\Users\\九龙坡郭富城\\Desktop\\1.png";
        String filePath="C:\\Users\\九龙坡郭富城\\Desktop\\究极风暴4快捷键.txt";
        sendEmail.sendComplexMail("2209473452@qq.com","title","<h1>123</h1>",imgPath,filePath);
    }
```



#### 3.11.3  发送模板邮件(推荐使用)

`1.导入Thymeleaf依赖`

```xml-dtd
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
 </dependency>	
```



`2.编写Thymeleaf模板页`

```HTML
<!--页面名称:EmailTemplate.html-->
<!DOCTYPE html>
<html lang="en" xmlns="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>邮件模板</title>
</head>
<body>
        <div>
            <span th:text="${userName}"  style="color: red">XXX</span>&nbsp;先生/女士，您好：
        </div>
        <p style="text-indent:2em">
           您的新用户验证码为
            <span th:text="${code}" style="color: cornflowerblue">123456</span>,请妥善保管。
        </p>
</body>
</html>
```

==注意：该模板页面放在resource/templates文件夹下==



`3.编写邮件类`

```JAVA
@Repository
public class SendEmail {

    @Resource
    private JavaMailSenderImpl javaMailSender;

    @Value("${spring.mail.username}")
    private String from;//发件人

   
    /**
     * @param to       收件人邮箱
     * @param subject   主题
     * @param text      模板
     * @param filePath  附件路径:可空,表示不发送附件
     * @return
     */
    public boolean sendTemplate(String to,String subject, String text,String filePath){
        MimeMessage message=javaMailSender.createMimeMessage();
        try {
            MimeMessageHelper helper=new MimeMessageHelper(message,true);
            helper.setFrom(from);
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(text,true);
            //存在附件则上传附件
            if(!StringUtils.isEmpty(filePath)){
                //设置邮件附件
                FileSystemResource fileSystemResource = new FileSystemResource(new File(filePath));
                String fileName=filePath.substring(filePath.lastIndexOf(File.separator));
                helper.addAttachment(fileName,fileSystemResource);
            }
            //发送邮件
            javaMailSender.send(message);
            return true;
        } catch (MessagingException e) {
            e.printStackTrace();
            return false;
        }
    }
}
```



`4.测试发送邮件`

```JAVA
    @Resource
    private TemplateEngine templateEngine;//注入thymeleaf模板引擎

    @Test
    public void test(){
        Context context=new Context();
        context.setVariable("userName","波仔");//根据thymeleaf的EL表达式设置值
        context.setVariable("code","987575");//根据thymeleaf的EL表达式设置值
        //关联thymeleaf模板:参数1是thymeleaf的文件名,参数2是Context对象
        String text = templateEngine.process("EmailTemplate", context);
        sendEmail.sendTemplate("2445995527@qq.com","标题",text,"附近的绝对路径地址");
    }
```



### 3.12 前后端分离验证JWT

`1.导入依赖`

```xml-dtd
    <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.5.0</version>
        </dependency>
```



`2.创建拦截器`

```JAVA
package com.eobard.interceptor;

@Component
public class JWTInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //获取前端请求header中的JWT令牌
        String token = request.getHeader("token");
        //如果想要传一些提示信息到前台,可以用response+json传回即可

        //验证JWT令牌是否有效:
        //                  有效则放行处理
        return JWTUtils.vertify(token)!=null?true:false;
    }
}
```

> 该拦截器用于拦截请求中JWT令牌是否有效才放行



`3.注册拦截器`

```JAVA
package com.eobard.config;

@Configuration
public class WebConfig extends WebMvcConfigurationSupport {

    @Resource
    private JWTInterceptor jwtInterceptor;

    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(jwtInterceptor)
                .addPathPatterns("/user/**")    //拦截/user/下的所有请求
                .excludePathPatterns("/login"); //排除登录请求:要生成token
    }
}
```

> 后期可以根据路径拦截请求，但是登录不能拦截：会生成JWT令牌



`4.注意事项`

​			后期开发中，前端传入token令牌的时候，应该把JWT令牌放入请求header中！

```HTML
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.js"></script>
<script>
    //模拟从本地获取JWT令牌
  localStorage.setItem("token","eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2Mzg3NjQwNzMsInVzZXJuYW1lIjoiZW9iYXJkIn0.hZj9e4OBeqczW8bW1jF4H5Vc403pwnMAn98fIpK_PhI")
    var item = localStorage.getItem("token");

    //ajax请求附带上JWT令牌
    $.ajax({
        type: "GET",
        url: "/user/get" ,
        headers: {'token': item}
    });
</script>
```

==详细使用见**`JWT使用2.3节`**==





### 3.13 Spring AOP的使用

> 概述：一种通过预编译和运行时动态代理的方式，在不修改源代码动态添加新功能



**原理**： 1. 将复杂的需要分解不同方面，将公共功能集中解决

​				2.采用动态代理机制，不改变源程序基础上，对代码进行增强处理  



#### **AOP相关术语**

* 切面：切面是通知和切点的结合，共同定义了切面的全部内容----它是什么，在何时何处完成其功能

  ​	实际上切面是一段程序代码，这段代码将被植入到程序流程中。



* 通知(Advice)：定义了切面是什么以及何时使用，还解决了何时执行。
  * 前置增强(Before):		在连接点方法调用之前处理
  * 后置返回增强(AfterReturning):	在连接点调用方法成功后处理
  * 异常增强(AfterThrowing):		在连接点方法抛出异常后处理
  * 最终增强(After):			在连接点方法调用后无论成功与否再处理
  * 环绕增强(Around):		在连接点方法调用前和调用后自定义;<font color="red">功能最强，包含了前面4种(用了环绕就不能用前面4种；反之)</font>



* 连接点对象：程序流程上的任意一点，对象的某一个操作，对象调用某一个方法，切面代码可以利用这点插入到应用的正常流程之中。

  ```java
  JoinPoint:
      getTarget()						//获取当前连接点对象的类
      getSignature().getName();		//获取当前连接点对象的方法名
      getArgs();						//获取当前连接点对象方法的参数列表
  
  ProceedingJoinPoint(用于环绕增强,方法同上):
  	proceed();						//该方法之前为前置增强逻辑,之后为后置返回增强逻辑
  	(MethodSignature) jp.getSignature();	//获取方法签名
  ```
  
  > **切入点：所有连接点的集合**



* 切点表达式(execution("表达式"))

```java
* com.service.*.*(..)  			//匹配com.service包下所有类的成员方法
* com.service..*.*(..) 			//匹配com.service包及其子包下所有类的成员方法
* com.service.Service.*()		//匹配com.service包下实体类为Service的任意无形参方法
* com.service.Service*.*()  	//匹配com.service包下实体类为Service开头的任意无形参方法
* com.service.Service*.*(..)	//匹配com.service包下实体类为Service开头的形参个数自定义的方法

public * com.service.BaseService.do(java.lang.String,..)
//匹配com.service包下实体类为BaseService中 public的第一个形参必须为String的do方法
```

==注意：其中在包名前面的第一个 `*`  代表的是任意一种返回类型，====若第一个`*`前面没有修饰符则代表所有类型的修饰方法（public、private、protected）都可以匹配。==



* 切点表达式(within("表达式"))

```java
com.eobard.service.*		//匹配com.eobard.service包下的所有类
com.eobard.service..*    	//匹配com.eobard.service包及其子包下的所有类
```

> **注：within针对的是某个类，粗粒度；execution精确的是某个方法，细粒度**





#### 细化切入点范围，提高性能

```java
//切入点:controller包下的所有类并且有LogInfo注解的所有方法
@Around("within(com.eobard.controller.*) && @annotation(com.eobard.annotation.LogInfo)")
```





#### 简单使用

1. 导入依赖

```xml
  <!--导入aop依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```



2. 编写切面

```java
package com.eobard.aspect;

@Component
@Aspect //标注为一个切面
@Slf4j
public class ApiLogAspect {

    //切入点: com.eobard.api包下所有实体类的所有方法
    @Pointcut("execution(* com.eobard.api.*.*(..))")
    public void pointCut() {
    }


    //环绕增强
    @Around("pointCut()")
    public Object around(ProceedingJoinPoint jp) {
        Object result = null;

        //开始时间
        long start = System.currentTimeMillis();

        try {
            log.error("开始调用当前控制器:{},方法名为:{},请求参数列表为:{}", jp.getTarget(), jp.getSignature().getName(), Arrays.toString(jp.getArgs()));

            result = jp.proceed();

            //结束时间
            long end = System.currentTimeMillis();
            //总耗时
            long total = end - start;
            log.error("结束调用当前控制器:{},方法名为:{},总耗时:{}ms", jp.getTarget(), jp.getSignature().getName(), total);
            log.error("结束调用当前控制器:{},方法名为:{},方法返回结果:{}",jp.getTarget(), jp.getSignature().getName(),result);
        } catch (Throwable e) {
            System.out.println("异常增强" + e.getMessage());
            e.printStackTrace();
        } finally {
            System.out.println("最终增强");
        }
        return result;
    }
}
```



3. 控制器方法

```java
@RestController
public class UserApi {


    @GetMapping("/get/{id}")
    public Map getUserById(@PathVariable Long id) {
        Map<String, Object> map = new HashMap();
        map.put("name", "zs");
        map.put("id", id);
        return map;
    }

    @GetMapping("list")
    public List<ObjectItem> objectItems() {
        List list = new ArrayList();

        for (int i = 0; i < 4; i++) {
            ObjectItem item = new ObjectItem();
            item.setSize(1000L + i);
            item.setObjectName("item_0" + i);
            list.add(item);
        }

        return list;
    }

}
```



4. 测试效果

```bash
2023-07-26 22:58:29.690 ERROR 19488 --- [nio-8080-exec-1] com.eobard.aspect.ApiLogAspect           : 开始调用当前控制器:com.eobard.api.UserApi@39954249,方法名为:objectItems,请求参数列表为:[]
2023-07-26 22:58:29.696 ERROR 19488 --- [nio-8080-exec-1] com.eobard.aspect.ApiLogAspect           : 结束调用当前控制器:com.eobard.api.UserApi@39954249,方法名为:objectItems,总耗时:7ms
2023-07-26 22:58:29.696 ERROR 19488 --- [nio-8080-exec-1] com.eobard.aspect.ApiLogAspect           : 结束调用当前控制器:com.eobard.api.UserApi@39954249,方法名为:objectItems,方法返回结果:[ObjectItem(objectName=item_00, size=1000), ObjectItem(objectName=item_01, size=1001), ObjectItem(objectName=item_02, size=1002), ObjectItem(objectName=item_03, size=1003)]
最终增强

2023-07-26 23:00:38.338 ERROR 19488 --- [nio-8080-exec-7] com.eobard.aspect.ApiLogAspect           : 开始调用当前控制器:com.eobard.api.UserApi@39954249,方法名为:getUserById,请求参数列表为:[3]
2023-07-26 23:00:38.338 ERROR 19488 --- [nio-8080-exec-7] com.eobard.aspect.ApiLogAspect           : 结束调用当前控制器:com.eobard.api.UserApi@39954249,方法名为:getUserById,总耗时:0ms
2023-07-26 23:00:38.338 ERROR 19488 --- [nio-8080-exec-7] com.eobard.aspect.ApiLogAspect           : 结束调用当前控制器:com.eobard.api.UserApi@39954249,方法名为:getUserById,方法返回结果:{name=zs, id=3}
最终增强
```







##### 注意

> #### **上面的切面也可以不用环绕通知去处理，可以达到同样的效果**

```java
package com.eobard.aspect;

@Component
@Aspect
@Slf4j
public class ControllerLogAspect {

    //切入点: com.eobard.api包下所有实体类的所有方法
    @Pointcut("execution(* com.eobard.api.*.*(..))")
    public void pointCut() {
    }

    private Long start;	//开始时间
    private Long end;	//结束时间

    //前置增强:在连接点方法调用之前处理
    @Before("pointCut()")
    public void before(JoinPoint jp) {
        start=System.currentTimeMillis();
        log.error("开始调用当前控制器:{},方法名为:{},请求参数列表为:{}", jp.getTarget(), jp.getSignature().getName(), Arrays.toString(jp.getArgs()));
    }


    //后置返回增强:在连接点调用方法成功后处理
    @AfterReturning(pointcut="pointCut()",returning ="result")
    public void afterReturning(JoinPoint jp,Object result) {
        end=System.currentTimeMillis();
        //总耗时
        long total = end - start;
        log.error("结束调用当前控制器:{},方法名为:{},方法返回结果:{},调用总耗时:{}ms", jp.getTarget(), jp.getSignature().getName(),result,total);
    }


    //后置最终增强:在连接点方法调用后无论成功与否再处理
    @After("pointCut()")
    public void afterFinally(JoinPoint jp) {
        System.out.println("最终增强");
    }

    //异常增强:在连接点方法抛出异常后处理
    @AfterThrowing(pointcut="pointCut()",throwing ="e" )
    public void afterThrowing(JoinPoint jp,RuntimeException e) {
        System.out.println("异常增强" + e.getMessage());
    }
}
```



测试结果

```bash
2023-07-26 23:16:25.290 ERROR 7236 --- [nio-8080-exec-1] com.eobard.aspect.ControllerLogAspect    : 开始调用当前控制器:com.eobard.api.UserApi@12a8b450,方法名为:getUserById,请求参数列表为:[10]
2023-07-26 23:16:25.297 ERROR 7236 --- [nio-8080-exec-1] com.eobard.aspect.ControllerLogAspect    : 结束调用当前控制器:com.eobard.api.UserApi@12a8b450,方法名为:getUserById,方法返回结果:{name=zs, id=10},调用总耗时:7ms
最终增强

2023-07-26 23:16:43.717 ERROR 7236 --- [nio-8080-exec-3] com.eobard.aspect.ControllerLogAspect    : 开始调用当前控制器:com.eobard.api.UserApi@12a8b450,方法名为:objectItems,请求参数列表为:[]
2023-07-26 23:16:43.718 ERROR 7236 --- [nio-8080-exec-3] com.eobard.aspect.ControllerLogAspect    : 结束调用当前控制器:com.eobard.api.UserApi@12a8b450,方法名为:objectItems,方法返回结果:[ObjectItem(objectName=item_00, size=1000), ObjectItem(objectName=item_01, size=1001), ObjectItem(objectName=item_02, size=1002), ObjectItem(objectName=item_03, size=1003)],调用总耗时:1ms
最终增强
```





#### 自定义注解+AOP

1. 自定义日志注解

```java
@Retention(RetentionPolicy.RUNTIME) //注解的生命周期:运行时
@Target(ElementType.METHOD)         //作用域方法上
public @interface LogInfo {

    String operationInfo() default "";			//操作日志

    LogType type() default LogType.CONSOLE;		//日志打印类型
}
```



2. 日志打印类型枚举类

```java
public enum LogType {

    CONSOLE,    //控制台打印
    FILE,       //文件输出
    EMAIL       //邮件通知
}
```



3. 切面

```java
@Component
@Aspect //标注为一个切面
@Slf4j
public class LogAspect {

    //切入点:针对所有方法上有LogInfo注解的方法
    @Pointcut("@annotation(com.eobard.annotation.LogInfo)")
    public void pointCut() {}

    @Around("pointCut()")
    public Object around(ProceedingJoinPoint jp) throws Throwable {
        //获取方法签名
        MethodSignature methodSignature = (MethodSignature) jp.getSignature();
        
        //获取当前方法名
        Method method = methodSignature.getMethod();
        
        //获取当前方法的自定义注解
        LogInfo logInfo = method.getAnnotation(LogInfo.class);

        if (!ObjectUtils.isEmpty(logInfo)) {
            //获取操作记录
            String info = logInfo.operationInfo();

            //获取日志记录类型
            LogType logType = logInfo.type();

            switch (logType) {
                case CONSOLE:
                    String now = DateTime.now().toString("yyyy-MM-dd HH:mm:ss");
                    log.error("当前时间:{},调用控制器:{},方法名为:{},请求参数列表:{},操作日志:{}", now, jp.getTarget(), method.getName(), Arrays.asList(jp.getArgs()), info);
                    break;
                case FILE:
                    //省略写入文件操作
                    break;
                case EMAIL:
                    //省略写入邮件操作
                    break;
            }
        }
        return jp.proceed();
    }
}
```

> **注：还可以在切入点中直接填方法的自定义注解的形参名即可，以上方法可以简写为**

```java
@Component
@Aspect
@Slf4j
public class LogAspect {

    //直接在切点表达式中填写自定义注解的形参名,在连接点对象后必须加入自定义注解的形参,两个变量名要一致
	@Around("@annotation(logInfo)")
    public Object around(ProceedingJoinPoint jp,LogInfo logInfo) throws Throwable {
      		......
        
            //获取操作记录
            String info = logInfo.operationInfo();

            //获取日志记录类型
            LogType logType = logInfo.type();

        	......
        	return jp.proceed();
    }
}
```







4. 方法上使用注解

```java
@RestController
public class UserController {

    //使用自定义注解
    @LogInfo(operationInfo = "根据id获取用户",type = LogType.CONSOLE)
    @GetMapping("get/{id}")
    public Map getUserById(@PathVariable Long id){
        Map<String,Object> map=new HashMap();
        map.put("id",id);
        map.put("name","eobard");
        map.put("age","23");
        return map;
    }

}
```



5. 运行效果

```java
2023-08-03 18:29:56.359 ERROR 6300 --- [nio-8080-exec-1] com.eobard.aop.LogAspect                 : 当前时间:2023-08-03 18:29:56,调用控制器:com.eobard.controller.UserController@4eb55259,方法名为:getUserById,请求参数列表:[20],操作日志:根据id获取用户
```





---

## 四. Spring Boot 原理相关

### 4.1 启动类纳入容器问题

> ### 1. 为什么Spring Boot启动的时候可以将所有的类加入容器中?

> ​		 因为在Spring Boot的启动器的注解上存在一个@ComponetScan的注解，它默认会扫描当前SpringBoot启动类所在的包，这也是为什么要将Spring Boot的启动器就放在其它业务层包的平级下，如 controller包的同级下



---

### 4.2 Spring Boot自动装配原理

>  1. 在springboot启动类的`@SpringBootApplication`注解中，里面有个`@EnableAutoConfiguration`注解，再次点入刚刚的注解里面有个`@AutoConfigurationPackage`注解，它的作用是将使用了该注解的类所在的包及其子包下所有组件扫描到Spring IOC容器中；里面还有一个`@Import`注解，它导入的并不是一个Configuration配置类，而是一个`AutoConfigurationImportSelector`类，而这个类实现了`ImportSelector`接口
>
>     ```java
>     @SpringBootConfiguration
>     @EnableAutoConfiguration
>     public @interface SpringBootApplication {}		//省略其它
>     
>     
>     @AutoConfigurationPackage
>     @Import({AutoConfigurationImportSelector.class})
>     public @interface EnableAutoConfiguration {}	//省略其它
>     ```
>
>     
>
>  2. `AutoConfigurationImportSelector`类中有一个`selectImports`的抽象方法并返回一个String[ ]，这个数组可以指定需要装配到Spring IOC容器的类，在`@Import`注解导入传进来的这个实现类的时候，会将实现类中返回的Class名称都装配到SPring IOC容器中
>
>     ```java
>         public String[] selectImports(AnnotationMetadata annotationMetadata) {
>             if (!this.isEnabled(annotationMetadata)) {
>                 return NO_IMPORTS;
>             } else {
>                 AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
>                 return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
>             }
>         }
>     ```
>
>     ==注意：通过实现`ImportSelector`注解传入`@Import`中可以决定哪些Bean的可以被Ioc容器选择性装配==

> ​	2)  这个AutoConfigurationImportSelector会根据getImportGroup的方法判断返回的类，如果实现了Group接口，它就会去**调用process方法里面又有一个获取所有的有效自动配置类的getAutoConfigurationEntry的方法**
>
> ​	3）之后它会**进入getCandidateConfigurations方法获取所有的配置类，总共有134个并放入集合中**，它们都是Spring boot的所有内置starter，都是叫XXXAutoConfiuration结尾，实际上它们都是Spring容器配置类
>
> ​	4）就比如常用的设置端口这个容器配置类， 它对应的实体类叫做ServerProperties，然后它类上面有一个@ConfigurationProperties注解它会将我们全局properties文件或者yml文件进行绑定，这也是为什么在全局配置文件配置了，Spring Boot能够自动更改的原因，就是因为在全局配置文件中，它们都有对应的XXXProperties的实体类，都是通过@ConfigurationProperties来进行一 一绑定的
>
> ​	5）然后**又会去调用一个SpringFactoriesLoader.loadFactoryNames的方法，它会从所有的jar包和类路径去找META-INF/spring.factories里面的文件**，这里面都是K-V的，如果我们后期要自定义一个starter，我们只需要在这个spring.factories里面去写上K-V形式的就可以了
>
> ​	6）找到所有的自动配置类过后，回到getAutoConfigurationEntry的方法，它又会走到一个叫做**getConfigurationClassFilter().filter()的方法，将之前的有效自动配置类根据你pom文件依赖的坐标starter，它会过滤出有效的配置类**，将其他的配置类过滤开就会得到最终自动装配好的配置类



---

### 4.3 启动原理问题

> ### 	**3. Spring Boot启动原理**

> ​		1)  执行Spring Boot项目创建的启动器run方法时：**初始化SpringApplication，同自动配置原理一样在构造方法中去读取spring.factories文件里面的listener和ApplicationContextInitializer**

```JAVA
 public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		//initial....
     
	   //读取spring.factories中的ApplicationContextInitializer             	
     this.setInitializers(				 this.getSpringFactoriesInstances(ApplicationContextInitializer.class));	
  	  //读取spring.factories中的listener包括一些热部署监听器、自动装配的等等
       this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));		
}
```



> ​	2)  运行**run方法**也是最关键的方法:**读取全局环境变量，配置信息,打印springboot的横幅**，计算启动时间等等

```JAVA
  public ConfigurableApplicationContext run(String... args) {
	// 创建一个StopWatch实例，用来记录SpringBoot的启动时间
 	StopWatch stopWatch = new StopWatch();

	//initial....

	//获取环境变量等等并绑定到SpringApplication中
	 ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);		

}
```



> ​	3) **创建SpringApplication上下文**：根据你是SERVLET还是REACTIVE来创建相应的上下文，默认是AnnotationConfigServletWebServerApplicationContext

```JAVA
public ConfigurableApplicationContext run(String... args) {
 	ConfigurableApplicationContext context = null;
	//initial....

	//创建上下文
	 context = this.createApplicationContext();
	//根据你环境时servlet还是reactive来将启动类放入上下文中		
     context.setApplicationStartup(this.applicationStartup);	
}
```



> ​	4) 预初始化上下文，读取启动类：也就是在new SpringApplication的构造方法中去推断当前运行线程中mian方法的类

```JAVA
 public ConfigurableApplicationContext run(String... args) {
	//initial....

 	SpringApplicationRunListeners listeners = this.getRunListeners(args);	
	//获取当前项目的启动类信息	
    listeners.starting(bootstrapContext, this.mainApplicationClass);		
}
```



> ​	5) **调用refresh方法加载ioc容器 ：就是将上面spring.factories中读取到的K-V,加载所有的自动配置类，创建相应的servlet容器**

```java
 public ConfigurableApplicationContext run(String... args) {
	//initial....
 	this.refreshContext(context);
    this.afterRefresh(context, applicationArguments);
}
```



> 6)  **在这些过程中，spring boot会调用很多的监听器去对外进行扩展**



---

### 4.4 SPI机制

> ### 4. JDK SPI机制和Spring Boot使用SPI

> #### 	JDK SPI机制：
>
> ​				服务提供商安装约定, 将具体的实现类名称放到/META-INF/services/xxx(顶级接口名)下, ServiceLoader就可以根据服务提供者的意愿, 加载不同的实现了, 避免硬编码写死逻辑, 从而达到解耦的目的.如 JDBC的DriverManager

`1.创建maven普通项目`

`2.创建查询的接口`

```JAVA
package com.eobard.service;

public interface Search {
      void searchInfo();
}
```



`3.创建查询的具体实现类`

```JAVA
//数据库查询
package com.eobard.service.impl;

public class DBSearchImpl implements Search {
    public void searchInfo() {
        System.out.println("from DB search..");
    }
}


//文件系统查询
package com.eobard.service.impl;

public class FileSearchImpl implements Search {
    public void searchInfo() {
        System.out.println("from local disk search..");
    }
}
```



`4.在resources文件夹下创建META-INF/services两个文件夹`

`5.在META-INF/services文件夹下创建顶层父接口限定名文件:com.eobard.service.Search,并在里面配置具体实现类`

```
com.eobard.service.impl.DBSearchImpl
com.eobard.service.impl.FileSearchImpl
```

`6.测试`

```JAVA
public class Test {

    @org.junit.Test
    public void test(){
   
        ServiceLoader<Search> loader = ServiceLoader.load(Search.class);
        for (Search search : loader) {
            //执行所有在com.eobard.service.Search文件中配置的实现类中的方法
            search.searchInfo();		
        }
    }
    
     @org.junit.Test
    public void test2() {
        ServiceLoader<Search> loader = ServiceLoader.load(Search.class);
        for (Search search : loader) {
            //根据自定义类型执行对应的方法
            if (search instanceof DBSearchImpl) {
                search.searchInfo();
            }
        }
    }
}
```

==注意：SPI机制已经定义好了加载服务的流程框架, 你只需要按照约定, 在META-INF/services目录下面,以接口的全限定名称为名创建一个文件(com.eobard.service.Search), 文件里面配置具体的实现类的全限定名称==





> #### 	Spring Boot使用SPI机制

> ​	Spring Boot在自动装配和启动原理中也同样使用了SPI机制

```
				SpringFactoriesLoader类所做的事情：
1. FACTORIES_RESOURCE_LOCATION: 正是指向我们上面所说的META-INF/spring.factories

2. loadFactories():  从META-INF/spring.factories查找指定的接口实现类并实例化, 其中查找是通过调用loadFactoryNames()

3. loadFactoryNames():从指定的位置查找特定接口的实现类的全限定名称 其中就是调用loadSpringFactories()一个一个的进去读取

4. instantiateFactory(): 实例化并且在实例化之前还要去重操作和检查是否是该类的实现类
```

==注意: 在SpringFactoriesLoader中有一个方法==

```JAVA
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		//获取当前线程的classloader
        ClassLoader classLoader = this.getClassLoader();
		//利用names去重，这里就是类似于"ServiceLoader"类，它是SpringFactoriesLoader
        Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		//利用反射实例化
        List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
        AnnotationAwareOrderComparator.sort(instances);
		//返回接口所有实例
        return instances;
    }

```



---

### 4.5 Servlet容器问题

> ### 	**5. 为什么可以通过配置的依赖自动使用不同的servlet容器**

> ​		因为在ServletWebServerFactoryAutoConfiguration类中通过注解导入了不同的容器配置类，然后每个配置类都有@ConditionalOnClass注解来根据当前的依赖自动选择不同的servlet容器，并且根据不同的servlet容器配置了servlet工厂类来启动servlet容器

```JAVA
//ServletWebServerFactoryAutoConfiguration类类导入了三个不同的web容器
@Import({EmbeddedTomcat.class, EmbeddedJetty.class, EmbeddedUndertow.class})
public class ServletWebServerFactoryAutoConfiguration {
    //....
}


//如tomcat容器
class ServletWebServerFactoryConfiguration {
   
    //依赖中如果有tomcat依赖这个注解就会生效,springboot就会使用tomcat容器
    @ConditionalOnClass({Servlet.class, Undertow.class, SslClientAuthMode.class})
    @ConditionalOnMissingBean(
        value = {ServletWebServerFactory.class},
        search = SearchStrategy.CURRENT
    )
    static class EmbeddedUndertow {
        
    }
    
 //tomcat如何启动的
public class TomcatServletWebServerFactory{
    
    //关键方法：启动tomcat容器
     public WebServer getWebServer(){
     }
}
```


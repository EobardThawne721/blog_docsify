# Spring MVC

## 一. 环境搭建

​		 项目使用maven项目构建，需要添加依赖以及配置相应环境文件

​		 ``a. pom.xml文件中添加依赖``

```xml-dtd
   <!--配置全局版本和maven编译版本-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!-- spring版本-->
        <spring.version>5.2.8.RELEASE</spring.version>
    </properties>


    <dependencies>
        <!-- springcontext -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- spring web-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- springmvc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- jsp-->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
            <scope>provided</scope>
        </dependency>
        <!-- servlet-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
```

​		 ``b.web.xml文件中添加配置``

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" version="4.0">

    <!-- 配置Spring MVC核心控制器 -->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <!-- 核心控制器 -->
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!-- 配置Servlet的初始化参数，读取spring mvc的配置文件，创建spring容器 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!-- 表示容器再启动时立即加载servlet -->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <!--配置Spring MVC url映射路径-->
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <!-- 所有请求都需要经过该核心控制器 -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>


    <!--配置解决中文乱码的过滤器-->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <!--配置过滤器的url映射路径-->
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
```

​		==**注意：DispatcherServlet(中央调度器)实体类的父类是HttpServlet，所以它初始化的时候也会执行init( )方法，并且在init( )方法中会创建WebApplicationContext的IOC环境，并且将WebApplicationContext容器放入this.getServletContext.setAttribute( )全局作用域中**==



​			``c.添加springmvc.xml文件``

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans
        xmlns="http://www.springframework.org/schema/beans"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!--扫描注解所在的包-->
    <context:component-scan base-package="com.eobard"/>

    <!--启用mvc注解支持-->
    <mvc:annotation-driven />


    <!--配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--页面前缀:表示所有的页面在webapp里面的WEB-INF里面的pages文件夹里的 -->
        <property name="prefix" value="/WEB-INF/pages/"/>
        <!--页面后缀:给页面名称拼接后缀为 .jsp-->
        <property name="suffix" value=".jsp"/>
    </bean>


            
        <!-- 加载静态资源 -->
        <!-- mapping属性：将静态资源映射到指定的路径下 -->
        <!-- location属性：本地静态资源文件所在的目录 -->
        <mvc:resources mapping="/statics/**" location="/statics/"/>
</beans>
```

​			==注意：建议把页面放在WEB-INF文件夹的下一层，优点：普通的request请求是不能访问的，但是有了springmvc的视图解析器可以访问到==

​			``d.新建一个statics文件夹放静态资源文件``

> ​				在Spring MVC中，当web.xml配置文件的DispatcherServlet前端控制器请求映射为"/"时，Spring MVC会
> ​                捕获Web容器所有的请求，包括静态资源的请求。Spring MVC会将它们当成一个普通请求处理，但是由
> ​                于找不到对应的处理器，所以按照常规的方式引用静态文件将无法正常访问。
> ​                **可以采用 <mvc:resources /> 标签即可解决静态资源无法访问问题，为了方便配置管理，建议将项**
> ​                **目中所有的静态资源文件 (js,css,images) 统一放到一个目录下 (如：webapp/statics) 。**
> ​          

​			``e.创建控制器``

```JAVA
@Controller
public class HelloController {

}
```

> controller使用注意：1. 控制器放在controller包下
> 									2.控制器以Controller结尾
>                         			   **3. 控制器加上@Controller注解注入ioc**
> 									4.控制器里面的方法可以返回多种不同类型的返回值，不一定只能返回String，区别于struts2

---

## 二. 请求路径&视图返回

### 2.1 请求路径

​			通过@RequestMapping注解，可以指定访问控制器具体某个方法的url，**每个url必须保证唯一**！

​			==使用注意：**1. @RequestMapping注解会自动匹配get或者post请求**==

​								==2.请求如果是get类型，控制器中@RequestMapping指定为post类型就会报405错误，因为请求类型不对应==

​								==3.里面有method=RequestMethod.XXX来指定具体某种请求==

​								==4.**前台请求的时候url最好加上${pageContext.request.contextPath}/xxx**==

#### 2.1.1 一级路径

​				通过在控制器的某个方法上加上@RequestMapping(value = "/自定义路径")注解即可

```JAVA
@Controller
public class HelloController {

    @RequestMapping("/hello2")
    public String hello2(){
        return "success";
    }

}

```

==注意：这里自定义路径为/hello2，在访问的时候路径就应该写 **http://localhost:8080/hello2** 。==



#### 2.1.2 多级路径

​			在控制器上加上@RequestMapping(value = "/自定义路径")注解表示一级路径，在某个方法上加上@RequestMapping(value = "/自定义路径")注解表示二级路径

```JAVA
@RequestMapping("/hello")
@Controller
public class HelloController {


   
    @RequestMapping("/index")
    public String hello2(){
        return "success";
    }

}

```

==注意：这时候采用了二级路径，在访问的时候路径就应该写 **http://localhost:8080/hello/index** 。==



### 2.2 视图返回

​			使用视图返回注意：返回页面的可以直接简写为页面的名称，不用后缀 。**前提是这个文件交给了配置视图解析器解析，且在springmvc.xml中的前缀里面；若不在里面就需要指定为完整的文件名**



#### 2.2.1 视图

​	SpringMVC解析jsp的时候**默认会使用InternalResourceView**(springmvc.xml中配置的)，如果发现jsp页面中有jstl语言则自动转为它的子类**JstlView**

JstlView：可以解析jstl、实现国际化(针对不同地区，不同国家，进行不同的显示)操作



#### 2.2.2 指定视图名

<font color="green">当页面在视图解析器的前缀文件夹里面</font>

```JAVA
@Controller
public class HelloController {


    //返回视图写法1(推荐使用！！！！)：通过字符串直接指定视图名
    @RequestMapping("/hello2")
    public String hello2(){
        return "success";			//存在视图解析器的前缀里面，直接返回文件名即可
    }

}
```

<font color="green">当页面在webapps的路径下</font>

```JAVA
@Controller
public class HelloController {

    @RequestMapping("/hello2")
    public String hello2(){
        return "/login.jsp";		//没有在视图解析器的前缀里面,存在于根路径,需要写出完整路径和文件名
    }

}
```



#### 2.2.3 ModelAndView对象

```JAVA
@Controller
public class HelloController {

    @RequestMapping(value = "/hello2")
    public ModelAndView hello(){
        //1.创建ModelAndView对象
        ModelAndView view=new ModelAndView();
        //2.设置返回的页面名称(不包含后缀名)
        view.setViewName("success");
        //3.返回对象：表示返回success.jsp页面
        return view;
    }
}
```

### 2.3 SpringMVC与Servlet混合使用

​	只需要在web.xml的url-pattern中的路径改为 .action类似的即可

```XML
    <!--修改Spring MVC url映射路径-->
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <!--所有mvc的请求必须要以.action结尾,否则全部是servlet的请求方式-->
        <url-pattern>.action</url-pattern>
    </servlet-mapping>
```

> 请求类型1： href="/login"  			//表示为servlet方式的请求，需要添加servlet相应的控制器类
>
> 请求类型2：href="/login.action"	//表示为springmvc方式的请求，需要添加相应的控制器类



### 2.4 通过Servlet解决静态资源方案

​		通过前面的配置   <mvc:resources /> 可以完成静态资源的访问，这里我们可以通过Servlet的方式来解决静态资源的访问。**如果springmvc有对应的@requestMapping则交给springmvc处理；如果没有对应的@requestMapping则交给服务器tomcat默认的Servlet处理。**

```xml-dtd
<!--在springmvc.xml文件中增加两个配置-->	
	<mvc:default-servlet-handler></mvc:default-servlet-handler>
	<mvc:annotation-driven ><mvc:annotation-driven />
```



### 2.5 组合路径(重点掌握)

> ​	@GetMapping：匹配GET方式请求
>
> ​	@PostMapping：匹配POST方式请求
>
> ​	@PutMapping：匹配PUT方式请求
>
> ​	@DeleteMapping：匹配DELETE方式请求

==注意：上面的几个注解就等同于之前的@RequestMapping(method=RequestMethod.XXX)==

---

## 三. 页面参数传递

### 	3.1 简单类型传参

```html
    <h1>简单数据传递</h1>
    <form action="/home3/get01" method="post">
        <input type="text"  name="id"> <br>
        <input type="text"  name="name"> <br>
        <input type="text"  name="salary"> <br>
        <input type=submit value="提交">
    </form>
```

```JAVA
@Controller
@RequestMapping("/home3")
public class Hello3Controller {


    @RequestMapping(value = "get01",method = RequestMethod.POST)
    public String  get01(Integer id,String name,Double salary){
        System.out.println("id = " + id);
        System.out.println("name = " + name);
        System.out.println("salary = " + salary);
        return "success";
    }
}
```

==注意：	1.**形参最好是包装类,这样即使不传值也不会报错；如果是基本类型不传值会报错**==
                 ==2.如果是form表单：**input表单的name值要和这里的形参名一致**；==



### 	3.2 实体类类型传参

```HTML
    <h1>类类型传递</h1>
    <form action="/home3/register" method="post">
        <input type="text"  name="id"> <br>
        <input type="text"  name="name"> <br>
        <input type=submit value="提交">
    </form>
```

```JAVA
public class User {
    private Integer id;
    private String name;
    
	//省略getter，setter
}
```

```JAVA
@Controller
@RequestMapping("/home3")
public class Hello3Controller {

     
    @RequestMapping("/register")
    public String register(User user){
        System.out.println("user = " + user);
        return "success";
    }
}
```

==注意：1.如果是form表单：**input表单的name值要和实体类的属性名一致**;==



### 3.3 包装类型传参

```HTML
    <h1>包装类型传递</h1>
    <form action="/home3/register" method="post">
        <input type="text"  name="id"> <br>
        <input type="text"  name="name"> <br>
        <input type="text"  name="order.oid"> <br>
        <input type=submit value="提交">
    </form>
```

```JAVA
public class UserVo extends User {
    private Integer id;
    private String name;
    
    private Order order
	//省略getter，setter
}
```

```JAVA
public class Order {
    private Integer oid;
   
	//省略getter，setter
}
```

```java
@Controller
@RequestMapping("/home3")
public class Hello3Controller {

     
    @RequestMapping("/register")
    public String register(UserVo user){
        System.out.println("userVo = " + user);
        return "success";
    }
}
```

==注意：如果是包装类传值，前台的name值为 **引用对象.属性名** 即可==



### 3.4 数组传参

```JAVA
@RestController
public class UserController {

    @GetMapping("/testArray")
    public String array(Integer[] ids){
        return Arrays.toString(ids);
    }
}
```

> ​	发起请求[localhost:8080/testArray?ids=1&ids=2](http://localhost:8080/testArray?ids=1&ids=2)或者提交表单中多个name值相同的数据，返回数据[1, 2]



### 3.5 @RequestParam

​			该注解可以**解决前端请求参数中与控制器参数名不一致的情况**

```html
 <form action="/home/get" method="get">
        <input type="text"  name="ID"> <br>
        <input type="text"  name="UserName"> <br>
        <input type=submit value="提交">
 </form>
```

```JAVA
@Controller
@RequestMapping("/home")
public class UserController {


    @RequestMapping(value = "get",method = RequestMethod.GET)
    public String  get(@RequestParam(value="ID")Integer id, @RequestParam(value="UserName")String name){
        System.out.println("id = " + id);
        System.out.println("name = " + name);
        return "success";
    }
}
```

==注意：@RequestParam注解默认为必填(如果前端不传值会400异常)，可以通过其中的required属性设置为非必填，eg：@RequestParam(value="ID",required="false")==



### 3.6 @CookieValue获取Cookie

​			cookie：服务端在接收客户端第一次请求的时候，会给客户端分配一个session，该session包含了一个sessionId，当服务端响应客户端的时候会将服务端的sessionId保存在cookie的JSessionId中发送给客户端。

```HTML
  <a href="/cookie">获取cookie的值</a>
```

```JAVA
@Controller
public class CookieController{
    
	   //获取客户端的JSessionId的值
	   @RequestMapping("cookie")
 	   public String cookie(@CookieValue("JSESSIONID")String jSessionId) {	
       		 System.out.println("jSessionId = " + jSessionId);
       		 return "success";
    	}
}
```

==注意：通过@CookieValue指定要获取前端Cookie里面的某个值==





### 3.7 @RequestBody获取JSON

> **使用@RequestBody接收前端json数据时，不能是get提交方式**

```java
    @DeleteMapping("batchRemove")
    //接收前台json的数组[1,2,3]转为对应java的list集合
    public Result batchRemove(@RequestBody List<Long> idList) {
        return roleService.removeByIds(idList) == true ? Result.ok(null) : Result.error(null);
    }
```

```java
    @PostMapping("save")
	//接收前台json数据并转为实体类
    public Result save(@RequestBody Admin user) {
        user.setPassword(MD5.encrypt(user.getPassword()));
        return adminService.save(user) == true ? Result.ok(null) : Result.error(null);
    }
```







---

## 四.控制器参数传递

### 4.1 Model形参对象

```JAVA
@Controller
@RequestMapping("/user")
public class Hello4Controller {

    //方法1：通过Model形参对象，需要在方法中声明Model形参
    @RequestMapping("/getOne")
    public String queryOne(Model model){
        User user=new User();
        user.setName("11");
        user.setId(1);
        model.addAttribute("user",user);//类似于request作用域，页面中可以通过el表达式获取数据
        return "queryOne";
    }
}
```

==注意：一定要在方法中声明Model形参对象，页面中可以直接通过EL表达式获取==



### 4.2 返回ModelAndView对象

```JAVA
@Controller
@RequestMapping("/user")
public class Hello4Controller {

    //方法2：通过ModelAndView方法
    @RequestMapping("/getOne2")
    public ModelAndView queryOne2(){
        ModelAndView view=new ModelAndView();
        view.setViewName("queryOne");

        User user=new User();
        user.setName("11");
        user.setId(1);

        view.addObject("user",user);//类似于servlet中的request作用域，页面中可以通过el表达式获取数据
        return view;
    }
}
```



### 4.3 原生作用域形参对象

```JAVA
@Controller
@RequestMapping("/user")
public class Hello4Controller {

    /**
     *  获取request response和session
     */
    @RequestMapping("/3scope")
    public void use3Scope(HttpServletRequest request, HttpServletResponse response) throws IOException {
        //通过原生response设置
        response.sendRedirect("/home/index");
        //通过原生request设置
        request.setAttribute("request","request");
        //通过原生session设置
        request.getSession().setAttribute("session","session");
    }
}
```

==注意：一定要在方法中声明原生作用域形参==



### 4.4 Map形参对象

```JAVA
@Controller
@RequestMapping("/user")
public class Hello4Controller {

 
    @RequestMapping("/MAP")
    public String queryOne(Map<String,Object> data){
       data.put("id",1);
       return "queryOne";
    }
}
```

==在页面中直接使用EL表达式获取键名即可：**${id**}==

---

## 五. 转发与重定向

 	1.若想要转发或重定向到另一个请求(如 控制器之间转发，控制器之间重定向跳转，重定向到某个页面)都要加上相应的关键字
 	2.转发到某个页面不需要加关键字,默认为隐式转发
 	3.显式的forward转发和redirect重定向要写上完整的路径名,因为显式转发/重定向不和视图解析器一起用



### 5.1 转发

#### 5.1.1 转发到页面

```JAVA
@Controller
public class Hello3Controller {

     
    @RequestMapping("/register")
    public String register(User user){
        System.out.println("user = " + user);
        return "success";			//隐式转发到视图前缀的success.jsp页面,默认为转发页面
    }
}
```

```JAVA
@Controller
public class Hello3Controller {

     
    @RequestMapping("/register")
    public String register(User user){
        System.out.println("user = " + user);
        return "forward:/WEB-INF/jsp/user.jsp";	//显式转发到视图前缀的user.jsp页面,写全路径
    }
}
```

==显式forward需要完整的写出视图前缀的路径==

#### 5.1.2 转发到控制器

```JAVA
@Controller
public class Forwar_RedirectController {

    @RequestMapping("/forward")
    public String forward(){
        return "forward:/home/index";   //控制器显式转发，转发到另外一个控制器的相应url上
    }
}
```

==注意：转发到控制器要加上foward关键字==



### 5.2 重定向

#### 5.2.1 重定向到被保护的内部(视图前缀)页面

​			如果想要**重定向到被保护的视图前缀里面的页面**，**单纯的加上视图名是不能跳转进去的**，因为SpringMVC的重定向不能跳转到WEB-INF里面的文件，所以想要跳转到被保护的视图前缀设置的页面里面去就**需要添加返回该视图页面的方法**

```JAVA
@Controller
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/login")
    public  String login(String account, String password, HttpSession session)  {
        User loginUser = userService.findUserByAccount(account, password);
        if(loginUser!=null){
          
            return "redirect:/index";		//表示直接显式重定向到路径为/index的方法上
        }
        return  "redirect:/login.jsp";		//表示显式跳转到根路径的login.jsp页面
    }


    @RequestMapping("/index")
    public String index(){
        return "index";				//index.jsp主页
    }
}
```

==注意：这里如果想要重定向到被保护视图前缀里面的index.jsp，**如果直接写“redirect：/index” 这样会出现404错误**，SpringMVC会默认跳转到路径为/index的方法上，所以应该写一个返回index.jsp页面的方法，然后重定向到该方法即可==





#### 5.2.2 重定向到外部页面

```JAVA
@Controller
public class Forwar_RedirectController {

    @RequestMapping("/error")
    public String error(){
        return "redirect:/error.jsp";	//注意该页面是在根路径的error.jsp页面
    }
}
```



#### 5.2.3 重定向到控制器

```JAVA
@Controller
public class Forwar_RedirectController {


    @RequestMapping("/redirect")
    public String redirect(){
        return "redirect:/home/index";  //控制器重定向，重定向到另外一个控制器的相应url上
    }

}
```

==小记：重定向不能跳转到被保护的(WEB-INF)内部的页面，而转发可以==



### 5.3 ModelAndView转发

```JAVA
@Controller
public class UserController {

    @RequestMapping("/register")
    public ModelAndView redirect(String account,String password){
        ModelAndView mv=new ModelAndView();
        mv.addObject("acc",account);
        mv.addObject("pwd",password);
        mv.setViewName("forward:/index.jsp");
    }

}
```



### 5.4 ModelAndView重定向

​		控制器返回ModelAndView对象重定向时，SpringMVC可以将上个页面的数据拿到重定向页面过后的页面，相当于重定向也可以拿到丢失的request作用域的数据

```HTML
<form action="/register" method="post">
    <input type="text" name="account" />
    <input type="text" name="password" />
    <input type="submit" value="注册" />
</form>
```

```JAVA
@Controller
public class UserController {

    @RequestMapping("/register")
    public ModelAndView redirect(String account,String password){
       //第一次请求,并将数据放入request的作用域
        ModelAndView mv=new ModelAndView();
        mv.addObject("acc",account);
        mv.addObject("pwd",password);
        //第二次请求,当前请求中没有任何request作用域对象
        mv.setViewName("redirect:/index.jsp");
    }
}
```

```HTML
<!--					index.jsp
	这里只能写${param.XXX},不能直接通过四大作用域获取值,因为这是两次请求并且这个页面是第二次请求的页面,它不能获取第一次请求中request作用域的值
-->	
重定向拿到第一次请求中request作用域的数据:${param.acc} <br>		
						 ${param.pwd} <br>


```

​		==注意：如果控制器用ModelAndView重定向返回页面前设置了request作用域， SpringMVC对第二次请求时会将第一次请求中request作用域的对象追加到第二次请求的url路径上(如这里的第二次请求路径就会变为：http:localhost/8080/register?account=XXX&password=XXX)==

---

## 六. Json数据交互

### 6.1 @RestController注解

​		这个注解是由多个注解组合而成的，**里面有@Controller和@ResponseBody两个注解**，表示这个**控制器类的所有方法都是无刷新页面**，**用于实体类上**

```JAVA
@RestController    //等同于@Controller和@ResponseBody两个注解
public class CRUDController {

    @RequestMapping("/add")
    public String add() {
        return "add...";
    }

    @RequestMapping("/delete")
    public String delete() {
        return "delete";
    }

    @RequestMapping("/update")
    public String update() {
        return "update";
    }

    @RequestMapping("/query")
    public String query() {
        return "query...";
    }
}
```

<font color="red">上述控制器里面的所有方法都是无刷新请求，即可以用Ajax直接调用！</font>



### 6.2 Fast Json

​		  a. 首先导入pom.xml依赖

```xml-dtd
 <!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.75</version>
        </dependency>
```

​		b. 编写控制器层

```JAVA
@Controller
public class AjaxController {


    //方法1：通过fast json返回
    @ResponseBody
    @RequestMapping("/findUser")
    public String findUser(){
        User u=new User();
        u.setId(1);
        u.setName("张三");
        u.setDate(new Date());
        return JSON.toJSONString(u);
    }
}
```

==使用注意：**1.返回json数据到前台必须加上这个注解 @ResponseBody表示无刷新**==
	            ==2.若要防止json乱码，要么方法加上@RequestMapping(value = "/findUser",produces = { "application/json;charset=UTF-8" })或者**在springmvc.xml中加入消息转换器(推荐)**==

```xml-dtd
  <!--启用mvc注解支持-->
    <mvc:annotation-driven >
        <!--设置json的相应配置-->
        <mvc:message-converters> 
            <!-- 配置JSON响应编码字符集 -->
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                        <value>text/plain;charset=UTF-8</value>
                        <value>application/xml;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

​        	     ==3.如果要返回日期，**必须在声明该属性的类上加入注解**@JSONField(format = "yyyy-MM-dd HH:mm:ss")，否则返回json的日期数据也会乱码==

```java
  @JSONField(format = "yyyy-MM-dd HH:mm:ss")
  private Date date;
```





### 6.3 Jackson

​		  a. 首先导入pom.xml依赖

```xml-dtd
     <!--jackson-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.11.2</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.11.2</version>
        </dependency>
```

​		b.编写控制器层

```JAVA
@Controller
public class AjaxController {


    //方法2：使用jackson：springmvc默认使用的json工具
    @ResponseBody
    @RequestMapping("/list")
    public List<User> userList(){
        ArrayList<User> users = new ArrayList<>();
        users.add(new User(1,"张三",new Date()));
        users.add(new User(2,"李四",new Date()));
        users.add(new User(3,"王五",new Date()));
        users.add(new User(4,"赵六",new Date()));
        return users;
    }

}

```

​		c. springmvc.xml文件配置消息转换器

```xml-dtd
 <!--启用mvc注解支持 -->
    <mvc:annotation-driven conversion-service="conversionService">
        <!--设置json的相应配置-->
        <mvc:message-converters>
            <!-- 配置Jackson消息转换器 -->
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="objectMapper">
                    <bean class="com.fasterxml.jackson.databind.ObjectMapper">
                        <!-- 格式化日期 -->
                        <property name="dateFormat">
                            <bean class="java.text.SimpleDateFormat">
                                <constructor-arg type="java.lang.String" value="yyyy-MM-dd HH:mm:ss" />
                            </bean>
                        </property>
                    </bean>
                </property>
            </bean>
            <!-- 配置JSON响应编码字符集 -->
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                        <value>text/plain;charset=UTF-8</value>
                        <value>application/xml;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

==注意：1.返回json数据到前台必须加上这个注解 @ResponseBody==
			==**2.使用jackson直接返回集合就可以了，它默认会给我们转换成json数据**==



### 6.4 Ajax上传注意

​			利用Ajax交互的时候，**如果前端要上传数组到后端，后端用数组接收可能会出现接收不到的情况**。<font color="red">最好的方式就是前端将数组转为字符串，然后再后端截取即可。</font>



---

## 七. Rest风格

### 7.1 介绍



​				从上

|       传统方式        |      Rest风格      |
| :-------------------: | :----------------: |
|  /user/findById?id=1  |  /user/findById/1  |
| /user/deleteById?id=1 | /user/deleteById/1 |

​	从上表可以看出，REST风格的URL中最明显的就是参数不再使用 ? 传递。这种风格的URL可读性更好， 使得项目架构清晰。在实际开发中，大多数会将传统方式和REST风格混合使用。



### 7.2 Rest风格使用

> Rest风格  使用注意：
> *                  1.若控制器方法中存在形参，需在控制器的方法形参中加入@PathVariable注解
> *                  2.@RequestMapping请求路径要更改，变为  @RequestMapping("/随意url/{控制器方法中的形参名}")
> *                  3.如果@PathVariable注解里面指定了值，那么在请求路径中 /随意url/{控制器方法中的形参名}，要把 {方法中的形参名}改为 注解指定的值
> *                  4.如果需要传多个值到控制器，**控制器不能用类类型来接收**，**要用基本类型来获取**，**且每个基本类型都要加入@PathVariable注解**，然后请求路径中直接变为  /随意url/{控制器方法中的形参名1}/{控制器方法中的形参名2}



​	<font color="green">Rest风格示例1</font>

```JAVA
@Controller
public class RestStyleController {

    @RequestMapping("/findUserByUserId/{id}")
    public String findUserById(@PathVariable Integer id){
        System.out.println("查询id = " + id);
        return "success";
    }
}
```

​	==url访问地址： "/findUserByUserId/1"==

​	<font color="green">Rest风格示例2</font>

```JAVA
@Controller
public class RestStyleController {

    @RequestMapping("/deleteById/{userId}")
    public String deleteById(@PathVariable("userId") Integer id){
        System.out.println("删除id = " + id);
        return "success";
    }
}
```

==url访问地址："/deleteById/2"==

​	<font color="green">Rest风格示例3</font>

```JAVA
@Controller
public class RestStyleController {

    @RequestMapping("/updateUser/{id}/{name}")
    public String updateUser(@PathVariable Integer id,@PathVariable String name){
        System.out.println("更新：id = " + id+" name = " + name);
        return "success";
    }
}
```

==url访问地址："/updateUser/2/张三"==



### 7.3 Rest风格增删改查

​	对于老的浏览器IE来说，只支持get和post请求，这时候就需要让springmvc加入过滤器去处理，让他们能够实现put和delete请求。

```xml
 <!--web.xml文件中配置过滤器-->
	<filter>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

​		**<font color="green">前端页面如果要使用put和delete请求都用post请求，但是要加上一行代码</font>**

```HTML
<form method="post" action="/put/1">
    <!--put请求:必须加上这行代码且不能修改任何属性的值-->
	<input type="hidden" name="_method" value="PUT" />	
</form>

<form method="post" action="/delete/1">
      <!--delete请求:必须加上这行代码且不能修改任何属性的值-->
	<input type="hidden" name="_method" value="DELETE" />	
</form>
```

```JAVA
//控制器
@Controller
public class RestController{
    
  @RequestMapping(value = "/put/{id}",method = RequestMethod.PUT)
    public String put(@PathVariable("id")Integer id){
        return "success";
    }

  @RequestMapping(value = "/delete/{id}",method = RequestMethod.DELETE)
    public String delete(@PathVariable("id")Integer id){
        return "success";
    }    
}
```

​	==注意：在跳转的页面最好加上 **<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" isErrorPage="true"%>** 头标签，否则会出现405 **或者控制器直接使用重定向到页面**==



---

## 八. 文件上传&下载

​			a. 首先在pom.xml文件中导入依赖

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

        <!--cos腾讯云文件上传-->
        <dependency>
            <groupId>com.qcloud</groupId>
            <artifactId>cos_api</artifactId>
            <version>5.6.8</version>
        </dependency>
```

​			b. springmvc.xml文件中配置

```xml-dtd
 <!-- 配置文件解析器对象，要求id名称必须是multipartResolver -->
    <bean id="multipartResolver"  class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 设置文件上传限制大小默认为10M 即10485760 ，这里改为20M-->
        <property name="maxUploadSize" value="20971520"/>
    </bean>
```

​			c. **前端页面的name值不要与实体类的属性名一致！！！**

​		

### 8.1 本地上传

#### 	8.1.1 单文件上传

```HTML
  <h1>单文件上传到本地</h1>
    <form action="/upload" method="post" enctype="multipart/form-data">
        <div style="margin-top: 20px">
            <input type="file" name="file">
        </div>
        <div style="margin-top: 20px">
            <input type="submit" value="上传">
        </div>
    </form>
```

```JAVA
@Controller
public class UploadController {


    /**
     * 单文件上传到本地
     */
    
    @RequestMapping("/upload")
    //这里的file与jsp页面的 <input type="file" name="file"> name一致
    public String upload(MultipartFile file, HttpServletRequest request) {  
            //判断是否选中文件
            if (!file.isEmpty()) {
                //获取文件上传路径(目标地址)
                String path = "E:upload/";
                //获取源文件名称
                String oldFileName = file.getOriginalFilename();
                //获取文件后缀名
                String suffix = FilenameUtils.getExtension(oldFileName);
                //重命名文件
                String newFileName = UUID.randomUUID().toString().replace("-"," ")+"."+suffix;
                //为了解决同一个文件夹文件过多的问题，使用日期作为文件夹管理
                String datePath = new SimpleDateFormat("yyyyMMdd").format(new Date());
                //组装文件名
                String finalName = datePath + "/" + newFileName;
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

              //将上传文件的路径保存在request作用域中
                if(file.getContentType().contains("image")){
                    request.setAttribute("url",finalName);
                }
            }
        return "success";
    }
}
```



#### 	8.1.2 多文件上传

```html
 <h1>多文件上传到本地</h1>
    <form action="/upload3" method="post" enctype="multipart/form-data">
        <div style="margin-top: 20px">
            <input type="file" name="files" multiple="multiple">
        </div>
        <div style="margin-top: 20px">
            <input type="submit" value="上传">
        </div>
    </form>
```

```JAVA
@Controller
public class UploadController {
	/**
     * 多文件上传到本地
     *      注意：上传多个文件的总大小不能超过springmvc.xml中配置的大小，否则会失败
     */
    
    @RequestMapping("/upload3")
    //这里的fileS与jsp页面的 <input type="file" name="fileS"> name一致
    public String upload(MultipartFile[] files, HttpServletRequest request) {  
        for (int i = 0; i < files.length; i++) {
            MultipartFile file=files[i];
            //判断是否选中文件
            if (!file.isEmpty()) {
                //获取文件上传路径(目标地址)
                String path = "E:upload/";
                //获取源文件名称
                String oldFileName = file.getOriginalFilename();
                //获取文件后缀名
                String suffix = FilenameUtils.getExtension(oldFileName);
                //重命名文件
                String newFileName = UUID.randomUUID().toString().replace("-"," ")+"."+suffix;
                //为了解决同一个文件夹文件过多的问题，使用日期作为文件夹管理
                String datePath = new SimpleDateFormat("yyyyMMdd").format(new Date());
                //组装文件名
                String finalName = datePath + "/" + newFileName;
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
            }
        }
        return "success";
    }
}
```



> **注意： 1. 如果需要将图片回显且是Idea项目，则要在idea中配置虚拟路径，同eclipse的配置**
>              	 **a.编辑tomcat的配置，打开编辑配置，选择development**
>              	 **b.选择 + 选择External Source，找到自己上传图片的文件夹即目标地址，这里为E:upload/的upload文件夹**
>              	 **c.Application Context的路径自定义，后期访问就为： /自定义/上传的文件名**
>             **2.获取上传的路径，页面访问就直接写 <img src="/自定义/url"   就可以了,这里的自定义就是上面Application Context的路径自定义**



### 8.2 腾讯元上传

#### 	8.2.1 <font color="green">Cos工具类</font>

```JAVA
//COS上传工具类
import com.qcloud.cos.COSClient;
import com.qcloud.cos.ClientConfig;
import com.qcloud.cos.auth.BasicCOSCredentials;
import com.qcloud.cos.auth.COSCredentials;
import com.qcloud.cos.model.ObjectMetadata;
import com.qcloud.cos.model.PutObjectRequest;
import com.qcloud.cos.model.PutObjectResult;
import com.qcloud.cos.model.StorageClass;
import com.qcloud.cos.region.Region;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.net.URL;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

public class CosUtils {
    // 密钥ID
    private static final String SECRET_ID = "自己的";
    // 密钥KEY
    private static final String SECRET_KEY = "自己的";
    // 所属地域
    private static final String REGIONID = "地区";
    // 存储桶名称
    private static final String BUCKETNAME = "桶名称";
    // COS的键需要手动设置 （类似于在哪个文件夹，若没有可以自动创建,这里用当前时间作为文件夹便于管理）
    private static String KEY = new SimpleDateFormat("yyyyMMdd").format(new Date()) +"/";
    // COSClient客户端对象
    private static COSClient cosClient;

    /**
     * 静态代码块加载
     */
    static {
        // 1 初始化用户身份信息(secretId, secretKey)
        COSCredentials cred = new BasicCOSCredentials(SECRET_ID, SECRET_KEY);
        // 2 设置bucket的区域, COS地域的简称
        ClientConfig clientConfig = new ClientConfig(new Region(REGIONID));
        // 3 生成cos客户端
        cosClient = new COSClient(cred, clientConfig);
    }

    //获取上传在哪个文件夹里面
    private static String getKey(){
        return KEY;
    }

    //手动设置上传的文件夹
    public void setKey(String key){
        KEY=key;
    }

    /**
     * 上传文件方法1：直接通过File对象
     *
     * @return 返回图片的访问地址
     */
    public static String uploadFile(File file) {
        String resultUrl = null;
        // 采用随机UUID+文件名 防止重复，可以保存该KEY，以后可以根据KEY删除文件
        String myKey = getKey() + UUID.randomUUID().toString().replace("-", "") + file.getName();
        // 上传文件
        PutObjectRequest putObjectRequest = new PutObjectRequest(BUCKETNAME, myKey, file);
        // 设置存储类型, 默认是标准(Standard)一般为标准的
        putObjectRequest.setStorageClass(StorageClass.Standard);
        try {
            PutObjectResult putObjectResult = cosClient.putObject(putObjectRequest);

            /**
             * putobjectResult会返回文件的etag String etag = putObjectResult.getETag();
             * System.out.println(etag);
             */
            // 获取访问连接
            Date expiration = new Date(new Date().getTime() + 5 * 60 * 10000);
            URL oldurl = cosClient.generatePresignedUrl(BUCKETNAME, myKey, expiration);
            String url = oldurl.toString();
            resultUrl = url.substring(0, url.indexOf("?"));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 关闭客户端
            cosClient.shutdown();
        }
        //可以保存URL，未来可以访问
        return resultUrl;
    }


    /**
     *  上传文件2：返回上传文件的路径
     *
     *  @return 返回图片的访问地址
     */
    public static String uploadFile2Cos(MultipartFile file) throws Exception {
        //判断文件是否为空：不为空就上传
            String originalFilename = file.getOriginalFilename();
            String extension = originalFilename.substring(originalFilename.lastIndexOf(".")).toLowerCase();
            //更改文件名防止重名
            String filename = UUID.randomUUID().toString().replace("-", "") + extension;
            try {
                InputStream inputStream = file.getInputStream();
                return uploadFile2Cos(inputStream, filename);
            } catch (Exception e) {
                throw new Exception("图片上传失败");
            }
    }


    // 上传文件的真正逻辑
    private static String uploadFile2Cos(InputStream instream, String fileName) {
        //String eTag = "";
        String resultUrl="";
        try {
            // 创建上传Object的Metadata
            ObjectMetadata objectMetadata = new ObjectMetadata();
            objectMetadata.setContentLength(instream.available());
            objectMetadata.setCacheControl("no-cache");
            objectMetadata.setHeader("Pragma", "no-cache");
            objectMetadata.setContentType(getcontentType(fileName.substring(fileName.lastIndexOf("."))));
            objectMetadata.setContentDisposition("inline;filename=" + fileName);
            // 上传文件这里的key就为filename
            String myKEY = getKey() + fileName;
            PutObjectResult putResult = cosClient.putObject(BUCKETNAME, myKEY, instream, objectMetadata);

            //获取eTag
            //eTag = putResult.getETag();

            // 设置URL过期时间为10年 3600l* 1000*24*365*10
            Date expiration = new Date(System.currentTimeMillis() + 3600L * 1000 * 24 * 365 * 10);
            String url = cosClient.generatePresignedUrl(BUCKETNAME, myKEY, expiration).toString();
            resultUrl = url.substring(0, url.indexOf("?"));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (instream != null) {
                    instream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return resultUrl;
    }


    /**
     * 获取上传的类型
     */
    private static String getcontentType(String filenameExtension) {
        if (filenameExtension.equalsIgnoreCase("bmp")) {
            return "image/bmp";
        }
        if (filenameExtension.equalsIgnoreCase("gif")) {
            return "image/gif";
        }
        if (filenameExtension.equalsIgnoreCase("jpeg") || filenameExtension.equalsIgnoreCase("jpg")
                || filenameExtension.equalsIgnoreCase("png")) {
            return "image/jpeg";
        }
        if (filenameExtension.equalsIgnoreCase("html")) {
            return "text/html";
        }
        if (filenameExtension.equalsIgnoreCase("txt")) {
            return "text/plain";
        }
        if (filenameExtension.equalsIgnoreCase("pptx") || filenameExtension.equalsIgnoreCase("ppt")) {
            return "application/vnd.ms-powerpoint";
        }
        if (filenameExtension.equalsIgnoreCase("docx") || filenameExtension.equalsIgnoreCase("doc")) {
            return "application/msword";
        }
        return "image/jpeg";
    }


}

```





#### 	8.2.2 单文件上传

```HTML
<h1>单文件上传到腾讯云</h1>
    <form action="/upload2" method="post" enctype="multipart/form-data">
        <div style="margin-top: 20px">
            <input type="file" name="file">
        </div>
        <div style="margin-top: 20px">
            <input type="submit" value="上传">
        </div>
    </form>
```

```JAVA
@Controller
public class UploadController {
	/**
     *  单文件通过工具类上传到腾讯云
     */
    
    @RequestMapping("/upload2")
    //这里的file与jsp页面的 <input type="file" name="file"> name一致
    public String upload2(MultipartFile file, Model model) throws  Exception{
        if(!file.isEmpty()){
            //获取源文件名称
            String oldFileName = file.getOriginalFilename();
            //获取文件后缀名
            String suffix = FilenameUtils.getExtension(oldFileName);
            //后期可以根据判断后缀名是否上传图片  eg :   Utils.isImg(suffix)

            String url = CosUtils.uploadFile2Cos(file);
            model.addAttribute("url",url);
            return "success";
        }
      return "redirect:/upload.jsp";//没有文件则重定向上传文件
    }
}
```



#### 	8.2.3 多文件上传

```HTMl
  <h1>多文件上传到cos</h1>
    <form action="/upload4" method="post" enctype="multipart/form-data">
        <div style="margin-top: 20px">
            <input type="file" name="files" multiple="multiple">
        </div>
        <div style="margin-top: 20px">
            <input type="submit" value="上传">
        </div>
    </form>
```



```JAVA
@Controller
public class UploadController {   
    
	/**
     *  多文件通过工具类上传到腾讯云
     */
    
    @RequestMapping("/upload4")
    //这里的fileS与jsp页面的 <input type="file" name="fileS"> name一致
    public String upload4(MultipartFile[] files) throws  Exception{
        for (int i = 0; i < files.length; i++) {
            MultipartFile file=files[i];
            if(!file.isEmpty()){
                //获取源文件名称
                String oldFileName = file.getOriginalFilename();
                //获取文件后缀名
                String suffix = FilenameUtils.getExtension(oldFileName);
                //后期可以根据判断后缀名是否上传  eg :   Utils.isImg(suffix)
                String url = CosUtils.uploadFile2Cos(file);
                System.out.println("第"+i+"次url的路径："+url);
            }
        }
        return "success";//没有文件则重定向上传文件
    }
}
```



### 8.3 文件下载

```JAVA
@Controller
public class DownloadController {

    @RequestMapping("/download")
    public void downloadFile(String fileName,HttpServletRequest request, HttpServletResponse response) throws IOException {
        fileName = "cat.jpg";
        if (fileName != null) {
            //获取file的路径或者绝对路径
            String realPath = "http://127.0.0.1:8080/files/cat.jpg";
            File file = new File(realPath);
            //二进制文件输出流
            OutputStream outputStream = null;
            //设置下载完毕不打开文件
            response.setContentType("application/force-download");
            //设置文件名
            response.setHeader("Content-Disposition", "attachment;filename=" + fileName);
            try {
                outputStream = response.getOutputStream();
                //二进制流读写文件（先从服务器读）
                outputStream.write(FileUtils.readFileToByteArray(file));
                outputStream.flush();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    outputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

```HTML
  <a href="/download">下载文件</a>
```



---

## 九. 拦截器 

​			Spring MVC 的拦截器（Interceptor）与 Servlet 的过滤器（Filter）类似，它主要用于拦截用户的请求并做 相应的处理，通常应用在权限验证、记录请求信息的日志、判断用户是否登录等功能上。



### 9.1 HandlerInterceptorAdapter类

	继承HandlerInterceptorAdapter类有两个比较重要的方法根据需求重写即可
			preHandle();	//拦截用户的请求,如登录
			postHandle();	//拦截服务器的响应,如将数据传回前端

### 9.2 拦截器的使用

​				``a. 登录页面``

```HTML
   		<form action="/user/login" method="post">
            id: 	<input type="text" name="id"> <br>
            name:	<input type="text" name="name"> <br>
            <input type="submit" value="LOGIN">
        </form>
```

​				``b.登录控制器``

```JAVA
@Controller
@RequestMapping("/user")
public class LoginController {

    @RequestMapping("/login")
    public String  login(User u, HttpServletRequest request){
        if(u.getId().equals(123456)&&u.getName().equals("admin")){
            request.getSession().setAttribute("loginUser",u);
            return "success";				//表示登录成功
        }		
        return "redirect:/login.jsp";		//否则登录失败，重定向到登录页面
    }
}
```

​			 ``c.登录拦截器的使用``

```JAVA
public class LoginInterceptor extends HandlerInterceptorAdapter {

    //这个方法表示在进入控制器方法之前拦截
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //判断是否登录
        if(request.getSession().getAttribute("loginUser")==null){
            response.sendRedirect("/login.jsp");
            return false;   //验证失败，拦截请求
        }
        return true;//验证通过，放行
    }
}
```

​			``d.springmvc.xml文件中注入拦截器``

```xml-dtd
  <!-- 配置拦截器 -->
    <mvc:interceptors>
        <mvc:interceptor>
            <!-- 设置拦截范围为所有请求路径，如果要指定某个范围 eg：可以写/dept/**，表示拦截类似以  /dept/路径名  所有的方法; /**表示拦截所有 -->
            <mvc:mapping path="/**"/>
            <!-- 排除拦截url范围为 /user/login的请求 -->
            <mvc:exclude-mapping path="/user/login"/>
            <!-- 注入自定义登录拦截器 -->
            <bean class="com.eobard.interceptor.LoginInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
```

==拦截器使用注意：1. 首先在拦截器根据session判断是否登录了==

​								==2.在springmvc文件中注入拦截器，并设置需要拦截的url地址和排除拦截的url地址==



### 9.3 拦截器链

​		多个拦截器的构成变成拦截器链，**按照springmvc.xml的配置顺序执行**，类似于栈但不是栈

```xml-dtd
  <!-- 配置拦截器 -->
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>          
            <bean class="com.eobard.interceptor.LoginInterceptor"/>
        </mvc:interceptor>

 		<mvc:interceptor>
            <mvc:mapping path="/**"/>          
            <bean class="com.eobard.interceptor.FcuntionInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
```

执行顺序：**LoginInterceptor 的preHandle()** =>**FcuntionInterceptor 的preHandle()**  => 控制器方法 => **FcuntionInterceptor 的postHandle()** =>**LoginInterceptor 的 postHandle()**

---

## 十.  异常处理器

​				基于AOP的思想，将项目中出现的错误通过异常处理器的捕捉，将页面指定跳转到自定义的错误页面，提供友好的用户体验。<font color="red">**最好是后面项目完整了再使用，否则开发中会看见不了错误信息！！！**</font>



### 10.1 局部异常处理器

​				**<font color="green">``控制器代码``</font>**

```java
@Controller
public class ExceptionController {

    //异常1
    @RequestMapping("/ex1")
    public String ex(){
        User u=null;
        u.getId();
        return "success";
    }

    //异常2
    @RequestMapping("/ex2")
    public String ex2(){
       int i=10/0;
        return "success";
    }


    //局部异常处理的方法
    @ExceptionHandler(value = {RuntimeException.class,NullPointerException.class})
    public  String handlerException(RuntimeException ex, Model model){
        //设置异常信息到request作用域
        model.addAttribute("error",ex.getMessage());
        return "exception";
    }
}
```

==使用局部异常处理注意：只能处理当前控制器的所有方法的异常，不能指定其它控制器的异常==



### 10.2 全局异常处理器

​				**<font color="green">``控制器代码``</font>**

```JAVA
@Component
public class GlobalExceptionResolver  implements HandlerExceptionResolver {
    
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
                                         Object o, Exception e) {
        ModelAndView view=new ModelAndView();
        view.setViewName("exception");					//设置异常跳转页面
        view.addObject("globalerror",e.getMessage());	//设置异常信息到request作用域
        return  view;
    }

}
```

==使用全局异常处理注意：需要将全局异常处理注入IOC容器，可以处理所有控制器的所有方法的异常==



### 10.3 全局异常处理器(推荐使用)

​				**<font color="green">``直接在springmvc.xml文件中配置``</font>**

```xml-dtd
    <!--注入spring自带的全局异常处理器：推荐使用-->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <props>
                <!--表示出现运行时异常，都去exception.jsp页面-->
                <prop key="java.lang.RuntimeException">exception</prop>
            </props>
        </property>
    </bean>
```

​				**<font color="green">``exception.jsp页面中获取错误信息``</font>**

```JSP
<!--springmvc中的全局异常处理用下面的方式获取信息-->
错误信息：${exception.message}
```

==注意：1.使用springmvc自带的异常处理器，需要在springmvc.xml文件中配置即可使用。2.错误信息只能通过上面**${exception.message}**的方式获取==



---

## 十一. 自定义类型转换器

### 11.1 简单注解局部转换(推荐使用)



​				<font color="green">**案例1：将前端页面输入的字符串转换为日期类型**</font>

>  在springmvc中，如果前端输入的格式为字符串的日期类型，如：2020-8-21 18:20:20,要想转化成日期类型，需要在对应的实体类的属性加上一个 @DateTimeFormat注解

​				``a.前端页面``

```HTML
 <h1>数据类型转换</h1>
    <form action="/date" method="post">
        <input type="text"  name="id"> <br>
        <input type="text"  name="name"> <br>
        <input type="text" name="date"><br>
        <input type=submit value="提交">
    </form>
```

​				``b.User实体类``

```JAVA
public class User {

    private Integer id;
    private String name;

    //pattern属性：指定转换格式
 	@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date date;
    
    //省略getter，setter
}
```

​				``c.控制器代码``

```JAVA
@Controller
public class DateTimeController {

   
   //方法1：通过注解简单转换，如果项目中有很多字符串转换为日期类型不推荐使用这种方式，只适用于局部的
    @RequestMapping("/date")
    public String date(User user, Model model){
        System.out.println(user);
        model.addAttribute("user",user);
        return "queryOne";
    }

}
```

​			`` d. queryOne.jsp页面``

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

	id：${user.id} <br>
	姓名：${user.name}<br>
	日期：<fmt:formatDate type="both"  value="${user.date}" pattern="yyyy-MM-dd HH:mm:ss" />

</body>
</html>
```

==注意：上面的日期用**Jstl格式化时间**，不然会变成CST时间：Sun Aug 09 18:20:30 CST 2020==



### 11.2 全局转换



​				<font color="green">**案例1：项目中有许多字符串类型需要转为日期类型**</font>

​					``a.前端页面``

```HTML
<h1>数据类型转换</h1>
    <form action="/date" method="post">
        <input type="text"  name="id"> <br>
        <input type="text"  name="name"> <br>
        <input type="text" name="date"><br>
        <input type=submit value="提交">
    </form>
```

​					``b.转换器``

```JAVA
public class String2DateConvert implements Converter<String, Date> {

    @Override
    public Date convert(String source) {
        //判断source是否为空
        if (StringUtils.isEmpty(source)) {
            throw new RuntimeException("参数不能为空");
        }
        //创建格式化对象
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        try {
            return format.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
            throw new RuntimeException("类型转换错误");
        }

    }
}
```

​					``c.配置springmvc.xml文件``

```xml-dtd
    <!--启用mvc注解支持 并且引用自定义转换器-->
    <mvc:annotation-driven conversion-service="conversionService">
        <!--设置json的相应配置-->
        <mvc:message-converters>
            <!-- 配置Jackson消息转换器 -->
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="objectMapper">
                    <bean class="com.fasterxml.jackson.databind.ObjectMapper">
                        <!-- 格式化日期 -->
                        <property name="dateFormat">
                            <bean class="java.text.SimpleDateFormat">
                                <constructor-arg type="java.lang.String" value="yyyy-MM-dd HH:mm:ss" />
                            </bean>
                        </property>
                    </bean>
                </property>
            </bean>
            <!-- 配置JSON响应编码字符集 -->
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                        <value>text/plain;charset=UTF-8</value>
                        <value>application/xml;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>


    <!-- 注册自定义转换器 -->
    <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <!-- 注入自定义类型转换器 -->
                <bean class="com.eobard.convert.String2DateConvert"/>
            </set>
        </property>
    </bean>
```

​					`` d. queryOne.jsp页面`` 

```JSP
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

	id：${user.id} <br>
	姓名：${user.name}<br>
	日期：<fmt:formatDate type="both"  value="${user.date}" pattern="yyyy-MM-dd HH:mm:ss" />

</body>
</html>
```

==注意：1.上面的日期用**Jstl格式化时间**，不然会变成CST时间：Sun Aug 09 18:20:30 CST 2020==

​			==2. 需要将全局转换器配置到springmvc.xml文件中==



---

## 十二. 数据校验与Spring标签

### 12.1 Spring mvc标签

​		==注意：1.使用Spring mvc的标签，进入这个页面的时候一定要传一个实体类过去绑定==

```JAVA
   //页面加载的时候需要传入一个实体类到页面去，页面中使用spring标签来绑定
    @RequestMapping(value = "/addUser",method = RequestMethod.GET)
    public String addUser(User user){
        return "register";
    }
```

​					==2.需要在jsp页面中引用标签：<%@taglib prefix="s" uri="http://www.springframework.org/tags/form" %>==



#### 12.1.1 form表单

```JSP
<form:form modelAttribute="控制器方法中的形参实体类属性名" method="post" action="xxx">
			
</form:form>
```



#### 12.1.2 input标签

```JSP
<form:form modelAttribute="控制器方法中的形参实体类属性名" method="post" action="xxx">
	<form:input path="该形参实体类属性的属性名"/>
</form:form>
```



#### 12.1.3 password标签

```JSP
<form:form modelAttribute="控制器方法中的形参实体类属性名" method="post" action="xxx">
	<form:password path="实体类属性的属性名"/>
</form:form>
```



#### 12.1.4 hidden标签

```jsp
<form:form modelAttribute="控制器方法中的形参实体类属性名" method="post" action="xxx">
	<form:hidden path="实体类属性的属性名"/>
</form:form>
```



#### 12.1.5 errors标签

```JSP
<form:form modelAttribute="控制器方法中的形参实体类属性名" method="post" action="xxx">
	<form:errors path="实体类属性的属性名"/>
</form:form>
```



### 12.2 数据验证

​				数据验证分为客户端验证和服务器端验证，客户端验证主要是过滤正常用户的误操作，通过JavaScript代码完成。服务器端验证是整个应用阻止非法数据的最后防线，通过在应用中编程实现。

```
常用注解验证：

      @NotNull：用于数字类型的非空验证
      @NotBlank：用于字符串类型的非空验证，可以去除前端输入框的前后空格
      @Past：日期必须是一个过去的日期
      @Future:日期必须是一个未来的日期
      @Min：输入的值不能小于最小值
      @Max：输入的值不能大于最大值
      ....

```



​			**<font color="green" size=5>``案例1:注册数据校验``</font>**

​			``a. 在pom.xml文件中导入依赖``

```xml-dtd
 <!-- JSR 303 数据校验依赖 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.4.3.Final</version>
        </dependency>
```

​			``b. 在实体类上使用数据校验的相关注解``

```JAVA
public class UserCheck {

    @NotNull(message = "id必填")
    @Min(value = 1,message = "id最小为1")
    @Max(value = 100,message = "id最大为100")
    private Integer id;

    @NotBlank(message = "用户名必填") 
    private String name;

    @Past(message = "生日必须是过去时间")
    @NotNull(message="日期不能为空")
    @DateTimeFormat(pattern = "yyyy/MM/dd")
    private Date date;

	//省略getter。setter
}
```

​			``c. 在控制器中要验证实体类，并且验证是否通过``

```JAVA
@Controller
public class DataCheckController {

    //页面加载的时候需要传入一个实体类到页面去，页面中使用spring标签来绑定，否则errors标签不会显示出错误信息
    @RequestMapping(value = "/addUserCheck",method = RequestMethod.GET)
    public String addUserCheck(UserCheck userCheck){
        return "register";
    }

    @RequestMapping(value = "/addUserCheck",method = RequestMethod.POST)
    public String addUserCheck(@Valid UserCheck userCheck, BindingResult result){
        //判断是否验证通过
        if(result.hasErrors()){
            return "register";//有错误就还是在注册页面
        }
        //没有错误就调用相应service然后返回相应页面
        return "queryOne";
    }
}

```

==注意：1.这里在进行注册页面的时候，一定要传一个实体类给页面进行标签的数据绑定，否则就无法获取错误信息==

​			==2.方法名以及访问url可以一致，但是前提是它们请求的method必须不同！**通常将get请求用于传实体类给视图进行标签绑定，post请求用于提交**==

​			``d.访问路径``

```HTML
 <a href="/addUserCheck">去到注册页面</a> 
```

​			``e.注册信息并获取错误信息``

```JSP
<%@taglib prefix="s" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>数据校验</h1>

    		<%--这里要通过传过来的实体类的属性名绑定--%>
    <s:form modelAttribute="userCheck" method="post" action="/addUserCheck">
            <s:input path="id"/>
            <s:errors path="id" cssStyle="color: red"/>
        <br>
            <s:input path="name"/>
            <s:errors path="name" cssStyle="color: red"/>
        <br>
            <s:input path="date"/>
            <s:errors path="date" cssStyle="color: red"/>
        <br>
        <input type=submit value="提交">
    </s:form>
</body>
</html>

```



---

## 十三. 常用的Jstl标签库使用

### 	13.1 输出标签

```jsp
		 <%
            request.setAttribute("req","request作用域");
        %>
        <h1>输出标签</h1>
        <c:out  value="${req}" default="若作用域的值为空，这里指定默认显示的值"/>
```



### 	13.2 判断标签

```JSP
  		<%
            session.setAttribute("session","true");
        %>

        <h1>判断标签</h1>
        <c:if test="${session=='true'}">
            是一样的
        </c:if>
```



### 	13.3 多分支标签

```JSP
 		<%
            request.setAttribute("flag","1");         
        %>


        <h1>多分支标签</h1>
        <c:choose>
            <c:when test="${flag=='1'}">输出1</c:when>
            <c:otherwise>输出2</c:otherwise>
        </c:choose>
```



### 	13.4 遍历标签

```JSP
		<%      
            List list=new ArrayList();
            list.add(1);
            list.add(2);
			request.setAttribute("list",list);
        %>


        <h1>循环标签</h1>
        <c:forEach var="item" items="${list}">
                 ${item} <br>
        </c:forEach>
```



### 	13.5 日期格式化标签

```jsp
   		<%
            Date d=new Date();
            request.setAttribute("date",d);
        %>

        <h1>日期格式化标签</h1>
        <fmt:formatDate type="both" value="${date}" pattern="yyyy-MM-dd HH:mm:ss" />
```


# Shiro学习

## 一. 环境搭建(了解)

`1.创建maven项目，导入依赖`

```xml-dtd
 <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <!-- shiro核心依赖 -->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>1.4.2</version>
        </dependency>
        <!-- slf4j日志依赖开始 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.21</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>1.7.21</version>
            <scope>compile</scope>
        </dependency>
        <!-- slf4j日志依赖结束 -->
```



`2.在resources文件下创建shiro.ini文件`

```ini
[users]
#定义两个静态用户的账号和密码
#格式： K=V
root=123456
admin=123456
```

==注意：==

> ​	shiro使用时可以连接数据库，也可以不连接数据库，不连接这在shiro.ini配置文件中配置静态数据。	
>
> ​	shiro.ini配置文件共有[main]，[users]，[roles]，[urls]共4部分组成
>
> 1. [main] 用于定义全局变量
> 2. [users] 用于定义用户名及密码
> 3. [roles] 用于定义角色
> 4. [urls] 用于定义访问url及拦截验证方式

> ​	.ini 配置文件类似于 Java 中的 properties（key=value），不过提供了将 key/value 分类的特性， key 每个部分不重复即可，而不是整个配置文件。



`3.使用`

```JAVA
 @Test
    public void test() {
        //1.读取配置文件，初始化工厂对象
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        //2.获取SecurityManager实例
        SecurityManager manager = factory.getInstance();
        //3.将SecurityManager绑定到工具类
        SecurityUtils.setSecurityManager(manager);
        //4.通过SecurityUtils得到当前登录的用户
        Subject currentUser = SecurityUtils.getSubject();
        //5.窗口登录令牌
        AuthenticationToken token = new UsernamePasswordToken("admin", "123456");
        //6.登录并传入令牌
        try {
            currentUser.login(token);
            System.out.println("登录成功！");
        } catch (AuthenticationException e) {
            e.printStackTrace();
            System.out.println("登录失败！");
        }
        //7.退出
        currentUser.logout();
    }
```

==注意：在登录的时候最好try-catch，防止出错==



`4.两个常见的异常`

​		<font color="red">异常一：账号有问题</font>

> org.apache.shiro.authc.UnknownAccountException

​		<font color="red">异常二：密码有问题</font>

> org.apache.shiro.authc.IncorrectCredentialsException



## 二. Realm域(了解)

**简述:**

> ​		Realm：域，Shiro 从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 Realm 得到用户相应的角色/权限进行验证用户是否能进行操作；可以把 Realm 看成 DataSource ，即安全数据源。如我们之前的ini配置方式 将使用org.apache.shiro.realm.text.IniRealm。
>
> ​	Shiro默认使用自带的IniRealm，IniRealm从ini配置文件中读取用户的信息，大部分情况下需要从系统的数据库中读取用户信息，所以需要自定义realm。





## 三. 验证角色(了解)

`1.resources文件夹下创建shiro-role.ini文件`

```INI
#定义用户信息
[users]
#用户名=密码，角色名1，角色名2，角色名3
admin=123456,role1,role2,role3
#用户名=密码，角色1
test=123,role1
```



`2.创建ShiroUtil工具类`

```JAVA
public class ShiroUtil {
    private static final String PREFIX="classpath:";
    private static  String fileNameSuffix = "shiro.ini";

    private ShiroUtil() {
    }

    public static Subject login(String userName, String password,String fileName){
        if(fileName!=""&&fileName!=null){
            fileNameSuffix=fileName;
        }
        //1.读取配置文件，初始化工厂对象
        Factory<SecurityManager> factory = new IniSecurityManagerFactory(PREFIX+fileNameSuffix);
        //2.获取SecurityManager实例
        SecurityManager manager = factory.getInstance();
        //3.将SecurityManager绑定到工具类
        SecurityUtils.setSecurityManager(manager);
        //4.通过SecurityUtils得到当前登录的用户
        Subject currentUser = SecurityUtils.getSubject();
        //5.窗口登录令牌
        AuthenticationToken token = new UsernamePasswordToken(userName,password);
        //6.登录并传入令牌
        try {
            currentUser.login(token);
            System.out.println("登录成功！");
        } catch (AuthenticationException e) {
            e.printStackTrace();
            System.out.println("登录失败！");
        }
        return currentUser;
    }
}
```



`3.Subject常用的API`

> ​		`boolean  isAuthenticated();`
>
> ​		`boolean hasRole(String roleName);`
>
> ​		`boolean[] hasRoles(List<String> roleNames);`

```JAVA
	@Test
	public void test1() {
		//得到当前用户
		Subject currentUser = ShiroUtil.login("classpath:shiro_role.ini", "admin", "123456");
        //判断是否验证过：当调用退出后会返回false
        System.out.println("is already checked:"+currentUser.isAuthenticated());
		
        //判断是否存在单个角色
        System.out.println(currentUser.hasRole("role2")?"有这个角色":"没有这个角色");
		
        //判断多个角色
		boolean [] results = currentUser.hasRoles(Arrays.asList("role1","role2","role3"));
		System.out.println(results[0]?"有role1这个角色":"没有role1这个角色");
		System.out.println(results[1]?"有role2这个角色":"没有role2这个角色");
		System.out.println(results[2]?"有role3这个角色":"没有role3这个角色");
		
        //退出
		currentUser.logout();
	}
```



## 四. Shiro整合Web(了解)

### 4.1 环境搭配

`1.创建maven web项目,导入依赖`

```xml-dtd
 <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!-- 添加servlet支持 -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.3.1</version>
    </dependency>
    <!-- 添加jstl支持 -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
    <!-- 添加日志支持 -->
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>
    <dependency>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
      <version>1.2</version>
    </dependency>
    <!-- 添加shiro支持 -->
    <dependency>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-core</artifactId>
      <version>1.2.4</version>
    </dependency>
    <dependency>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-web</artifactId>
      <version>1.2.4</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.12</version>
    </dependency>
  </dependencies>
```



`2.更改web.xml文件`

```xml-dtd
 <!-- 配置shiro监听 -->
    <listener>
        <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
    </listener>
    <!-- 添加shiro支持 -->
    <filter>
        <filter-name>ShiroFilter</filter-name>
        <filter-class>org.apache.shiro.web.servlet.ShiroFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>ShiroFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```



`3.resources文件夹下创建shiro.ini`

```INI
#定义用户
[users]
#用户名=密码，角色（管理员）
kazu=123456,admin
#用户名=密码，角色（教师）
jack=123456,teacher
#用户名=密码
tom=123456
```

==注意：整合在Servlet中，Shiro默认从shiro.ini 配置文件读取，其它名字会报错==



### 4.2 案例一： 匿名访问

`1.创建LoginServlet`

```JAVA
@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("login：get");
        req.getRequestDispatcher("login.jsp").forward(req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}
```



`2.在shiro.ini中添加匿名访问`

```INI
#省略其它....

#访问路径
[urls]
#请求为/login的设置为anon（匿名请求，不登录也可访问）
/login=anon
```



### 4.3 案例二：登录

`1.编写前端页面`

```JSP
    登录
    <form action="/login" method="post">
        username: <input type="text" name="username" />
        <br>
        password: <input type="text" name="password" />
        <input type="submit" value="login" />
    </form>
    <font color="red">${error}</font>
```



`2.编写LoginServlet的Post方法`

```JAVA
@Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        //Shiro自带工具类获取Subject对象
        Subject subject = SecurityUtils.getSubject();
        //从shiro.ini文件中判断静态用户账号和密码
        UsernamePasswordToken token = new UsernamePasswordToken(username,password);
        try {
            subject.login(token);
            resp.sendRedirect("index.jsp");
        } catch (Exception e) {
            e.printStackTrace();
            req.setAttribute("error","账号或密码有问题");
            req.getRequestDispatcher("login.jsp").forward(req,resp);
        }
    }
```



### 4.4 案例三：身份验证

`1.编写一个功能菜单的Servlet`

```JAVA
@WebServlet("/func")
public class FunctionServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.sendRedirect("/func.jsp");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.sendRedirect("/func.jsp");
    }
}
```



`2.shiro.ini添加身份验证`

```INI
[main]
#表示身份认证未通过时,进入某个请求或者页面
authc.loginUrl=/login

#省略其它....

#访问路径
[urls]
#请求为/func的设置为authc(必须进行身份验证才可以访问)
/func=authc
```



### 4.5 案例四：角色权限验证

`1.shiro.ini添加角色验证`

```INI
[main]
#表示没有权限时，进入没有权限的页面
roles.unauthorizedUrl=/unauthorized.jsp

#定义用户
[users]
#用户名=密码，角色（管理员）
kazu=123456,admin
#用户名=密码，角色（教师）
jack=123456,teacher

#定义角色
[roles]
#admin管理员角色拥有用户所有权限
admin=user:*
#teacher教师角色拥有学生所有权限
teacher=student:*

#访问路径
[urls]
#请求为/student的设置为roles[自定义角色](必须拥有该权限才可以访问)
/student=roles[teacher]

#省略其它....
```



`2.编写Student的Servlet`

```JAVA
@WebServlet("/student")
public class StudentServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.sendRedirect("/studentIndex.jsp");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}
```

> ​	当用kazu登录时，进入unauthorized.jsp页面
>
> ​	当用jack登录时，进入/student请求



## 五. Url匹配方式(重点)

### 5.1 匹配任意字符

```INI
#访问路径
[urls]
#匹配以/testXXXX  开头的0-n个字符
/test1*=authc
```

> ​	访问： /test1
>
> ​				/test1a
>
> ​				/test1bcdede



### 5.2 匹配子目录所有路径

```INI
#访问路径
[urls]
#匹配以/test2/XXX  子目录下的所有路径
/test2/**=authc
```

> 访问：	/test2
>
> ​				/test2/a
>
> ​				/test2/bcde



### 5.3 任意匹配

```ini
#访问路径
[urls]
#匹配以 /test2XXX/XXX  该目录及其子目录下任何的所有路径
/test3*/**=authc
```

> 访问：	/test3
>
> ​				/test3a
>
> ​				/test3a/a
>
> ​				/test3absew/awewqeqwe



## 六. Shiro加密(重点)

### 6.1 Base64

​			Base64加密算法不安全可逆向获取原值，不用来存储重要的信息，**可以用来存储如博客内容之类的信息(博客内容可能会出现一些转义符号之类的，直接存储在数据库再次读取可能会乱码)**



### 6.2 MD5加密

​		通过在散列算法上加入了”盐“，指定”盐”为唯一的干扰数据，可以让改MD5散列加密更难破解，可以适用于密码加密

> ​	特点：MD5算法不可逆，如果内容相同无论加密N次，MD5生成结果都是一致的
>
> ​	应用场景： 
>
> ​					如何比较a.txt和b.txt的内容一致？ **采用MD5**，它可以对文件内容进行校验





### 6.3 加密工具类

```JAVA
public class EncryptionUtils {

    private EncryptionUtils(){}

    /**
     * base64加密
     */
    public static String encBase64(String str){
        return Base64.encodeToString(str.getBytes());
    }

    /**
     * base64解密
     */
    public static String decBase64(String str){
        return Base64.decodeToString(str);
    }

    /**
     * md5加密
     * @param str	密码
     * @param salt	盐值(佐料),建议值是唯一的
     */
    public static String md5Hash(String str,String salt) {
        return new Md5Hash(str, salt).toString();
    }

    /**
     * md5加密
     * @param str	密码
     * @param salt	盐值(佐料),建议值是唯一的
     * @param count	加密次数:	最好是1024或者2048次
     */
    public static String md5Hash(String str,String salt,int count) {
        return new Md5Hash(str, salt,count).toString();
    }

}
```

==注意：项目开发加密的时候获取盐值，可以使用以下方法：==

> ByteSource salt = ByteSource.Util.bytes(String str);

​	



## 七. SSM使用Shiro(重点)

### 7.1 创建ssm maven项目

#### `1.添加依赖`

```xml-dtd
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <spring.version>5.2.8.RELEASE</spring.version>
    </properties>

<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <!-- 添加Servlet支持 -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>javax.servlet.jsp-api</artifactId>
        <version>2.3.1</version>
    </dependency>
    <!-- 添加jstl支持 -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
	<!--  spring web  -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!--  spring mvc  -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.8.7</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.2.3</version>
    </dependency>
    <!-- 添加日志支持 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <!-- 添加mybatis支持 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.3</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.3</version>
    </dependency>
    <!-- jdbc驱动包  -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.37</version>
    </dependency>
 		<!--  slf4j  -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.12</version>
    </dependency>
	 <!--  shiro整合  -->
  	<dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>1.2.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-web</artifactId>
        <version>1.2.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.2.4</version>
    </dependency>
    <!--  DBCP数据源  -->
    <dependency>
        <groupId>commons-dbcp</groupId>
        <artifactId>commons-dbcp</artifactId>
        <version>1.4</version>
    </dependency>

</dependencies>

<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.properties</include>
            </includes>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```



#### `2.修改web.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- Spring配置文件 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    <!-- 编码过滤器 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <async-supported>true</async-supported>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!-- Spring监听器 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- 添加对springmvc的支持 -->
    <servlet>
        <servlet-name>springMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
        <async-supported>true</async-supported>
    </servlet>
    <servlet-mapping>
        <servlet-name>springMVC</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```



#### `3.创建applicationContext.xml文件`

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 扫描注解所在的包 -->
    <context:component-scan base-package="com.eobard.service"/>


    <!-- 引入database.properties属性文件 -->
    <bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
        <property name="location" value="classpath:db.properties" />
    </bean>

    <!-- 配置数据源dataSource -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
        <!--驱动-->
        <property name="driverClassName" value="${jdbc.driver}"/>
        <!--连接字符串-->
        <property name="url" value="${jdbc.url}"/>
        <!--数据库用户名-->
        <property name="username" value="${jdbc.user}"/>
        <!--数据库密码-->
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- 配置SqlSessionFactoryBean:如果未来要使用MP，这里的class就需要换成MP的类 -->
    <bean id ="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--  引用数据源 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 加载MyBatis配置文件 -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>

    <!-- 加载mapper所在的包 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 扫描mapper接口所在的位置 -->
        <property name="basePackage" value="com.eobard.dao"/>
    </bean>

    <!-- 定义事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 开启注解式事务 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>

</beans>
```



#### `4.创建db.properties文件`

```properties
jdbc.driver = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/db_shiro?useUnicode=true&characterEncoding=utf-8
jdbc.user = root
jdbc.password = 123456
```



#### `5.创建log4j.properties文件`

```properties
log4j.rootLogger=debug,con
log4j.appender.con=org.apache.log4j.ConsoleAppender
log4j.appender.con.layout=org.apache.log4j.PatternLayout
log4j.appender.con.layout.ConversionPattern=%d{MM-dd HH:mm:ss}[%p]%c%n -%m%n
```



#### `6.创建mybatis-config.xml文件`

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <!-- 显示日志信息 -->
        <setting name="logImpl" value="LOG4J"/>
    </settings>

    <typeAliases>
        <!-- 设置别名 -->
        <package name="com.eobard.entity"/>
    </typeAliases>
    
</configuration>
```



#### `7.创建springmvc.xml文件`

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/mvc
           http://www.springframework.org/schema/mvc/spring-mvc.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 加载控制器所在的包 -->
    <context:component-scan base-package="com.eobard.controller"/>

    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 视图前缀（页面在根路径） -->
        <property name="prefix" value="/"/>
        <!-- 视图后缀(页面的后缀名是什么) -->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- 开启注解支持 -->
    <mvc:annotation-driven />

</beans>
```



### 7.2 SSM+Shiro实现登录

#### `1.各层代码`

**<font color="green">Dao层</font>**

```JAVA
public interface UserMapper {
    User findUserByName(String userName) throws Exception;
}
```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC " - //mybatis.org//DTDMapper3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.dao.UserMapper">

    <select id="findUserByName" resultType="com.eobard.entity.User">
        select * from t_user where username =#{userName}
    </select>
</mapper>
```



**<font color="green">Service层</font>**

```JAVA
public interface UserService {

    User findUserByName(String userName) throws  Exception;
}
```

```JAVA
@Service
@Transactional
public class UserServiceImpl implements UserService {

    @Resource
    private UserMapper userMapper;

    @Override
    public User findUserByName(String userName) throws Exception {
        return userMapper.findUserByName(userName);
    }
}
```



**<font color="green">Controller层</font>**

```java
@Controller
public class UserController {

    @PostMapping("/login")
    public String login(User user, Model model){
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(user.getUserName(), user.getPassword());
        try {
            subject.login(token);
            Session session = subject.getSession();
            session.setAttribute("currentUser",user.getUserName());
            return "index";
        } catch (Exception e) {
            e.printStackTrace();
            model.addAttribute("error","用户名或密码错误");
            return "forward:/login";
        }
    }

    @GetMapping("/login")
    public String login(){
        return "login";
    }
}
```



#### `2.自定义realm实现登录认证`

```JAVA
public class UserRealm extends AuthorizingRealm {

    @Resource
    private UserService userService;


    /**
     * 授权：为当前角色授予权限和角色
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    /**
     * 身份验证：为当前角色进行登录身份验证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取当前用户信息
        String userName=(String) token.getPrincipal();
         //当前realm对象的唯一名字，调用父类的getName()方法
        String realmName = getName();
        try {
            User user = userService.findUserByName(userName);
            if(user!=null){
                SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user.getUserName(), user.getPassword(), realmName);
                //验证成功
                return authenticationInfo;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        //验证失败
        return null;
    }
}
```

==注意：1.这里通过调用父类的getName()方法保证了SimpleAuthenticationInfo构造方法realmName的唯一性==			==2.  SimpleAuthenticationInfo构造方法可以有四个参数: 账号,密码,盐值,realm值==



#### `3.注入shiro配置`

**<font color="green">添加配置到web.xml</font>**

```xml-dtd
   <!-- shiro配置 -->
    <!-- shiro过滤器定义 -->
    <filter>
        <filter-name>shiroFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <!-- 该值缺省为false,表示生命周期由SpringApplicationContext管理,设置为true则表示由ServletContainer管理 -->
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>shiroFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```



**<font color="green">配置applicationContext.xml</font>**

```xml-dtd
<!--Shiro配置-->
    <!-- 1.注入自定义realm -->
    <bean id="myRealm" class="com.eobard.realm.UserRealm" />
    <!-- 2.将realm注入到securityManager安全管理器 -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="realm" ref="myRealm" />
    </bean>
    <!-- 3.Shiro过滤器 -->
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <!-- Shiro的核心安全接口,这个属性是必须的 -->
        <property name="securityManager" ref="securityManager" />
        <!-- 身份认证失败，则跳转到登录页面的配置 -->
        <property name="loginUrl" value="/login.jsp" />
        <!-- Shiro连接约束配置,即过滤链的定义 -->
        <property name="filterChainDefinitions">
            <value>
                <!-- 登录请求可匿名访问  -->
                /login = anon
				 <!-- 静态路径请求可匿名访问  -->
        		/resources/**=anon
                <!-- 以下所有请求必须经过身份验证才可访问 -->
                /** = authc
            </value>
        </property>
    </bean>
    <!-- 4.保证实现了Shiro内部lifecycle函数的bean执行 -->
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
    <!-- 5.开启Shiro注解 -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" depends-on="lifecycleBeanPostProcessor" />
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager" />
    </bean>
```



==注意：使用MD5加密==

```xml
<!--注入自定义realm-->   
<bean id="userRealm" class="com.ang.elearning.shiro.UserRealm">
        <!-- 配置密码匹配器 -->
        <property name="credentialsMatcher">
            <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
                <!-- 加密算法为MD5 -->
                <property name="hashAlgorithmName" value="MD5"></property>
                <!-- 加密次数 -->
                <property name="hashIterations" value="1024"></property>
            </bean>
        </property>
    </bean>
```

```JAVA
//realm认证的时候修改代码	
	@Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取当前用户信息
        String userName=(String)token.getPrincipal();
        //省略其它层
        //User user = userService.findUserByName(userName);
        //模拟从数据库获取账号
        if("root".equals(userName)){
            //模拟从数据库获取密码和盐值
            String password="2ce37c45225404a9d3f7b0132f6f15f7";
            String salt="xxxx";
            return new SimpleAuthenticationInfo(userName,password, ByteSource.Util.bytes(salt),this.getName());
        }
        //验证失败
        return null;
    }
```



### 7.3 权限配置

#### `1.DB`

```
数据库：
		user			userid(主键)		username	password	roleid(外键)	 	
        					1				admin		123			1
        					2				test		123			2
        					3				tom			123			3
        				
        			
        				
        
        role			id(主键)	rolename
        				 1			admin
        				 2			teacher
        				 3			student
        				 
        
        permission		id(主键) 	permissionname	roleid(外键)		
        				 1			user:*			1
        				 2			student:*		2
        				 3			user:create		3
```

> 一个role对应多个user;	
>
> 一个role对应多个permission



#### `2.各层代码`

**<font color="green">Dao层</font>**

```JAVA
public interface UserMapper {

    User findUserByName(String userName) throws Exception;

    //根据用户名查询角色名
    Set<String> findUserWithRoleName(String userName) throws Exception;

    //根据用户名查询权限
    Set<String> findUserWithFunctionName(String userName) throws Exception;
}
```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC " - //mybatis.org//DTDMapper3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.dao.UserMapper">

    <select id="findUserByName" resultType="com.eobard.entity.User">
        select * from user where username =#{userName}
    </select>

    <select id="findUserWithRoleName" resultType="java.lang.String">
        SELECT
            role.roleName
        FROM
            USER INNER JOIN role ON USER.roleid = role.id
        WHERE
            username =#{userName}
    </select>

    <select id="findUserWithFunctionName" resultType="java.lang.String">
        SELECT
            permission.permissionName
        FROM
            user
            INNER JOIN role ON user.roleid = role.id
            inner JOIN permission on role.id=permission.roleid
            where username=#{userName}
    </select>
</mapper>
```



**<font color="green">Service层</font>**

```JAVA
public interface UserService {
    User findUserByName(String userName) throws  Exception;

    //根据用户名查询角色名
    Set<String> findUserWithRoleName(String userName) throws Exception;

    //根据用户名查询权限
    Set<String> findUserWithFunctionName(String userName) throws Exception;
} 
```

```java
@Service
@Transactional
public class UserServiceImpl implements UserService {

    @Resource
    private UserMapper userMapper;

    @Override
    public User findUserByName(String userName) throws Exception {
        return userMapper.findUserByName(userName);
    }

    @Override
    public Set<String> findUserWithRoleName(String userName) throws Exception {
        return userMapper.findUserWithRoleName(userName);
    }

    @Override
    public Set<String> findUserWithFunctionName(String userName) throws Exception {
        return userMapper.findUserWithFunctionName(userName);
    }
}
```



#### `3.自定义realm实现权限授权`

```JAVA
public class UserRealm extends AuthorizingRealm {

    @Resource
    private UserService userService;


    /**
     * 授权：为当前角色授予权限和角色
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //获取当前登录用户信息
        String userName = (String) principalCollection.getPrimaryPrincipal();
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();

        try {
            //设置当前登录用户的角色
            authorizationInfo.setRoles(userService.findUserWithRoleName(userName));
            //设置当前登录用户的权限
            authorizationInfo.setStringPermissions(userService.findUserWithFunctionName(userName));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return authorizationInfo;
    }

    /**
     * 身份验证：为当前角色进行身份验证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取当前用户信息
        String userName = (String) token.getPrincipal();
        //当前realm对象的唯一名字，调用父类的getName()方法
        String realmName = getName();
        try {
            User user = userService.findUserByName(userName);
            if (user != null) {
                SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user.getUserName(), user.getPassword(), realmName);
                //验证成功
                return authenticationInfo;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        //验证失败
        return null;
    }
}
```



#### `4.配置applicationContext.xml`

```xml-dtd
<!--Shiro配置-->
    <!-- 1.注入自定义realm -->
    <bean id="myRealm" class="com.eobard.realm.UserRealm" />
    <!-- 2.将realm注入到securityManager安全管理器 -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="realm" ref="myRealm" />
    </bean>
    <!-- 3.Shiro过滤器 -->
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <!-- Shiro的核心安全接口,这个属性是必须的 -->
        <property name="securityManager" ref="securityManager" />
        <!-- 身份认证失败，则跳转到登录页面的配置 -->
        <property name="loginUrl" value="/login.jsp" />
        <!-- 权限认证失败，则跳转到指定页面 -->
         <property name="unauthorizedUrl" value="/unauthorized.jsp" />
        <!-- Shiro连接约束配置,即过滤链的定义 -->
        <property name="filterChainDefinitions">
            <value>
                <!-- 登录请求可匿名访问  -->
                /login = anon
				 <!-- 静态路径请求可匿名访问  -->
        		/resources/**=anon
                <!-- 以admin开头所有请求必须进行身份验证且用户必须拥有admin角色 -->
                /admin*/** = roles[admin]
                <!-- 以student开头所有请求必须拥有teacher角色 -->
                /student*/** = authc,roles[teacher]
                <!-- 以student开头所有请求必须拥有user:create的权限 -->
                /teacher*/** = perms["user:create"]
            </value>
        </property>
    </bean>
    <!-- 4.保证实现了Shiro内部lifecycle函数的bean执行 -->
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
    <!-- 5.开启Shiro注解 -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" depends-on="lifecycleBeanPostProcessor" />
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager" />
    </bean>
```



#### `5.测试`

**<font color="red">登录：admin admin</font>**

> ​			访问 ：	/admin  	√
>
> ​							/student	×
> ​							/teacher	√		

==注意：因为admin 123 的权限是user:*表示拥有所有权限==	***代表通配符，表示所有**

**<font color="red">登录：test 123</font>**

> ​		访问 ：	/admin  	×
>
> ​						/student	√	
> ​						/teacher	×		

**<font color="red">登录：tom123456</font>**

> ​		访问 ：	/admin  	√
>
> ​						/student	×
> ​						/teacher	√		



### 7.4 解决多个角色拥有相同功能路径(重点)

​		如果在applicationContext.xml设置了该路径下的功能所属角色，那么这个该路径下就只有这个角色才可以访问了；如果有不同的角色都拥有该路径下的功能，那么在配置文件的方式就不能够实现了，就可以用注解完成

> ​	/admin*/ = roles[admin]



**@RequiresRoles**

> ​	当前Subject必须拥有所有指定的角色时，才能访问被该注解标注的方法。如果当前Subject不同时拥有所有指定角色，**则方法不执行还会抛出AuthorizationException异常。可以配合使用SpringMVC的全局异常处理：添加一个处理AuthorizationException异常的类将错误信息跳转到自定义权限不足页面**

==注意：该注解可以放在控制器方法上，表示该路径的方法必须要有指定的角色；也可以放在整个控制器的类上，表示整个类路径下的方法都要有指定的角色，**建议使用这个注解可以省去applicationContext.xml的多余配置代码**==



**<font color="green">eg:</font>**

```JAVA
@Controller
@RequestMapping("/admin")
public class UserController {

    @GetMapping("/add")
    //表示当前Subject需要角色 admin 和user
    //等同于之前配置文件的 	 /admin*/** = roles[admin,user]
   	@RequiresRoles(value={“admin”, “user”}, logical= Logical.AND)
    public  String toAdd(){
        return "/user/add";
    }

    @GetMapping("/update")
    //表示当前Subject需要角色 admin 或者 user
	@RequiresRoles(value={"user","admin"},logical=Logical.OR)
    public  String toUpdate(){
        return "/user/update";
    }

    //没有权限的页面
    @GetMapping("/noAutho")
    public String toNoAutho(){
        return "/unauthorized";
    }
}
```









## 八.SpringBoot使用Shiro(重点)

### 8.1创建SpringBoot项目

#### `1.添加依赖`

```xml-dtd
<!--thymeleaf-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <!-- shiro与spring整合依赖 -->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.4.0</version>
        </dependency>

        <!-- 添加热部署 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- 导入mybatis相关的依赖 -->
        <!-- 数据源 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.0</version>
        </dependency>
        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!-- SpringBoot的Mybatis启动器 -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.1.1</version>
        </dependency>


  <build>
   			<!--配置mapper文件-->
	<resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
  </build>
```



#### `2.全局配置文件`

```properties
#加载驱动
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#数据库连接路径
spring.datasource.url=jdbc:mysql://localhost:3306/db_shiro?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
#数据库用户名
spring.datasource.username=root
#数据库密码
spring.datasource.password=123456
#数据源类型（阿里巴巴）
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
#mybatis取别名
mybatis.type-aliases-package=com.eobard.entity

spring.thymeleaf.suffix=.html
```



### 8.2 SpringBoot+Shiro实现登录

#### `1.各层代码`

**<font color="green">Dao层</font>**

```JAVA
public interface UserMapper {

    User findUserByName(String userName) throws Exception;
}
```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC " - //mybatis.org//DTDMapper3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.dao.UserMapper">

    <select id="findUserByName" resultType="com.eobard.entity.User">
        select * from t_user where username =#{userName}
    </select>
</mapper>
```



**<font color="green">Service层</font>**

```JAVA
public interface UserService {
    User findUserByName(String userName) throws  Exception;
}
```

```JAVA
@Service
@Transactional
public class UserServiceImpl implements UserService {

    @Resource
    private UserMapper userMapper;

    @Override
    public User findUserByName(String userName) throws Exception {
        return userMapper.findUserByName(userName);
    }
}
```



**<font color="green">Controller层</font>**

```JAVA
@Controller
public class IndexController {

    @RequestMapping("/index")
    public String index(){
        return "/login";
    }

    @PostMapping("/login")
    public String login(User user, Model model){
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(user.getUserName(), user.getPassword());
        try {
            subject.login(token);
            Session session = subject.getSession();
            session.setAttribute("currentUser",user.getUserName());
            //代表登录状态5秒后失效,需要重新认证
            session.setTimeout(5000L);
            return "success";
        } catch (Exception e) {
            e.printStackTrace();
            model.addAttribute("error","用户名或密码错误");
            return "login";
        }
    }



    @GetMapping("/add")
    public  String toAdd(){
        return "/user/add";
    }

    @GetMapping("/update")
    public  String toUpdate(){
        return "/user/update";
    }

}
```

==注意：设置登陆过期时间，单位毫秒，这里设置30分钟 , 1s=1000ms==

> ​	SecurityUtils.getSubject().getSession().setTimeout(1800000);



#### `2.在resources/templates文件夹创建html静态页面`

```HTML
<!--login.html-->
<h2>登录</h2>
    <form action="/login" method="post">
        <input type="text" name="userName"> <br>
        <input type="text" name="password"> <br>
        <input type="submit" value="login">
    </form>
        <p style="color: red" th:text="${error}"></p>

<!--success.html-->
     登录成功
        <!--注意：如果是session作用域中，取值为session.xxx-->
        <p style="color: red" th:text="${session.currentUser}"></p>
        <a href="/add">用户添加</a> <br>
        <a href="/update">用户修改</a> <br>
        <hr>
```

**另外在templates文件夹在创建一个user文件夹，并添加add.html和update.html即可**



#### `3.自定义realm实现登录认证`

```JAVA
public class UserRealm extends AuthorizingRealm {

    @Resource
    private UserService userService;

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    //身份验证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取当前用户信息
        String userName=(String)token.getPrincipal();
        //当前realm对象的唯一名字，调用父类的getName()方法
        String realmName = getName();
        try {
            User user = userService.findUserByName(userName);
            if(user!=null){
                SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user.getUserName(), user.getPassword(), realmName);
                //验证成功
                return authenticationInfo;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        //验证失败
        return null;
    }
}
```

​	==注意： SimpleAuthenticationInfo构造方法可以有四个参数: 账号,密码,盐值,realm值==



#### `4.创建配置类`

```JAVA
@Configuration
public class ShiroConfig {

    //1.注入自定义realm
    @Bean
    public UserRealm getUserRealm() {
        return new UserRealm();
    }

    //2.创建DefaultWebSecurityManager对象，关联自定义realm
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联自定义realm对象
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    //3.创建ShiroFilterFactoryBean对象，关联DefaultWebSecurityManager
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager defaultWebSecurityManager) {
        ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
        //关联DefaultWebSecurityManager对象
        factoryBean.setSecurityManager(defaultWebSecurityManager);
        //添加啊Shiro内置过滤器
        /**
         *  常用的过滤器简称
         *          anon：不需要认证(登录)可以访问
         *          authc：必须认证才可以访问
         *          perms：必须有对应的权限才可以访问
         *          roles：必须有对应的角色才可以访问
         *			logout: 退出
         */
        
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        
        /*****设置放行的请求路径******/
        filterChainDefinitionMap.put("/index","anon");
        filterChainDefinitionMap.put("/login","anon");
         /*****设置静态路径的请求路径******/
        filterChainDefinitionMap.put("/resources/**","anon");
        
       /*****设置身份验证的请求路径******/
        filterChainDefinitionMap.put("/add","authc");
        filterChainDefinitionMap.put("/update","authc");
        
       //所有请求必须经过身份验证，此配置放在最后
        filterChainDefinitionMap.put("/**","authc");
        
        //设置过滤器链
        factoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        //身份认证失败，则跳转到登录页面的配置，默认去login.jsp
        factoryBean.setLoginUrl("/index");  
        return factoryBean;
    }
}
```

==注意：LinkedHashMap创建的元素有序！HashMap创建的元素无序==



### 8.3 权限配置

#### `1.DB`

```
数据库：
		user			userid(主键)		username	password	roleid(外键)	 	
        					1				admin		123			1
        					2				test		123			2
        					3				tom			123			3
        				
        			
        				
        
        role			id(主键)	rolename
        				 1			admin
        				 2			teacher
        				 3			student
        				 
        
        permission		id(主键) 	permissionname	roleid(外键)		
        				 1			user:*			1
        				 2			student:*		2
        				 3			user:create		3
```

> 一个role对应多个user;	
>
> 一个role对应多个permission



#### `2.各层代码`

**<font color="green">Dao层</font>**

```java
public interface UserMapper {

    User findUserByName(String userName) throws Exception;

    //根据用户名查询角色名
    Set<String> findUserWithRoleName(String userName) throws Exception;

    //根据用户名查询权限
    Set<String> findUserWithFunctionName(String userName) throws Exception;
}
```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC " - //mybatis.org//DTDMapper3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.dao.UserMapper">

    <select id="findUserByName" resultType="com.eobard.entity.User">
        select * from user where username =#{userName}
    </select>

    <select id="findUserWithRoleName" resultType="java.lang.String">
        SELECT
            role.roleName
        FROM
            USER INNER JOIN role ON USER.roleid = role.id
        WHERE
            username =#{userName}
    </select>

    <select id="findUserWithFunctionName" resultType="java.lang.String">
        SELECT
            permission.permissionName
        FROM
            user
            INNER JOIN role ON user.roleid = role.id
            inner JOIN permission on role.id=permission.roleid
            where username=#{userName}
    </select>
</mapper>
```



**<font color="green">Service层</font>**

```java
public interface UserService {
    User findUserByName(String userName) throws  Exception;

    //根据用户名查询角色名
    Set<String> findUserWithRoleName(String userName) throws Exception;

    //根据用户名查询权限
    Set<String> findUserWithFunctionName(String userName) throws Exception;
} 
```

```JAVA
@Service
@Transactional
public class UserServiceImpl implements UserService {

    @Resource
    private UserMapper userMapper;

    @Override
    public User findUserByName(String userName) throws Exception {
        return userMapper.findUserByName(userName);
    }

    @Override
    public Set<String> findUserWithRoleName(String userName) throws Exception {
        return userMapper.findUserWithRoleName(userName);
    }

    @Override
    public Set<String> findUserWithFunctionName(String userName) throws Exception {
        return userMapper.findUserWithFunctionName(userName);
    }
}
```



**<font color="green">Controller层</font>**

```JAVA
@Controller
public class IndexController {

    @RequestMapping("/index")
    public String index(){
        return "/login";
    }

    @PostMapping("/login")
    public String login(User user, Model model){
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(user.getUserName(), user.getPassword());
        try {
            subject.login(token);
            Session session = subject.getSession();
            session.setAttribute("currentUser",user.getUserName());
            return "success";
        } catch (Exception e) {
            e.printStackTrace();
            model.addAttribute("error","用户名或密码错误");
            return "login";
        }
    }


    @GetMapping("/add")
    public  String toAdd(){
        return "/user/add";
    }

    @GetMapping("/update")
    public  String toUpdate(){
        return "/user/update";
    }


    //没有权限的页面
    @GetMapping("/noAutho")
    public String toNoAutho(){
        return "/unauthorized";
    }
}
```



#### `3.自定义realm实现权限授权`

```java
public class UserRealm extends AuthorizingRealm {

    @Resource
    private UserService userService;

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //获取当前登录用户信息
        String userName = (String) principalCollection.getPrimaryPrincipal();
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();

        try {
            //设置当前登录用户的角色
            authorizationInfo.setRoles(userService.findUserWithRoleName(userName));
            //设置当前登录用户的权限
            authorizationInfo.setStringPermissions(userService.findUserWithFunctionName(userName));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return authorizationInfo;
    }

    //身份验证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取当前用户信息
        String userName=(String)token.getPrincipal();
        //当前realm对象的唯一名字，调用父类的getName()方法
        String realmName = getName();
        try {
            User user = userService.findUserByName(userName);
            if(user!=null){
                SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(user.getUserName(), user.getPassword(), realmName);
                //验证成功
                return authenticationInfo;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        //验证失败
        return null;
    }
}
```



#### `4.修改配置类`

```JAVA
@Configuration
public class ShiroConfig {

    //1.注入自定义realm
    @Bean
    public UserRealm getUserRealm() {
        return new UserRealm();
    }

    //2.创建DefaultWebSecurityManager对象，关联自定义realm
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联自定义realm对象
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    //3.创建ShiroFilterFactoryBean对象，关联DefaultWebSecurityManager
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager defaultWebSecurityManager) {
        ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
        //关联DefaultWebSecurityManager对象
        factoryBean.setSecurityManager(defaultWebSecurityManager);
        //添加啊Shiro内置过滤器
        /**
         *  常用的过滤器简称
         *          anon：不需要认证(登录)可以访问
         *          authc：必须认证才可以访问
         *          perms：必须有对应的权限才可以访问
         *          roles：必须有对应的角色才可以访问
         *			logout: 退出
         */
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        /*****设置放行的请求路径******/
        filterChainDefinitionMap.put("/index","anon");
        filterChainDefinitionMap.put("/login","anon");
        /*****设置静态路径的请求路径******/
        filterChainDefinitionMap.put("/resources/**","anon");

        /*****设置身份验证的请求路径******/
        filterChainDefinitionMap.put("/add","authc");
        filterChainDefinitionMap.put("/update","authc");

        /*****设置角色的请求路径******/
        filterChainDefinitionMap.put("/add","roles[admin]");
        filterChainDefinitionMap.put("/update","roles[teacher]");

        /*****设置权限的请求路径******/
        filterChainDefinitionMap.put("/add","perms[user:add]");
        //同时设置角色必须为teacher并且权限要有user:update才可以访问
//       filterChainDefinitionMap.put("/update","perms[user:update],roles[teacher]");


        //所有请求必须经过身份验证，此配置放在最后
        //filterChainDefinitionMap.put("/**","authc");
        
        //设置过滤器链
        factoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        //身份认证失败，则跳转到登录页面的配置，默认去login.jsp
        factoryBean.setLoginUrl("/index");
//        //权限认证失败，则跳转到指定页面
        factoryBean.setUnauthorizedUrl("/noAutho");
        return factoryBean;
    }
}
```



#### `5.测试`

##### 5.1 角色验证访问

**<font color="red">登录：admin admin</font>**

> ​					/add			√
>
> ​					/update		×



**<font color="red">登录：test 123</font>**

> ​					/add			  ×
>
> ​					/update		√



##### 5.2 权限验证访问

注释角色请求路径的代码

 		 //filterChainDefinitionMap.put("/add","roles[admin]");
 	     //filterChainDefinitionMap.put("/update","roles[teacher]");

取消注释  filterChainDefinitionMap.put("/update","perms[user:update],roles[teacher]");



**<font color="red">登录：admin admin</font>**

> ​					/add			√
>
> ​					/update		×

**<font color="red">登录：test 123 或 tom 123456</font>**

> ​					/add			×
>
> ​					/update		×



**<font color="red">登录：test 123</font>**

> ​					/add			×
>
> ​					/update		√



**<font color="red">登录：admin admin 或 tom 123456</font>**

> ​					/add			×
>
> ​					/update		×





### 8.4	解决多个角色拥有相同功能路径

​	同样在控制器的方法或实体类上**使用注解@RequiresRoles(value={“admin”, “user”}, logical= Logical.AND)**，同7.4使用方法



### 8.5 Spring Boot 使用MD5加密登录

#### `1.修改配置类`

```JAVA
   //1.注入自定义realm
    @Bean
    public UserRealm getUserRealm() {
        UserRealm userRealm = new UserRealm();
        userRealm.setCredentialsMatcher(hashedCredentialsMatcher());
        return userRealm;
    }

    //使用MD5加密,加密次数1024次
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher(){
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        // 使用md5 算法进行加密
        hashedCredentialsMatcher.setHashAlgorithmName("md5");
        // 设置散列次数： 加密次数(这个地方没有盐值也不会影响密码对比)
        hashedCredentialsMatcher.setHashIterations(1024);
        return hashedCredentialsMatcher;
    }
```



#### `2.修改自定义realm`

```JAVA
 	@Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //获取当前用户信息
        String userName=(String)token.getPrincipal();
        //省略其它层
        //User user = userService.findUserByName(userName);
        //模拟从数据库获取账号
        if("root".equals(userName)){
            //模拟从数据库获取密码和盐值
            String password="2ce37c45225404a9d3f7b0132f6f15f7";
            String salt="xxxx";
            return new SimpleAuthenticationInfo(userName,password, ByteSource.Util.bytes(salt),this.getName());
        }
        //验证失败
        return null;
    }
```

==注意：盐值需要从数据库中读取出来==



## 九. 常见的过滤器类

> 常用的过滤器简称
>               			anon：不需要认证(登录)可以访问
>           				authc：必须认证才可以访问
>             			  perms：必须有对应的权限才可以访问
>             			  roles：必须有对应的角色才可以访问
>    					   logout:  退出





## 十. Shiro标签

### 1.常用标签

> ### 	1.游客标签
>
> ```
> <shiro:guest>
>  	游客访问
> </shiro:guest>
> ```
>
> 
>
> ### 	2.身份标签
>
> ```
> <shiro:principal/>
> ```
>
> **注意：在new SimpleAuthenticationInfo(第一个参数,..)**
>
> * 第一个参数放的如果是一个username ，那么就可以直接用。											
>
> * 第一个参数放的是对象，比如User对象。那么如果要取username字段。
>
> > ​	**<shiro:principal property="username"/>**
>
> 
>
> ###  	3.认证成功标签
>
> <!--user标签：用户已经通过认证\记住我 登录后显示响应的内容-->
>
> ```
> <shiro:user>
>    欢迎  <shiro:principal/> 登录成功
> </shiro:user>
> ```
>
> 
>
> ### 	4. 角色标签
>
> <!--name属性指定角色类型-->
>
> ```
> <shiro:hasRole name="admin">
> 	角色为admin可以看得到
> </shiro:hasRole>
> 
> <shiro:hasAnyRoles name="admin,teacher">
> 	角色为admin或者teacher都可以看得到
> </shiro:hasAnyRoles>
> ```
>
> 
>
> ### 	5.权限标签
>
> <!--name属性指定权限字符串：可以使用通配符* -->
>
> ```
> <shiro:hasPermission name = "user:add">
>              用户<shiro:principal/> 拥有权限user:add <br>
> </shiro:hasPermission>
> ```

==注意：1.在页面中使用标签来区分不同身份的功能，要么根据角色标签进行划分功能，要么根据资源(权限字符串)进行划分功能==

​			==**2.整合了Thymeleaf可以直接在html标签中的属性使用Shiro标签**==

```html
<a shiro:user>111</a>
```



### 2.SSM使用标签

` jsp导入标签头`

```jsp
<%@taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>
```



### 3.SpringBoot使用

#### `a.导入pom依赖`

```xml-dtd
 <!--在thymeleaf使用shiro-->
        <dependency>
            <groupId>com.github.theborakompanioni</groupId>
            <artifactId>thymeleaf-extras-shiro</artifactId>
            <version>2.0.0</version>
        </dependency>
```



#### `b.ShiroConfig配置类`

```JAVA
  	//4.配置thymeleaf使用shiro标签
    @Bean
    public ShiroDialect shiroDialect() {
        return new ShiroDialect();
    }
```



#### `c.页面导入路径`

```HTML
<html lang="en" xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
```



## 十一. 缓存使用

​				当用户每次访问权限的页面的时候，每次都会访问数据库，但实际上用户的权限基本是固定了的，可以**采用缓存机制来避免用户每次访问时直接访问数据库，从而起到系统优化的效果。**



### 11.1 SpringBoot使用Ehcache缓存

#### `a.导入依赖`

```xml-dtd
    <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
			<version>1.4.0</version>	<!--注意对应shiro的版本-->
        </dependency>
```



#### `b.ShiroConfig配置文件`

```JAVA
	//1.注入自定义realm
    @Bean
    public UserRealm getUserRealm() {
        UserRealm userRealm = new UserRealm();
        //开启缓存
        userRealm.setCacheManager(new EhCacheManager());
        userRealm.setCachingEnabled(true);//开启全局缓存
        userRealm.setAuthenticationCachingEnabled(true);//开启认证缓存
        userRealm.setAuthorizationCachingEnabled(true);//开启授权缓存
        userRealm.setAuthenticationCacheName("authenticationCache");
        userRealm.setAuthorizationCacheName("authorizationCache");
        
        return userRealm;
    }
```



## 十二. 其它

### 12.1 获取登录后的对象

```JAVA
//获取账号
	Subject subject = SecurityUtils.getSubject();

//实现登录
	UsernamePasswordToken token = new UsernamePasswordToken(userVo.getAccount(), userVo.getPassword());

//获取登录对象
TUser currentUser = (TUser) subject.getPrincipal();
```





### 12.2 记住我功能

`1.前台`

```html
<input type="checkbox" name="rememberMe"  checked  />记住我
```



`2.控制器`

```java
	//登录逻辑
    @PostMapping("/login")
    public String login(UserVo userVo, Model model,boolean rememberMe) {
        //获取账号
        Subject subject = SecurityUtils.getSubject();
        
        //实现登录
        UsernamePasswordToken token = new UsernamePasswordToken(userVo.getAccount(), userVo.getPassword());
        try {
            //设置记住我
            token.setRememberMe(rememberMe);
            //登录
            subject.login(token);
	 
         //省略其他
     
        }
    }
```



`3.ShiroConfig配置`

```java
  // 实现记住我，所需要的配置
    @Bean
    public SimpleCookie simpleCookie() {
        // 这个参数是cookie的名称，对应前端的checkbox的name = rememberMe
        SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
        //设为true后，只能通过http访问，javascript无法访问
        //防止xss读取cookie
        simpleCookie.setHttpOnly(true);
        // 记住我cookie生效时间1小时，单位秒
        simpleCookie.setMaxAge(60);
        return simpleCookie;
    }

    // 实现记住我，所需要的配置
    @Bean
    public CookieRememberMeManager cookieRememberMeManager() {
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(simpleCookie());
        // rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度(128 256 512 位)
        cookieRememberMeManager.setCipherKey(Base64.decode("4AvVhmFLUs0KTA3Kprsdag=="));
        return cookieRememberMeManager;
    }

	 //更改DefaultWebSecurityManager对象
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联自定义realm对象
        securityManager.setRealm(userRealm);
        // 实现记住我，所需要的配置
        securityManager.setRememberMeManager(cookieRememberMeManager());
        return securityManager;
    }


```








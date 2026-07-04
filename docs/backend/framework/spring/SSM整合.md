# SSM整合

## 1. 创建Maven项目，导入依赖

```xml-dtd
 <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!--  锁定版本信息  -->
        <!--  spring相关版本  -->
        <spring.version>5.2.8.RELEASE</spring.version>
        <!--  mybatis版本  -->
        <mybatis.version>3.5.5</mybatis.version>
        <!--  mybatis-spring版本  -->
        <mybatis-spring.version>2.0.5</mybatis-spring.version>
        <!--  mysql版本  -->
        <mysql.version>5.1.37</mysql.version>
        <!--  fast json版本  -->
        <fast-json.version>1.2.75</fast-json.version>
        <!--  jackson版本  -->
        <jackson.version>2.11.2</jackson.version>
    </properties>

    <!--  依赖坐标  -->
    <dependencies>
        <!-- spring相关 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
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
        <!-- mybatis相关 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>${mybatis-spring.version}</version>
        </dependency>
        <!--  mysql  -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <!--日志-->
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
        <!--  DBCP数据源  -->
        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.4</version>
        </dependency>
        <!--  log4j  -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <!--  单元测试  -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
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
        <!--  jsp  -->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
            <scope>provided</scope>
        </dependency>
        <!--  servlet  -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        <!--  JSR 303依赖  -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.4.3.Final</version>
        </dependency>
        <!--  fast json  -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fast-json.version}</version>
        </dependency>
        <!--jackson-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <!--  commons-io  -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.7</version>
        </dependency>
        <!--  commons-fileupload  -->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.4</version>
        </dependency>
       <!--jstl标签-->
        <dependency>
            <groupId>jstl</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <!--  pagehelper  -->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.2.0</version>
        </dependency>
    </dependencies>

```



==注意：若想要把Mapper的映射文件放在src/main/java目录下，则要在pom.xml文件中添加以下配置==



```xml-dtd
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



---



## 2. 在 resources文件夹下创建相应配置文件

### 2.1 applicationContext.xml

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx.xsd
           http://www.springframework.org/schema/aop
		   http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 扫描注解所在的包 -->
    <context:component-scan base-package="com.eobard.dao,com.eobard.service"/>


    <!-- 引入database.properties属性文件 -->
    <bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
        <property name="location">
            <value>classpath:db.properties</value>
        </property>
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
        <!-- 加载src/main/resource文件下的SQL映射文件 -->
        <property name="mapperLocations" value="classpath:mapper/**/*.xml"/>
			<!--若想要加载src/main/java文件下则使用下列属性-->
        <!--
			<property name="mapperLocations" value="classpath:com/eobard/dao/**/*.xml"/>
		-->
    </bean>

<!--     使用mybatis-plus的MybatisSqlSessionFactoryBean
    <bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="typeAliasesPackage" value="com.eobard.entity"></property>
        <property name="mapperLocations" value="classpath:mapper/**/*.xml" />
    </bean>-->



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

==注意：不管使用了src/main/java还是src/main/resource的方法，都要在对应的路径下创建Mapper.xml映射文件，否则SSM环境搭建好了之后会启动不了==



### 2.2 db.properties

```properties
jdbc.driver = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/ssm?useUnicode=true&characterEncoding=utf-8
jdbc.user = root
jdbc.password = 123456
```



### 2.3 log4j.properties

```properties
log4j.rootLogger=debug,con
log4j.appender.con=org.apache.log4j.ConsoleAppender
log4j.appender.con.layout=org.apache.log4j.PatternLayout
log4j.appender.con.layout.ConversionPattern=%d{MM-dd HH:mm:ss}[%p]%c%n -%m%n
```



### 2.4 mybatis-config.xml

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



### 2.5 springmvc.xml

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/mvc
           http://www.springframework.org/schema/mvc/spring-mvc.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!-- 加载控制器所在的包 -->
    <context:component-scan base-package="com.eobard.controller"/>

    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 视图前缀（页面在哪个目录下） -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!-- 视图后缀(页面的后缀名是什么) -->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- 加载静态资源 -->
    <mvc:resources mapping="/statics/**" location="/statics/"/>

    <!-- 开启注解支持 -->
    <mvc:annotation-driven>
        <mvc:message-converters>
            <!-- 配置编码字符集 -->
            <!-- 配置响应编码字符集 -->
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

            <!-- 配置Jackson消息转换器 -->
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="objectMapper">
                    <bean class="com.fasterxml.jackson.databind.ObjectMapper">
                        <!-- 格式化日期 -->
                        <property name="dateFormat">
                            <bean class="java.text.SimpleDateFormat">
                                <constructor-arg type="java.lang.String" value="yyyy-MM-dd" />
                            </bean>
                        </property>
                    </bean>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>

    <!-- 配置文件解析器对象，要求id名称必须是multipartResolver -->
    <bean id="multipartResolver"
          class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 设置文件上传限制大小为10M -->
        <property name="maxUploadSize" value="10485760"/>
    </bean>

</beans>
```



### 2.6 根据路径创建Mapper文件

​			根据applicationContext.xml文件的mapperLocations属性，在对应的路径下创建映射文件，该文件夹用于存放Mapper.xml文件



---



## 3. 修改webapp的web.xml

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">


    <!-- 设置欢迎界面 -->
    <welcome-file-list>
        <welcome-file>login.jsp</welcome-file>
    </welcome-file-list>

    <!-- 加载spring配置文件 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- 上下文参数配置 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <!-- 使用*号通配符，通配符前面的字符要一致 -->
        <param-value>classpath:applicationContext*.xml</param-value>
    </context-param>

    <!-- 加载springmvc配置文件 -->
    <servlet>
        <!-- servlet名称，名称自定义，名称唯一 -->
        <servlet-name>springmvc</servlet-name>
        <!-- 前端控制器，SpringMVC核心控制器 -->
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- web项目启动，立即加载springmvc配置文件 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- springmvc配置文件位置 -->
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <!-- servlet名称，名称自定义 -->
        <servlet-name>springmvc</servlet-name>
        <!-- 访问路径-->
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- 配置过滤器 -->
    <!--配置解决中文乱码的过滤器-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
  <!--配置过滤器的url映射路径-->
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
```



---



## 4. 根据配置文件，创建相应的包结构

> com.eobard.entity
>
> com.eobard.dao
>
> com.eobard.service
>
> com.eobard.controller
>
> ....省略其它包



## 5.项目截图

<img src="./SSM整合_images/image-20211122200905824.png" alt="image-20211122200905824" align=left />


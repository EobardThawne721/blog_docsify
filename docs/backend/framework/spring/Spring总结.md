# Spring

## 一 .  Spring IoC

### 	1.1 Spring Ioc概念

​			Ioc—Inversion of Control，即“控制反转”，不是什么技术，而是一种设计思想。在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。

​			

* **谁控制谁，控制什么(控制)：**传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建；**谁控制谁？当然是IoC 容器控制了对象；控制什么？那就是主要控制了外部资源获取（不只是对象包括比如文件等）以及掌控对象的生命周期**

* **为何是反转，哪些方面反转了(反转)：**有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？**因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。**

  ​     ==IoC 不是一种技术，只是一种编程思想/设计模式，一个重要的面向对象编程的法则，有了IoC容器后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是 松散耦合 ，其实IoC对编程带来的最大改变不是从代码上，而是从思想上，发生了“主从换位”的变化。应用程序原本是老大，要获取什么资源都是主动出击，但是在IoC/DI思想中，应用程序就变成被动的了，被动的等待IoC容器来创建并注入它所需要的资源了==



> IoC很好的体现了面向对象设计法则之一   好莱坞法则：“don't call me ,i will call you ”；即由IoC容器帮对象找相应的依赖对象并注入，而不是由对象主动去找



---



### 1.2 Spring Ioc & DI 解析及原理

#### 			1.2.1 DI(依赖注入)解析

*  **谁依赖于谁：**当然是**应用程序依赖于IoC容器**；
*  **为什么需要依赖：**应用程序需要IoC容器来提供对象需要的外部资源；
*  **谁注入谁：**很明显是**IoC容器注入应用程序某个对象，即应用程序依赖的对象**
*  **注入了什么：**就是**注入某个对象所需要的外部资源（包括对象、资源、常量数据）**



>  如果一个对象A需要另一个对象B才能实现某一个功能：Service层需要Dao层的依赖，这时就可以说A依赖于B；而Spring容器在创建A对象时，会自动将A对象需要的B对象注入到A对象中，这个过程就叫做依赖注入





#### 1.2.2 Ioc 解析

>  举个例子来说
>
> ​	传统方式找工作：我们需要自己去找公司然后将自己的信息(属性)介绍给它们，然后讲解自己的特点和爱好之类的，这其中可能就会有许多的过程了就类比于自己手动设置一些setter
>
> ​	通过Ioc来找工作：在自己和公司之间建立了一个第三方(如BOSS直聘）,上面管理了许多公司的信息，我们需要什么样的公司只需要告知它们，它们就会自动的给我们提供；如果它们提供的公司与我们的不符合要求，我们就会pass掉，并且不舒服，也就是会报异常。

​			==Spring所倡导的开发方式所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。==



#### 		1.2.3 Ioc实现原理

​				反射（通过默认无参构造new对象）+内省机制



---



### 	1.3 Spring Ioc实现依赖注入(DI)

#### 				1.3.1 Setter注入

​					通过无参构造实例，灵活性好，通过setter方式注入			

```java
public class User{
    private String name;
    
    public String getName(){
        return this.name;
    }
    
    public void setName(String name){
        this.name=name;//为属性赋值
    }
}
```



#### 		1.3.2  构造器注入

​				通过匹配构造方法实例，建议保留无参构造，灵活性差

```java
public class User{
    private String name;
    
    public User(){}
    
    public User(String name){ 
        this.name=name;//为属性赋值
    }
}
```



---



## 二.Ioc容器的管理

### 	2.1 BeanFactory概念及管理

​		BeanFacrory在调用getBean()之前是不会实例化任何对象,只有在需要创建JavaBean的实例对象时才会为其分配空间。==**这使得它更适用于物理资源受限制的应用程序，尤其是内存限制的环境**==



---



### 	2.2 ApplicationContext的应用及配置文件的拆分

#### 2.2.1 ApplicationContext的应用

​			ApplicationContext扩展了BeanFactory容器并添加了一些强大的功能，它有三个实现类，可以实例化其中一个来创建spring的ApplicationContext容器。

* ClassPathXmlApplicationContext: 从当前src根路径开始来创建

```java
ApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");//在src根路径
```





* FileSystemXmlApplicationContext: 从通过参数指定文件的位置寻找，可以获取src根路径之外的资源

```java
ApplicationContext context=new FileSystemXmlApplicationContext("D:/project/bean.xml");//在其他路径
```





* WebApplicationContext：是spring的web 应用容器，在servlet的web.xml文件中配置spring的ContextLoadeListener监听器

```xml
		<!--监听 -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<context-param>
		<!--加载以spring开头的配置文件 -->
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>	
	</context-param>

```

#### 		

#### 2.2.2 配置文件的拆分

​			拆分applicationContext.xml文件，将不同模块拆分成几个小模块，在主文件引入即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation=" 
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd 
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd 
 http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd 
">
    <import resource="spring-dao.xml" />
    <import resource="spring-service.xml" />
    <import resource="spring-controller.xml" />
</beans>
```



---



### 	2.3 BeanFactory和ApplicationContext的区别

​				1.BeanFactory采用了<font color="red">**延迟加载**</font>方案，只有在调用getBean()才会实例Bean, 如果某一个Bean的属性没有注入，则会在getBean的时候抛出异常

​				2.ApplicationContext则在初始化自检，在容器启动后会<font color="red">**一次性创建**</font>所有的Bean，会检查所有依赖的属性是否注入了

​		==在实际开发中，通常情况下会选择ApplicationContext，而只有在系统资源较少时，才考虑BeanFactory==





---





## 三. Bean的配置

### 	3.1  注入方式

#### 		3.1.1 XML配置文件注入

​		==需要注入的对象1==

```java
//一个普通的JavaBean
public class User{
    private int id;
    private String name;
    
    public User(){}
    
    public User(int id,String name){ 
        this.id=id;
        this.name=name;
    }
  //省略getter、setter
}
```

​		==需要注入的对象2==

```java
public class Collection{
    private List<Integer> list; //注入集合
	private Map<Integer, String> map;//注入map
	private Set<String> set;//注入set
	private Object[] array;//注入数组
	private Properties prop;//注入属性
    
    private User user; //引用user
   
    public Collection(){}
  
    public Collection(User user){ 
       this.user=user;
    }
    
    //省略getter、setter
}
```

​		==applicationContext.xml文件开始注入==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation=" 
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd 
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd 
 http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd 
">

<!--1.简单注入User对象-->
<bean id="user" class="com.gxp.entity.User" />

<!--2.setter注入User对象并设置属性值-->
<bean id="user2" class="com.gxp.entity.User">
		<!-- property属性:属性注入,实体类中的属性必须要有setter方法且这里的name值为实体类中的属性名-->	
		<property name="name" value="gxp" />
</bean>

<!--2.构造器注入User对象并设置属性值-->
<bean id="user3" class="com.gxp.entity.User">
    	<!-- constructor-arg属性:实体类要有构造方法,建议加上无参构造;要求注入值的顺序要对应构造方法的顺序-->	
   		<constructor-arg value="11" index="0" />
		<constructor-arg value="gxp" index="1" />
</bean>

<!--3.P命名空间(实则也是setter注入)注入User对象并设置属性值
	注入方式 p:属性名="值"
-->   
<bean id="user4" class="com.gxp.entity.User" p:id="1" p:name="刘德华"  />
    
<!--4.注入复杂类型并设置属性值--> 
<bean id="collection" class="com.gxp.entity.Collection">
    	<property name="list">
			<list>
				<value>2</value>
				<value>1</value>
			</list>
		</property>
    
		<property name="map">
			<map>
				<entry key="1" value="gxp" />
				<entry key="2" value="cgx" />
			</map>
		</property>
    
		<property name="set">
			<set>
				<value>郭富城</value>
				<value>陈冠希</value>
			</set>
		</property>
    
		<property name="array">
			<array>
				<value>1</value>
				<value>2</value>
			</array>
		</property>

		<property name="prop">
			<props>
				<prop key="url">jdbc</prop>
				<prop key="name">root</prop>
			</props>
		</property>
	</bean>

<!--5.注入引用类型并设置属性值三大方式-->
    
    <!--a.第一种按照属性注入：通过ref来指定Ioc容器中依赖对象的id-->
<bean id="collection2" class="com.gxp.entity.Collection">
    <property name="user" ref="user" />
</bean> 
    
    <!--b.第二种按照构造器注入：通过ref来指定Ioc容器中依赖对象的id-->
<bean id="collection3" class="com.gxp.entity.Collection">
   <constructor-arg ref="user" index="0"/>
</bean>    
    
    <!--c.第三种按照p命名空间注入:通过p:属性名-ref="Bean的id值"-->
<bean id="collection4" class="com.gxp.entity.Collection" p:user-ref="user" />  

</beans>



```

>  小技巧：可以在Eclipse中下载Spring的插件,可以在XML文件书写有提示。 打开帮助按钮->Eclipse MarketPlace->搜索Spring即可



---



#### 		3.1.2 注解注入(推荐使用!)

​			1.通过@Controller,@Service,@Repository,@Component都可以注入一个Bean，还可以在里面指定Bean的id**(如果不指定，则id默认为类名且首字母小写)**; <font color="red" size=4>但是一般规定前面三个最好是指定为控制器,业务逻辑层，dao层的Bean，最后一个指定为组件类(工具类)的Bean</font>  

​			2.在application.xml文件中开启注解扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation=" 
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd 
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd 
 http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd 
">
	<!--使用了注解的模式必须要扫描,多个包之间可以用逗号隔开:
 		base-package="com.eobard.dao,com.eobard.service,com.eobard.coontroller" 
  -->
    
    <!--直接扫描com.eobard包下所有-->
<context:component-scan base-package="com.eobard"/>

</beans>
```





​			==需要注入的对象1==

```java
//一个普通的JavaBean
@Component("user")     //相当于注入一个bean的id为user的对象
public class User{
    
	@Value("gxp") 		//通过暴力反射注入值,破坏了封装性
    private String name;
    
    private int id;
    @Value("123") 		//推荐使用set方法注入 
    public void setId(int id){ 
        this.id=id;
    }
    
  //省略其它getter、setter
}
```

​		

​			==需要注入的对象2==

```java
//接口
public interface UserDao{
    
}
//实现类
@Repository("userDao")
public class UserDaoImp implements UserDao{

}

```

​			==需要注入的对象3==

```java
//接口
public interface UserService {
}
//实现类
@Service("userService")
public class UserServiceImp implements UserService {

}

```

​			==需要注入的对象4==

```java
@Controller("userController")
public class UserController {	
}

```



---

### 3.2 ApplicationConxtex.xml读取外部配置文件&动态获取值

#### 3.2.1 读取外部配置文件

```properties
#外部配置文件连接信息  db.properties
jdbc.user=root
jdbc.password=123456
jdbc.url=jdbc:mysql://localhost:3306/ssh_shop?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&rewriteBatchedStatements=TRUE
jdbc.class=com.mysql.jdbc.Driver
```

```xml-dtd
<!--ApplicationContext.xml文件-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation=" 
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd 
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd 
 http://www.springframework.org/schema/tx 
 http://www.springframework.org/schema/tx/spring-tx-2.5.xsd 
">
	<!--开启注解 -->
	<context:component-scan base-package="com.eobard" />
	
	<!-- 配置外部配置文件 -->
	<bean
		class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="location">
			<value>classpath:db.properties</value>
		</property>
	</bean>

	<!--获取外部配置文件信息 -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="user" value="${jdbc.user}" />
		<property name="password" value="${jdbc.password}" />
		<property name="jdbcUrl" value="${jdbc.url}" />
		<property name="driverClass" value="${jdbc.class}" />
	</bean>

</beans>
```

==注意：这种形式十分常用，通过类似于EL表达式的形式可以动态的获取外部配置文件值==



#### 3.2.2 动态获取实体类注入的值

```java
//工具类
@Component
public class Utils {

	@Value("1.0")
	private String version;
	
	@Value("admin")
	private String identity;
	
	@Value("2021-8-9 12:00:00")
	private String time;

	//省略getter，setter
}

//系统信息类
public class SysInfo {

	private String version;
	
	private String identity;
	
	private String time;

 	//省略其它属性和getter，setter
}
```

```xml-dtd
<!--applicationContext.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation=" 
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd 
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd 
 http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd 
">

<context:component-scan base-package="com.eobard" />
	
    <bean id="sys" class="com.eobard.domain.SysInfo">
        <property name="version" value="#{utils.version}"/>
        <property name="identity" value="#{utils.identity}"/>
        <property name="time" value="#{utils.time}"/>
    </bean>

</beans>
```



==注意：通过类似于Ognl表达式的形式可以动态获取类中注入的值，通过 类的id.属性名 就可以获取，**前提是该类一定是先注入了值的**==

---



### 	3.3  Bean的相关属性介绍

#### 3.3.1 default-lazy-init属性:

​			在`<beans>`根标签下设置，指定改标签下所有bean是否都延迟加载 

#### 3.3.2lazy-init属性 :

​			在`<bean>`标签下设置，表示延迟加载,单例模式中运行容器默认会加载所有Bean,运用了延迟加载,只有在用到这个类的时候才会实例化

#### 3.3.3 init-method 属性:

​			在`<bean>`标签下设置, bean对象创建完之后运行初始化方法，对应该bean内部的某个方法名

#### 3.3.4 destroy-method 属性

​			在`<bean>`标签下设置,容器关闭之后运行销毁方法(如果是多例模式,不会执行这个方法)

```java
public class Test {
	private Integer id;
	private String sts;

	//省略getter，setter
    
	public void init()
	{
		System.out.println("bean初始化");
	}
	public void des() 
	{
		System.out.println("bean销毁");
	}

}
```

```xml
	<!--applicationContext.xml里面的设置-->
<bean id="t1" class="com.eobard.spring.Test" init-method="init" destroy-method="des" />
```



#### 3.3.5 depends-on属性

​			在`<bean>`标签下设置，在该Bean初始化之前强制执行指定的Bean初始化

```xml
	<!--applicationContext.xml里面的设置-->
	<bean class="com.eobard.spring.domain.Son" id="son" depends-on="father">
		<property name="father" ref="father" />
	</bean>
	
	<bean class="com.eobard.spring.domain.Father" id="father" />
```





---



### 3.4  Bean的实例化

​		==需要注入的对象==

```java
//一个User的JavaBean
public class User {

	private int id;
	private String name;
    //省略getter,settter
}
```



#### 3.4.1 构造器实例化(默认)

```xml
<bean id="user" class="com.eobard.domain.User" />
```



#### 3.4.2  静态工厂实例化

```java
/**
* 	通过静态方法实例化User
*/
public class StaticBeanFactory {

	public static User getUser() {
		return new User();
	}
}
```

```xml
		<!-- 使用静态工厂实例化:
			class：工厂的类限定名
			factory-method：工厂的静态方法
		 -->
<bean id="user" class="com.eobard.config.StaticBeanFactory" factory-method="getUser" />
```



#### 3.4.3  实例工厂实例化

```java
/**
*	通过实例化工厂来实例化User
*/
public class BeanFactory {
	public BeanFactory() {
		System.out.println("工厂方式实例化，正在调用工厂的构造方法...");
	}
	
	public User getUser() {
		return new User();
	}
}
```

```xml
 	<!--	1.首先要将工厂实例化
			2.然后在需要创建的对象指定工厂bean和工厂方法即可，无需声明class
	-->
<bean id="beanFactory" class="com.eobard.config.BeanFactory" />
<bean id="userByFactory" factory-bean="beanFactory" factory-method="getUser"/>
```



---



### 3.5 Bean的获取

​		通过ApplicationContext的实现类来获取指定id的Bean

```java
//1.获取IOC容器，并初始化所有Bean
ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
//2.获取id为指定的Bean,如果找不到指定id的Bean则会异常
User user1=ac.getBean("user",User.class);
User user2=ac.getBean("user",User.class);
//3.判断两个对象是否一致：true，说明IOC容器默认为单例模式
System.out.println(user1==user2);
```

---



### 	3.6   Bean的装配及简化三层

#### 3.6.1 注解装配其他Bean

##### 3.6.1.1 byType 注入

​				通过@AutoWired注入，若出现多个类型相同并注入则会异常

​		==正常注入1==

```java
//Dao层
@Repository
public class DaoImp implements Dao {

}

//service层
@Service
public class ServiceImp implements Service{	
	@Autowired
	private Dao dao; 	//通常是引用接口层，这叫做面向接口开发  
	//注解注入不需要提供getter，setter
}
```

​		==正常注入2==

```java
//service层
@Service
public class PayService1 {

}
@Service
public class PayService2 {

}

//控制器层
@Controller
public class PayController {
	private PayService1 payService1;
	private PayService2 payService2;
    
    public PayController(){}
    
	//autowired通过构造方法注入多个service
	@Autowired
	public PayController(PayService1 payService1, PayService2 payService2) {
		this.payService1 = payService1;
		this.payService2 = payService2;
	}
}

```

​	

​		==异常注入==

```java
//Dao层:有两个实现类
@Repository
public class DaoImp1 implements Dao {
}

@Repository
public class DaoImp2 implements Dao {
}

//service层
@Service
public class ServiceImp implements Service{	
	@Autowired
	private Dao dao; 	 
}
```

​	<font color="red" size=4>这样注入会异常，因为ServiceImp是按照类型注入，而Dao的实现类有两个，ServiceImp不知道注入谁</font>



​	**解决方法1**：在Dao层任意一个实现类中加上注解@Primary,则会默认注入这个,而不会注入另外一个

```java
//Dao层:有两个实现类
@Repository
@Primary
public class DaoImp1 implements Dao {
}

@Repository
public class DaoImp2 implements Dao {
}

//service层
@Service
public class ServiceImp implements Service{	
	@Autowired
	private Dao dao; 	 
}
```



​	**解决方法2**：配合@Qualifier手动指定

```java
//Dao层:有两个实现类
@Repository
public class DaoImp1 implements Dao {
}

@Repository
public class DaoImp2 implements Dao {
}

//service层
@Service
public class ServiceImp implements Service{	
	@Autowired
    @Qualifier("daoImp1")
	private Dao dao; 	 
}
```

---



##### 3.6.1.2 byName注入

​			通过@Qualifier注入，可以通过指定对应bean的id注入

```java
//Dao层
@Repository
public class DaoImp implements Dao {
}

//service层
@Service
public class ServiceImp implements Service{	
	@Qualifier("daoImp")
	private Dao dao; 	 
}
```

---



##### 3.6.1.3 Resource注入(推荐使用)

​			通过@Resource来注入，可以通过指定name属性对应bean的id注入，若找不到则可以指定type属性按照类型注入，<font color="red">这是J2EE的注解，并不是spring提供的</font>

```java
//Dao层
@Repository
public class DaoImp implements Dao {
}

//service层
@Service
public class ServiceImp implements Service{	
	@Resource(name="daoImp")
	private Dao dao; 	 
}
```

---



#### 3.6.2  简化三层

##### 3.6.2.1 注解形式(推荐使用!)

```java
//控制器
@Controller
public class Controller{
    @Autowired
	private Service service;
}

//业务逻辑层
public interface Service{    
}

@Service
public class ServiceImp implements Service {
	@Autowired
	private Dao Dao;
}

//数据访问层
public interface Dao{
}

@Repository
public class DaoImp implements Dao{
}


```



##### 3.6.2.2 xml文件形式

```java
//控制器
public class Controller{
	private Service service;
    //提供setter方便setter注入
    public void setService(Service service){
        this.service=service;
    }
}

//业务逻辑层
public interface Service{    
}

@Service
public class ServiceImp implements Service {
	private Dao dao;
    //提供setter方便setter注入
	public void setUserDao(Dao dao) {
		this.dao = dao;
	}
}

//数据访问层
public interface Dao{
}

@Repository
public class DaoImp implements Dao{
}

```



---



### 	3.7 Bean的作用域

#### 			3.7.1 singleton(单例，默认值)

​		Spring以单例模式创建Bean的实例，该容器中Bean的实例只有一个。 单例模式对于无会话状态的Bean(如Dao组件,Service组件)是最理想的选择。

​		==注解形式==

```java
@Controller
@Scope("singleton")  //默认值，可以省略不写
public class Dao {
    
}
```

​		==XML文件形式==

```xml
<!--applicationContext.xml里面的设置-->
<bean id="user" class="com.eobard.spring.User" scope="singleton" />
```



#### 			3.7.2 prototype(原型)

​			每次从容器中获取Bean时，都会创建一个新的实例。对需要保持会话状态的Bean(如struts的Action)，在每次请求都会创建一个新的实例

​			==注解形式==

```java
@Controller
@Scope("prototype")
public class UserAction {
    
}
```

​			==XML文件形式==

```xml
<!--applicationContext.xml里面的设置-->
<bean id="user" class="com.eobard.spring.User" scope="prototype" />
```



#### 			3.7.3 request

​			用于web应用环境，针对每次HTTP请求都会创建一个新的实例

#### 			3.7.4 session

​			用于web应用环境，同一个会话共享一个实例，不同的会话使用不同的实例

#### 			3.7.5 global session

​			尽在Portlet的web应用环境中，同一个全局会话共享一个实例；对非Portlet的环境，等同于session

---

## 四.Spring AOP

### 4.1 AOP概述&原理

​			   概述：一种通过预编译和运行时动态代理的方式，在不修改源代码动态添加新功能

​			   原理： 1. 将复杂的需要分解不同方面，将公共功能集中解决

​						    2.采用动态代理机制，不改变源程序基础上，对代码进行增强处理  



### 4.2 熟悉动态代理

​			SpringAOP 有两种代理(优先使用Proxy代理): **1.Proxy:被代理的类必须要实现接口 2.Cglib:被代理的类不能被final修饰,基于继承。**

​			==Proxy实现动态代理==

```java
//service层
public interface UserService {
	void add();
	void delete();
}

public class UserServiceImp implements UserService {
	@Override
	public void add() {
		System.out.println("add");
	}

	@Override
	public void delete() {
		System.out.println("delete");

	}
}


//代理类: 对service层进行增强处理,手动开启/提交事务
public class UserProxy {

	public static UserService getUserServiceProxy(UserService us) {
        
        //Proxy.newProxyInstance(参数1,参数2,参数3)
        //参数1: 	 代理类的字节码的加载器
        //参数2:	  被代理类的类字节码的接口
        //参数3:	new InvocationHandler()匿名内部类
        
        // method.invoke(参数1, 参数2);  动态增强就需要在这句话前后增强
        //参数1：传入的被代理类对象
        //参数2：方法调用的参数 
		return (UserService) Proxy.newProxyInstance(UserProxy.class.getClassLoader(), 
								us.getClass().getInterfaces(),
								new InvocationHandler() {
									@Override
									public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
										System.out.println("开启service层的事务");
										Object invoke = method.invoke(us, args);
										System.out.println("提交/回滚事务");
										return invoke;
									}
								});
	}
}

//测试类
		UserService service = UserProxy.getUserServiceProxy(new UserServiceImp());
		service.add();
/**
输出结果：
*		开启service层的事务
			add
		关闭/回滚事务
*/

```

​		

### 4.2 AOP相关术语

#### 			4.2.1 切面

> ​	切面是通知和切点的结合，共同定义了切面的全部内容----它是什么，在何时何处完成其功能
>
> ​	实际上切面是一段程序代码，这段代码将被植入到程序流程中

​							

#### 			4.2.2  通知(Advice)

> ​	定义了切面是什么以及何时使用，还解决了何时执行



* ​						前置增强(Before):		在连接点方法调用之前处理

* ​                        后置返回增强(AfterReturning):	在连接点调用方法成功后处理

* ​                        异常增强(AfterThrowing):		在连接点方法抛出异常后处理

* ​                        最终增强(After):			在连接点方法调用后无论成功与否再处理

* ​                        环绕增强(Around):		在连接点方法调用前和调用后自定义;<font color="red">功能最强，包含了前面4种(用了环绕就不能用前面4种；反之)</font>



#### 			4.2.3 连接点对象

>  连接点：程序流程上的任意一点，对象的某一个操作，对象调用某一个方法，切面代码可以利用这点插入到应用的正常流程之中

​								JoinPoint ：

​													getTarget()		//获取当前连接点对象的类

​													getSignature().getName();//获取当前连接点对象的方法名

​													getArgs();		//获取当前连接点对象方法的参数列表

​								ProceedingJoinPoint(用于环绕增强,方法同上)

​													proceed();//该方法之前为前置增强逻辑,之后为后置返回增强逻辑

​						==切入点：所有连接点的集合==



#### 			4.2.4 切点表达式(execution("表达式"))

```
* com.service.*.*(..)  			//匹配com.service包下所有类的成员方法
* com.service..*.*(..) 			//匹配com.service包及其子包下所有类的成员方法
* com.service.Service.*()		//匹配com.service包下实体类为Service的任意无形参方法
* com.service.Service.*()  		//匹配com.service包下实体类为Service开头的任意无形参方法
* com.service.Service*.*(..)	//匹配com.service包下实体类为Service开头的形参个数自定义的方法

public * com.service.BaseService.do(java.lang.String,..)
//匹配com.service包下实体类为BaseService中 public的第一个形参必须为String的do方法
```

==注意:其中在包名前面的第一个*代表的是任意一种返回类型，====若第一个*前面没有修饰符则代表所有类型的修饰方法都可以匹配==



#### 4.2.5 切点表达式(within("表达式"))

```java
com.eobard.service.*		//匹配com.eobard.service包下的所有类
com.eobard.service..*    	//匹配com.eobard.service包及其子包下的所有类
```

> **注：within针对的是某个类，粗粒度；execution精确的是某个方法，细粒度**





#### 4.2.6 细化切入点范围，提高性能

```java
//切入点:controller包下的所有类并且有LogInfo注解的所有方法
@Around("within(com.eobard.controller.*) && @annotation(com.eobard.annotation.LogInfo)")
```







### 4.3 注解形式实现AOP(推荐使用)

**<font color="red">		注意：以下两种方法都是AspectJ开发，一种是基于AspectJ的注解开发，一种是基于AspectJ注解开发(推荐使用基于AspectJ注解开发)</font>**



#### 4.3.1 注解AOP流程

> AOP注解方式:
>
> ​		1.首先在类上定义切面	@Aspect
>
> ​		2.编写空切点方法并定义切点表达式		@Pointcut("execution("切点表达式")")
>
> ​		3.根据需要编写通知(Advice),且每个方法的注解都要引入切点表达式的方法
>
> ​					a.前置:		@Before("切点方法()")
>
> ​					b.后置返回:	@AfterReturning("切点方法()",returning="方法返回值形参")
>
> ​					c.后置最终:	@After("切点方法()")
>
> ​					d.异常增强:	@AfterThrowing("切点方法()",throwing="方法异常形参")
>
> ​					e.环绕增强:	@Around("切点方法()")
>
> ​		4.在XML文件注入切面实体类,不能通过注解来注入!



#### 4.3.2 注解AOP使用

​					首先应在applicationContext.xml文件中配置注解扫描AOP

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation=" 
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd 
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd 
 http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd 
">

<!--开启AOP注解支持  -->
<aop:aspectj-autoproxy />

<!--用了注解AOP的方式必须要将该类注入IOC容器中(推荐使用@Component注解注入) -->
<bean id="aopLogger" class="com.eobard.annotation.aop.AllServiceLogger" />
<bean id="aop1" class="com.eobard.annotation.aop.Aop1"/>

</beans>
```

​				==简单的AOP(不包含环绕)==

```java
//定义切面
@Aspect
public class AllServiceLogger {

	//定义切点方法并标识切点表达式
	@Pointcut("execution(* com.eobard.annotation.service.*.*(..))")
	public void pointCut() {}
	
	//前置增强,里面的方法表示引用切点表达式的方法
	@Before("pointCut()")
	public void before(JoinPoint jp) {
		System.out.println("注解前置增强：" + jp.getTarget() + "=> " + jp.getSignature().getName() +  ",参数列表："+ Arrays.toString(jp.getArgs()));
	}
	
	
	//后置返回增强,里面的方法表示引用切点表达式的方法,且接收返回值为形参的返回值
	@AfterReturning(pointcut="pointCut()",returning ="result")
	public void afterReturning(JoinPoint jp,Object result) {
        System.out.println("注解后置返回增强：" + jp.getTarget() + "=>" + jp.getSignature().getName() + ",参数列表"+ Arrays.toString(jp.getArgs()) + ",返回值类型:" + result);
	}
	
	
	//后置最终增强,里面的方法表示引用切点表达式的方法
	@After("pointCut()")
	public void afterFinally(JoinPoint jp) {
         System.out.println("注解最终增强：" + jp.getTarget() + "=>" + jp.getSignature().getName());
	}
	
	//异常增强,里面的方法表示引用切点表达式的方法,且接受异常返回值为形参的返回值
	@AfterThrowing(pointcut="pointCut()",throwing ="e" )
	public void afterThrowing(JoinPoint jp,RuntimeException e) {
           System.out.println("注解异常增强：" + jp.getTarget() + "=> " + jp.getSignature().getName() + "" + ",异常:" + e.getMessage());
	}
	
	
}

```



​					==环绕增强==

```java
@Aspect
public class Aop1 {
	
	@Pointcut("execution(* com.eobard.annotation.service.*(..))")
	public void pointCut() {}

	@Around("pointCut()")
	public Object around(ProceedingJoinPoint jp){
		Object result=null;
	try {
         System.out.println("前置增强"+jp.getTarget().toString()+" "+jp.getSignature().getName());
			result=jp.proceed();
         System.out.println("环绕增强"+jp.getTarget().toString()+" "+jp.getSignature().getName()+"返回值:"+result);
        
	} catch (Throwable e) {
			System.out.println("异常增强"+e.getMessage());
	}
	finally{
			System.out.println("最终增强");
	}
		
		return result;
	}
	
}

```





### 4.4 XML文件实现AOP

​					 首先要在applicationContext.xml配置

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>


<!-- 
	default-lazy-init属性(true/false):在<beans>标签下设置指定改标签下所有bean是否都延迟加载 
-->

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation=" 
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd 
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd 
 http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd 
">
<!--以下是AOP切面的配置 -->
	<!--1.注入aop增强处理的实体类 -->
	<bean id="userServiceLogger" class="com.gxp.aop.AllServiceLogger" />
	<!-- 2.配置切面 -->
	<aop:config>
		<!-- 3.定义切点,规则:当前包及其子包下所有类的所有方法 -->
		<aop:pointcut id="pointcut"
			expression="execution(* com.gxp.service..*.*(..))" />
		<!-- 4.使用切面编程 -->
		<aop:aspect ref="userServiceLogger">
			<!-- 前置增强 :这里的method与增强处理的前置增强方法名一致 -->
			 <aop:before method="before" pointcut-ref="pointcut" /> 
			<!-- 后置(返回)增强:这里的method与增强处理的后置增强方法名一致,这里的result与通知(Advice)的返回形参一致 -->
			 <aop:after-returning method="afterReturning" pointcut-ref="pointcut" 
				returning="result"/>
			<!-- 异常增强:这里的method与增强处理的异常增强方法名一致,这里的e与通知(Advice)的返回形参一致 -->
			 <aop:after-throwing method="afterThrowing" pointcut-ref="pointcut" 
				throwing="e"/> 
			<!--后置增强:这里的method与增强处理的后置增强一致 -->
			 <aop:after method="afterFinally" pointcut-ref="pointcut"/> 

			<!-- 环绕增强(一般用了环绕就不用其他的增强了,环绕增强包含了其它几个):这里的method与增强处理的环绕增强一致 -->
			<!--<aop:around method="around" pointcut-ref="pointcut" />-->
		</aop:aspect>
	</aop:config>


</beans>
```

​	

​				AOP实体类

```java
/**
 * 面向切面编程:将所有BLL层作为一个切面(Aspect),将日志输出抽取出来进行增强处理
 */
public class AllServiceLogger {

	/**
	 * 前置增强 JoinPoint 连接点对象 jp.getArgs()获取参数列表 Arrays.toString(jp.getArgs())显示出参数列表
	 */
	public void before(JoinPoint jp) {
         System.out.println("前置增强--  调用了" + jp.getTarget() + "的 " + jp.getSignature().getName() + "" + ",方法参数列表"+ Arrays.toString(jp.getArgs()))
	}

	/**
	 * 后置返回增强
	 */
	public void afterReturning(JoinPoint jp, Object result) {
         System.out.println("后置增强-- 调用了" + jp.getTarget() + "的 " + jp.getSignature().getName() + "" + ",方法参数列表"+ Arrays.toString(jp.getArgs()) + "方法返回值类型:" + result);
	}

	/**
	 * 异常增强:当发生异常的时候,捕捉异常可以将异常信息记录在日志中
	 */
	public void afterThrowing(JoinPoint jp, RuntimeException e) {
         System.out.println("异常增强-- 调用了" + jp.getTarget() + "的 " + jp.getSignature().getName() + "" + ",异常信息是" + e.getMessage());
	}

	/**
	 * 后置增强：类似于try-finally的finally块可以用于资源的关闭或者释放
	 */
	public void afterFinally(JoinPoint jp) {.
         System.out.println("后置增强---调用了" + jp.getTarget() + "的" + jp.getSignature().getName());
	}

	/**
	 * 	环绕增强(形参必须是ProceedingJoinPoint )，功能最强的一个：可以包含前置、后置、异常、最终增强
	 *	
	 */
	public Object around(ProceedingJoinPoint jp) {
		Object result = null;
		try {
            System.out.println("环绕增强之前的处理逻辑：" + jp.getTarget() + ":处理方法是" + jp.getSignature().getName() + ",方法参数列表"+ Arrays.toString(jp.getArgs()));
			result=jp.proceed();// 在当前方法处理之前的逻辑是前置增强，处理之后的逻辑是后置增强,还可以在catch语句中获取最终增强
             System.out.println("环绕增强之后的处理逻辑" + jp.getTarget() + "的 " + 									jp.getSignature().getName() + "" + ",方法参数列表"+ Arrays.toString(jp.getArgs()) + "方法返回值类型:" + result);
            
		} catch (Throwable e) {
              System.out.println("环绕增强的异常信息-- 调用了" + jp.getTarget() + "的 " + jp.getSignature().getName() + "" + ",异常信息是"+ e.getMessage());
			e.printStackTrace();
		} finally {
             System.out.println("环绕增强的finally块---调用了" + jp.getTarget() + "的" + jp.getSignature().getName());
		}
		return result;
	}
}

```





### 4.5 web环境下监听session的应用

​				web环境下用aop来记录用户日志需要获取session这样就必须在web.xml中监听RequestContextHolder就要配置

```xml

 		<!-- 监听request：方便aop获取session -->
  	<listener>
        <listener-class>
       		 org.springframework.web.context.request.RequestContextListener
        </listener-class>
   </listener>
```

---





## 五. JdbcTemplate操作

### 5.1常用API

```
常用的API:
		1.通常执行不带返回值的sql语句,如创建表,增删改
			void execute(String sql);
			
		2.针对单行单列查询,返回单个值(参数2:：基本类型数据的字节码 或者  类类型 :new BeanPropertyRowMapper<T>(T.class))
			Object queryForObject(String sql,Class type,Object...args);
			T queryForObject(String sql,RowMapper<T> rowMapper,Object...args);
		
		3.针对多行
			List query(String sql,new BeanPropertyRowMapper<T>(T.class));
```





### 5.2 不依靠Srping Ioc容器操作JdbcTemplate

​					不借助Spring Ioc<font color="red">需要依赖C3P0连接池来操作，所以先要配置好C3P0</font>

```java
/**
 * 	使用jdbcTemplate开发 : 不借助IOC容器
 */
public class DeptDaoImp {

	private static ComboPooledDataSource dataSource;
	//jdbc模板
	private JdbcTemplate template = new JdbcTemplate(dataSource);

	static {
        //省略了C3P0的配置XML
		dataSource = new ComboPooledDataSource("MySql");
	}

	//增加
	public void save(Dept dept) {
		String sql="insert into dept values(null,?,?)";
		template.update(sql, dept.getDeptName(),dept.getDeptCareer());
	}

	//删除
	public void deleteById(Integer id) {
		String sql="delete from dept where deptNo=?";
		template.update(sql, id);
	}

	//修改
	public void update(Dept dept) {
		String sql="update dept set deptName=?,deptCareer=? where deptNo=?";
		template.update(sql, dept.getDeptName(),dept.getDeptCareer(),dept.getDeptNo());
	}


	//查询单个
	public Dept findDeptById(Integer id) {
		Dept d = template.queryForObject("select * from dept where deptNo=?", new BeanPropertyRowMapper<Dept>(Dept.class), id);
		return d;
	}
	
	//查询数量
	public Integer getDeptCount() {
		return	template.queryForObject("select count(*) from dept", Integer.class);
	}

	//查询全部
	public List<Dept> findAll() {
		List<Dept> list = template.query("select * from dept", new BeanPropertyRowMapper<Dept>(Dept.class));
		return list;
	}
	
}
```

```
//测试
	DeptDaoImp dao=new DeptDaoImp();
	dao.XXX()
```



### 5.3 依靠Spring Ioc容器操作

```java
/**
 * 	使用jdbcTemplate开发 : 借助IOC容器注入
 */
@Repository("deptDao")
public class DeptDaoImp  {

	@Autowired
	private JdbcTemplate template;

	// 增加
	public void save(Dept dept) {
		String sql = "insert into dept values(null,?,?)";
		template.update(sql, dept.getDeptName(), dept.getDeptCareer());
	}

	// 删除
	public void deleteById(Integer id) {
		String sql = "delete from dept where deptNo=?";
		template.update(sql, id);
	}

	// 修改
	public void update(Dept dept) {
		String sql = "update dept set deptName=?,deptCareer=? where deptNo=?";
		template.update(sql, dept.getDeptName(), dept.getDeptCareer(), dept.getDeptNo());
	}

	// 查询单个
	public Dept findDeptById(Integer id) {
		Dept d = template.queryForObject("select * from dept where deptNo=?",
				new BeanPropertyRowMapper<Dept>(Dept.class), id);
		return d;
	}

	// 查询数量
	public Integer getDeptCount() {
		return template.queryForObject("select count(*) from dept", Integer.class);
	}

	// 查询全部
	public List<Dept> findAll() {
		List<Dept> list = template.query("select * from dept", new BeanPropertyRowMapper<Dept>(Dept.class));
		return list;
	}

}

```

```xml
<!--applicationContext.xml中-->
<context:component-scan base-package="com.eobard" />
	<!--注入C3P0连接池  -->
	<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="user" value="root" />
		<property name="password" value="123456" />
		<property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test_db?useUnicode=true&amp;characterEncoding=utf-8" />
		<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="checkoutTimeout" value="30000"/>
		<property name="maxPoolSize" value="20" />
		<property name="initialPoolSize" value="10" />
	</bean>
```

```
//测试
	ApplicationContext ac=new ClassPathXmlApplicationContext("applicationContext.xml");
	DeptDaoImp dao=ac.getBean("deptDao",DeptDaoImp.class);
	dao.XXX()
```



---

## 六. Spring整合Quartz

### 6.1 Quartz 相关介绍

​			在某一个有规律的时间点干某件事。并且时间的触发的条件可以非常复杂（比如每月最后一个工作日的17:50），复杂到需要一个专门的框架来干这个事。 Quartz就是来干这样的事，你给它一个触发条件的定义，它负责到了时间点，触发相应的Job起来干活。

```ABAP
Quartz:定时任务框架(异步,多个线程执行)
        任务: 做什么事情   eg:调用 service层
        触发器: 定义时间   eg:某个任务多久触发
        调度器:将任务和时间一一关联起来
        
Cron表达式:从左到右（用空格隔开）：秒  分 小时   月中的某一天   月份   星期中的星期几    年份	
	字段：		取值范围:							特殊字符			
       	秒		0~59的整数							, - * /    			四个字符
        分		0~59的整数							, - * /    			四个字符
        小时		0~23的整数							, - * /    			四个字符
        日期		1~31的整数							,- * ? / L W C  	八个字符  
        月份		1~12的整数或者 JAN-DEC			    , - * /    			四个字符
        星期		1~7的整数或者 SUN-SAT(1是星期天)		 , - * ? / L C # 	九个字符    
        年(可选)	1970~2099						  , - * /    			四个字符

特殊常用字符:
			*  :表示匹配该域的任意值
			?:只能用在日期和星期两个域(只能同时出现一个).它也匹配域的任意值
			-:表示范围:  	  			 eg(在分钟域用):5-20表示5到20分钟每分钟执行一次
			,:表示列出枚举值: 			 eg(在分钟域用):5,20表示5和20分钟时执行
			L:表示最后,只能用在日期和星期两个域:	 eg(在星期域用)：5L表示最后一个星期4执行
					
			例子:	https://www.cnblogs.com/yanghj010/p/10875151.html
			
SimpleScheduleBuilder常用API：	
			repeatSecondlyForever();//默认每秒钟执行一次
			repeatSecondlyForever(10);//每10秒执行一次
			
             repeatSecondlyForTotalCount(int count) ;//每秒执行count次
             repeatSecondlyForTotalCount(int count,int seconds);//每seconds秒执行count次
             
             withIntervalInSeconds(int seconds);//间隔的时间
             withRepeatCount(int count);//需要重复几次 (进行的总共次数=count+1;如果输入2，则会重复3次)

CronScheduleBuilder常用API          
             CronScheduleBuilder.cronSchedule(表达式);
		
```



### 6.2 Quartz 单独使用

#### 	6.2.1 简单时间使用

<font color="green">每隔2秒备份数据库</font>

```java
//任务类: 实现备份数据库
//该实体类必须实现接口！！！！！
public class DB4BackUpJob  implements Job{

    //省略具体备份代码
	private DBService dbSerive=new DBServiceImpl();
    
	@Override
	public void execute(JobExecutionContext arg0) throws JobExecutionException {
		//实现备份
        dbSerive.backUp();
	}

}
```

```JAVA
//测试
public class Test {

	public static void main(String[] args) throws SchedulerException, InterruptedException {
        // 创建 JobDetail任务
        JobDetail jobDetail=JobBuilder.newJob(DB4BackUpJob.class)
                											.withIdentity("job1","group1")
                											.build();
        //创建 Trigger触发器
        Trigger trigger= TriggerBuilder.newTrigger()
                								.withIdentity("trigger1","group1") //设置标识
                								.startNow()                															.withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(2))
                									.build();
		//获取Scheduler调度器
        Scheduler scheduler= StdSchedulerFactory.getDefaultScheduler();
        //关联 job和 trigger
        scheduler.scheduleJob(jobDetail,trigger);
        //启动 scheduler
        scheduler.start();
	}
}

```



#### 	6.2.2 自定义时间使用

<font color="green">在自定义的日期间隔中每隔2秒备注数据库</font>

```JAVA
//任务类: 实现备份数据库
//该实体类必须实现接口！！！！！
public class DB4BackUpJob  implements Job{

    //省略具体备份代码
	private DBService dbSerive=new DBServiceImpl();
    
	@Override
	public void execute(JobExecutionContext arg0) throws JobExecutionException {
		//实现备份
        dbSerive.backUp();
	}

}
```

```java
public class Test2 {

	public static void main(String[] args) throws SchedulerException, ParseException {
		 // 创建 JobDetail任务
        JobDetail jobDetail=JobBuilder.newJob(MeetingJob.class)
                											.withIdentity("job1","group1")
                											.build();
        //设置开始和结束时间
        SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
       	Date start = sdf.parse("2021-8-9 17:41:30");
		Date end = sdf.parse("2021-8-9 17:42:00");
        
        //创建 Trigger触发器
        Trigger trigger= TriggerBuilder.newTrigger()
                									.withIdentity("trigger1","group1") //设置标识
                									.startAt(start)
                									.endAt(end)      									.withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(2))
                									.build();
        
		 //获取Scheduler调度器
        Scheduler scheduler= StdSchedulerFactory.getDefaultScheduler();
        //关联 job和 trigger
        scheduler.scheduleJob(jobDetail,trigger);
        //启动 scheduler
        scheduler.start();
	}
}

```





#### 	6.2.3 复杂时间使用

<font color="green">在2021年的8月份的每个星期一的整数秒备注数据库</font>

```JAVA
//任务类: 实现备份数据库
//该实体类必须实现接口！！！！！
public class DB4BackUpJob  implements Job{

    //省略具体备份代码
	private DBService dbSerive=new DBServiceImpl();
    
	@Override
	public void execute(JobExecutionContext arg0) throws JobExecutionException {
		//实现备份
        dbSerive.backUp();
	}

}
```

```java
public class Test3 {
	public static void main(String[] args) throws SchedulerException, ParseException {
		 // 创建 JobDetail任务
        JobDetail jobDetail=JobBuilder.newJob(MeetingJob.class)
                											.withIdentity("job1","group1")
                											.build();
        
        //创建 Trigger触发器
        Trigger trigger= TriggerBuilder.newTrigger()
                							.withIdentity("trigger1","group1") //设置标识
                							//cron表达式 
            	.withSchedule(CronScheduleBuilder.cronSchedule("10,20,30,40,50, * * * 8 MON 2021"))
                									.build();
        
		 //获取Scheduler调度器
        Scheduler scheduler= StdSchedulerFactory.getDefaultScheduler();
        //关联 job和 trigger
        scheduler.scheduleJob(jobDetail,trigger);
        //启动 scheduler
        scheduler.start();	
        }
}
```



​			==注意：1. 任务类必须要实现Job接口，并重写方法==

​						==2.调度器如果需要关闭必须要让主线程的运行时间>调度器使用的时间，常见的办法就是让主线程休眠一定时间==



### 6.3 Spring整合Quartz

#### 	6.3.1 简单时间使用

​		<font color="green">每隔1秒重复备份数据库两次</font>

```java
//任务的配置类：用于配置常见信息
@Component
public class ScheduleJob {
	@Value("id1")
	private String jobId; 
	
	@Value("任务1")
	private String jobName;
	
	@Value("group1")
	private String jobGroup;

	@Value("1000")
	private int repeatInterval;//重复间隔时间
    
	@Value("2")
	private int repeatCount;//重复次数
	
    //省略getter，setter

}
```

```JAVA
//任务类: 实现备份数据库
//该实体类必须实现接口！！！！！
public class DB4BackUpJob  implements Job{

    //省略具体备份代码
	private DBService dbSerive=new DBServiceImpl();
    
	@Override
	public void execute(JobExecutionContext arg0) throws JobExecutionException {
		//实现备份
        dbSerive.backUp();
	}

}
```

```xml-dtd
<!--Spring配置文件-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation=" 
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd 
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd 
 http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd 
">

	<!--使用了注解的模式必须要扫描,多个包之间可以用逗号隔开 -->
	<context:component-scan base-package="com.spring" />

	
		<!-- 配置job任务 -->
	<bean id="jobDetail" class="org.springframework.scheduling.quartz.JobDetailFactoryBean" >
		<property name="jobClass"  value="com.spring.job.DB4BackUpJob"/>
		<property name="jobDataAsMap" >
			<map>
				<entry key="scheduleJob" ><ref bean="scheduleJob"/></entry>
			</map>
		</property>
	</bean>
	
	<!-- 配置触发器  -->
	<bean id="simpleTrigger" class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">
			<property name="repeatInterval" value="#{scheduleJob.repeatInterval}"></property>
			<property name="repeatCount" value="#{scheduleJob.repeatCount}"></property>
			<property name="jobDetail" ref="jobDetail"></property>
	</bean>
	
	
	<!-- 配置调度器  -->
	<bean id="schedulerFactoryBean" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
			<property name="triggers" ref="simpleTrigger" />	
	</bean>
	
</beans>
```

```JAVA
//测试
public class Test1 {

	public static void main(String[] args) {
		ApplicationContext context=new ClassPathXmlApplicationContext("spring.xml");
		StdScheduler bean =(StdScheduler) context.getBean("schedulerFactoryBean");
		try {
			bean.start();
		} catch (SchedulerException e) {
			e.printStackTrace();
		}
	}
}
```





#### 	6.3.2 复杂时间使用

​	<font color="green">在2021年的8月份的每个星期一的整数秒备注数据库</font>

```JAVA
//任务的配置类：用于配置常见信息
@Component
public class ScheduleJob {
	@Value("id1")
	private String jobId; 
	
	@Value("任务1")
	private String jobName;
	
	@Value("group1")
	private String jobGroup;
	
	@Value("10,20,30,40,50, * * * 8 MON 2021")
	private String cronExpression; //cron表达式

	
    //省略getter，setter

}
```

```JAVA
//任务类: 实现备份数据库
//该实体类必须实现接口！！！！！
public class DB4BackUpJob  implements Job{

    //省略具体备份代码
	private DBService dbSerive=new DBServiceImpl();
    
	@Override
	public void execute(JobExecutionContext arg0) throws JobExecutionException {
		//实现备份
        dbSerive.backUp();
	}

}
```

```xml-dtd
<!--Spring配置文件-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation=" 
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd 
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd 
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd 
 http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd 
">

	<!--使用了注解的模式必须要扫描,多个包之间可以用逗号隔开 -->
	<context:component-scan base-package="com.spring" />

	
		<!-- 配置job任务 -->
	<bean id="jobDetail" class="org.springframework.scheduling.quartz.JobDetailFactoryBean" >
		<property name="jobClass"  value="com.spring.job.DB4BackUpJob"/>
		<property name="jobDataAsMap" >
			<map>
				<entry key="scheduleJob" ><ref bean="scheduleJob"/></entry>
			</map>
		</property>
	</bean>
	
	<!-- 配置触发器  -->
	<bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
		<property name="cronExpression" value="#{scheduleJob.cronExpression}"></property>
		<property name="jobDetail" ref="jobDetail"></property>
	</bean>
	
	
	<!-- 配置调度器  -->
	<bean id="schedulerFactoryBean" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
			<property name="triggers" ref="cronTrigger" />	
	</bean>
	
</beans>
```

```JAVA
//测试
public class Test1 {

	public static void main(String[] args) {
		ApplicationContext context=new ClassPathXmlApplicationContext("spring.xml");
		StdScheduler bean =(StdScheduler) context.getBean("schedulerFactoryBean");
		try {
			bean.start();
		} catch (SchedulerException e) {
			e.printStackTrace();
		}
	}
}
```



---



## 七. Spring整合Junit

```java
@RunWith(SpringJUnit4ClassRunner.class)		   //运用junit测试
@ContextConfiguration("classpath:spring.xml")  //扫描配置文件
public class SpringAddJunit4 {
	
	@Resource(name="dao")
	private Dao dao;
    
    @AutoWired
    private Service service;
	
	@Test
	public void test1() {
		//测试代码
	}

}

```




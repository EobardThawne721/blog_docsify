# Spring Cloud Alibaba

## SpringBoot实战核心

### SpringBoot的自动装配原理

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
>     ==注意：通过实现`@ImportSelector`接口传入`@Import`中可以决定哪些Bean的可以被Ioc容器选择性装配，具体查看`ImportSelector实战演练`这一章的内容==
>
>  3. 通过Spring提供的`SpringFactoriesLoader`机制，扫描classpath下的`META-INF/spring.factories`，读取需要自动装配的配置类
>
>  4. 通过条件筛选出不符合条件的配置类，最终完成自动装配







### **ImportSelector实战演练**

1. 使用@Import手动导入配置类的方式加入IOC容器中

> **众所周知，SpringBoot启动类会将当前所在包及其子包的配置加入IOC容器中，如果想要把FirstConfig和SecondConfig的配置信息加入到SpringIOC容器中则必须需要将其使用@Import注解的方式导入进来**

<img src="./Spring Cloud Alibaba_images/image-20240930192223396.png" alt="image-20240930192223396" style="zoom:150%;" /> 

```java
//假设的第一个配置类
@Component
public class FirstConfig {
    //bean的名称默认为方法名
    @Bean
    public User getUserByFirstConfig1() {
        return new User(1,"配置1");
    }
}
======================================================================================
//假设的第二个配置类
@Component
public class SecondConfig {
    //bean的名称默认为方法名
    @Bean
    public User getUserByFirstConfig2() {
        return new User(2,"配置2");
    }
}
```

```java
@Configuration
//如果不使用@Import导入的话,SpringIOC容器中不会存在对应的Bean
public class GlobalConfiguration {
	
}

===========================================================================================
 @SpringBootApplication
public class ThinkingInSpringCloudApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(ThinkingInSpringCloudApplication.class, args);

        User user = context.getBean("getUserByFirstConfig1", User.class);
        System.out.println(user);

        User user2 = context.getBean("getUserByFirstConfig2", User.class);
        System.out.println(user2);
    }

}
```

<img src="./Spring Cloud Alibaba_images/image-20240930192852519.png" alt="image-20240930192852519" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20240930193031659.png" alt="image-20240930193031659" style="zoom:80%;" />





2. 使用实现ImportSelector接口的方式实现批量装配

<img src="./Spring Cloud Alibaba_images/image-20240930194137828.png" alt="image-20240930194137828" style="zoom:80%;" /> 

```java
public class CustomSelector implements ImportSelector {

    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        //将定义的两个配置Bean的名称(com.thawne.FirstConfig,com.thawne.SecondConfig)会被装配到容器中
        return new String[]{FirstConfig.class.getName(), SecondConfig.class.getName()};
    }
}

==========================================================================================

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import({CustomSelector.class}) //导入CustomSelector具体实现了ImportSelector
public @interface EnableCustomAutoConfiguration {

}
```

<img src="./Spring Cloud Alibaba_images/image-20240930194420254.png" alt="image-20240930194420254" style="zoom:80%;" />





### @Conditional条件装配

>  **`@Conditional`可以根据特定条件决定是否加载某个配置或组件，一般与`@Configuration`和`@Bean`配合使用，简单来说Spring在解析@Configuration配置类时，如果该配置类增加了@Conditional注解，那么会根据该注解配置的条件来决定是否装配对应的Bean。它接受一个或多个实现了 `Condition` （函数式接口，提供了matches方法，返回true表示可以注入Bean反之不行）接口的类作为参数**



#### @Conditional使用

```java
public class CustomCondition implements Condition {

    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        //从环境变量中获取操作系统的名称,如果是windows则返回true,否则返回false
        String osName = conditionContext.getEnvironment().getProperty("os.name");
        System.out.println("osName = " + osName);
        if ("windows 10".equalsIgnoreCase(osName)) {
            return true;
        }
        return false;
    }
}

======================================================================================

@Configuration
public class GlobalConfig {

    @Bean
    @Conditional(CustomCondition.class)
    public User user() {
        return new User(1, "eobard thawne");
    }
}
    
```

<img src="./Spring Cloud Alibaba_images/image-20241003161230141.png" alt="image-20241003161230141" style="zoom:80%;" />



#### 常用注解

* @ConditionalOnBean/@ConditionalOnMissingBean：容器中存在某个类或不存在时进行Bean装载

  ```java
  //通过name指定对应bean名称的类存在才加载或者直接通过指定对应bean的class文件
  @ConditionalOnBean(User.class)
  @ConditionalOnBean(name = "user")
  ```

* @ConditionalOnExpression：基于SpEL表达式的条件判断

  ```java
  //从配置文件中获取的值要一致才可以装配,如果是字符串类型的需要在EL表达式加上单引号
  @ConditionalOnExpression("${my.age}==24 and '${my.name}'=='eobard'")
  ```

* @ConditionalOnProperty：全局配置文件中指定的属性是否有对应的值

  ```java
  //根据全局配置文件获取my.sex的值如果是男则装配; 如果不存在属性值或属性值不对,则都不装配
  @ConditionalOnProperty(name = "my.sex",havingValue = "男",matchIfMissing = false)
  ```

* @ConditionalOnResource：classpath文件下是否存在指定资源

  ```java
  //判断指定资源名称是否存在于classpath下,存在则装配,反之不装配
  @ConditionalOnResource(resources = "classpath:my.config")
  ```

* @ConditionalOnClass：判断classpath类路径下是否存在给定的类才装配Bean

  ```java
   //在classpath环境下存在Redisson这个类的时候才装配(即当项目里面有这个依赖的时候)
  @ConditionalOnClass(Redisson.class)    
  ```

  





### 实现自定义的Starter

> **官方命名格式为：spring-boot-starter-模块名称，比如spring-boot-starter-web；而我们自定义命名格式为：模块名称-spring-boot-starter，比如mybatis-spring-boot-starter。这里我们实现一个`redisson-spring-boot-starter`**

<img src="./Spring Cloud Alibaba_images/image-20241003202743196.png" alt="image-20241003202743196" style="zoom:80%;" /> 



0. 创建springboot项目，导入依赖

```xml-dtd
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--redisson-->
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.12.0</version>
        </dependency>
    </dependencies>
```



1. 创建一个redisson的配置文件`redis.properties`

```properties
redisson.host=127.0.0.1
redisson.port=6379
```

2. 创建一个配置Bean跟配置文件对应起来

```java
@Data
@Component
@ConfigurationProperties(prefix = "redisson")       //指定配置文件的前缀
@PropertySource("classpath:redis.properties")    //指定自定义的配置文件
public class RedissonProperties {
    private String host;
    private String password;
    private int port = 6379;
}
```

3. 配置自动装配类

> **注意：`@EnableConfigurationProperties(RedissonProperties.class)注解的作用与上面的@Component作用一致,是为了将配置文件中的属性与实体类绑定,并将该类的实例装配为一个Spring Bean,这样可以在自动装配类中的形参中使用该Bean的值`。如果RedissonProperties不使用@Component注解标记为一个Bean，则需要再下面自动装配类中加上该注解；反之如果在RedissonProperties使用@Component注解后，不需要再自动装配类中使用该注解**

```java
import com.eobard.properties.RedissonProperties;
import org.redisson.Redisson;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.redisson.config.SingleServerConfig;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.util.StringUtils;

@Configuration
//在classpath环境下存在Redisson这个类的时候才装配(即当项目pom.xml里面引入这个依赖的时候)
//@EnableConfigurationProperties(RedissonProperties.class)
@ConditionalOnClass(Redisson.class) 
public class RedissonAutoConfiguration {

    @Bean
    RedissonClient redissonClient(RedissonProperties redissonProperties) {
        String prefix = "redis://";
        Config config = new Config();
        SingleServerConfig singleServerConfig = config.useSingleServer()
                .setAddress(prefix + redissonProperties.getHost() + ":" + redissonProperties.getPort());

        if (!StringUtils.isEmpty(redissonProperties.getPassword())) {
            singleServerConfig.setPassword(redissonProperties.getPassword());
        }
        return Redisson.create(config);
    }
}
```

4. 在`resuorces/META-INF`下创建`spring.factories`文件，并配置自动装配类

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.eobard.config.RedissonAutoConfiguration
```



5. 在我们自己的项目导入依赖、注入`RedissonClient`即可

```xml
 <!--添加我们自己的依赖-->
        <dependency>
            <groupId>com.eobard</groupId>
            <artifactId>redis-spring-boot-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```

<img src="./Spring Cloud Alibaba_images/image-20241003202931575.png" alt="image-20241003202931575" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20241003202940348.png" alt="image-20241003202940348" style="zoom:80%;" /> <img src="./Spring Cloud Alibaba_images/image-20241003202957283.png" alt="image-20241003202957283" style="zoom:80%;" />







### 整合Dubbo

> **创建一个空项目，在空项目中创建3个模块，分别定义接口工程、生产者工程、消费者工程。并在生产者工程及消费者工程中引入接口工程。接口工程存放表的实体类及服务接口，生产者工程提供服务接口的实现,，消费者工程调用服务接口**



#### 点对点服务实现通信

<img src="./Spring Cloud Alibaba_images/image-20241006135911681.png" alt="image-20241006135911681" style="zoom:80%;" />





##### sample-api模块

```java
//定义服务接口
public interface IUserService {
    public String getUserInfo(String id);
}
```





##### sample-provider模块

1. 导入依赖

```xml
 <!--导入api依赖-->
        <dependency>
            <groupId>com.eobard</groupId>
            <artifactId>sample-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--springboot-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>2.3.7.RELEASE</version>
        </dependency>
        <!--dubbo的springboot starter-->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.8</version>
        </dependency>
```



2. 具体实现api接口

```java
@DubboService
public class UserServiceImpl implements IUserService {
    @Value("${dubbo.application.name}")
    private String serverName;


    @Override
    public String getUserInfo(String id) {
        return String.format("userId:[%s],serverName:[%s]", id, serverName);
    }
}
```



3. springboot启动器类

```java
@SpringBootApplication
//使用dubbo并扫描Dubbo提供的@DubboService注解所在的路径
@EnableDubbo
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```



4. 全局配置文件

```properties
# 服务提供方的应用名称
dubbo.application.name=sample-provider-dubbo-demo
# 服务提供方的协议信息:默认采用dubbo协议
dubbo.protocol.name=dubbo
# 服务提供方商暴露的端口号
dubbo.protocol.port=20880
# 注册中心的地址,如果不需要注册中心,则填写N/A
dubbo.registry.address=N/A
```



##### sample-consumer模块

1. 导入依赖

```xml
 <!--springboot-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>2.3.7.RELEASE</version>
        </dependency>
        <!--dubbo的springboot starter-->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.8</version>
        </dependency>
        <!--导入api依赖-->
        <dependency>
            <groupId>com.eobard</groupId>
            <artifactId>sample-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```



2. 启动类

```java
@SpringBootApplication
public class ConsumerApplication {

    //dubbo暴露的地址
    @DubboReference(url = "dubbo://127.0.0.1:20880/com.eobard.service.IUserService")
    private IUserService userService;


    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class,args);
    }

    @Bean
    public ApplicationRunner runner(){
        return args -> System.out.println(userService.getUserInfo("11"));
    }
}
```



3. 全局配置文件

```properties
server.port=8081
# 消费者方的应用名称
dubbo.application.name=sample-consumer-dubbo-demo
```

<img src="./Spring Cloud Alibaba_images/image-20241006140955195.png" alt="image-20241006140955195" style="zoom:80%;" />





## 一.概念

### 1.1 微服务

> **"微服务其实是一种架构风格，我们在开发一个应用的时候应该是由一组小型服务组成，每个小型的服务都运行在自己的进程内；小服务之间通过HTTP的方式进行互联互通。"**

<img src="./Spring Cloud Alibaba_images/image-20220314201701744.png" style="zoom:80%;" />

* 水平扩展：服务实例水平增加
* 垂直扩展：硬件升级





### 1.2 微服务架构图

<img src="./Spring Cloud Alibaba_images/image-20220314203702706.png" alt="image-20220314203702706" style="zoom:150%;" />



### 1.3 快速启动微服务

1. 找到根项目的`.idea`文件夹下的`workspace.xml`文件

2. 添加下列代码

   ```xml
   <component name="RunDashboard">
       <option name="configurationTypes">
         <set>
           <option value="SpringBootApplicationConfigurationType" />
         </set>
       </option>
     </component>
   ```

   

## 二.基本环境搭建

### 2.1 创建分布式项目

（1）创建SpringBoot项目，  类型为Maven POM，不需要任何的场景启动器

<img src="./Spring Cloud Alibaba_images/image-20220315092640985.png" alt="image-20220315092640985" style="zoom:150%;" />



（2）修改父项目的pom.xml

```xml
	<!--设置打包方式为：pom-->
    <packaging>pom</packaging>
```



（3）在父项目下创建两个子Maven项目的module（订单服务和库存服务）

<img src="./Spring Cloud Alibaba_images/image-20220315093758040.png" alt="image-20220315093758040" style="zoom:150%;" />

<img src="./Spring Cloud Alibaba_images/image-20220315094107406.png" alt="image-20220315094107406" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220315094131656.png" alt="image-20220315094131656" style="zoom:80%;" />



（4）分别在两个子Maven项目的pom.xml添加web场景启动器依赖

```xml
    <dependencies>
        <!--添加web场景启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```





（5）创建两个子项目的启动类

```java
//Order子项目启动器类
@SpringBootApplication
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }


    //后期开发应该将该配置放入配置类中,这里为了方便起见直接注入
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder){
        return builder.build();
    }

}
```

```JAVA
//Stock子项目启动器类
@SpringBootApplication
public class StockApplication {

    public static void main(String[] args) {
        SpringApplication.run(StockApplication.class,args);
    }

    //后期开发应该将该配置放入配置类中,这里为了方便起见直接注入
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder){
        return builder.build();
    }
}
```

<img src="./Spring Cloud Alibaba_images/image-20220315100540655.png" alt="image-20220315100540655" style="zoom:80%;" />



（6）创建子项目全局配置文件，修改端口号

```properties
#Order项目端口号为8081
server.port=8081
```

```properties
#Stock项目端口号为8082
server.port=8082
```

<img src="./Spring Cloud Alibaba_images/image-20220315101853667.png" alt="image-20220315101853667" style="zoom:80%;" />





（7）创建两个服务的控制器类，并添加两个方法

```JAVA
//Order子项目
@RestController
@RequestMapping("/order")
public class OrderController {

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/buy")
    public String purchase(){
        /**
         * 参数1：远程url
         * 参数2：返回数据的类型
         * 参数3：参数的类型(可变参数)
         */
        String message = restTemplate.getForObject("http://localhost:8082/stock/reduct", String.class);
        System.out.println("message = " + message);
        return"下单成功！";
    }

}
```

```JAVA
//Stock子项目
@RestController
@RequestMapping("/stock")
public class StockController {

    @RequestMapping("/reduct")
    public String reduct(){
        return"库存商品-1";
    }
}
```



（8）启动两个子项目，运行测试

<img src="./Spring Cloud Alibaba_images/image-20220315102742116.png" alt="image-20220315102742116" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220315102758195.png" alt="image-20220315102758195" style="zoom:80%;" />



> 从上面步骤可知，当服务器发生变迁（每个IP地址都会不一样），每个服务的功能维护起来的成本十分大，这时候就可以通过注册中心来解决这个问题，让每个服务都能够快速的调用新的地址



### 2.2 改造微服务项目

> **版本说明：**[版本说明 · alibaba/spring-cloud-alibaba Wiki (github.com)](https://github.com/alibaba/spring-cloud-alibaba/wiki/版本说明)

<img src="./Spring Cloud Alibaba_images/image-20220315105309348.png" alt="image-20220315105309348" style="zoom:150%;" />

<img src="./Spring Cloud Alibaba_images/image-20220315105342780.png" alt="image-20220315105342780" style="zoom:150%;" />



**`修改父项目的pom.xml文件`**

```xml
 <!--以后公司开发可以使用parent去继承公司自定义的父maven项目
        <parent>
        </parent>
    -->

    <properties>
        <java.version>1.8</java.version>
        <spring.cloud.alibaba.version>2.1.4.RELEASE</spring.cloud.alibaba.version>
        <spring.boot.version>2.1.13.RELEASE</spring.boot.version>
        <spring.cloud.version>Greenwich.SR6</spring.cloud.version>
    </properties>

    <!--设置打包方式为：pom-->
    <packaging>pom</packaging>

	<dependencyManagement>
       <!--Spring Cloud Alibaba的版本管理，通过dependency完成继承-->
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <!--Spring Boot的版本管理-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <!--spring cloud的版本管理-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```



## 三. Nacos

### 3.1 概念

> Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。`集 注册中心+配置中心+服务管理 平台`

<img src="./Spring Cloud Alibaba_images/image-20220316094314405.png" alt="image-20220316094314405" style="zoom: 150%;" />



心跳机制：在服务中保留定时任务，每隔一定时间发送信息到注册中心，可以动态的维护最新的一个注册表。

* 若注册中心超过指定时间没收到心跳，就视为当前服务已经挂掉，将状态修改为down；
* 若时间过长都还没收到，则就从注册中心将这条服务给剔除掉；
* 若某个服务手动停止了，可以采用注销接口将从注册中心注销接口，同样也可以从注册表中剔除掉



### 3.2 搭建Nacos服务端

* 根据Spring Cloud Alibaba版本下载对应的Nacos：[Releases · alibaba/nacos (github.com)](https://github.com/alibaba/nacos/releases)

  <img src="./Spring Cloud Alibaba_images/image-20220316101959445.png" alt="image-20220316101959445" style="zoom:80%;" />

  > **注意：Nacos默认为集群，开发时需要改为单机版；账号和密码都是nacos**



#### Windows版本

* 解压文件，进入bin目录，编辑startup.cmd并保存

<img src="./Spring Cloud Alibaba_images/image-20220316104828723.png" alt="image-20220316104828723" style="zoom:80%;" />



* 运行startup.cmd

<img src="./Spring Cloud Alibaba_images/image-20220316105425499.png" alt="image-20220316105425499" style="zoom:80%;" />



* 浏览器输入：http://192.168.117.1:8848/nacos/index.html

<img src="./Spring Cloud Alibaba_images/image-20220316105547613.png" alt="image-20220316105547613" style="zoom:80%;" />



#### Linux版本

* 创建nacos文件夹

```bash
cd /usr/local/software
mkdir nacos
cd nacos
```



* 上传文件到该路径

<img src="./Spring Cloud Alibaba_images/image-20220316111450275.png" alt="image-20220316111450275" style="zoom:80%;" /> 



* 解压文件

```bash
tar -zxf nacos-server-2.0.3.tar.gz 
```



* 单机运行

```BASH
cd nacos/bin
sh startup.sh -m standalone &
```

<img src="./Spring Cloud Alibaba_images/image-20220316115354899.png" alt="image-20220316115354899" style="zoom:80%;" />



* 开放端口，供Windows访问

```bash
firewall-cmd --zone=public --add-port=8848/tcp --permanent
systemctl reload firewalld
```



* 浏览器输入：http://192.168.2.102:8848/nacos/index.html

<img src="./Spring Cloud Alibaba_images/image-20220316115653438.png" alt="image-20220316115653438" style="zoom:80%;" />





#### 集群搭建

（1）在`/usr/local/software/nacos`路径里面继续解压两次，并重命名

```bash
tar -zxf nacos-server-2.0.3.tar.gz 
```

<img src="./Spring Cloud Alibaba_images/image-20220317080653135.png" alt="image-20220317080653135" style="zoom:80%;" />



（2）进入nacos_8849文件夹，修改配置文件

```bash
cd nacos_8849/conf/
vim application.properties
```

<img src="./Spring Cloud Alibaba_images/image-20220318114011888.png" alt="image-20220318114011888" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220324112614924.png" alt="image-20220324112614924" style="zoom:80%;" /> 



（3）将conf文件夹里面的nacos-mysql.sql运行在Linux中的mysql

> **注意：这里以Docker为例，需要根据Docker的Mysql端口来调整db.url.0的端口号**

<img src="./Spring Cloud Alibaba_images/image-20220317083012324.png" alt="image-20220317083012324" style="zoom:80%;" /> 



（4）拷贝conf文件夹的cluster.conf.example，并编辑

```BASH
cp cluster.conf.example cluster.conf
vim cluster.conf
```

<img src="./Spring Cloud Alibaba_images/image-20220317083324415.png" alt="image-20220317083324415" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220317083618743.png" alt="image-20220317083618743" style="zoom:80%;" /> 





（5）修改startup.sh

```bash
cd ../bin/
vim startup.sh
```

<img src="./Spring Cloud Alibaba_images/image-20220317084037519.png" alt="image-20220317084037519" style="zoom:80%;" />



（6）将Nacos_8849修改的文件复制到其余两个文件

```bash
cd /usr/local/software/nacos
```

<img src="./Spring Cloud Alibaba_images/image-20220317084925622.png" alt="image-20220317084925622" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220317085101002.png" alt="image-20220317085101002" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220317085212455.png" alt="image-20220317085212455" style="zoom:80%;" />



（7）修改8850和8851的全局配置文件端口号

```bash
vim nacos_8850/conf/application.properties
vim nacos_8851/conf/application.properties
```

<img src="./Spring Cloud Alibaba_images/image-20220317085344212.png" alt="image-20220317085344212" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220317085413527.png" alt="image-20220317085413527" style="zoom:80%;" /> 



（8）分别启动三个Nacos服务，并访问

> **注意：由于本机分配的内存较小，只能开启两个服务，可通过`free -h`查看可用的内存（若剩余的内存很小，则通过`vim /etc/fstab`将`/dev/mapper/centos-swap`注释掉）**

```BASH
sh nacos_8849/bin/startup.sh 
sh nacos_8850/bin/startup.sh 
sh nacos_8851/bin/startup.sh 

#需要开放端口或关闭防火墙供Windows访问
```

<img src="./Spring Cloud Alibaba_images/image-20220317194429117.png" alt="image-20220317194429117" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220317194450247.png" alt="image-20220317194450247" style="zoom:80%;" />



（9）配置Nginx负载均衡

```BASH
cd /usr/local/software/nginx/conf
vim nginx.conf 
```

```bash
upstream nacoscluster{
		#1. 负载均衡地址,未分配权重,默认采用轮询机制
        server 192.168.2.102:8849;
        server 192.168.2.102:8851;
    }

    server {
    	#2. 监听8847端口号,当访问该端口号时,反向代理下面路径
        listen       8847;

        location /nacos/ {
        #3. 路径Url,当访问 http://IP地址:8847/nacos/时 自动访问负载均衡的地址
            proxy_pass http://nacoscluster/nacos/;
            }
        }

```

<img src="./Spring Cloud Alibaba_images/image-20220318094602773.png" alt="image-20220318094602773" style="zoom:80%;" />



（10）修改两个子Maven项目的全局配置文件，并运行

```properties
#访问集群nacos+负载均衡
spring.cloud.nacos.server-addr=192.168.2.102:8847
```

<img src="./Spring Cloud Alibaba_images/image-20220318114623633.png" alt="image-20220318114623633" style="zoom:80%;" />

> **`注意：如果nacos版本为2.0+，可能会出现"client not connected ，current status：STARTING"，需要在Linux开放端口或直接将cloud版本降级`**

<img src="./Spring Cloud Alibaba_images/image-20220318114821928.png" alt="image-20220318114821928" style="zoom:80%;" />







### 3.3 搭建Nacos客户端

（1）两个子Maven项目分别导入依赖

```xml
	 <!--nacos服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
```



（2）修改两个子Maven项目的全局配置文件

```properties
#Order子项目
#nacos将应用名称当做服务器名称
spring.application.name=order-service
#nacos地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos账号
spring.cloud.nacos.discovery.username=nacos
#nacos密码
spring.cloud.nacos.discovery.password=nacos
#nacos命名空间
spring.cloud.nacos.discovery.namespace=public
```

```properties
#Stock子项目
#nacos将应用名称当做服务器名称
spring.application.name=stock-service
#nacos地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos账号
spring.cloud.nacos.discovery.username=nacos
#nacos密码
spring.cloud.nacos.discovery.password=nacos
#nacos命名空间
spring.cloud.nacos.discovery.namespace=public
```



（3）修改Order启动类，便于调用Stock服务

```JAVA
@SpringBootApplication
@EnableDiscoveryClient//开启服务端nacos注册发现功能
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }

    //后期开发应该将该配置放入配置类中,这里为了方便起见直接注入
    @Bean
    /**
     * 添加负载均衡器：默认采用轮询的机制
     *  eg：若有多个库存服务，当前订单服务会依次轮流调用其它那几个库存服务
     */
    @LoadBalanced
    public RestTemplate restTemplate(RestTemplateBuilder builder){
        return builder.build();
    }

}

```



（4）修改Order的控制器访问路径

>  **`注意：这里url不能写IP地址，要改为注册中心的nacos服务器名称`**

```JAVA
@RestController
@RequestMapping("/order")
public class OrderController {

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/buy")
    public String purchase(){
        /**
         * 参数1：远程url
         * 参数2：返回数据的类型
         * 参数3：参数的类型(可变参数)
         */
        String message = restTemplate.getForObject("http://stock-service/stock/reduct", String.class);
        System.out.println("message = " + message);
        return"下单成功！";
    }

}
```



（5）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220316202831127.png" alt="image-20220316202831127" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220316202852890.png" alt="image-20220316202852890" style="zoom:80%;" />



#### 默认负载均衡

（1）修改Stock控制器代码，测试负载均衡端口号

```JAVA
@RestController
@RequestMapping("/stock")
public class StockController {


    @Value("${server.port}")
    private String port;

    @RequestMapping("/reduct")
    public String reduct(){
        return port+"端口，库存商品-1";
    }
}
```



（2）水平扩容库存服务

> **`注意：通过Ideal的Copy Configuration可以快速的复制服务`**

<img src="./Spring Cloud Alibaba_images/image-20220316203402284.png" alt="image-20220316203402284" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220316203817167.png" alt="image-20220316203817167" style="zoom:80%;" />



（3）运行三个服务，并查看运行结果

<img src="./Spring Cloud Alibaba_images/image-20220316204313366.png" alt="image-20220316204313366" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220316204350369.png" alt="image-20220316204350369" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220316204327366.png" alt="image-20220316204327366" style="zoom:80%;" />





### 3.4 总结

<img src="./Spring Cloud Alibaba_images/image-20220316201719273.png" alt="image-20220316201719273" style="zoom:80%;" />





## 四. Ribbon

​		**Ribbon是基于Netflix发布的负载均衡器**（简称LoadBalancer，LB），为Ribbon配置服务提供者地址后，Ribbon就可基于某种负载均衡算法，自动地帮助服务消费者去请求。Ribbon默认为我们提供了很多负载均衡算法，例如轮询、随机等。我们也可为Ribbon实现自定义的负载均衡算法。



> 服务端负载均衡：客户端所有请求统一交给nginx，由nginx进行实现负载均衡请求转发， 既请求由nginx服务器端进行转发

eg：大量的消费者(服务消费者)去饭店吃饭，为了解决人流量，饭店可以开连锁(水平扩展)，消费者可以通过手机(Nginx)推荐哪家近、哪家好吃、哪家不排队的优势从而决定去哪家饭店吃饭。

<img src="./Spring Cloud Alibaba_images/image-20220319100105093.png" alt="image-20220319100105093" style="zoom:80%;" />



> 客户端负载均衡：Ribbon是从eureka注册中心上获取服务注册信息列表，缓存到本地，然后在本地实现轮询负载均衡策略。既在客户端实现负载均衡。

eg：消费者(服务消费者)提前就知道了有多家饭店(水平扩展)，他根据自己的爱好(某种负载均衡算法)决定去哪家消费

<img src="./Spring Cloud Alibaba_images/image-20220319094228502.png" alt="image-20220319094228502" style="zoom:80%;" />





### 4.1 负载均衡策略

> 注意：以下的负载均衡策略的父接口是`IRule`

* `RandomRule`：随机选取一个服务器实例，在它的无参构造对象中初始化了一个Random对象，每次利用Random对象生成一个不大于服务器实例总数的随机数，并将随机数作为下标获取下一个服务器实例。
* `RoundRobinRule`：轮询负载均衡，轮询index，选择index对应位置的Server；
* `RetryRule`：在轮询基础上重试，在轮询基础上选择一个服务实例，如果实例正常则返回；否则在失效时间deadline之前不断重试，若超过了deadline就返回null
* `WeightedResponseTimeRule`：根据每一个实例运行情况计算权重，在挑选实例的时候根据权重进行挑选；如果一个服务的平均响应时间很短则权重越大，该服务实例就越有几率被选中。
* `BestAvailableRule`：过滤掉失效的服务实例，找出并发请求最小的服务实例来调用。
* `ZoneAvoidanceRule(默认规则)`：就近原则，根据服务部署所在区域和服务可用性选择服务器，过滤成功后继续使用线性轮询方式



### 4.2  修改默认负载均衡策略

#### 4.2.1 通过启动类

（1）复制上一个`子消费者`项目修改相应的Maven结构

<img src="./Spring Cloud Alibaba_images/image-20220320105013085.png" alt="image-20220320105013085" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220320105133247.png" alt="image-20220320105133247" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220320105332095.png" alt="image-20220320105332095" style="zoom:80%;" />



（2）导入依赖、修改相应文件

```xml
 <!--nacos服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
 <!--添加web场景启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

```properties
#Order-ribbon项目端口号为8840
server.port=8840
#nacos将应用名称当做服务器名称
spring.application.name=order-ribbon
#nacos地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos账号
spring.cloud.nacos.discovery.username=nacos
#nacos密码
spring.cloud.nacos.discovery.password=nacos
#nacos命名空间
spring.cloud.nacos.discovery.namespace=public
```

```JAVA
@RestController
@RequestMapping("/order")
public class OrderController {

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping("/buy")
    public String purchase(){
        String message = restTemplate.getForObject("http://stock-service/stock/reduct", String.class);
        System.out.println("message = " + message);
        return"下单成功！";
    }

}
```



<img src="./Spring Cloud Alibaba_images/image-20220320102509520.png" alt="image-20220320102509520" style="zoom:80%;" /> 

==注意：一定是消费者的子项目，因为Ribbon是服务消费者的负载均衡；在这个项目中存在下单和库存两个服务：下单为服务消费者，库存为服务提供者==



（3）添加配置类、修改主启动类

> **注意：这里的配置类一定不能被启动类的@ComponentScan扫描到！**

```JAVA
@Configuration
public class RibbonConfig {

    //方法名一定要叫iRule
    @Bean
    public IRule iRule(){
        return new RandomRule();
    }

}
```

```JAVA
@SpringBootApplication
@RibbonClients(value = {
        //name：为哪个服务提供商去运用这个负载均衡策略
        //configuration：配置类
        @RibbonClient(name = "stock-service",configuration = RibbonConfig.class)
})
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }

    //后期开发应该将该配置放入配置类中,这里为了方便起见直接注入
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(RestTemplateBuilder builder){
        return builder.build();
    }

}
```

<img src="./Spring Cloud Alibaba_images/image-20220320105822140.png" alt="image-20220320105822140" style="zoom:80%;" />







（4）水平扩容订单服务并运行，见3.3.(2)

<img src="./Spring Cloud Alibaba_images/image-20220320112101747.png" alt="image-20220320112101747" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220320112158111.png" alt="image-20220320112158111" style="zoom:80%;" />





#### 4.2.2 通过配置文件

（1）在全局配置文件配置即可，不用在启动类加注解也不用添加配置类

```properties
#nacos将应用名称当做服务器名称
spring.application.name=order-ribbon
#nacos地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos账号
spring.cloud.nacos.discovery.username=nacos
#nacos密码
spring.cloud.nacos.discovery.password=nacos
#nacos命名空间
spring.cloud.nacos.discovery.namespace=public

#等同于在主启动类的@RibbonClients注解：使用按权重的负载均衡策略
stock-service.ribbon.NFLoadBalancerRuleClassName=com.alibaba.cloud.nacos.ribbon.NacosRule
```

<img src="./Spring Cloud Alibaba_images/image-20220320123507799.png" alt="image-20220320123507799" style="zoom:80%;" />



### 4.3 自定义负载均衡策略

（1）创建类

```JAVA
public class CustomLoadBalancerRule extends AbstractLoadBalancerRule {

    public Server choose(Object o) {
        //获取当前请求服务的实例
        ILoadBalancer loadBalancer = this.getLoadBalancer();
        List<Server> servers = loadBalancer.getReachableServers();

        //通过线程安全的ThreadLocalRandom类随机的获取服务
        int serverIndex = ThreadLocalRandom.current().nextInt(servers.size());

        //获取下标对应的server
        Server server = servers.get(serverIndex);

        return server;
    }

    //获取初始化配置
    public void initWithNiwsConfig(IClientConfig iClientConfig) {

    }
}
```



（2）**消费者方**通过`4.2.2`或`4.2.1`任意一种方式引用

```properties
#这里通过配置文件方式引用
stock-service.ribbon.NFLoadBalancerRuleClassName=com.ribbon.CustomLoadBalancerRule
```



（3）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220321094525498.png" alt="image-20220321094525498" style="zoom:80%;" />



#### 4.3.1 注意

>  **注意：通过控制台打印结果可知，`DynamicServerListLoadBalancer`具有懒加载的特点，当第一次请求会出现很慢的特点，可以通过下列配置解决**

```properties
#消费者全局配置文件开启ribbon立即加载
ribbon.eager-load.enabled=true
#配置立即加载的服务名,多个用逗号隔开
ribbon.eager-load.clients=stock-service
```



### 4.4 使用LoadBalancer

> `Spring Cloud LoadBalancer`是Spring Cloud 官方自己提供的客户端负载均衡器，用来替代Ribbon；提供了两种负载均衡的客户端：RestTemplate、WebClient ；详细实现方式查看源码`RoundRobinLoadBalancer`

（1）复制一份消费者服务，并将所有Ribbon的类和依赖排除

```XML
  <!--nacos服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <!--移除Ribbon-->
                <exclusion>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
                </exclusion>
            </exclusions>
        </dependency>


    <!--添加loadbalancer依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-loadbalancer</artifactId>
        </dependency>
```



（2）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220321105558345.png" alt="image-20220321105558345" style="zoom:80%;" />





## 五. Open Feign

### 5.1 概念

> **`通过前面的例子可知，每次我们调用远程服务的时候都要在Controller中写详细的url地址，这样对于后期的维护十分庞大，有没有一种简洁的方式调用呢？`**

1. Feign是开发的声明式、模板化的HTTP客户端；Feign可帮助我们更加便捷、优雅地调用HTTP API

2. Spring Cloud Open Feign对Feign进行了增强，使其支持Spring MVC注解，另外还**整合了Ribbon和Nacos**，从而使得Feign的使用更加方便

3. **Spring Cloud Open Feign可以做到使用HTTP请求访问远程服务，就像调用本地方法一样的，开发者完全感知不到这是在调用远程方法，更感知不到在访问HTTP请求**。而不需要通过常规的Http Client构造请求再解析返回数据。
4. **采用动态代理机制**





### 5.2 快速使用

（1）创建一个新的订单子模块，编写依赖、配置

```xml
 <!--添加web场景启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--nacos服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <!--添加openfeign依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

```properties
#Order-openfeign项目端口号为8841
server.port=8841
#nacos将应用名称当做服务器名称
spring.application.name=order-openfeign
#nacos地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos账号
spring.cloud.nacos.discovery.username=nacos
#nacos密码
spring.cloud.nacos.discovery.password=nacos
#nacos命名空间
spring.cloud.nacos.discovery.namespace=public
```



（2）编写FeignService对应远程服务

> **注意：在Feign里面写远程路径时，若是Rest风格则要在形参中详细写完占位符的参数名**
>
> **eg： @GetMapping("/{id}")**
>
> ​			**public String get(@PathVariable("id") int id)；**

```JAVA
/**
 * 参数1：调用其它服务的服务名
 * 参数2：调用其它服务接口Controller上的@RequestMapping地址
 */
@FeignClient(name = "stock-service",path = "/stock")
public interface StockFeginService {

    //声明需要调用的rest接口对应的方法
    @RequestMapping("/reduct")
    String reduct();
}
```

<img src="./Spring Cloud Alibaba_images/image-20220322135902441.png" alt="image-20220322135902441" style="zoom:80%;" />



（3）启动类

```JAVA
@SpringBootApplication
//开启Fegin
@EnableFeignClients
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }

}
```



（4）编写控制器

```JAVA
@RestController
@RequestMapping("/order")
public class OrderController {

    @Resource
    private StockFeginService stockFeginService;

    @RequestMapping("/buy")
    public String purchase(){
        String message = stockFeginService.reduct();
        System.out.println("message = " + message);
        return"下单成功！";
    }

}
```



（5）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220322140115791.png" alt="image-20220322140115791" style="zoom:80%;" />





#### 其它配置（熔断器与404）

<img src="./Spring Cloud Alibaba_images/image-20241003205229886.png" alt="image-20241003205229886" style="zoom:80%;" />





### 5.3 日志配置

> 有时候我们遇到Bug：接口调用失败、参数没收到等问题，或者想看看调用性能，就需要配置Feign的日志了，以此让Feign把请求信息输出来.

**通过源码可以看到日志等级有4种，分别是:**

* NONE【性能最佳，适用于生产】：不记录任何日志(默认值)
* BASIC【适用于生产环境追踪问题】∶仅记录请求方法、URL、响应状态代码以及执行时间。
* HEADERS：记录BASIC级别的基础上，记录请求和响应的header。
* FULL【比较适用于开发及测试环境定位问题】︰记录请求和响应的header、body和元数据。



#### 5.3.1 全局配置

（1）创建配置类

```JAVA
/**
 * 全局配置：使用@Configuration会将配置作用到所有服务提供方
 * 局部配置：如果只想针对某一个服务配置，就不用加@Configuration注解
 */
@Configuration
public class FeignConfig {

    //配置远程服务日志级别
    @Bean
    public Logger.Level feignLevel(){
        return Logger.Level.FULL;
    }

}
```



（2）全局配置文件

```properties
#设置远程调用service 日志输出级别为debug,默认为info>debug
logging.level.com.eobard.order.fegin=debug
```



（3）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220323140618617.png" alt="image-20220323140618617" style="zoom:80%;" />



#### 5.3.2 局部配置

##### 配置文件(推荐)

（1）全局配置文件

```properties
#设置当前包的日志输出级别为debug,默认为info>debug
logging.level.com.eobard.order.fegin=debug

#局部配置:设置服务名为stock-service日志级别为basic
feign.client.config.stock-service.logger-level=basic
```



（2）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220323141141989.png" alt="image-20220323141141989" style="zoom:80%;" />







##### 配置类

（1）配置类

```JAVA
//局部配置：针对某一个服务配置，不加@Configuration注解
public class FeignConfig {

     //配置远程服务日志级别
    @Bean
    public Logger.Level feignLevel(){
        return Logger.Level.FULL;
    }

}
```



（2）FeignService类

```JAVA
/**
 * 采用动态代理机制
 * 参数1：调用的服务名
 * 参数2：调用接口Controller上的@RequestMapping地址
 * 参数3：设置stock-service服务的配置
 */
@FeignClient(name = "stock-service",path = "/stock",configuration = FeignConfig.class)
public interface StockFeginService {

    //声明需要调用的rest接口对应的方法
    @RequestMapping("/reduct")
    String reduct();
}
```



（3）配置文件

```properties
#设置远程调用service 日志输出级别为debug,默认为info>debug
logging.level.com.eobard.order.fegin=debug
```



（4）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220323141455842.png" alt="image-20220323141455842" style="zoom:80%;" />





### 5.4 契约配置(了解)

Open Feign 默认使用Spring MVC 契约，也就是Spring MVC的注解，要想使用Feign的默认契约，也就是使用Feign原生的注解，则需要如下改动。

（1）配置类

```properties
#设置为默认的契约(还原成原生注解)
feign.client.config.stock-service.contract=feign.Contract.Default
```



（2）修改FeignService

```JAVA
@FeignClient(name = "product-service",path="/product")
public interface ProductFeignService {

    @RequestLine("GET /{id}")
    public String get(@Param("id") Integer id);

//还原成原生的注解后，若使用Spring MVC的注解则会报错
//    @RequestMapping("/{id}")
//    public String get(@PathVariable("id") Integer id);
}
```



### 5.5 超过时间配置

> **若服务间的连接或调用超过了指定配置时间，将会引发异常**

#### 全局配置

```JAVA
//使用@Configuration会将配置作用到所有服务提供方
@Configuration
public class FeignConfig {

     //配置远程服务日志级别
    @Bean
    public Logger.Level feignLevel(){
        return Logger.Level.FULL;
    }


    @Bean
    public Request.Options options(){
        //设置服务间的连接时间为5秒,请求处理超时时间3秒
        return new Request.Options(5000,3000);
    }
}
```



#### 局部配置

```properties
#当前服务的全局配置文件

# 针对于某个微服务
#调用服务名为stock-service的连接超时时间：5s
feign.client.config.stock-service.connect-timeout=5000
#调用服务名为stock-service的请求处理时间：3s
feign.client.config.stock-service.read-timeout=3000


# 配置全部微服务
# feign的超时时间配置相关,default表示应用全部远程服务
feign.client.config.default.connect-timeout=5000
feign.client.config.default.read-timeout=3000
```



### 5.6 自定义拦截器

​			消费端调用服务提供端的时候进行拦截，可以实现日志、授权认证等操作

<img src="./Spring Cloud Alibaba_images/image-20220324114504916.png" alt="image-20220324114504916" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220324114519285.png" alt="image-20220324114519285" style="zoom:80%;" /> 



#### 配置文件

（1）创建拦截器

```JAVA
public class CustomFeignInterceptor implements RequestInterceptor {

    Logger logger= LoggerFactory.getLogger(this.getClass());

    public void apply(RequestTemplate requestTemplate) {
        requestTemplate.header("xxx","设置请求头参数");
        logger.info("自定义拦截器记录日志：访问了{}",requestTemplate.url());
    }
}
```



（2）消费者服务全局配置文件

```properties
#自定义拦截器
feign.client.config.stock-service.request-interceptors[0]=com.eobard.order.interceptor.CustomFeignInterceptor
```



（3）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220324104428974.png" alt="image-20220324104428974" style="zoom:80%;" />



#### 配置类

（1）创建拦截器

```JAVA
public class CustomFeignInterceptor implements RequestInterceptor {

    Logger logger= LoggerFactory.getLogger(this.getClass());

    public void apply(RequestTemplate requestTemplate) {
        requestTemplate.header("xxx","设置请求头参数");
        logger.info("自定义拦截器记录日志：访问了{}",requestTemplate.url());
    }
}
```



（2）配置类注入自定义拦截器

```JAVA
//使用@Configuration会将配置作用到所有服务提供方
@Configuration
public class FeignConfig {

//    //配置远程服务日志级别
//    @Bean
//    public Logger.Level feignLevel(){
//        return Logger.Level.FULL;
//    }
//
//
//    @Bean
//    public Request.Options options(){
//        //设置服务间的连接时间为5秒,请求处理超时时间3秒
//        return new Request.Options(5000,3000);
//    }
    
      //注入自定义拦截器
    @Bean
    public CustomFeignInterceptor requestInterceptor(){
        return new CustomFeignInterceptor();
    }
}
```



（3）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220324104203729.png" alt="image-20220324104203729" style="zoom:80%;" />





## 六. Nacos配置中心

​				动态配置服务可以让你以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置。**动态配置消除了配置变更时重新部署应用和服务的需要，让配置管理变得更加高效和敏捷。**

<img src="./Spring Cloud Alibaba_images/image-20220324111255093.png" alt="image-20220324111255093" style="zoom:80%;" />



配置中心具有以下优点：

* 动态的修改配置
* 配置中心挂了也不影响配置的使用
* 配置是可以多个服务共享的
* 配置可以回滚
* 支持权限管理，只有授予权限的人才能查看和修改配置



### 6.1 快速配置

（1）打开Nacos控制台，添加配置

<img src="./Spring Cloud Alibaba_images/image-20220324113139913.png" alt="image-20220324113139913" style="zoom:80%;" />



（2）添加订单关于Redis配置

<img src="./Spring Cloud Alibaba_images/image-20220324113310694.png" alt="image-20220324113310694" style="zoom:80%;" />



（3）创建结果

<img src="./Spring Cloud Alibaba_images/image-20220324113513216.png" alt="image-20220324113513216" style="zoom:80%;" />



（4）编辑内容，便于`6.2`读取

<img src="./Spring Cloud Alibaba_images/image-20220325135414836.png" alt="image-20220325135414836" style="zoom:80%;" />





### 6.2 读取配置

（1）创建新的`Nacos-Config配置中心模块`，导入依赖并初始化为SpringBoot项目

```xml-dtd
 <dependencies>
        <!--添加web场景启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--配置中心-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>
```

```JAVA
@SpringBootApplication
public class NacosConfigApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(NacosConfigApplication.class, args);
        String name = applicationContext.getEnvironment().getProperty("user.name");
        String age = applicationContext.getEnvironment().getProperty("user.age");
        System.out.println("name = " + name);
        System.out.println("age = " + age);

    }
}
```

```properties
#全局配置文件

server.port=8042
#配置中心的服务名:对应配置列表的Data ID
spring.application.name=com.eobard.order.redis
```

<img src="./Spring Cloud Alibaba_images/image-20220325134928525.png" alt="image-20220325134928525" style="zoom:80%;" /> 





（2）创建`bootstrap.properties`文件，配置相关属性

```properties
#bootstrap.properties：配置中心的相关配置,该文件和全局配置文件所属一个地方

#配置中心的服务名:对应配置列表的Data ID
spring.application.name=com.eobard.order.redis
#配置中心的地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos的账号
spring.cloud.nacos.username=nacos
#nacos的密码
spring.cloud.nacos.config.password=nacos
```

> **`注意：`**2023-01-11 10:45:34.783  INFO 7408 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: CompositePropertySource {name='NACOS', propertySources=[NacosPropertySource {name='**wemall-coupon.properties**'}, NacosPropertySource {name='wemall-coupon'}]}
>
> **`可以启动项目查看输出日志中，有个配置中心的对应的默认文件名，然后在nacos创建对应的配置文件就可以对应起来`**



（3）启动运行

> **基本原理：nacos客户端每10ms去注册中心根据MD5进行判断，若不一致则拉取注册中心的配置信息**

<img src="./Spring Cloud Alibaba_images/image-20220325142400508.png" alt="image-20220325142400508" style="zoom:80%;" />



（4）改变注册中心的属性值，再次查看控制台

<img src="./Spring Cloud Alibaba_images/image-20220325142743895.png" alt="image-20220325142743895" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220325142724666.png" alt="image-20220325142724666" style="zoom:80%;" />



### 6.3 其它扩展配置

#### 6.3.1文件扩展名

（1）若修改了注册中心配置文件的配置格式，则在Nacos-Config服务中也要修改，默认为Properties类型

<img src="./Spring Cloud Alibaba_images/image-20220326091731631.png" alt="image-20220326091731631" style="zoom:80%;" /> 



（2）修改Nacos-Config的`bootstrap.properties`配置文件并运行

```properties
#配置中心的服务名:对应配置列表的Data ID
spring.application.name=com.eobard.order.redis
#配置中心的地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos的账号
spring.cloud.nacos.username=nacos
#nacos的密码
spring.cloud.nacos.config.password=nacos
#修改了注册中心的文件格式，通过file-extension来设置，默认为properties
spring.cloud.nacos.config.file-extension=yaml
```

<img src="./Spring Cloud Alibaba_images/image-20220326092857598.png" alt="image-20220326092857598" style="zoom:80%;" /> 





#### 6.3.2 命名空间

> 利用命名空间来做环境隔离：开发可用dev环境，测试可用test环境，生产可用prod环境
>
> 利用模块来隔离：member模块可用member环境，coupon模块可用coupon环境，order模块可用order环境...

（1）新建一个dev命名空间，并将public的文件克隆到dev里面，修改值便于测试

<img src="./Spring Cloud Alibaba_images/image-20220326095920183.png" alt="image-20220326095920183" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220326094939806.png" alt="image-20220326094939806" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220326095015304.png" alt="image-20220326095015304" style="zoom:80%;" /> 



（2）修改bootstrap.properties文件

```properties
#配置中心的服务名:对应配置列表的Data ID
spring.application.name=com.eobard.order.redis
#配置中心的地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos的账号
spring.cloud.nacos.username=nacos
#nacos的密码
spring.cloud.nacos.config.password=nacos

#修改了注册中心的文件格式，通过file-extension来设置，默认为properties
spring.cloud.nacos.config.file-extension=yaml

#设置注册中心的命名空间,如果有多个命名空间,默认获取public命名空间
spring.cloud.nacos.config.namespace=dev

#设置配置中心的所属分组
spring.cloud.nacos.config.group=DEFAULT_GROUP
```

<img src="./Spring Cloud Alibaba_images/image-20220326095930436.png" alt="image-20220326095930436" style="zoom:80%;" /> 



#### 6.3.3 自定义data id

（1）创建自定义的配置文件

<img src="./Spring Cloud Alibaba_images/image-20220326103719853.png" alt="image-20220326103719853" style="zoom:80%;" />



（2）修改bootstrap.properties文件

> **配置文件优先级：相互的配置文件可以形成互补**

**profile > 默认配置文件 > extension-configs(下标越大,优先级越大) >  shared-configs(下标越大,优先级越大)**

==extension-configs和shared-configs是一模一样的用法，只是优先级不一样==



```properties
#配置中心的服务名:对应配置列表的Data ID
spring.application.name=com.eobard.order.redis
#配置中心的地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos的账号
spring.cloud.nacos.username=nacos
#nacos的密码
spring.cloud.nacos.config.password=nacos

#修改了注册中心的文件格式，通过file-extension来设置，默认为properties
spring.cloud.nacos.config.file-extension=yaml
#设置注册中心的命名空间
spring.cloud.nacos.config.namespace=dev

#设置自定义的配置文件data id
spring.cloud.nacos.config.shared-configs[0].data-id=custom-redis.properties
#设置是否自动刷新配置文件
spring.cloud.nacos.config.shared-configs[0].refresh=true
```



（3）修改启动类

```JAVA
@SpringBootApplication
public class NacosConfigApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(NacosConfigApplication.class, args);
        String name = applicationContext.getEnvironment().getProperty("user.name");
        String age = applicationContext.getEnvironment().getProperty("user.age");
        String config = applicationContext.getEnvironment().getProperty("user.config");
        System.out.println("name = " + name);
        System.out.println("age = " + age);
        System.out.println("config = " + config);

    }
}
```

<img src="./Spring Cloud Alibaba_images/image-20220326104226143.png" alt="image-20220326104226143" style="zoom:80%;" /> 



### 6.4 @RefreshScope(重点)

​			若在某个接口中需要动态的获取配置中心的动态值，则可以通过`@RefreshScope`注解完成

 

（1）创建控制器

```JAVA
@RestController
@RefreshScope
public class ConfigController {

    @Value("${user.config}")
    private String config;

    @Value("${user.name}")
    private String name;

    @GetMapping("/get")
    public String info(){
        return config+"====="+name;
    }

}
```



（2）运行测试

<img src="./Spring Cloud Alibaba_images/image-20220326105527518.png" alt="image-20220326105527518" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220326105625447.png" alt="image-20220326105625447" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220326105710232.png" alt="image-20220326105710232" style="zoom:80%;" /> 



### 6.5 加载多配置集（重点）

（1）设置多个配置文件：在coupon命名空间，在dev组中

<img src="./Spring Cloud Alibaba_images/image-20230111122800003.png" alt="image-20230111122800003" style="zoom: 80%;" />



（2）在项目中加载多配置集

```properties
#设置配置中心的服务名,对应的配置中心Data Id默认为 wemall-coupon.properties
spring.application.name=wemall-coupon
#设置配置中心的地址
spring.cloud.nacos.config.server-addr=127.0.0.1:8848
#设置配置中心的命名空间
spring.cloud.nacos.config.namespace=aff35074-b571-4f79-adc6-8afa5e39188c
#设置配置中心的默认分组
spring.cloud.nacos.config.group=dev

#加载nacos多配置集
    #设置需要加载配置的data-id
spring.cloud.nacos.config.ext-config[0].data-id=datasource.yml
    #设置需要加载配置的group组
spring.cloud.nacos.config.ext-config[0].group=dev
    #设置需要加载配置自动刷新,默认为false
spring.cloud.nacos.config.ext-config[0].refresh=true

spring.cloud.nacos.config.ext-config[1].data-id=mybatisplus.yml
spring.cloud.nacos.config.ext-config[1].group=dev
spring.cloud.nacos.config.ext-config[1].refresh=true

spring.cloud.nacos.config.ext-config[2].data-id=others.yml
spring.cloud.nacos.config.ext-config[2].group=dev
spring.cloud.nacos.config.ext-config[2].refresh=true
```



（3）启动项目可从nacos获取到配置信息

<img src="./Spring Cloud Alibaba_images/image-20230111123025416.png" alt="image-20230111123025416" style="zoom:80%;" />





## 七. sentinel

​				随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel是面向分布式服务架构的流量控制组件，**`主要以流量为切入点，从限流、流量整形、熔断降级、系统负载保护、热点防护等多个维度来帮助开发者保障微服务的稳定性。`**

* **流控主要是在服务的提供端**，控制访问流量和线程，作为流量的防卫兵

* **熔断主要是在服务的消费端**，查看某些服务是否出现了慢调用、异常等，此时就给它熔断停止访问，通过熔断时长来实现“自我修复”，保证了服务的可用性

[如何使用 · alibaba/Sentinel Wiki (github.com)](https://github.com/alibaba/Sentinel/wiki/如何使用)



### 7.1 服务雪崩

> 服务雪崩效应是一种因**服务提供者**的不可用导致**服务调用者**的不可用，并将不可用逐渐放大的过程。​

​			在服务提供者不可用的时候，会出现大量重试的情况：用户重试、代码逻辑重试，这些重试导致进一步加大请求流量。导致雪崩效应的最根本原因是：大量请求线程同步等待造成的资源耗尽。当服务调用者使用同步调用时，会产生大量的等待线程占用系统资源。一旦线程资源被耗尽，服务调用者提供的服务也将处于不可用状态，于是服务雪崩效应产生了。



<img src="./Spring Cloud Alibaba_images/image-20220327095555562.png" alt="image-20220327095555562" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220327095620942.png" alt="image-20220327095620942" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220327095637094.png" alt="image-20220327095637094" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220327095646778.png" alt="image-20220327095646778" style="zoom:80%;" /> 

常见的导致雪崩的情况有以下几种：

- 程序bug导致服务不可用，SQL的慢查询
- 缓存击穿，导致调用全部访问某服务，导致down掉
- 访问量的突然激增。
- 硬件问题





### 7.2 服务熔断

> 远程服务不稳定或网络抖动时暂时关闭，就叫服务熔断。

软件世界的断路器可以这样理解：实时监测应用，如果发现在一定时间内失败次数失败率达到一定阈值，就"跳闸"。断路器打开：请求直接返回，而不去调用原本调用的逻辑。跳闸一段时间后(例如 10秒)，断路器会进入半开状态，这是一个瞬间态，此时允许一次请求调用原本该调用的逻辑，如果成功，则断路器关闭，应用正常调用；如果调用依然不成功，断路器继续回到打开状态，过段时间 再进入半开状态尝试；通过"跳闸"，应用可以保护自己，而且避免浪费资源；而通过半开的设计，可实现应用的"自我修复"。

<img src="./Spring Cloud Alibaba_images/image-20220327101657015.png" alt="image-20220327101657015" style="zoom:80%;" />



### 7.3 服务降级
​				当某个服务熔断之后，服务将不再被调用，此时客户端可以自己准备一个本地的fallback回调，返回一个默认值。例如：(备用接口/缓存)。这样做，虽然服务水平下降，但好歹可用，比直接挂掉要强。



### 7.4 控制台部署

（1）根据Spring Cloud Alibaba的版本，下载对应的sentinel版本：[Releases · alibaba/Sentinel (github.com)](https://github.com/alibaba/Sentinel/releases)

<img src="./Spring Cloud Alibaba_images/image-20220328094804598.png" alt="image-20220328094804598" style="zoom:80%;" />



（2）运行jar包

```bash
java -jar sentinel-dashboard-1.8.0.jar
```



（3）访问页面

<img src="./Spring Cloud Alibaba_images/image-20220328095215708.png" alt="image-20220328095215708" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220328095227418.png" alt="image-20220328095227418" style="zoom:80%;" />



#### 创建批处理文件

（1）将`sentinel-dashboard-1.8.0.jar`放到固定的位置，如`D:\sentinel\`下面

（2）桌面创建批处理文件`sentinel.bat`进行编辑

```bash
java -Dserver.port=8858 -Dsentinel.dashboard.auth.username=root -Dsentinel.dashboard.auth.password=123456 -jar D:\sentinel\sentinel-dashboard-1.8.0.jar pause
```

* -Dserver.port=8858：代表端口号为8858
* -Dsentinel.dashboard.auth.username=root：代表登录账号为root
* Dsentinel.dashboard.auth.password=123456 ：代表登录密码为123456



（3）运行批处理文件即可

<img src="./Spring Cloud Alibaba_images/image-20220328100455668.png" alt="image-20220328100455668" style="zoom:80%;" />





### 7.5 整合Alibaba

（1）创建一个新的Order-Sentinel子模块项目

<img src="./Spring Cloud Alibaba_images/image-20220328101907963.png" alt="image-20220328101907963" style="zoom:80%;" /> 



（2）导入依赖

```xml
	<!--添加web场景启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--添加sentinel场景启动器-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```



（3）全局配置文件

```properties
server.port=8045
#设置注册到sentinel控制台项目名称
spring.application.name=order-sentinel
#设置sentinel控制台的地址
spring.cloud.sentinel.transport.dashboard=localhost:8858
```



（4）创建一个简单的控制器，并运行启动

```JAVA
@RestController
public class OrderController {

    @GetMapping("/index")
    public String index(){
        return "hello world";
    }
}
```



（5）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220328103318657.png" alt="image-20220328103318657" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220328103336014.png" alt="image-20220328103336014" style="zoom:80%;" /> 





### 7.6 @SentinelResource

​				通过反射+切面实现流控降级后的处理方法



#### 7.6.1 注解属性

* value：定义资源名
* blockHandler：设置流控降级后的处理方法名(默认该方法必须声明在同一个方法中)
*  blockHandlerClass：设置流控降级处理方法的所在类(必须是static静态方法)
*  fallback:当接口出现了异常，可以交给fallback指定的方法进行处理,用法同流控降级(若和流控逻辑同时设置，则流控逻辑优先级更高) 



 **自定义流控逻辑注意事项**

1. 必须是public
2. 返回值必须和源方法(被@SentinelResource注解标记的)保持一致
3.  方法中必须包含源方法的参数、顺序也要保持一致
4. 可在参数最后添加BlockException





#### 7.6.2 单独设置

（1）控制器

```JAVA
@RestController
public class OrderController {

    @GetMapping("/index")
 	@SentinelResource(value = "index",blockHandler = "flowBackHandler")
"customFlowBackHandler",blockHandlerClass = CustomBlockHandler.class)
    public String index(){
        return "hello world";
    }
    
      public String flowBackHandler(BlockException e){
        return "被流控了";
    }
}
```



（2）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220331104314643.png" alt="image-20220331104314643" style="zoom:80%;" /> 



#### 7.6.3 复用设置

（1）将流控降级的处理方法抽取成一个单独类

```JAVA
public class CustomBlockHandler {

    public static String customFlowBackHandler(BlockException e){
        return "自定义类的方式被流控了！！";
    }
}
```



（2）控制器引用

```JAVA
@RestController
public class OrderController {
    
	@SentinelResource(value = "index",blockHandler = "customFlowBackHandler",blockHandlerClass = CustomBlockHandler.class)
    public String index(){
        return "hello world";
    }
    
}
```



（3）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220331103055264.png" alt="image-20220331103055264" style="zoom:80%;" /> 



#### 7.6.4 统一异常处理

​				设置统一异常处理适合对BlockException返回的信息处理是一样的，如果不一样则还是需要使用@SentinelResource自定义即可



（1）定义全局异常处理类

```JAVA
@Component
public class GlobalExceptionHandler implements BlockExceptionHandler {

    Logger logger= LoggerFactory.getLogger(this.getClass());

    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse response, BlockException e) throws Exception {
        //getRule返回资源、规则的详细信息
        logger.info("BlockExceptionHandler BlockException================"+e.getRule());

        Result r = null;
        if(e instanceof FlowException){
            r = Result.error(100,"接口被限流了");
        }else if (e instanceof DegradeException){
            r = Result.error(101,"服务降级了");
        }else if (e instanceof ParamFlowException){
            r = Result.error(102,"热点参数限流了");
        }else if (e instanceof AuthorityException){
            r = Result.error(104,"授权规则不通过");
        }

        //返回Json数据
        response.setStatus(500);
        response.setCharacterEncoding("UTF-8");
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        new ObjectMapper().writeValue(response.getWriter(),r);
    }
}
```



（2）异常实体类

```JAVA
public class Result<T> {
    private Integer code;
    private String msg;
    private T data;

    public Result() {
    }

    public Result(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public static Result error(Integer code, String msg){
        return new Result(code,msg);
    }

	//省略getter、setter
}
```



（3）控制器

```java
    @GetMapping("/gloalException")
    public String gloalException(){
        return "正常访问";
    }
```



（4）运行结果

<img src="./Spring Cloud Alibaba_images/1648702220634.png" alt="1648702220634" style="zoom:80%;" /> 





### 7.6 QPS流控规则

​				监控应用流量的**QPS（每秒请求数，服务器在一秒内处理多少个请求）**或并发线程数等指标，当达到指定的阈值时对流量进行控制，以避免被瞬时的流量高峰冲垮，从而保障应用的高可用性。

<img src="./Spring Cloud Alibaba_images/image-20220328104002772.png" alt="image-20220328104002772" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220328104652197.png" alt="image-20220328104652197" style="zoom:80%;" />



（1）创建流控规则

<img src="./Spring Cloud Alibaba_images/image-20220328105053851.png" alt="image-20220328105053851" style="zoom:80%;" /> 



（2）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220328105132868.png" alt="image-20220328105132868" style="zoom:80%;" /> 



#### 自定义流控逻辑

（1）自定义流控处理逻辑，并重新启动

```JAVA
@RestController
public class OrderController {

    @GetMapping("/index")
    @SentinelResource(value = "index",blockHandler = "flowBackHandler")
    public String index(){
        return "hello world";
    }


    //自定义流控逻辑
    public String flowBackHandler(BlockException e){
        return "被流控了";
    }
}
```



（2）重新添加规则

<img src="./Spring Cloud Alibaba_images/image-20220328110648013.png" alt="image-20220328110648013" style="zoom:80%;" />



（3）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220328110726113.png" alt="image-20220328110726113" style="zoom:80%;" /> 



### 7.7 并发线程数

​				并发数控制用于保护业务线程池不被慢调用耗尽，例如：当应用所依赖的下游应用由于某种原因导致服务不稳定、响应延迟增加，对于调用者来说，意味着吞吐量下降和更多的线程数占用，极端情况下甚至导致线程池耗尽。为应对太多线程占用情况，业内有使用隔离的方案，比如通过不同业务逻辑使用不同线程地来隔离业务自身之间的资源争抢(线程池隔离)，这种隔离方案虽然隔离性比较好，但是代价就是线程数目太多，线程上下文切换的overhead比较大，特别是对低延时的调用有比较大的影响。**Sentinel并发控制不负责创建和管理线程池，而是简单统计当前请求上下文的线程数目，如果超出阈值，新的请求会被立即拒绝。并发数控制通常在调用端进行配置。**

<img src="./Spring Cloud Alibaba_images/image-20220331105217656.png" alt="image-20220331105217656" style="zoom:80%;" />





（1）控制器代码

```java
 	public String flowBackHandler(BlockException e){
        return "被流控了";
    }


    @GetMapping("/thread")
    @SentinelResource(value = "thread",blockHandler = "flowBackHandler")
    public String thread(){
        try {
            //线程休眠10s
            Thread.sleep(1000*10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "正常访问";
    }
```



（2）添加流控规则：一次只允许一个线程

<img src="./Spring Cloud Alibaba_images/image-20220331110151185.png" alt="image-20220331110151185" style="zoom:80%;" /> 



（3）运行结果

> 注意：最好通过两个浏览器访问同一个地址，这样可以清晰的看出效果

<img src="./Spring Cloud Alibaba_images/image-20220331110238749.png" alt="image-20220331110238749" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220331110213886.png" alt="image-20220331110213886" style="zoom:80%;" />





### 7.8 关联流控模式

> **当指定接口关联的接口达到限流条件时，开启对指定接口开启限流。**

关联：
			当两个资源之间具有资源争抢或者依赖关系的时候，这两个资源便具有了关联。比如对数据库同一个字段的读操作和写操作存在争抢，读的速度过高会影响写得速度，写的速度过高会影响读的速度。如果放任读写操作争抢资源，则争抢本身带来的开销会降低整体的吞吐量。可使用关联限流来避免具有关联关系的资源之间过度的争抢。



（1）控制器

```java
 	@GetMapping("/buy")
    public String buy(){
        return "下单成功";
    }

    @GetMapping("/query")
    public String query(){
        return "查询订单";
    }
```



（2）添加关联流控规则

<img src="./Spring Cloud Alibaba_images/image-20220402103731628.png" alt="image-20220402103731628" style="zoom:80%;" />



（3）jmeter压力测试

<img src="./Spring Cloud Alibaba_images/image-20220402103949183.png" alt="image-20220402103949183" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220402104008160.png" alt="image-20220402104008160" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220402104028700.png" alt="image-20220402104028700" style="zoom:80%;" /> 



（4）浏览器访问`/query`接口

<img src="./Spring Cloud Alibaba_images/image-20220402104050341.png" alt="image-20220402104050341" style="zoom:80%;" /> 





### 7.9 Warm Up（预热）

​			预热冷启动方式：当系统长期处于低水位的情况下，当流量突然增加时(`激增流量的情况`)，直接把系统拉升到高水位可能瞬间把系统压垮。通过冷启动，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。

> 冷加载因子: Code Factor默认是3，即请求QPS 从 threshold(阈值) / 3开始自增，经预热时长逐渐升至设定的QPS阈值。





（1）控制器

```java
 	@GetMapping("warmup")
    public String warmup() {
        return "预热访问";
    }
```



（2）添加流控规则

<img src="./Spring Cloud Alibaba_images/image-20220404092906719.png" alt="image-20220404092906719" style="zoom:80%;" />





（3）jmeter设置1秒10个线程

<img src="./Spring Cloud Alibaba_images/image-20220404093619575.png" alt="image-20220404093619575" style="zoom:80%;" />



（4）查看sentinel控制台实时监控结果

<img src="./Spring Cloud Alibaba_images/image-20220404093448020.png" alt="image-20220404093448020" style="zoom:80%;" /> 





### 7.10 排队等待

​				`适用于脉冲流浪`；匀速排队方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。该方式的作用如下图所示

<img src="./Spring Cloud Alibaba_images/image-20220404094330932.png" alt="image-20220404094330932" style="zoom:80%;" />

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景， 在某一秒有大量的请求到来, 而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求,而不是在第一秒直接拒绝多余的请求。

> `注意：匀速排队模式暂不支持QPS > 1000的情况`



（1）控制器

```java
 	@GetMapping("wait")
    public String Wait() {
        return "排队等待正常访问";
    }
```



（2）添加流控规则

<img src="./Spring Cloud Alibaba_images/image-20220404102109184.png" alt="image-20220404102109184" style="zoom:80%;" /> 



（3）jmeter模拟脉冲流量访问

<img src="./Spring Cloud Alibaba_images/image-20220404102237565.png" alt="image-20220404102237565" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220404102311728.png" alt="image-20220404102311728" style="zoom:80%;" /> 



（4）sentinel控制台实时监控结果

<img src="./Spring Cloud Alibaba_images/image-20220404101850895.png" alt="image-20220404101850895" style="zoom:80%;" />



### 7.11 熔断降级规则

​			微服务架构都是分布式的，由非常多的服务组成。不同服务之间相互调用，组成复杂的调用链路。以上的问题在链路调用中会产生放大的效果。复杂链路上的某一环不稳定，就可能会层层级联，最终导致整个链路都不可用。**因此我们需要对不稳定的弱依赖服务调用进行熔断降级，暂时切断不稳定调用，避免局部不稳定因素导致整体的雪崩。熔断降级作为保护自身的手段，通常在客户端（调用端）进行配置。**




#### 7.11.1 慢调用比例

​				选择以慢调用比例作为阈值，需要设置允许的慢调用RT(即最大的响应时间)，请求的响应时间大于该值则统计为慢调用。**`当单位统计时长内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态(HALF-OPEN状态)，若接下来的一个请求响应时间小于设置的慢调用RT则结束熔断，若大于设置的慢调用RT则会再次被熔断。`**



（1）控制器

```java
@GetMapping("/queryOne")
    public String queryOne(){
        try {
            //测试慢调用
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "查询数据库！";
    }
```



（2）新增降级规则

<img src="./Spring Cloud Alibaba_images/image-20220409095654642.png" alt="image-20220409095654642" style="zoom:80%;" />





（3）Jmeter测试，1秒内10个线程访问，并在接下来的5秒内通过浏览器访问；过了5秒再次访问

`当请求完后，此时进入熔断状态`

<img src="./Spring Cloud Alibaba_images/image-20220409101248922.png" alt="image-20220409101248922" style="zoom:80%;" /> 

`熔断状态内访问直接降级`

<img src="./Spring Cloud Alibaba_images/image-20220409101347073.png" alt="image-20220409101347073" style="zoom:80%;" /> 

`超过熔断时长，此时为探测恢复状态状态：若下一次的请求小于设置的慢调用RT则结束熔断，若大于设置的慢调用RT则会再次被熔断，继续重复上述操作`

<img src="./Spring Cloud Alibaba_images/image-20220409101418576.png" alt="image-20220409101418576" style="zoom:80%;" /> 





#### 7.11.2 异常比例

（1）控制器

```JAVA
  	@GetMapping("/err")
    public String err(){
        int a=1/0;
        return "";
    }
```



（2）新增降级规则

<img src="./Spring Cloud Alibaba_images/image-20220409102718817.png" alt="image-20220409102718817" style="zoom:80%;" /> 



（3）Jmeter测试1秒10个线程访问

`当请求完后，此时进入熔断状态`

<img src="./Spring Cloud Alibaba_images/image-20220409102817125.png" alt="image-20220409102817125" style="zoom:80%;" /> 

`熔断状态内访问直接降级`

<img src="./Spring Cloud Alibaba_images/image-20220409102829529.png" alt="image-20220409102829529" style="zoom:80%;" /> 

`超过熔断时长，此时为探测恢复状态状态：若下一次的请求正常访问则结束熔断，若失败会再次被熔断，继续重复上述操作`

<img src="./Spring Cloud Alibaba_images/image-20220409102844478.png" alt="image-20220409102844478" style="zoom:80%;" /> 



#### 7.11.3 异常数

<img src="./Spring Cloud Alibaba_images/image-20220409104115435.png" alt="image-20220409104115435" style="zoom:80%;" /> 



#### 7.11.4 整合OpenFeign

（1）复制Order-OpenFeign模块为Order-OpenFeign-Sentinel新模块

<img src="./Spring Cloud Alibaba_images/image-20220409113941776.png" alt="image-20220409113941776" style="zoom:80%;" /> 



（2）修改代码

```xml
       <!--sentinel-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

```properties
#项目端口号为8843
server.port=8843
#nacos将应用名称当做服务器名称
spring.application.name=order-openfeign-sentinel
#nacos地址
spring.cloud.nacos.server-addr=127.0.0.1:8848
#nacos账号
spring.cloud.nacos.discovery.username=nacos
#nacos密码
spring.cloud.nacos.discovery.password=nacos
#nacos命名空间
spring.cloud.nacos.discovery.namespace=public
#openfeign整合sentinel
feign.sentinel.enabled=true
```



（3）在Stock模块创建新接口

<img src="./Spring Cloud Alibaba_images/image-20220409113917667.png" alt="image-20220409113917667" style="zoom:80%;" />



（4）编写Feign接口和降级处理逻辑

```java
/**
 * 采用动态代理机制
 * name：调用的服务名
 * path：调用接口Controller上的@RequestMapping地址
 * configuration：设置stock-service服务的配置
 * fallback:降级的处理类
 */
@FeignClient(name = "stock-service",path = "/stock",fallback = StockFeignServiceFallBack.class)
public interface StockFeignService {

    //声明需要调用的rest接口对应的方法
    @GetMapping("/send")
    String send();
}
```

```java
//降级处理规则
@Component
public class StockFeignServiceFallBack implements StockFeignService {

    public String send() {
        return "远程服务降级了！";
    }
}
```



（5）控制器

```java
   	@GetMapping("/send")
    public String send() {
        return  stockFeginService.send();
    }
```



（6）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220409114514077.png" alt="image-20220409114514077" style="zoom:80%;" /> 



### 7.12 热点参数限流

​			何为热点?热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的数据，并对其访问进行限制。

<img src="./Spring Cloud Alibaba_images/image-20220410092628075.png" alt="image-20220410092628075" style="zoom:80%;" />



（1）控制器

```java
	 /**
     * 热点规则必须使用@SentinelResource
     * @return
     */
    @GetMapping("/testHot")
    @SentinelResource(value = "HotKey",blockHandler = "deal_testHotKey")
    public String testHotKey(@RequestParam(value = "p1",required = false) String p1) {
        return "------testHotKey";
    }

    public String deal_testHotKey (String p1, BlockException exception){
        return "热点参数被流控了";
    }
```



（2）新增热点规则

<img src="./Spring Cloud Alibaba_images/image-20220410112400301.png" alt="image-20220410112400301" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220410112641195.png" alt="image-20220410112641195" style="zoom:80%;" /> 



（3）运行结果

<img src="./Spring Cloud Alibaba_images/image-20220410114144342.png" alt="image-20220410114144342" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220410114154307.png" alt="image-20220410114154307" style="zoom:80%;" /> 



## 八. Seata

### 8.1 分布式事务概念

​				指事务的操作位于不同的节点上，需要服务与服务之间远程协作才能完成事务操作，这种分布式系统环境下由不同的服务之间通过网络远程协作完成事务称之为分布式事务。**简单的说，就是一次大的操作由不同的小操作组成，这些小的操作分布在不同的服务器上，且属于不同的应用，分布式事务需要保证这些小操作要么全部成功，要么全部失败。本质上来说，分布式事务就是为了保证不同数据库的数据一致性。**

<img src="./Spring Cloud Alibaba_images/image-20220416095205724.png" alt="image-20220416095205724" style="zoom:80%;" />





### 8.2 二阶段提交协议(2PC）

<img src="./Spring Cloud Alibaba_images/er.png" style="zoom:80%;" />

> - undo：记录更新前数据，用于保证事务原子性，作为回滚。
> - redo： 记录更新后数据，用于保证事务的持久性，作为提交。

第一阶段:  （1）协调者向参与者发送事务请求：询问是否执行事务操作，然后等待参与者的响应

​						（2）参与者接收到协调者请求后，执行事务操作，并将undo和redo信息记录事务日志中

​						（3）成功执行了事务、写入了undo和redo，向协调者返回ack：yes；否则返回ack：no



第二阶段：（1）协调者向参与者发送commit请求；参与者收到commit请求：执行事务提交，提交完成后释放占用									 的资源。

​						（2）参与者执行事务提交后向协调者发送ack：yes响应

​						（3）协调者接收所有参与者ack响应后，完成事务提交



**2PC存在的问题----同步阻塞：参与者等待协调者指令时是处于阻塞状态，无法进行其他操作；协调者宕机，参与者会一直阻塞占用事务资源**





### 8.3 AT模式(重点！)

​				AT模式的一阶段、二阶段提交和回滚均由Seata框架自动生成，一阶段事务的控制在DB内。用户只需编写业务SQL"，便能轻松接入分布式事务，AT模式是一种对业务无任何侵入的分布式事务解决方案。

<img src="./Spring Cloud Alibaba_images/image-20220416111528815.png" alt="image-20220416111528815" style="zoom:80%;" />



第一阶段：（1）Seata拦截并解析业务SQL，找到要更新的数据将其保存成before image(undo)；然后执行更新业务									 SQL，将更新之后的数据保存成after image(redo)

​						（2）将当前数据生成行锁，保证了一阶段操作的原子性

<img src="./Spring Cloud Alibaba_images/image-20220416111412493.png" alt="image-20220416111412493" style="zoom:80%;" />





第二阶段：（1）执行成功，Seata框架只需要将一阶保存的快照数据和行锁删掉，完成数据清理

<img src="./Spring Cloud Alibaba_images/image-20220416111955218.png" alt="image-20220416111955218" style="zoom:80%;" />

​					  （2）执行失败进行业务回滚：首先对比after image和数据库当前业务数据，如果一致就用before image执行update操作还原数据，并且删除中间数据；如果不一致说明数据已经被修改，需要人工介入

<img src="./Spring Cloud Alibaba_images/image-20220416112250868.png" alt="image-20220416112250868" style="zoom:80%;" />



### 8.4 TCC模式

​			TCC模式需要用户根据自己的业务场景实现 Try、Confirm和Cancel三个操作；事务发起方在一阶段执行Try方法，在二阶段提交执行Confirm方法，若有回滚操作，则在二阶段回滚执行Cancel方法。

* 侵入性强，并且得自己实现事务相关控制逻辑
* 整个过程基本没有锁，性能更强

<img src="./Spring Cloud Alibaba_images/image-20220416122745729.png" alt="image-20220416122745729" style="zoom:80%;" />



> **一个下单请求示例**

<img src="./Spring Cloud Alibaba_images/image-20220416114414453.png" alt="image-20220416114414453" style="zoom: 80%;" />

* 在一阶段执行try中的方法逻辑体，一般不会真正的修改数据值
* 在二阶段confirm方法中去真正的修改数据值，如有回滚则执行cancel的方法





### 8.5  服务搭建(db+nacos高可用集群方式)

1. 根据spring cloud alibaba版本下载对应的服务端文件 [Releases · seata/seata (github.com)](https://github.com/seata/seata/releases)

<img src="./Spring Cloud Alibaba_images/image-20220417100555561.png" alt="image-20220417100555561" style="zoom:80%;" />



2. 解压压缩文件，并更改`conf/file.conf`文件

<img src="./Spring Cloud Alibaba_images/image-20220417101456047.png" alt="image-20220417101456047" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220417110711953.png" alt="image-20220417110711953" style="zoom:80%;" />



3. 创建数据库，并执行sql

<img src="./Spring Cloud Alibaba_images/image-20220417110815526.png" alt="image-20220417110815526" style="zoom:80%;" />

```sql
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(96),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```

<img src="./Spring Cloud Alibaba_images/image-20220417110859614.png" alt="image-20220417110859614" style="zoom:80%;" /> 





4. 启动nacos服务，并修改conf/registry.conf文件

<img src="./Spring Cloud Alibaba_images/image-20220418090820239.png" alt="image-20220418090820239" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220418091202477.png" alt="image-20220418091202477" style="zoom:80%;" /> 





5. 下载文件 [seata: Seata-Gitee.com](https://gitee.com/seata-io/seata/tree/develop)，并将script文件夹复制到本地seata的位置

<img src="./Spring Cloud Alibaba_images/image-20220418092807043.png" alt="image-20220418092807043" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220418092824225.png" alt="image-20220418092824225" style="zoom:80%;" />





6. 编辑script下的文件

<img src="./Spring Cloud Alibaba_images/image-20220418093147164.png" alt="image-20220418093147164" style="zoom:80%;" />







7. 执行文件

<img src="./Spring Cloud Alibaba_images/image-20220418093326154.png" alt="image-20220418093326154" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220418093753290.png" alt="image-20220418093753290" style="zoom: 80%;" />



> **eg：配置远程nacos地址**

<img src="./Spring Cloud Alibaba_images/image-20220418093929146.png" alt="image-20220418093929146" style="zoom:80%;" />









8. **执行seata.bat**

<img src="./Spring Cloud Alibaba_images/image-20220418100433120.png" alt="image-20220418100433120" style="zoom:80%;" />



> **注意：若想要更改端口号，使用下列参数命令（针对于linux版本！）**

<img src="./Spring Cloud Alibaba_images/image-20220418100554751.png" alt="image-20220418100554751" style="zoom:80%;" /> 



9. 运行结果

<img src="./Spring Cloud Alibaba_images/image-20220418100855236.png" alt="image-20220418100855236" style="zoom:80%;" />



### 8.6 本地事务存在的问题

#### 8.6.1 创建order子服务

1. 环境搭建

```xml
 	<!--添加web场景启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

		<!--添加openfegin依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <!-- MyBatis Plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.1</version>
        </dependency>
        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
```

```properties
#配置nacos服务名称
spring.application.name=seata-order
server.port=8008
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://192.168.2.102:3307/order
spring.datasource.username=root
spring.datasource.password=123456
#打印sql语句
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

2. 启动器

```java
@SpringBootApplication
@MapperScan({"com.eobard.order.mapper"})//扫描包
//使用openFeign
@EnableFeignClients
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }

}
```

3. 代码

```java
//dao层
public interface UserMapper extends BaseMapper<User> {
}

//feign层
@FeignClient(name = "seata-stock",path = "/stock")
public interface StockFeign {

     @RequestMapping("/reduct")
     String reduct();
}


//实体类
@TableName("t_user")
public class User {

    @TableId(value = "id",type = IdType.INPUT)
    private Integer id;
    private String name;

    public User(Integer id, String name) {
        this.id = id;
        this.name = name;
    }
    
    //省略getter、setter
}
```

4. 控制器

```java
@RestController
@RequestMapping("/order")
public class OrderController {

    @Resource
    private StockFeign stockFeign;
    
    @Resource
    private UserMapper mapper;

    @Transactional
    @GetMapping("buy/{productId}")
    public String buy(@PathVariable Integer productId) {
        User order = new User(productId,"zs");
        System.out.println(order);
        mapper.insert(order);
        
        //调用远程服务
        String msg = stockFeign.reduct();
        
        //创造异常
        int a=1/0;
        return msg;
    }

}
```





#### 8.6.2 创建stock子服务

1. 环境搭建

```xml
   <!--添加web场景启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!-- Spring Data JPA -->
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
```

```properties
#配置nacos服务名称
spring.application.name=seata-stock
server.port=8009
#数据库配置:
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://192.168.2.102:3307/stock
spring.datasource.username=root
spring.datasource.password=123456
#整合数据连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.jpa.show-sql=true
#JPA正向工程
spring.jpa.hibernate.ddl-auto=update
```

2. 启动器

```java
@SpringBootApplication
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class,args);
    }

}
```

3. 代码

```java
//dao层
public interface ProductRepository extends JpaRepository<Product,Integer> {

}


//实体类
@Entity //标识是一个实体类
@Table(name = "product") //指定生产数据库的表名
public class Product {

    @Id //主键
    @GeneratedValue(strategy = GenerationType.AUTO) //主键类型:自增类型
    private Integer id;
    private String name;
    private Integer count;

    public Product() {
    }
    public Product(Integer id, String name, Integer count) {
        this.id = id;
        this.name = name;
        this.count = count;
    }
      //省略getter、setter
}
```

4. 控制器

```java
@RestController
@RequestMapping("/stock")
public class StockController {

    @Resource
    private ProductRepository repository;

    @RequestMapping("/reduct")
    @Transactional
    public String reduct(){
        Product one = repository.getOne(1);
        one.setCount(one.getCount()-1);
        repository.save(one);
        return"下单成功，扣减库存-1";
    }
}
```



#### 8.6.3 运行结果

1. 初始结果

<img src="./Spring Cloud Alibaba_images/image-20220419130533635.png" alt="image-20220419130533635" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220419130618689.png" alt="image-20220419130618689" style="zoom:80%;" /> 



2. 启动order子服务，扣减库存

<img src="./Spring Cloud Alibaba_images/image-20220419130809161.png" alt="image-20220419130809161" style="zoom:80%;" />

3. 查看数据库

<img src="./Spring Cloud Alibaba_images/image-20220419130852036.png" alt="image-20220419130852036" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220419130935076.png" alt="image-20220419130935076" style="zoom:80%;" /> 

4. 总结

<img src="./Spring Cloud Alibaba_images/image-20220419131120119.png" alt="image-20220419131120119" style="zoom:80%;" />







### 8.7 改进为分布式事务(AT)

1. 上面两个子服务分别导入依赖

```xml
  <!--nacos服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <!--seata依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
        </dependency>

        <!--添加openfegin依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```



2. 两个子服务的数据库分别执行以下SQL

```sql
CREATE TABLE IF NOT EXISTS `undo_log`
(
    `id`            BIGINT(20)   NOT NULL AUTO_INCREMENT COMMENT 'increment id',
    `branch_id`     BIGINT(20)   NOT NULL COMMENT 'branch transaction id',
    `xid`           VARCHAR(100) NOT NULL COMMENT 'global transaction id',
    `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
    `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
    `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
    `log_created`   DATETIME     NOT NULL COMMENT 'create datetime',
    `log_modified`  DATETIME     NOT NULL COMMENT 'modify datetime',
    PRIMARY KEY (`id`),
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8 COMMENT ='AT transaction mode undo table';
```

<img src="./Spring Cloud Alibaba_images/image-20220514095132348.png" alt="image-20220514095132348" style="zoom:80%;" /> 





3. 配置事务分组

<img src="./Spring Cloud Alibaba_images/image-20220514115823167.png" alt="image-20220514115823167" style="zoom:80%;" />



4. 在两个子服务配置文件分别配置

```properties
#设置事务分组：必须和config.txt的vgroupMapping里面的对应
spring.cloud.alibaba.seata.tx-service-group=chongqing
seata.service.vgroup-mapping.chongqing=default
seata.enabled=true

#配置seata注册中心：告诉微服务怎么访问seata-server
seata.registry.type=nacos
seata.registry.nacos.server-addr=127.0.0.1:8848
seata.registry.nacos.username=nacos
seata.registry.nacos.password=nacos
seata.registry.nacos.application=seata-server
seata.registry.nacos.group=SEATA_GROUP

#配置seata配置中心
seata.config.type=nacos
seata.config.nacos.server-addr=127.0.0.1:8848
seata.config.nacos.username=nacos
seata.config.nacos.password=nacos
seata.config.nacos.group=SEATA_GROUP
```



5. 将order远程调用服务的本地事务注解更换为全局事务注解

<img src="./Spring Cloud Alibaba_images/image-20220514115635073.png" alt="image-20220514115635073" style="zoom:80%;" />

 <img src="./Spring Cloud Alibaba_images/image-20220514115647373.png" alt="image-20220514115647373" style="zoom:80%;" />

> **`注意：调用远程服务的消费者才加@GlobalTransactional注解`**



#### 运行结果

* **注释1/0的异常：发现事务正常处理**

<img src="./Spring Cloud Alibaba_images/image-20220514120058797.png" alt="image-20220514120058797" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220514120116154.png" alt="image-20220514120116154" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220514120130838.png" alt="image-20220514120130838" style="zoom:80%;" /> 





* **制造1/0的异常：事务正常回滚，`不会出现8.6.3的情况`**

<img src="./Spring Cloud Alibaba_images/image-20220514120303961.png" alt="image-20220514120303961" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220514120358103.png" alt="image-20220514120358103" style="zoom:80%;" /> 

<img src="./Spring Cloud Alibaba_images/image-20220514120414683.png" alt="image-20220514120414683" style="zoom:80%;" /> 





## 九. Gateway网关组件

### 9.1 核心概念

1. **Route(路由)**：路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由，目标URI会被访问。
2. **Predicate(断言)**：这是一个java 8的Predicate，可以使用它来匹配来自HTTP请求的任何内容，如：请求头和请求参数。断言的输入类型是一个ServerWebExchange。

​			`注意:当访问gateway时,使用断言对请求进行匹配:匹配成功就路由转发,否则返回404`

3. **Filter(过滤器)**：指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者后对请求进行修改。

> ​			**web请求通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。predicate就是匹配条件，而filter，就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标URI，就可以实现具体的路由了。**





### 9.2 简单使用

1. 搭建Gateway子模块，导入依赖

```xml-dtd
   <!--gateway依赖-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
```



2. 全局配置

<img src="./Spring Cloud Alibaba_images/image-20220521102414238.png" alt="image-20220521102414238" style="zoom:80%;" />

```yaml
server:
  port: 8010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        - id: stock_route
          uri: http://localhost:8082
          predicates:
            - Path=/stock-serv/**
          filters:
            - StripPrefix=1
```



3. 访问

<img src="./Spring Cloud Alibaba_images/image-20220521102556714.png" alt="image-20220521102556714" style="zoom:80%;" /> 





### 9.3 配置多个路由

```yaml
server:
  port: 8200

spring:
  cloud:
    # nacos地址
    nacos:
      discovery:
        server-addr: localhost:8848

    # gateway配置
    gateway:
      discovery:
        locator:
          enabled: true   # 开启路由发现
      # 路由配置
      routes:
        # 权限服务模块
        - id: wegou-service-acl
          uri: lb://wegou-service-acl
          predicates:   
            - Path=/*/acl/**

        # 商品模块
        - id: wegou-service-product
          uri: lb://wegou-service-product
          predicates:
            - Path=/*/product/**

        # 活动模块
        - id: wegou-service-activity
          uri: lb://wegou-service-activity
          predicates:
            - Path=/*/activity/**

		# 订单模块
        - id: wegou-service-order
          uri: lb://wegou-service-order
          predicates:
            - Path=/*/order/**

		# 支付模块
        - id: wegou-service-payment
          uri: lb://wegou-service-payment
          predicates:
            - Path=/*/payment/**
```

**注意：`/*/order/** `  ，`*`表示前面一层路径，`**`表示多层路径，eg : /admin/order/get/1就可以匹配到订单模块的断言**





### 9.4  整合Nacos

1. 导入依赖

```xml
 <!--nacos服务注册发现-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
```



2. 修改配置文件

```yaml
server:
  port: 8010	  #端口号
spring:
  application:
    name: api-gateway  #服务名称
  cloud:
    #网关配置
    gateway:
      routes:
        - id: stock_route           #路由的唯一标识
          uri: lb://stock-service   # 需要转发的应用地址:   lb://nacos服务名
          #其中lb表示使用本地负载均衡策略

          # 断言规则
          #  只有输入: 网关地址/stock-serv/stock/reduct
          #  才会转发到:
          #     http://nacos中stock-service的ip地址/stock-serv/stock/reduct
          predicates:
            - Path=/stock-serv/**

          #转发真正路径之前去除第一层路径,变为真正的地址
          #     http://nacos中stock-service的ip地址/stock/reduct
          filters:
            - StripPrefix=1
#nacos配置
    nacos:
      server-addr: 127.0.0.1:8848
      discovery:
        namespace: public
        username: nacos
        password: nacos

```



3. 访问

<img src="./Spring Cloud Alibaba_images/image-20220521110226263.png" alt="image-20220521110226263" style="zoom:80%;" /> 



4. **`注意事项`**

```java
//通常来说网关服务不需要数据源自动装配类,排除数据源自动装配类
@SpringBootApplication(exclude =DataSourceAutoConfiguration.class)
```







### 9.5 内置路由断言工厂

**`注意：路由断言可以写多个条件，表示and关系`**



* 基于Datetime类型的断言工厂

  * AfterRoutePredicateFactory： 接收一个日期参数，判断请求日期是否晚于指定日期
  * BeforeRoutePredicateFactory： 接收一个日期参数，判断请求日期是否早于指定日期
  * BetweenRoutePredicateFactory： 接收两个日期参数，判断请求日期是否在指定时间段内

  ```yaml
  #可以通过获取日期值:ZonedDateTime.now()
   - After=2022-05-22T10:28:22.765+08:00[Asia/Shanghai]	
  ```

  

  

* 基于远程地址的断言工厂

  * RemoteAddrRoutePredicateFactory：接收一个IP地址段，判断请求主机地址是否在地址段中
  
  ```yaml
   - RemoteAddr=192.168.1.1/24
  ```
  
  

* 基于Cookie的断言工厂

  * CookieRoutePredicateFactory：接收两个参数，cookie 名字和一个正则表达式。 判断请求cookie是否具有给定名称且值与正则表达式匹配。

  ```yaml
   - Cookie=cookie的K,cookie具体value值或正则表达式
  ```

  

* 基于Header的断言工厂

  * HeaderRoutePredicateFactory：接收两个参数，标题名称和正则表达式。 判断请求Header是否具有给定名称且值与正则表达式匹配。

  ```yaml
   - Header=X‐Request‐Id,\d+
  ```

  

* 基于Host的断言工厂

  * HostRoutePredicateFactory：接收一个参数，主机名模式。判断请求的Host是否满足匹配规则。

  ```yaml
   - Host=**.testhost.org
  ```

  

* 基于Method请求方法的断言工厂

  * MethodRoutePredicateFactory：接收一个参数，判断请求类型是否跟指定的类型匹配。

  ```yaml
   - Method=GET
  ```

  

* 基于Path请求路径的断言工厂

  * PathRoutePredicateFactory：接收一个参数，判断请求的URI部分是否满足路径规则。

  ```yaml
   - Path=/get/{segment}
   - Path=/get/**
  ```

  

* 基于Query请求参数的断言工厂

  * QueryRoutePredicateFactory ：接收两个参数，请求param和正则表达式， 判断请求参数是否具有给定名称且值与正则表达式匹配。

  ```yaml
   - Query=key,value
  ```

  

* 基于路由权重的断言工厂

  * WeightRoutePredicateFactory：接收一个[组名,权重], 然后对于同一个组内的路由按照权重转发

  ```yaml
   -id: weight_low
     uri: host1
     predicates: 
       - Path=/product/**
       - Weight=group1, 2 
  #请求10次有8次匹配下面，剩余两次匹配上面
   - id: weight_high
     uri: host2
     predicates:
       - Path=/product/**
       - Weight= group1, 8
  ```

  

#### 示例

```yaml
  #访问方法是GET、在指定时间之后、请求头token数据是数字、请求参数name的值是zs或者ls、路径带有stock-serv的才可以转发
   predicates:
            - Path=/stock-serv/**
            - After=2022-05-22T10:28:22.765+08:00[Asia/Shanghai]
            - Method=GET
            - Query=name,zs|ls
            - Header=token,\d+
```

* 成功访问

<img src="./Spring Cloud Alibaba_images/image-20220522105714377.png" alt="image-20220522105714377" style="zoom:80%;" />



* 不满足任一条件都访问失败

<img src="./Spring Cloud Alibaba_images/image-20220522105759390.png" alt="image-20220522105759390" style="zoom:80%;" /> 







### 9.6 自定义路由断言工厂

* 必须作为spring组件bean

* 自定义路由类必须以RoutePredicateFactory结尾

* 必须继承AbstractRoutePredicateFactory

* 必须声明静态内部类，声明属性来接收配置文件的信息

* 需要结合shortcutFieldOrder来进行数据绑定

* 通过apply来进行逻辑判断是否匹配

```java
@Component
public class CheckNameRoutePredicateFactory extends AbstractRoutePredicateFactory<CheckNameRoutePredicateFactory.Config> {

    public CheckNameRoutePredicateFactory() {
        super(CheckNameRoutePredicateFactory.Config.class);
    }


    //进行数据的绑定
    public List<String> shortcutFieldOrder() {
        return Arrays.asList("name");
    }

    //进行逻辑判定是否转发路由
    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        return new GatewayPredicate() {
            public boolean test(ServerWebExchange exchange) {
               if(config.getName().equals("eobard")){
                   return true;
               }
                return false;
            }
        };
    }

    //用于接收全局配置文件断言信息的值
    @Validated
    public static class Config {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```



访问

```yaml
 predicates:
            - Path=/stock-serv/**
            - CheckName=eobard
```

<img src="./Spring Cloud Alibaba_images/image-20220522113115752.png" alt="image-20220522113115752" style="zoom:80%;" /> 





### 9.7 内置局部过滤器

​				在Gateway中，Filter的生命周期只有两个：“pre”和“post”：

- **PRE**：这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。
- **POST**：这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。

> 局部过滤器针对于某个路由，需要在路由中进行配置

<img src="./Spring Cloud Alibaba_images/image-20220711160837304.png" alt="image-20220711160837304" style="zoom:80%;" />





#### 9.7.1 过滤器作用

详细过滤器见[Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#the-addrequestheader-gatewayfilter-factory)

<img src="./Spring Cloud Alibaba_images/image-20220711160628245.png" alt="image-20220711160628245" style="zoom:80%;" />



#### 9.7.2 示例1

> 使用`AddRequestHeader`和`AddRequestParameter`两个过滤器模拟前后端分离

1. 修改网关的配置文件

   ```yaml
   server:
     #端口号
     port: 8010
   spring:
     application:
       #服务名称
       name: api-gateway
     cloud:
       #网关配置
       gateway:
         routes:
           - id: stock_route           #路由的唯一标识
             uri: lb://stock-service   # 需要转发的地址:   lb://nacos服务名
   
            
             predicates:
               - Path=/stock-serv/**
               
             filters:
               - AddRequestHeader=token,wqeqweqwe	#添加key为token的请求头，值为wqe..
               - AddRequestParameter=username,eobard #添加请求的key为username，值为eobard
               - StripPrefix=1
   #nacos配置
       nacos:
         server-addr: 127.0.0.1:8848
         discovery:
           namespace: public
           username: nacos
           password: nacos
   
   ```



2. stock-nacos子服务模块

```java
@RestController
@RequestMapping("/stock")
public class StockController {


    @Value("${server.port}")
    private String port;

    @RequestMapping("/reduct")
    public String reduct(){
        return port+"端口，库存商品-1";
    }


    @RequestMapping("/getHead")
    public String reduct(@RequestHeader("token")String header, @RequestParam("username")String name){
        return port+header+" ==="+name;
    }

}
```



3. 访问

<img src="./Spring Cloud Alibaba_images/image-20220711161833744.png" alt="image-20220711161833744" style="zoom:80%;" />



#### 9.7.2 示例2（路径重写）

网关配置文件

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: admin_route
#负载均衡到nacos中renren-fast的服务中
          uri: lb://renren-fast
#前端项目的请求需要满足/api/xxxx 才能匹配网关
          predicates:
            - Path=/api/**
#路径重写:前端发送请求 网关地址/api/xxx.jpg  会被重写成请求 nacos中renren-fast的服务地址/renren-fast/xxx.jpg
          filters:
            - RewritePath=/api/(?<segment>.*),/renren-fast/$\{segment}
```





#### 9.7.3 路由优先级(重点)

```yaml
spring:
  cloud:
    gateway:
      routes:
#管理员模块
        - id: admin_route
          #负载均衡到nacos中renren-fast的服务中
          uri: lb://renren-fast
          #前端项目的请求需要满足/api/xxxx 才能匹配网关
          predicates:
            - Path=/api/**
          #路径重写:前端发送的请求 http://localhost:88/api/xxx.jpg 会被重写成请求 nacos中renren-fast的服务地址/renren-fast/xxx.jpg
          filters:
            - RewritePath=/api/(?<segment>.*),/renren-fast/$\{segment}
#商品模块
        - id: product_route
          #负载均衡到nacos中wemall-product的服务中
          uri: lb://wemall-product
          predicates:
            - Path=/api/product/**
          filters:
            #路径重写:前端发送的请求 http://localhost:88/api/product/xxx 会被重写成请求 nacos中wemall-product的服务地址/xxx
            - RewritePath=/api/(?<segment>.*),/$\{segment}
```

当我们访问 http://localhost:88/api/product/xxx时，会首先被第一个请求拦截，如果第一个有spring security管理的话，则我们不能进入product_route的url，这时候我们可以调整路径断言的顺序来改变优先级，即把商品模块放在管理员模块之前生效，这样就优先匹配/api/product/xx然后再匹配/api/**

> **如果断言中有Path前缀相同的情况，可以根据自己的需要调整生效顺序**





### 9.8 全局过滤器

针对于所有路由请求，一旦定义就会投入使用

1. 自定义日志全局过滤器

```java
@Component
public class LogFilter implements GlobalFilter {
    Logger logger= LoggerFactory.getLogger(this.getClass());

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String url=exchange.getRequest().getPath().value();
        logger.info(url);
        return chain.filter(exchange);
    }
}
```

2. 访问

<img src="./Spring Cloud Alibaba_images/image-20220711171758607.png" alt="image-20220711171758607" style="zoom:80%;" />



### 9.9 跨域处理

#### 9.9.1 通过配置文件

```yaml
spring:
  application:
    #服务名称
    name: api-gateway
  cloud:
    #网关配置
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
              allowedOrigins: "*"      # 允许哪些网站的跨域请求，这里设置为允许所有
#                - "http://localhost:8090"
#                - "http://www.leyou.com"
              allowedMethods: # 允许的跨域ajax的请求方式
                - "GET"
                - "POST"
                - "DELETE"
                - "PUT"
```



#### 9.9.2 通过配置类

```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsWebFilter corsWebFilter(){
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedMethod("*");   //允许的method
        config.addAllowedOrigin("*");   //允许的来源
        config.addAllowedHeader("*");   //允许的请求头参数

        //允许访问的资源：import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;
        UrlBasedCorsConfigurationSource source=new UrlBasedCorsConfigurationSource(new PathPatternParser());
        source.registerCorsConfiguration("/**",config);
        return new CorsWebFilter(source);
    }
}
```





### 9.10 网关整合sentinel限流

```xml-dtd
  <!--添加sentinel场景启动器-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>

        <!--sentinel整合gateway-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
        </dependency>
```

```yaml
#sentinel配置
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 127.0.0.1:8858
```





#### 9.10.1 根据QPS流控

<img src="./Spring Cloud Alibaba_images/image-20220716203828528.png" alt="image-20220716203828528" style="zoom:80%;" />





#### 9.10.2 针对请求属性

> * 精确：参数属性的值必须和匹配串的值相等
> * 子串：参数属性的值与匹配串前面相等即可(类似于subString操作)
> * 正则：参数属性的值和正则表达式的值匹配即可

<img src="./Spring Cloud Alibaba_images/image-20220716203957452.png" alt="image-20220716203957452" style="zoom:80%;" />



<img src="./Spring Cloud Alibaba_images/image-20220716204400962.png" alt="image-20220716204400962" style="zoom:80%;" />





#### 9.10.3 针对API分组

<img src="./Spring Cloud Alibaba_images/image-20220716205609764.png" alt="image-20220716205609764" style="zoom:80%;" />



<img src="./Spring Cloud Alibaba_images/image-20220716205724459.png" alt="image-20220716205724459" style="zoom:80%;" />





#### 9.10.4 自定义异常返回处理

##### 方法一:实体类

```java
public class Result<T> {
    private Integer code;
    private String msg;
    private T data;

    public Result() {
    }

    public Result(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public static Result error(Integer code, String msg){
        return new Result(code,msg);
    }

    //省略getter、setter
}
```

```java
@Configuration
public class GatewayConfig {

    @PostConstruct
    public void init(){
        BlockRequestHandler handler = new BlockRequestHandler() {
           //实例化json返回字符串
       Result r=Result.error(HttpStatus.TOO_MANY_REQUESTS.value(),"当前访问过多，限流了！");
            @Override
            public Mono<ServerResponse> handleRequest(ServerWebExchange serverWebExchange, Throwable throwable) {
                // 打印异常对应的实体类        
				// System.out.println( throwable);
                return ServerResponse.status(HttpStatus.OK)
                        .contentType(MediaType.APPLICATION_JSON)
                        .body(BodyInserters.fromObject(r));
            }
        };
        GatewayCallbackManager.setBlockHandler(handler);
    }
}
```





##### 方法二:配置文件

```yaml
spring:
  cloud:
     scg:
        fallback:
          mode: response
          response-body: "{'code':429,'msg':'当前访问过多，限流了！'}"
```







## 十.链路追踪SkyWalking

​				Skywalking是分布式系统的应用程序性能监视工具，专为微服务，云原生架构和基于容器（Docker，K8S,Mesos）架构而设计，它是一款优秀的APM（Application Performance Management）工具，包括了分布式追踪，性能指标分析和服务依赖分析等。



### SkyWalking服务端搭建

1. 打开网站  [Index of /dist/skywalking/8.5.0 (apache.org)](https://archive.apache.org/dist/skywalking/8.5.0/)下载文件

<img src="./Spring Cloud Alibaba_images/image-20220717193618845.png" alt="image-20220717193618845" style="zoom:80%;" />

2. 解压缩下载的文件夹到非中文路径<img src="./Spring Cloud Alibaba_images/image-20220717203952744.png" alt="image-20220717203952744" style="zoom:80%;" />

3. 修改解压缩文件里面的webapp的webapp.yml文件的端口号

<img src="./Spring Cloud Alibaba_images/image-20220717194836930.png" alt="image-20220717194836930" style="zoom:80%;" />





4. 运行bin文件夹的startup.bat即可

<img src="./Spring Cloud Alibaba_images/image-20220717194946527.png" alt="image-20220717194946527" style="zoom:80%;" />



### 接入多个微服务

1. 首先在每个微服务的vm运行参数添加以下代码

* 参数1：skywalking-agent.jar的具体位置
* 参数2：当前微服务的服务名称
* 参数3：设置SkyWalking的collector地址

```
-javaagent:D:\Java\SkyWalking\apache-skywalking-apm-bin-es7\agent\skywalking-agent.jar
-DSW_AGENT_NAME=stock-service
-DSW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800
```

<img src="./Spring Cloud Alibaba_images/image-20220717201353774.png" alt="image-20220717201353774" style="zoom:80%;" />





2. 浏览器就看查看各个微服务情况

<img src="./Spring Cloud Alibaba_images/image-20220717201758703.png" alt="image-20220717201758703" style="zoom:80%;" />



3. 将复制的文件放入plugins文件夹，并重启服务

<img src="./Spring Cloud Alibaba_images/image-20220717202920503.png" alt="image-20220717202920503" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220717203540415.png" alt="image-20220717203540415" style="zoom:80%;" />







### 使用mysql持久化

1. 打开config文件夹，修改配置

<img src="./Spring Cloud Alibaba_images/image-20220717204513601.png" alt="image-20220717204513601" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220717204626532.png" alt="image-20220717204626532" style="zoom:80%;" />



<img src="./Spring Cloud Alibaba_images/image-20220717210054543.png" alt="image-20220717210054543" style="zoom:80%;" />





2. 建立数据库：swtest

<img src="./Spring Cloud Alibaba_images/image-20220717204839704.png" alt="image-20220717204839704" style="zoom:80%;" /> 



3. 将mysql的驱动器jar包放入oap-libs文件中

<img src="./Spring Cloud Alibaba_images/image-20220717205259328.png" alt="image-20220717205259328" style="zoom:80%;" />



4. 关闭服务，再次打开服务，发现数据没有丢失





### 自定义链路追踪

> **注意：SkyWalking链路追踪，默认追踪controller线路，如果想要追踪dao层、service层则需要增加注解**



1. 在对应的微服务导入依赖，需要和版本对应！

```xml
 <!--SkyWalking 工具类：跟服务版本一致-->
        <dependency>
            <groupId>org.apache.skywalking</groupId>
            <artifactId>apm-toolkit-trace</artifactId>
            <version>8.5.0</version>
        </dependency>
```



2. 以service层为例，添加两个方法

```java
@Service
public class StockServiceImpl {

    //加上该注解,可在SkyWalking链路追踪到该方法
    @Trace
    /**
     * 该注解可以添加额外的信息
     *      key: 方法名
     *      value: returnedObj 返回的对象值
     *           : arg[i]     方法参数列表的下标,从0开始
     */
    @Tag(key = "getMsg",value = "returnedObj")
    public String getMsg(){
        return "这是BLZ层获取的信息";
    }

    @Trace
    @Tags({ @Tag(key = "getInfo", value = "returnedObj"),
            @Tag(key = "param",value = "arg[0]"),
            @Tag(key = "param",value = "arg[1]")
            })
    public String getInfo(String name,Integer id){
        return "======"+name+"---"+id;
    }
}
```

==注意：要想添加额外信息，则必须和@Trace注解一起使用==



3. 运行结果

<img src="./Spring Cloud Alibaba_images/image-20220723163327979.png" alt="image-20220723163327979" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220723163344693.png" alt="image-20220723163344693" style="zoom:80%;" />





### 日志

1. 在对应的微服务导入依赖，需要和版本对应！

```xml
 <!-- skywalking 日志记录  -->
        <dependency>
            <groupId>org.apache.skywalking</groupId>
            <artifactId>apm-toolkit-logback-1.x</artifactId>
            <version>8.5.0</version>
        </dependency>
```



2. 在微服务resources文件夹下添加logback-spirng.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 引入 Spring Boot 默认的 logback XML 配置文件  -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>


    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 日志的格式化 -->
        <encoder  class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <Pattern>-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} [%tid] %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}</Pattern>
            </layout>
        </encoder>

    </appender>

    <appender name="grpc-log" class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.log.GRPCLogClientAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.mdc.TraceIdMDCPatternLogbackLayout">
                <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%X{tid}] [%thread] %-5level %logger{36} -%msg%n</Pattern>
            </layout>
        </encoder>
    </appender>

    <!-- 设置 Appender -->
    <root level="INFO">
        <appender-ref ref="console"/>
        <appender-ref ref="grpc-log"/>
    </root>

</configuration>
```



3. 运行结果

<img src="./Spring Cloud Alibaba_images/image-20220723171535713.png" alt="image-20220723171535713" style="zoom:80%;" />



### 告警

1. 以网关服务为例，建立告警实体类

```java
public class SwAlarmDTO {
    private int scopeId;
    private String scope;
    private String name;
    private String id0;
    private String id1;
    private String ruleName;
    private String alarmMessage;
    private List<Tag> tags;
    private long startTime;
    private transient int period;
    private transient boolean onlyAsCondition;

    public static class Tag{
        private String key;
        private String value;
          //省略getter、setter
    }
    
    //省略getter、setter
}    
```



2. 告警控制器：以模拟发送邮件为主，发现了慢接口或异常，可第一时间发送邮件

```java
@RestController
@RequestMapping("/alarm")
public class SwAlarmController {

    Logger log=LoggerFactory.getLogger(this.getClass());

    /**
     * 接收skywalking服务的告警通知并发送至邮箱
     *
     * 必须是post
     */
    @PostMapping("/receive")
    public void receive(@RequestBody List<SwAlarmDTO> alarmList) {
       /* SimpleMailMessage message = new SimpleMailMessage();
        // 发送者邮箱
        message.setFrom(from);
        // 接收者邮箱
        message.setTo(from);
        // 主题
        message.setSubject("告警邮件");
        String content = getContent(alarmList);
        // 邮件内容
        message.setText(content);
        sender.send(message);*/
        String content = getContent(alarmList);
        log.info("告警邮件已发送..."+content);
    }

    private String getContent(List<SwAlarmDTO> alarmList) {
        StringBuilder sb = new StringBuilder();
        for (SwAlarmDTO dto : alarmList) {
            sb.append("scopeId: ").append(dto.getScopeId())
                    .append("\nscope: ").append(dto.getScope())
                    .append("\n目标 Scope 的实体名称: ").append(dto.getName())
                    .append("\nScope 实体的 ID: ").append(dto.getId0())
                    .append("\nid1: ").append(dto.getId1())
                    .append("\n告警规则名称: ").append(dto.getRuleName())
                    .append("\n告警消息内容: ").append(dto.getAlarmMessage())
                    .append("\n告警时间: ").append(dto.getStartTime())
                    .append("\n标签: ").append(dto.getTags())
                    .append("\n\n---------------\n\n");
        }

        return sb.toString();
    }
}
```



3. 在其它微服务模拟慢接口

```java
  	@GetMapping("getInfo")
    public String getInfo() throws Exception{
        TimeUnit.SECONDS.sleep(2);
        return stockService.getInfo("eobard",22);
    }
```



4. 修改`SkyWalking\apache-skywalking-apm-bin-es7\config\alarm-settings.yml`

<img src="./Spring Cloud Alibaba_images/image-20220723180636756.png" alt="image-20220723180636756" style="zoom:80%;" />



5. 运行结果

<img src="./Spring Cloud Alibaba_images/image-20220723180748524.png" alt="image-20220723180748524" style="zoom:80%;" />

<img src="./Spring Cloud Alibaba_images/image-20220723180804983.png" alt="image-20220723180804983" style="zoom:80%;" />


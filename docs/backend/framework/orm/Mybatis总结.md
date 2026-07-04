# Mybatis

## 一.入门

### 		1.1 入门程序

​			a. 导入相关jar包

​			b.在Eclipse 根路径创建db.properties文件

```properties
jdbc.driver= com.mysql.jdbc.Driver
jdbc.url =jdbc:mysql://localhost:3306/mybatis
jdbc.username= root
jdbc.password =123456
```

​			c. 在Eclipse 根路径创建文件Log4j.properties

```properties
log4j.rootLogger=DEBUG,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
log4j.logger.org.apache=INFO
```

​			d. 在Eclipse 根路径创建 mybatis-config.xml配置文件

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
"http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
	<!--  读取外部properties配置文件来动态获取数据库信息  -->
	<properties resource="db.properties" />

		<!--environments里面可以配置多个环境 -->
	<environments default="mysql">
		<environment id="mysql">
			<!-- 使用jdbc的事务 -->
			<transactionManager type="JDBC" />
			<!-- 使用连接池连接数据库 -->
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>		
	</environments>
	
		<!--加载映射文件 -->
	<mappers>
		<mapper resource="mappers/UserMapper.xml"/>
	</mappers>
</configuration>

```

​			e.在相应包下创建User实体<font color="red">(最好是将User实体类和数据库的表属性一致)</font>

```java
public class User {

	private Integer u_id;
	private String u_name;
	private String u_password;
	private Character u_sex;
	private String u_phone;
	//省略getter，setter
}
```



​			f. 在相应包下创建映射文件<font color="red">(最好以实体类名+Mapper命名)</font>

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
        
<mapper namespace="UserMapper">
	<select id="selectUser" resultType="com.eobard.domain.User">
		select * from user
	</select>
</mapper>
        
```

​			g. 测试

```java
public class Test{
    @Test
    public void test(){
        //读取mybatis配置文件
		InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
		//创建会话工厂对象
		SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
		//打开Session
		SqlSession sqlSession = sessionFactory.openSession();
         sqlSession.selectList("UserMapper.selectUser").forEach(System.out::println);
    }
}
```

##### 	

### 1.2 MybatisUtils工具类

```java
public class MybatisUtils {

    //全局工厂
	private static SqlSessionFactory sessionFactory;
	
	//静态代码块加载资源
	static {
		try {
			InputStream in=Resources.getResourceAsStream("mybatis-config.xml");
			sessionFactory=new SqlSessionFactoryBuilder().build(in);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	//获取session
	public static  SqlSession getSession() {
		//true为自动提交事务
		return sessionFactory.openSession(false);
	}
	
	//关闭session
	public static void closeSession(SqlSession session) {
		if(session!=null)
		{
			session.close();
		}
	}
}
```



---



## 二. Mybatis-config相关子标签属性介绍

​			==**注意：以下标签都是在Mybatis-config.xml中的`<configuration>`根标签中设置，且里面的标签有严格的先后顺序**==



### 	2.1 读取外部配置文件

```xml-dtd
	<!--读取外部properties配置文件来动态获取数据库信息  -->
	<properties resource="db.properties" />
```



### 2.2 settings标签

```xml
<settings>	
		<!--取值：NONE：取消自动映射
 				PARTIAL:只会自动映射，没有定义嵌套结果集映射的结果集
				FULL:会自动映射任意复杂的结果集（无论是否嵌套)，如 关联查询时
			-->
		<setting name="autoMappingBehavior" value="FULL"/>
    	<!--
		  如果不手动加上log4j.properties的日志,可以用自带的日志工厂来打印SQL等信息
  	       value: STDOUT_LOGGING 和  LOG4J(需要额外导包+建立log4j.properties文件)
		-->
    	<setting name="logImpl" value="STDOUT_LOGGING"/>
    
    	<!-- 开启延迟加载,默认值是true -->
		<setting name="lazyLoadingEnabled" value="true" />
			<!-- 设置积极懒加载,默认值是true -->
		<setting name="aggressiveLazyLoading" value="false"/>
			<!--指定哪个对象的方法触发一次延迟加载。默认值：equals,clone,hashCode,toString  -->
		<setting name="lazyLoadTriggerMethods" value=""/>
    		<!--默认开启驼峰命名映射
					如果数据库的属性名为stu_name,则在实体类的属性名就可以写为stuName，它会自动映射成驼峰命名
					如果数据库的属性名就位stuname，则需要将这里的true改为false，不然myabtis默认会驼峰命名映射	
			-->
    	<setting  name="mapUnderscoreToCamelCase" value="false"/>
</settings>
```



### 	2.3 类型别名

#### 			2.3.1 类型别名配置

​			==不要设置了某一个别名后又再一次重复设置，配置关联关系的时候最好别用==

```xml
	<!-- 类型别名配置：将resultType里面的类型可以简化,如parameterType="User"不用写全类名
		通过包配置会扫描该包下的所有文件,以对象类名作为别名,不区分大小写,推荐使用小写
	  -->
	<typeAliases>
         <!--单独设置一个实体类,取别名为coun-->	
         <typeAlias type="com.eobard.utils.Country" alias="coun"/>
         <!--批量设置整个domain包下的-->
		<package name="com.eobard.domain"/>
	</typeAliases>
```

​			

#### 		2.3.2 常见别名配置

|    别名    | 映射的类型 |
| :--------: | :--------: |
|   _byte    |    byte    |
|   _long    |    long    |
|   _short   |   short    |
|    _int    |    int     |
|  integer   |  Integer   |
|  _double   |   double   |
|   _float   |   float    |
|  _boolean  |  boolean   |
|   string   |   String   |
|    byte    |    Byte    |
|  boolean   |  Boolean   |
| collection | Collection |
|            |            |



### 2.4 自定义类型转换器

#### 				2.4.1 类型转换器概念

​								Mybatis用类型转换器将java类型的数据自动转化为不同的DB类型，这取决于Mybatis底层自带的类型转换器，**正因为存在类型转换器，Mybatis可以将不同类型的数据根据底层数据库的不同而插入不同类型的值**



#### 				2.4.2 自定义简单类型转换器

​								<font color="green">需求：根据用户的布尔值来设置用户的性别</font>

```
数据库设计：
				id  主键 
				sex bit类型1位 				
```

```java
//实体类
public class User{
	private int id;
	private boolean sex;
    //省略getter，setter
}
```

```JAVA
//UserMapper
public interface UserMapper {
	List<User> findAll();
	void insertOne(User u);
}
```

```xml-dtd
<!--UserMapper.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mapper.UserMapper">

	<select id="findAll" resultType="com.eobard.domain.User">
		select * from user
	</select>

	<insert id="insertOne" parameterType="com.eobard.domain.User">
	 	insert into user(sex) values(#{sex})
	</insert>
</mapper>
```

```java
/**
 *  将java类型boolean值转化为DB的bit的类型处理器
 *  false：0
 *  true：1
 */
public class Boolean2BitTypeHandler  extends BaseTypeHandler<Boolean>{

	//DB类型->java类型
	@Override
	public Boolean getNullableResult(ResultSet rs, String colunmName) throws SQLException {
		 byte b = rs.getByte(colunmName);
		 return b==0?false:true;
	}
	
	//DB类型->java类型
	@Override
	public Boolean getNullableResult(ResultSet rs, int colunmnIndex) throws SQLException {
		 byte b = rs.getByte(colunmnIndex);
		 return b==0?false:true;
	}

	//DB类型->java类型
	@Override
	public Boolean getNullableResult(CallableStatement cs, int colunmnIndex) throws SQLException {
		 byte b = cs.getByte(colunmnIndex);
		 return b==0?false:true;
	}

	//java类型->DB类型
	@Override
	public void setNonNullParameter(PreparedStatement ps, int index, Boolean value, JdbcType type) throws SQLException {
		if(value)
		{
			ps.setByte(index, (byte) 1);
		}
		else {
			ps.setByte(index, (byte) 0);
		}
	}

}
```

```xml-dtd
<!--mybatis-config.xml文件配置-->
<typeHandlers>
		<typeHandler handler="com.eobard.convert.Boolean2BitTypeHandler" javaType="Boolean" 							jdbcType="BIT"/>
</typeHandlers>
```



#### 				2.4.3 自定义枚举类型转换器(常用)

​										<font color="green">需求：根据用户枚举来设置用户的性别</font>

```
数据库设计：
				id  主键 
				gender bit类型1位 	
```

```JAVA
//枚举类
public enum Gender {
	MAN, WEMEN;
    
	public static Gender getUserGender(byte code) {
		switch (code) {
		case 0:
			return MAN;
		case 1:
			return WEMEN;
		default:
			return MAN;
		}
	}

}

```

```JAVA
//实体类
public class User {
	private int id;
	private com.eobard.domain.Gender gender;
	//省略getter，setter
}
```

```JAVA
//UserMapper
public interface UserMapper {
	List<User> findAll();
	void insertOne(User u);
}
```

```xml-dtd
<!--UserMapper.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mapper.UserMapper">

	<select id="findAll" resultType="com.eobard.domain.User">
		select * from user
	</select>

	<insert id="insertOne" parameterType="com.eobard.domain.User">
	 	insert into user(sex) values(#{gender})
	</insert>
</mapper>
```

```java
/**
 *  将java类型枚举值转化为DB的bit的类型处理器
 *  MAN：0
 *  WEMAN：1
 */
public class Enum2BitTypeHandler extends BaseTypeHandler<Gender>{

	//DB类型->java类型
		@Override
		public Gender getNullableResult(ResultSet rs, String colunmName) throws SQLException {
			 byte b = rs.getByte(colunmName);//0,1
			 return  Gender.getUserGender(b);
		}
		
		//DB类型->java类型
		@Override
		public Gender getNullableResult(ResultSet rs, int colunmnIndex) throws SQLException {
			 byte b = rs.getByte(colunmnIndex);
			 return  Gender.getUserGender(b);
		}

		//DB类型->java类型
		@Override
		public Gender getNullableResult(CallableStatement cs, int colunmnIndex) throws SQLException {
			 byte b = cs.getByte(colunmnIndex);
			 return  Gender.getUserGender(b);
		}

		//java类型->DB类型
		@Override
		public void setNonNullParameter(PreparedStatement ps, int index, Gender value, JdbcType type) throws SQLException {
			if(value==Gender.WEMEN)
			{
				ps.setByte(index, (byte) 1);
			}
			else {
				ps.setByte(index, (byte) 0);
			}
		}

}
```

```xml-dtd
<!--mybatis-config.xml文件配置-->
<typeHandlers>
			<typeHandler handler="com.eobard.convert.Enum2BitTypeHandler" 				               					javaType="com.eobard.domain.Gender" jdbcType="BIT"/>
</typeHandlers>
```



==注意：使用类型转换器一定要继承`BaseTypeHandler<T>`类并重写方法，然后在mybatis-config.xml文件去配置即可==



### 2.5 导入相关插件

```xml
	<plugins>
			<!-- 5.0版本之前为PageHelper，5.0版本以后为PageInterceptor  -->
		<plugin interceptor="com.github.pagehelper.PageInterceptor">
				<!--
					reasonable:分页合理化参数，默认为false，根据参数查询	
							为true时   当前页码<0  查询第一页，当前页码>总页码    查询最后一页
				  -->
			<property name="reasonable" value="trues"/>
		</plugin>
        
        <!--省略其他插件-->
	</plugins>
```





### 	2.6 环境配置和切换

```xml
		<!--environments里面可以配置多个环境，通过default="id名"来指定切换不同环境  -->
	<environments default="mysql">
		<environment id="mysql">
			<!-- 使用jdbc的事务 -->
			<transactionManager type="JDBC" />
			<!-- 使用连接池连接数据库 -->
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>
		
		<environment id="oracle">
			<!-- 使用jdbc的事务 -->
			<transactionManager type="JDBC" />
			<!-- 使用连接池连接数据库 -->
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}"/>
				<property name="url" value="${jdbc.url}"/>
				<property name="username" value="${jdbc.username}"/>
				<property name="password" value="${jdbc.password}"/>
			</dataSource>
		</environment>
		
	</environments>
```



### 	2.7 加载映射文件

```xml
	<mappers>
		<!-- 找到映射文件 三种常用方式
			1.直接使用resource找到(Dao式开发)
			
  			2.加载注解实现CRUD;  如果是使用了mapper动态代理的形式,必须保证mapper.xml和接口要在同一个包下
			然后使用class="接口限定名"即可
			
			3.★★★(推荐使用) 通过package name="包名"即可,必须保证mapper.xml和接口要在同一个包下 
						 	且是mapper动态代理形式
		-->
		<mapper resource="mappers/UserMapper.xml"/>
		<mapper class="com.eobard.mapper.UserMapper"/> 
		<package name="com.eobard.mapper"/>
	</mappers>
```



---



## 三. Mapper映射文件相关属性介绍

​			**<font color="red">注意：在映射文件里面的sql语句最好不要出现查询相同的属性，可能会出现查询结果不完整的情况，如：关联查询一对多，多对多时，两个表都有主键叫id，最好把其中一个id取一个别名</font>**



### 	3.1 根标签namespace属性

​				namespace: 命名空间，在mapper.xml中的根标签`<mapper>`中设置,一个映射文件可能有多个mapper用这个来区分，唯一标识

### 	3.2 CRUD标签的id属性

​				id: 表示标签的名称，唯一标识

​		==小tips：如果是Dao式开发，则直接用 mapper根标签命名空间+CRUD标签id 获取对应的SQL语句==



### 	3.3 CRUD标签的parameterType属性

​			parameterType：表示参数的类型，可以省略不写，可以自动匹配；<font color="red">如果是类类型则要写完整的类限定名</font>；<font color="green">如果用了类型别名，则直接写类名即可</font>



### 	3.4 CRUD标签的resultType属性

​			resultType：表示需要返回数据的类型，<font color="red">如果是类类型则要写完整的类限定名</font>；<font color="green">如果用了类型别名，则直接写类名即可</font>

​			==要求：实体类的每个属性名必须与数据库表列名一致！==



### 	3.5 CRUD标签的resultMap属性

​		    resultMap：<font color="red">当实体类的属性和数据库表的列不一致时，需要手动映射(如一对多，多对多关联查询)</font>, property为JavaBean的属性名,通过column手动指定数据库的字段名，type就是返回值的类型(这里使用了类型别名),  id为标识，不能重复

```java
//实体类属性与数据库表列名不一致
public class country{
    private Integer id;
	private String name;

    //省略getter，settter
}
```

```xml
<!--映射文件-->
<mapper namespace="CountryMapper">

	<!--resultMap是手动映射
				id不能重复，用于区分不同resultMap，标识出resultMap引用哪个手动映射
				property为JavaBean的属性名
				column手动指定数据库的字段名
		  		type就是返回值的类型(这里使用了类型别名)
	  -->

    	<!--定义id为c的resultMap手动映射-->
	<resultMap type="com.eobard.domain.Country" id="c">
        	<!--定义主键属性-->
		<id  property="id" column="c_id"/>
        	<!--定义其它属性-->
		<result property="name" column="c_name"/>
	</resultMap>
    
    	<!--引用id为c的resultMap-->
	<select id="selectAll"  resultMap="c">
		select * from country 
	</select>
</mapper>
```

​		==resultMap和resultType的区别(不能同时出现)==

​						1.resultType是自动映射,数据库的字段与JavaBean的字段要一一对应

​						2.resultMap是手动映射,适用于关联查询或数据库表与实体类属性不一致



​			两点注意：<font color="red">1.resultMap标签里面的配置默认会匹配 type类下的属性名和数据库的列名，假设实体类只有一两个属性名与数据库列名不一致，就只需要配置不一样的即可(不用配置全部)，resultMap会自动映射其它相同属性名</font>

​								<font color="red">2. 在关联关系查询时，可以通过extends 来实现resultMap标签的映射完整性</font>

```xml
	<resultMap type="com.eobard.domain.XXX" id="Base">
		<!--基类映射-->
	</resultMap>

	<resultMap type="com.eobard.domain.XXX" id="other" extends="Base">
		<!--扩展映射：如关联查询-->
	</resultMap>
```



### 3.6 CRUD标签的useGeneratedKeys属性

​				对于支持自动生成记录主键的数据库(mysql和sqlserver)，此时**设置useGeneratedKeys参数值为true**，在执行添加记录之后可以获取到数据库自动生成的主键ID。**需要配合keyProperty属性同时使用**

```xml-dtd
	<!--
		useGeneratedKeys:是否使用自动生成主键
		keyProperty:通常指定为该表主键对应的实体类属性名，即User实体类的主键属性 u_id
		-->
	<insert id="addUser" useGeneratedKeys="true" keyProperty="u_id" parameterType="com.eobard.domain.User"> 
		insert into user(u_name,u_password,u_sex,u_phone) values(#{u_name},#{u_password},#{u_sex},#{u_phone})
	</insert>
```

```JAVA
@Test
	public void test3() {
			User user=new User();
			user.setU_name("自增");
			user.setU_password("000");
			user.setU_phone("1232");
			user.setU_sex('男');
        	//没有主键
			System.out.println(user);
			userMapper1.addUser(user);
			session1.commit();
        	//在提交事务后，会将自动生成的主键返回
			System.out.println(user);
			
	}
```

```
User [u_id=null, u_name=自增, u_password=000, u_sex=男, u_phone=1232, c_id=null]
[main] DEBUG [com.eobard.cache.UserMapper.addUser] - ==>  Preparing: insert into user(u_name,u_password,u_sex,u_phone) values(?,?,?,?) 
[main] DEBUG [com.eobard.cache.UserMapper.addUser] - ==> Parameters: 自增(String), 000(String), 男(String), 1232(String)
[main] DEBUG [com.eobard.cache.UserMapper.addUser] - <==    Updates: 1
User [u_id=14, u_name=自增, u_password=000, u_sex=男, u_phone=1232, c_id=null]
```



​	==注意：useGeneratedKeys设置为 true 时，表示如果插入的表id以自增列为主键，则允许 JDBC 支持自动生成主键，并可将自动生成的主键id返回。**useGeneratedKeys参数只针对 insert 语句生效，默认为 false；**==



### 	3.7 SQL语句中的#{}

​				占位符,通过Ognl表达式来让sql语句参数化     用 #{任意参数名}来代替 ;<font color="red">如果参数是类类型,则#{类里面的属性名}</font>，输入的参数若为String，会自动在sql语句上加上引号

​			

### 	3.8 SQL语句中的${}

​				%${value}%：字符串的拼接(如模糊查询), 只能用这种方式来书写；不安全且容易被SQL注入，**输入什么参数，会直接显示在sql语句中，不会自动加上引号，需要手动加**



​			==#{}和${}的区别==

```sql
select * from user where u_name like "%"#{name}"%" 
select * from user where u_name like concat('%',#{name},'%')    1和2都是 #占位符方式(推荐使用)
select * from user where u_name like '%${value}%'   #字符串拼接
```

​							1. #{}是预编译处理,调用PreparedStatement的set方法来赋值；<font color="red">尽量使用占位符</font>

​							 2.${}是字符串替换,存在sql注入的风险,传入数据直接显示在生成的sql中

​							3. **${value} 适合于根据动态列名升序/降序排列的查询语句**









---



## 四. Mybatis相关API介绍

|           对象           |             作用              |         生命周期         |
| :----------------------: | :---------------------------: | :----------------------: |
| SqlSessionFactoryBuilder | 用于创建SqlSessionFactory对象 |         局部变量         |
|    SqlSessionFactory     |    用于创建SqlSession对象     |       **全局变量**       |
|        SqlSession        |           执行CRUD            | **局部变量(线程不安全)** |





### 		4.1 SqlSession的selectOne

​			用于查询Mapper映射文件中select标签中sql语句返回单个值的结果(如查询聚合函数)，<font color="red">不能用于查询sql语句返回多个值的情况</font>

```xml
	<!--省略其它标签-->
	<select id="selectUserCount" resultType="integer">
		select count(1) from user
	</select>
```

```java
	//省略获取sqlSession
	int count=sqlSession.selectList("UserMapper.selectUserCount");
```



### 		4.2 SqlSession的selectList

​			用于查询Mapper映射文件中select标签中sql语句返回多个值的结果。

```xml
	<!--省略其它标签-->
	<select id="selectUser" resultType="com.eobard.domain.User">
		select * from user
	</select>
```

```java
	//省略获取sqlSession
	sqlSession.selectList("UserMapper.selectUser").forEach(System.out::println);
```



### 4.3 SqlSession的insert

​			用于执行Mapper映射文件中insert标签的sql语句

```xml
	<!--省略其它标签-->
	<insert id="insertUser" parameterType="com.eobard.domain.User">
		insert into user values(null,#{u_name},#{u_password},#{u_sex},#{u_phone})
	</insert>
```

```java
	//省略获取sqlSession
	sqlSession.insert("UserMapper.insertUser",u);
	sqlSession.commit();
```



### 4.4 SqlSession的update

​			用于执行Mapper映射文件中insert标签的sql语句

```xml
	<!--省略其它标签-->
	<update id="updateUser" parameterType="com.eobard.domain.User">
		update user set u_name=#{u_name},u_password=#{u_password},u_sex=#{u_sex},u_phone=#{u_phone} where u_id=#{u_id}
	</update>
```

```java
	//省略获取sqlSession
	sqlSession.insert("UserMapper.updateUser",user);
	sqlSession.commit();
```



### 4.5 SqlSession的delete

​			用于执行Mapper映射文件中delete标签的sql语句

```xml
	<!--省略其它标签-->
	<delete id="deleteUserById" parameterType="Integer">
		delete from user where u_id=#{id}
	</delete>
```

```java
	//省略获取sqlSession
	sqlSession.insert("UserMapper.deleteUserById",1);
	sqlSession.commit();
```



### 4.6 SqlSession的commit&rollback

​			commit: 用于提交增删改操作的事务，不提交事务会导致更新不到数据库的数据

​			rollback: 用于回滚事务



### 4.7 SqlSession的getMapper

​			用于获取Mapper动态代理的接口

```java
		//省略获取SqlSession的操作
		T tMapper = sqlSession.getMapper(T.class);
```



### 4.8  SqlSession的批量操作

​			mybatis的Executor是SQL语句的执行器，它默认是simple类型的；如果要想使用批量的大数据的增删改，则需要Batch类型的执行器

```JAVA
		InputStream in = Resources.getResourceAsStream("mybatis-config.xml");
		 SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(in);
		 //这里指定为ExecutorType.BATCH
		 SqlSession session = sessionFactory.openSession(ExecutorType.BATCH);
		 BatchTestMapper mapper = session.getMapper(BatchTestMapper.class);
		 long start = System.currentTimeMillis();
		 for(int i=0;i<100000;i++) {
			 mapper.addBatch(new com.eobard.domain.BatchTest(UUID.randomUUID().toString().substring(0,9)));
		 }
		 long end = System.currentTimeMillis();
		 System.out.println(end-start);
		 session.commit();
		 MybatisUtils.closeSession(session);
```

通过日志可知  **Batch：预编译SQL一次，大量的增删改操作只会给你参数值；**

​						 **Simple：预编译SQL N次，大量的增删改操作会执行**

==注意：如果通过simple类型的执行器，大量的增删改用的时间会很久；但是如果是用Batch类型的执行器，大量的增删改用的时间会很快==



---



## 五. Mybatis 多参数入参

​					<font color="red">在MyBatis中,多个参数入参是不允许的，也就是说方法中只能传入一个参数</font>，在实际应用中,方法的参数(参数个
数、参数数据类型)会有各式各样的选择,根据不同的业务需要选择相应的入参方式。

​					实现多参数入参常用的方式有3种，分别是:  @Param注解入参、 类对象入参， map集合入参 。

```
//如果传入了多个参数，会引发以下错误
Parameter 'id' not found. Available parameters are [arg1, arg0, param1, param2]
```



### 5.1 @Param注解(重点)

```java
public interface UserMapp{
    List<User> selectUserByMoreCondtions(@Param("sex")String sex,@Param("id")int id);
}
```

```xml
<!--UserMapper.xml文件中-->
	<select id="selectUserByMoreCondtions" resultType="user">
		select * from user where u_id >#{id} and u_sex like concat("%",#{sex},"%")
	</select>
```

​	==一旦方法参数加入了@param注解，那么映射文件中sql语句的Ognl的名称(占位符)就必须跟接口方法中@param的名称一致==



### 5.2 类对象入参

​		<font color="orange"> 简单类对象</font>

```java
public interface UserMapp{
    List<User> selectUserByMoreCondtions(User user);
}
```

```xml
<!--UserMapper.xml文件中-->
	<select id="selectUserByMoreCondtions" resultType="user">
		select * from user where u_id >#{u_id} and u_sex like concat("%",#{u_sex},"%")
	</select>
```

​	<font color="orange"> 注解+类对象</font>

```java
public interface UserMapp{
    List<User> selectUserByMoreCondtions(@Param("u")User user);
}
```

```xml
<!--UserMapper.xml文件中-->
<select id="selectUserByMoreCondtions" resultType="user">
		select * from user where u_id >#{u.u_id} and u_sex like concat("%",#{u.u_sex},"%")
	</select>
```

​	==1.使用了类类型作为参数，一定要记住映射文件中sql语句的Ognl的名称(占位符)就必须跟User实体类的属性名称一致==

​    ==2.如果是使用了注解+类对象，则sql语句的Ognl的名称(占位符)就必须为@Param的名称 . User实体类的属性名==



### 5.3 map集合入参(重点)

​			<font color="red">**当实体类的参数很多的时候，我们就可以不用实体类作为参数，反而用Map集合作为参数，这样就可以不用new 实体类来设置各个属性值了**</font>

```java
public interface UserMapp{
    List<User> selectUserByMoreCondtions(Map<String,Object> map);
}
```

```xml
<!--UserMapper.xml文件中-->
<select id="selectUserByMoreCondtions" resultType="user">
		select * from user where u_id >#{id} and u_sex like concat("%",#{sex},"%")
	</select>
```

```java
//测试
	@Test
	public void test() {
		SqlSession session2 = MybatisUtils.getSession();
		UserMapper userMapper = session2.getMapper(UserMapper.class);
		Map<String, Object> map=new HashMap<String, Object>();
		map.put("sex", '男');
		map.put("id", 2);
		userMapper.selectUserByMap(map).forEach(System.out::println);
	}
```

​	==注意：使用了Map作为参数，一定要记住映射文件中sql语句的Ognl的名称(占位符)就必须跟Map的 K名称一致==



### 5.4 类类型+普通类型入参

```JAVA
public interface UserMapp{
    List<User> selectUserByMoreCondtions(@Param("id")int id,@Param("user")User u);
}
```

```xml-dtd
<!--UserMapper.xml文件中-->
<select id="selectUserByMoreCondtions" resultType="user">
		select * from user where u_id >#{id} and u_sex like concat("%",#{user.sex},"%")
</select>
```

==注意：如果是普通类型+实体类类型入参，则同样需要@Param注解，然后**在sql映射文件中拿实体类的属性值直接通过@Param里面的注解名.实体类对应的属性名即可**==



---



## 六.  Mapper动态代理开发

### 	6.1 Mapper动态代理五大原则+1个注意

​				<font color="red">a*</font>. Mapper.xml映射文件的namespace与对应接口限定名一致<font color="red">且最好处于同一个包下</font>

​				<font color="red">b*</font>. 接口方法名与Mapper.xml要调用的方法名的id一致

​				<font color="red">c*</font>. <font color="purple">接口里面的方法不能出现重载</font>

​				d. 接口的形参类型要与 Mapper.xml的parameterType一致

​				e. 接口的返回类型要与 Mapper.xml的resultType一致

​				f. <font color="green">Mapper动态代理会根据返回类型动态的选择selectList或者selectOne,最好可以让接口名与Mapper名一致</font>

### 	6.2 Mapper实现动态代理

```java
//UserMapper接口：相当于是dao层,以前接口为UserDao；mybatis最好定义为UserMapper
public interface UserMapper {

	User selectUserById(Integer id);

	List<User> selectUserByPhone(String phone);

	void insertUser(User user);

	void updateUser(User user);
}
```

```xml-dtd
<!--UserMapper.xml文件-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mapper.UserMapper">

	<select id="selectUserById" parameterType="Integer" resultType="user">
		select * from user where u_id=#{id}
	</select>
	
	<select id="selectUserByPhone" parameterType="String" resultType="user">
		select * from user where u_phone like "%"#{phone}"%"
	</select>
	
    <insert id="insertUser" parameterType="user">
		insert into user values(null,#{u_name},#{u_password},#{u_sex},#{u_phone})
	</insert>
	
	<update id="updateUser" parameterType="user">
		update user set u_name=#{u_name},u_password=#{u_password},u_sex=#{u_sex},u_phone=#{u_phone} 				where u_id=#{u_id}
	</update>
</mapper>
```



```xml
<!--mybatis-config导入映射文件-->
	<mappers>
		<!-- 找到mapper动态代理映射文件
			1.加载单个mapper动态代理：必须保证mapper.xml和接口要在同一个包下，然后使用class="接口限定名"即可
			
			2.加载多个mapper动态代理(★★★推荐使用)： 通过package name="包名"即可,必须保证mapper.xml和接口要在同一个包下且是mapper动态代理形式
		-->
		
		<mapper class="com.eobard.mapper.UserMapper"/> 
		<package name="com.eobard.mapper"/>
	</mappers>

```



```java
//测试Mapper动态代理开发
public class Test{
    @Test
    public void test(){
        //读取mybatis配置文件
		InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
		//创建会话工厂对象
		SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
		//打开Session
		SqlSession sqlSession = sessionFactory.openSession();
         //获取mapper动态代理接口
         UserMapper userMapper = session.getMapper(UserMapper.class);
         //调用任意一个方法
         User user=userMapper.selectUserById(1);
    }
}
```



---



## 七.动态SQL

### 7.1 where标签

​			where标签(可以去掉开头的and 或者 or)：如果里面的都不成立,则sql语句不会生成where

```xml
	<select id="selectUserByMoreCondition" resultType="User" parameterType="com.eobard.domain.User">
		select * from user 
		<where>
			<if test="u_name!=null and u_name!=''">
				and u_name like "%"#{u_name}"%"
			</if>
			<if test="u_sex!=null and u_sex!=''">
				and  u_sex=#{u_sex}
			</if>
			<if test="c_id!=null and c_id!=''">
				and c_id>=#{c_id}
			</if>
		</where>
	</select>
```



### 7.2 if标签

​			if标签：如果if标签中的test判断Ognl表达式的值成立则会让sql语句追加里面的查询,反之不会

```xml
	<delete id="deleteUser" parameterType="com.eobard.domain.User">
		delete from user where  1=1   
        		<if test="u_id!=null and u_id!=''">
					and	 u_id=#{u_id},
				</if>	
	</delete>
```

​	==注意：只有字符串类型的才会多加一句判断  属性名!=' '，其他类型只需要加上 属性名 !=null即可；所有判断一定要先判断为null，再判断为' '==

​				==如果传入的是类类型的参数，只需要判断属性就可以了==

### 7.3 set标签

​			set标签(可以去掉后面的 ,):可以让里面的if标签成立中的语句结尾的逗号省略

```xml
	<update id="updateUserByDynamicSQL" parameterType="com.eobard.domain.User" >
			update user
			<set>
					<if test="u_name!=null and u_name!=''">
						u_name=#{u_name},
					</if>
					<if test="u_password!=null and u_password!=''"> 
						u_password=#{u_password},
					</if>
					<if test="u_sex!=null and u_sex!=''">
						 u_sex=#{u_sex},
					</if>
				
			</set>
			where u_id=#{u_id}
	</update>
```



### 7.4 foreach标签

​			foreach标签(当查询语句有in的情况下): 遍历<font color="green">传入集合</font>或 <font color="green">数组数据</font> 或<font color="green">包装类的集合或数组属性名(需提供getter，setter)</font>

​							collection属性：array/ list/工具类集合或数组的属性名

​							separator：分隔符，中间以什么分隔开这些数据

​							item：表示循环的数据别名，可通过ognl表达式获取

​							open：循环之前需要加上的字符

​							close：循环结束后要加上的字符



#### 			7.4.1 传入数组

```java
//通过mapper动态代理形式
public interface UserMapper{
    List<User> selectUserByArrayIds(int[] ids);
}
```

```xml-dtd
<!--UserMapper.xml文件中-->
<select id="selectUserByArrayIds" resultType="User" parameterType="Integer">
	select * from user where u_id in 
		<foreach collection="array" separator="," item="id" open="(" close=")">
			#{id}
		</foreach>
</select>
```



#### 			7.4.2 传入集合

```java
//通过mapper动态代理形式
public interface UserMapper{
    List<User> selectUserByListIds(List<Integer> ids);
}
```

```xml-dtd
<!--UserMapper.xml文件中-->
<select id="selectUserByListIds" resultType="User" parameterType="Integer">
	select * from user where u_id in 
		<foreach collection="list" separator="," item="id" open="(" close=")">
			#{id}
		</foreach>
</select>
	
```

==注意：如果需要加上if标签判断是否有值，if里面的ognl表达式判断的就是array/list这两个，并不是接口中的形参名！！==



#### 			7.4.3 传入工具类数组

```java
//UserVo工具类
public class UserVo {

	private int[] ids;

	public int[] getIds() {
		return ids;
	}
	public void setIds(int[] ids) {
		this.ids = ids;
	}	
}

```

```xml-dtd
<!--UserMapper.xml文件中-->	
<select id="selectUserByVoIds" resultType="User" parameterType="com.eobard.domain.UserVo" >
	select * from user where u_id in 
		<foreach collection="ids" separator="," item="id" open="(" close=")">
			#{id}
		</foreach>
</select>
```



### 7.5 sql标签和include标签

​			sql标签：可以提取公共重复的sql语句片段使之重用，id属性标识符，不可重复

​			include标签：通过refid="sql标签的id名称"导入公共的sql标签

```java
//通过mapper动态代理形式
public interface UserMapper{
    List<User> findAll();
}
```

```xml
<!--UserMapper.xml文件中-->
  	<sql id="select*">
	  	select * from user
	  </sql>

<select id="findAll" resultType="User">
	<include refid="select*" />
</select>


```

==注意：如果当前Mapper.xml文件想要使用另外Mapper.xml文件中的SQL标签，则使用方式：**另外Mapper.xml文件的namespace.SQL标签的id即可**==



### 7.6 注意事项

​	在写动态SQL标签的时候，如果出现了">="这种判断符号，需要转义，否则在xml文件中识别不了

> <! [CDATA[   数据库列名>=20   ]]>

---



## 八 . Mybatis Generator使用

### 8.1 Mybatis Generator上手

​				a.  首先将相应MBG的jar包导入项目中

​				b.  在项目Eclipse 根路径创建generatorConfig.xml配置文件

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
     
    <context id="MyGenerator" targetRuntime="MyBatis3">
    	
    	<!--这个标签可以去掉生成文件的注释  -->
    	<commentGenerator>
    		<!-- 去掉注释 -->
				<property value="true" name="suppressAllComments"/>
			<!-- 去掉时间戳 -->
				<property value="true" name="suppressDate"/>
    	</commentGenerator>
    
        <!-- 数据库连接信息：jdbc配置 -->
        <jdbcConnection
                driverClass="com.mysql.jdbc.Driver"
                connectionURL="jdbc:mysql://localhost:3306/mybatis?useUnicode=true&amp;characterEncoding=utf-8"
                userId="root"
                password="123456">
        </jdbcConnection>

		<!--java jdbc之间的类型转换   -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- javaBean的生成
        			targetPackage：javabean的输出路径在哪个包下
        			targetProject:输出项目的位置
         -->
        <javaModelGenerator targetPackage="com.eobard.domain"
                            targetProject="src">
               <!-- 是否开启子包：即在该包下又加上一个schema的子包 -->
            <property name="enableSubPackages" value="false" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        
        
        <!-- xml映射文件的生成 -->
        <sqlMapGenerator targetPackage="com.eobard.mapper"
                         targetProject="src">
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>
        
        
        <!-- mapper接口的生成 -->
        <javaClientGenerator type="XMLMAPPER"  targetPackage="com.eobard.mapper" targetProject="src">
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>
        
        
        <!-- 指定逆向分析哪些表，根据表创建JavaBean
        		tableName:数据库的表名
        		domainObjectName:表对应的类名。若不指定，则默认为首字母大写表名
        -->
        <table tableName="user" domainObjectName="User"></table>
        <table tableName="country" domainObjectName="Country"></table>
    </context>
</generatorConfiguration>
```



​				c. 创建自动生成文件的类并运行，<font color="red">只能运行一次，否则会追加</font>

```java
public class Generator {

	public static void main(String[] args) throws Exception {	
		List<String> warnings = new ArrayList<String>();
		boolean overwrite = true;
		File configFile = new File("src/generatorConfig.xml");
		ConfigurationParser cp = new ConfigurationParser(warnings);
		Configuration config = cp.parseConfiguration(configFile);
		DefaultShellCallback callback = new DefaultShellCallback(overwrite);
		MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
		myBatisGenerator.generate(null);
	}
}

```





### 8.2 Mybatis Generator 使用&注意事项

#### 			8.2.1 使用

​						Mapper.xml生成的方法

​								a.	xxxByExample:根据复杂条件来CRUD

​								b.	xxxByPrimaryKey：根据主键来CRUD

​								c.	xxxSelective：属性有选择性的CRUD(如增加时有些属性不想赋值)

​								d.	xxxByExampleSelective：根据复杂条件, 属性有选择性的CRUD

​								e.	xxxByPrimaryKeySelective：根据主键, 属性有选择性的CRUD



```java
public class TestMBG{
    private InputStream in=null;
	private SqlSession session=null;
	
	@Before
	public void init() throws Exception {
		 	in = Resources.getResourceAsStream("mybatis-config.xml");
			session=new SqlSessionFactoryBuilder()
												.build(in)
												.openSession();
	}
    
    @Test
	public void test() {
		UserMapper userMapper = session.getMapper(UserMapper.class);
        //通过主键查询
//		User user = userMapper.selectByPrimaryKey(1);
//		System.out.println(user);

//		UserExample example=new UserExample();
        //通过复杂条件查询
//		example.createCriteria().andUIdGreaterThanOrEqualTo(1);
//		userMapper.selectByExample(example).forEach(System.out::println);
		
		UserExample example=new UserExample();
		example.createCriteria().andUSexEqualTo("男");
		long count = userMapper.countByExample(example);
		System.out.println(count);
	}
    
}
```





#### 			8.2.2 注意事项

​						1. 如果已经通过MBG生成，再次生成会继续添加在原有基础上且会报错

>  Result Maps collection already contains value for xxxMapper.BaseResultMap
>
> <font color="red">只需要将生成的文件删除再次生成1次即可！</font>

​						2. 生成的javaBean有两个：一个是对应数据库的javaBean,一个是对该javaBean进行各种条件封装的javaBean	

* XXX.java   实体类

* XXXExample.java	 封装该实体各种查询条件的类

  ​					

  ​					3. 复杂条件指的是动态生成的XXXExample类						

  ```java
  			XXXExample example=new XXXExample();
  		 	example.createCriteria().andXXXX()  //Criteria支持链式编程
  ```

  ​	
  
  ​				   4. MBG的模糊查询是通过${value}的方式，所以用模糊查询的时候**需要自己手动加上  %值% **

---



## 九.关联查询

​			**<font color="red">在单表查询时，resultMap可以不用映射实体类属性名与数据库列名相同的，只需要映射不同的即可；但是在关联查询时，<font color="black">主表</font>查询什么就必须映射什么，否则查询的结果为null</font>**



### 9.0 Vo类关联其它类属性(重点)

​		需求：在查询当前表的数据还要获取其它表的某几个字段，这时候就可以借助Vo类来关联其它实体类的属性了

```java
public class UserVo extends User{
    private int classId;		//其它类的属性1
    private String className;	//其它类的属性2
}
```



### 9.1一对一关联查询

​				association标签

```xml
//配置一对一关联标签
<association property="当前实体引用另一个实体类的属性名" javaType="另一个实体类的类型(类的限定名)">
	//配置另一个类的主键
	<id property="" column="" />
	//配置另一个类的其它属性
	<result property="" column=""/>
</association>
```

```
数据库设计：
			identity： id 主键
					  user_id  外键
					  identity_name
					  
			user：   u_id主键
					u_name
					u_sex
```

​		<font color="green">用户与身份证的一对一：</font>

```java
//该实体类与user表属性一致
public class User {
	private Integer u_id;
	private String u_name;
	private String u_password;
	private Character u_sex;
	//省略getter，setter
}

//该实体类与identity表除主键之外属性不一致
public class Identity{
    private Integer id;
	private Integer userId;
	private String identityName;
    
    //配置一对一
	private User user;
    //省略getter，setter
}
```

```java
//IdentityMapper接口
public interface IdentityMapper {
	List<Identity> getIdentityList();
}
```

```xml-dtd
<!--IdentityMapper.xml映射文件 -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mapper.IdentityMapper">

	<!--手动映射 Identity全部属性 -->
	<resultMap type="com.eobard.domain.Identity" id="BaseIdentity">
		<id column="id" property="id"/>
		<result column="user_id" property="userId"/>
		<result column="identity_name" property="identityName"/>
	</resultMap>
	
	<!--配置一对一关联映射，继承手动映射 Identity的resultMap  -->
	<resultMap type="com.eobard.domain.Identity" id="userIdentity" extends="BaseIdentity">
		<association property="user" javaType="com.eobard.domain.User">
			<!--根据mybatis-config.xml中的是否修改autoMappingBehavior配置,默认需要手动配置！ -->
			<id column="u_id" property="u_id"/>
			<result column="u_name" property="u_name"/>
			<result column="u_password" property="u_password"/>
			<result column="u_sex" property="u_sex"/>
		</association>
	</resultMap>
	
	<select id="getIdentityList" resultMap="userIdentity">
		select i.* , u.u_id,u.u_name,u.u_password,u.u_sex from identity i INNER join user u on i.user_id=u.u_id
	</select>
	
</mapper> 

```

```
//结果
Identity [id=1, userId=1, identityName=1215454565, user=User [u_id=1, u_name=九龙坡陈冠希, u_password=654321, u_sex=男]]

Identity [id=2, userId=2, identityName=465789725, user=User [u_id=2, u_name=李四, u_password=654321, u_sex=男]

Identity [id=3, userId=3, identityName=8798325654, user=User [u_id=3, u_name=王五, u_password=123456, u_sex=男]
```



​		==注意事项：==

```xml
<!--mybatis-config.xml配置文件中-->
<settings>	
	<!-- <setting name="autoMappingBehavior" value="PARTIAL"/> -->
	<setting name="autoMappingBehavior" value="FULL"/>
</settings>
```

>  	   	   如果用默认的PARTIAL：则关联级别需要手动配置所查的其它属性，<font color="red">否则会出现关联的实体类属性为空</font>
>  	   		如果是是FULL:则关联级别不需要手动配置其它属性，会自动映射并查找与其它实体类属性名相同的数据库列名

**一对一关联的时候返回接收数据的类最好是引用了另外一个实体类的类，即这里的Identity类**



### 9.2 一对多关联查询	

​				collection标签

```
// 配置一对多标签	 
 <collection property="一方引用多方集合的属性名" ofType="多方实体类的类限定名">
				//配置另一个类的主键
			<id property="" column="" />
				//配置另一个类的其它属性
			<result property="" column=""/>
</collection>
```



​	<font color="green">城市与人的一对多</font>

```
数据库设计：
			country：c_id  主键
					c_name 
					
			user：  u_id   主键
					u_name
					u_sex
					c_id	外键
```

```java
//该实体类与country表属性不一致
public class Country {
		private Integer id;
		private String name;	
    	
    	//配置一对多
		private List<User> userList;
   		 //省略getter，setter
}

//该实体类与user表属性一致
public class User {
	private Integer u_id;
	private String u_name;
	private Character u_sex;
	//省略getter，setter
}


```

```java
//CountryMapper接口
public interface CountryMapper {
		List<Country> findCountryWithUser();
}
```

```xml-dtd
<!--CountryMapper.xml映射文件 -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mapper.CountryMapper">

		<!--手动映射 Country全部属性 -->
	<resultMap type="com.eobard.domain.Country" id="base">
		<id property="id" column="c_id" />
		<result property="name" column="c_name" />
	</resultMap>


	<resultMap type="com.eobard.domain.Country" id="countryListWithUser" extends="base">
		<collection property="userList" ofType="com.eobard.domain.User">
					<!--  配置别名-->
				<id property="u_id" column="user_id"/>
				<result property="u_name" column="u_name"/>
				<result property="u_sex" column="u_sex"/>
		</collection>
	</resultMap>

	<select id="findCountryWithUser" resultMap="countryListWithUser">
			select c.* ,u.u_id as user_id  ,u.u_name,u.u_sex  from country c 
				inner JOIN user u on u.c_id=c.c_id
	</select>

</mapper>
```

```
//测试结果
Country [id=1, name=中国, userList=[User [u_id=1, u_name=九龙坡陈冠希, u_sex=男, c_id=1], User [u_id=3, u_name=王五, u_password=null, u_sex=男, c_id=1]]]

Country [id=2, name=美国, userList=[User [u_id=2, u_name=李四, u_sex=男,  c_id=2]]]

Country [id=3, name=俄罗斯, userList=[User [u_id=4, u_name=二狗,  u_sex=女,  c_id=3]]]

```



​		==注意事项：配置多表查询时，当表之间的属性名重名时(如主键)，需要给其中一个表的属性取别名，不然可能会出现查询结果不完整的情况==





### 9.3 多对一关联查询

​				多对一可以理解为一对一的形式来做

```
数据库设计：
			country：c_id  主键
					c_name 
					
			user：   u_id   主键
					u_name
					u_sex
					c_id	外键
```

<font color="green">用户对应一个城市</font>

```java
//UserTmp实体类
public class UserTmp {
	private Integer u_id;
	private String u_name;
	
    //对应一个城市
	private CountryTmp country;
    
    //省略getter，setter
}

//CountryTmp实体类
public class CountryTmp {
	private int c_id;
	private String c_name;
  //省略getter，setter
}
```

```java
//UserTmpMapper接口
public interface UserTmpMapper {

	List<UserTmp> getUserTmpWithCountryTmp();
}

```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mapper.UserTmpMapper">

	<resultMap type="com.eobard.domain.UserTmp" id="userTmpWithCountry">
		<id property="u_id" column="u_id"/>
		<result property="u_name" column="u_name"/>
			<!--跟一对一关联一样的配置映射查询的列 -->
		<association property="country"  javaType="com.eobard.domain.CountryTmp">
			<id column="c_id" property="c_id"/>
			<result column="c_name" property="c_name"/>
		</association>
	</resultMap>

			<!--多对一关联-->
	<select id="getUserTmpWithCountryTmp" resultMap="userTmpWithCountry">
		SELECT u.u_id,u.u_name,c.* FROM `user` u inner join country c on u.c_id=c.c_id
	</select>
</mapper>  
```

```
//测试结果
UserTmp [u_id=1, u_name=九龙坡陈冠希, country=CountryTmp [c_id=1, c_name=中国]]
UserTmp [u_id=2, u_name=李四, country=CountryTmp [c_id=2, c_name=美国]]
UserTmp [u_id=3, u_name=王五,  country=CountryTmp [c_id=1, c_name=中国]]
UserTmp [u_id=4, u_name=二狗,  country=CountryTmp [c_id=3, c_name=俄罗斯]]
UserTmp [u_id=5, u_name=六四,  country=CountryTmp [c_id=4, c_name=澳大利亚]]
UserTmp [u_id=9, u_name=周杰伦, country=CountryTmp [c_id=6, c_name=叙利亚]]
```

==多对一关联就是类似于一对一关联的操作==



### 9.4 多对多关联查询

​				多对多关联可以拆分成两个一对多，即多的一方对应中间表，中间表对应另一个多的 一方

```
数据库设计：
		user： u_id 主键
			   u_name
               
         user_fun
         		id 主键
         		u_id  user表的外键
         		fun_Id func表的外键
         
         func   id 主键
         		function_name
```

​		<font color="green">一个用户有多个功能权限</font>

```java
//用户实体类
public class User {
	private Integer u_id;
	private String u_name;	
	private Character u_sex;
    
    //对应中间表：一个User有多个User_Func中间表数据
	private List<User_Func> userFuncList;

    //省略getter，setter   
}


//中间表实体类
public class User_Func {
	private Integer id;
	private Integer uId;
	private Integer funId;
    
  	//对应另一个多方表：一个User_Func中间表的数据有多个Func功能表数据
	private List<Func> funcList;

    //省略getter，setter
}


//功能实体类
public class Func {

	private Integer id;
	private String functionName;
	 //省略getter，setter
}
```

```java
//userMapper接口
public interface UserMapper {
		List<User> findUserFunctionInfo();
}
```

```xml-dtd
<!--userMapper.xml映射文件 -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mapper.UserMapper">

		<!--手动映射 User全部属性 -->
	<resultMap type="com.eobard.domain.User" id="baseUser">
		<id property="u_id" column="u_id"/>
		<result property="u_name" column="u_name"/>
		<result property="u_sex" column="u_sex"/>
	</resultMap>	


	<resultMap type="com.eobard.domain.User" id="userFunctionList" extends="baseUser">
		<!-- 一个user表有多个user_func的中间表 : 配置一对多中查询的列 -->
			<collection property="userFuncList" ofType="com.eobard.domain.User_Func">
				<id property="id" column="uf_id"/>
				
				<!--嵌套  一个user_func的中间表有多个func表:配置一对多  -->
					<collection property="funcList" ofType="com.eobard.domain.Func">
						<id property="id" column="function_id"/>
						<result property="functionName" column="function_name"/>
					</collection>
			</collection>
	</resultMap>


	<!--多对多查询 -->
	<select id="findUserFunctionInfo" resultMap="userFunctionList">
		SELECT
			u.u_id ,
			u.u_name,
			u.u_sex,
			uf.id AS uf_id,
			f.id AS function_id,
			f.function_name 
		FROM
			USER u
			INNER JOIN user_fun uf ON u.u_id = uf.u_id
			INNER JOIN func f ON f.id = uf.fun_id
	</select>
	

</mapper>
```

```
//测试结果
用户id：1  用户名：九龙坡陈冠希  用户性别：男
		 功能id：1  功能名：权限1
		 功能id：2  功能名：权限2

用户id：2  用户名：李四  用户性别：男
		 功能id：1  功能名：权限1
		 功能id：2  功能名：权限2
		 
用户id：3  用户名：王五  用户性别：男
		 功能id：4  功能名：权限4
		 功能id：5  功能名：权限5
```

​		==注意：1.Mybatis的多对多关联不像Hibernate那样是双向关联，它是单向关联，所以不需要在两个多方的实体类上都加上对方的List集合==

​					==2. 配置多对多的是分成两个一对多来查询，所以在配置XML文件的时候是<font color="red">嵌套collection元素</font>，表示<font color="red">多的其中一方</font>有多个中间表，中间表又有多个<font color="red">多的另一方</font>==



### 9.5  一对一查询:懒加载(掌握)

<font color="green">一个用户只有一个身份证</font>

```
数据库设计：
			identity： id 主键
					  user_id  外键:对应user表的u_id
					  identity_name
					  
			user：   u_id主键
					 u_name
					 u_sex
```

```JAVA
//user实体类
public class User {

	private int u_id;
	private String u_name;
	private Character u_sex;
	
    //一对一关联identity类
	private Identity identity;
    
    //省略getter，settter
}

//identity实体类
public class Identity {

	private Integer id;
	private String identityName;
    
      //省略getter，settter
}
```

```JAVA
//UserMapper
public interface UserMapper {
    
	User selectUserById(Integer id); //根据自身id查询用户
    
}

//IdentityMapper
public interface IdentityMapper {

	Identity getIdentityByUserId(int userid);  //根据外键id来查询身份证
}

```

```xml-dtd
<!--UserMapper.XML-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.lazy.UserMapper">


	<select id="selectUserById"  resultMap="user_identity_map">
		select u_id,u_name,u_sex from user where u_id=#{u_id}
	</select>
	
	<resultMap type="com.eobard.lazy.User" id="user_identity_map">
			<id property="u_id" column="u_id"/>
			<result property="u_name" column="u_name"/>
			<result property="u_sex" column="u_sex"/>
			
			<!-- column：主表对应数据库的主键列名
				 select：对应另外一个根据外键查询的方法(用namespace+id)
			-->
			<association property="identity" javaType="com.eobard.lazy.Identity" select="com.eobard.lazy.IdentityMapper.getIdentityByUserId" column="u_id"></association>
	</resultMap>
	
</mapper>
```

```xml-dtd
<!--IdentityMapper.XML-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.lazy.IdentityMapper">

		<!-- 这里要根据主表的外键来查询副表自身  -->
	<select id="getIdentityByUserId" resultMap="identityMap">
		select * from identity where user_id=#{id}
	</select>
	
	<resultMap type="com.eobard.lazy.Identity" id="identityMap">
		<result property="identityName" column="identity_Name"/>
	</resultMap>
</mapper> 

```

```xml-dtd
<!--mybatis-config.xml配置-->
<settings>
   			<!-- 开启延迟加载,默认值是true -->
		<setting name="lazyLoadingEnabled" value="true" />
			<!-- 设置积极懒加载,默认值是true -->
		<setting name="aggressiveLazyLoading" value="false"/>
			<!--指定哪个对象的方法触发一次延迟加载。默认值：equals,clone,hashCode,toString  -->
		<setting name="lazyLoadTriggerMethods" value=""/>
</settings>
```

==注意：主表与副表懒加载的时候，association标签里面的column指的是外键名称，即通过主表哪个主键来确定(**简记：就写当前主表的主键名称即可**)，里面的select写的是另外一个mapper文件中根据外键来查询的方法==





### 9.6  一对多查询:懒加载(掌握)

<font color="green">一个国家有多个人</font>

```
数据库设计：
			country：c_id  主键
					c_name 
					
			user：  u_id   主键
					u_name
					u_sex
					c_id	外键:对应country的主键c_id
```

```JAVA
//country实体类
public class Country {
	private Integer id;
	private String name;
    
    //一对多关联User
	private List<User> userList;

    //省略getter，setter
}

//user实体类
public class User {

	private int u_id;
	private String u_name;
	private Character u_sex;
	    
    //省略getter，settter
}

```

```JAVA
//CountryMapper
public interface CountryMapper {

	Country getCountryById(int cid);
}


//UserMapper
public interface UserMapper {

	List<User> getUsersWithCountryByCid(int c_id);
}

```

```xml-dtd
<!--CountryMapper.XML-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.lazy.CountryMapper">

	<select id="getCountryById" resultMap="countryMap">
		select * from country where c_id=#{c_id}
	</select>
	
	<resultMap type="com.eobard.lazy.Country" id="countryMap">
		<id column="c_id" property="id"/>
		<result property="name" column="c_name"/>

		<!-- column：主表对应数据库的主键列名
				 select：对应另外一个根据外键查询的方法(用namespace+id)
			-->
		<collection property="userList" column="c_id" fetchType="lazy" select="com.eobard.lazy.UserMapper.getUserWithCountryByCid" ofType="com.eobard.lazy.User" />
	</resultMap>
</mapper> 

```

```xml-dtd
<!--UserMapper.XML-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.lazy.UserMapper">

			<!-- 这里要根据主表的外键来查询副表自身  -->
	<select id="getUserWithCountryByCid" resultType="com.eobard.lazy.User">
			select u_id,u_name,u_sex from user where c_id=#{c_id}
	</select>
</mapper>
```

```xml-dtd
<!--mybatis-config.xml配置-->
<settings>
   			<!-- 开启延迟加载,默认值是true -->
		<setting name="lazyLoadingEnabled" value="true" />
			<!-- 设置积极懒加载,默认值是true -->
		<setting name="aggressiveLazyLoading" value="false"/>
			<!--指定哪个对象的方法触发一次延迟加载。默认值：equals,clone,hashCode,toString  -->
		<setting name="lazyLoadTriggerMethods" value=""/>
</settings>
```

==注意：主表与副表懒加载的时候，collection标签里面的column指的是外键名称，即通过主表哪个主键来确定(**简记：就写当前主表的主键名称即可**)，里面的select写的是另外一个mapper文件中根据外键来查询的方法==



---



## 十.  PageHelper分页插件

​			 首先导入jar包，然后要在配置文件去配置PageHelper的拦截器

​	

```java
//UserMapper接口
public class UserMapper{
    
    public List<User> findUserByPageHelper();
}
```

```xml-dtd
<!--UserMapper.xml映射文件-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mapper.UserMapper">
    
    <select id="findUserByPageHelper" resultType="com.eobard.domain.User">
    	select * from user
    </select>
</mapper>
```

```java
//测试类
public class Test{

    /**
    *  普通方式
    */
	@Test
	public void test5() {
		SqlSession session = MybatisUtils.getSession();
		UserMapper userMapper = session.getMapper(UserMapper.class);
        //设置从第一页开始，读取三条数据
		PageHelper.startPage(1,3);
        
   		List<User> userList = userMapper.selectAllBySQLTag();
        //将结果封装到PageInfo中，有更加强大的API可以提供
		PageInfo<User> pageInfo = new PageInfo<User>(userList);
        
        //常用的API
		System.out.println("当前页码：" + pageInfo.getPageNum());
		System.out.println("每页数量：" + pageInfo.getPageSize());
		System.out.println("总记录数：" + pageInfo.getTotal());
		System.out.println("总页码：" + pageInfo.getPages());
        
        System.out.println("上一页"+pageInfo.getPrePage());
		System.out.println("下一页"+pageInfo.getNextPage());
        
        System.out.println("是否有下一页："+ pageInfo.isHasNextPage());
		System.out.println("是否有下一页："+pageInfo.isHasPreviousPage());
        
		System.out.println("是否是第一页："+pageInfo.isIsFirstPage());
		System.out.println("是否是最后一页："+pageInfo.isIsLastPage());
        
        //分页的数据 
     	pageInfo.getList().forEach(System.out::println);
         
		MybatisUtils.closeSession(session);
	}  
    
   /**
    *  lambda表达式
    */
    @Test
    public void test6(){
        SqlSession session = MybatisUtils.getSession();
	 	UserMapper userMapper = session.getMapper(UserMapper.class);
        
        //设置从第一页开始，读取三条数据并调用相应的mapper
        Page<User> page2 = PageHelper.startPage(4, 3).doSelectPage(() -> userMapper.selectAllBySQLTag());
        
		System.out.println("当前页码：" + page2.getPageNum());
		System.out.println("每页数量：" + page2.getPageSize());
		System.out.println("总记录数：" + page2.getTotal());
		System.out.println("总页码：" + page2.getPages());
        
         //分页的数据 
		List<User> result = page2.getResult();
		result.forEach(System.out::println);
		MybatisUtils.closeSession(session);
    }
}
```

```xml-dtd
<!--mybatis-config.xml文件中-->
<plugins>
		<!-- 5.0版本之前为PageHelper，5.0版本以后为PageInterceptor  -->
	<plugin interceptor="com.github.pagehelper.PageInterceptor">
			<!--
				reasonable:分页合理化参数，默认为false，根据参数查询	
						为true时   当前页码<0  查询第一页，当前页码>总页码    查询最后一页
			  -->
		<property name="reasonable" value="trues"/>
	</plugin>
</plugins>
```

​	==注意：1. 使用了PageHelper插件，SQL语句只需要写相应查询就行，不用加上limit ? , ?==

​				==2. 一定要在查询前去设置PageHelper.startPage( ) 来初始化分页查询==

​				==3. PageInfo有相关分页的方法不用手动继续去查==

---



## 十一. 注解开发

​			**注解开发只是针对于Dao层开发并不是动态代理的注解开发,能够实现简单的CRUD语句，对于关联语句较为复杂，还是推荐使用XML形式**

```xml
<!--mybatis-config.xml映射注解注入-->
<mappers>
	<package name="com.eobard.anno"/>
</mappers>
```

### 	11.1 @Insert新增

```java
public interface UserMapper {

	//新增
	@Insert("insert into user(u_name,u_password,u_sex,u_phone) values(#{u_name},#{u_password},#{u_sex},#{u_phone})")
	public void addUser(User user);
	
}
```

```
//测试
		SqlSession session = MybatisUtils.getSession();
		UserMapper userMapper = session.getMapper(UserMapper.class);
		//省略设置值
		userMapper.addUser(user);
		session.commit();
```



### 	11.2 @Delete删除

```java
public interface UserMapper {

	//删除
	@Delete("delete from user where u_id=#{id}")
	public void delUser(int id);
	
	
}
```

```
//测试
		SqlSession session = MybatisUtils.getSession();
		UserMapper userMapper = session.getMapper(UserMapper.class);
		userMapper.delUser(10);
		session.commit();
```



### 	11.3 @Update修改 

```java
public interface UserMapper {

	//修改
	@Update("update user set u_name=#{u_name},u_sex=#{u_sex} where u_id=#{u_id}")
	public void updateUser(User user);
	
}
```

```
//测试
		SqlSession session = MybatisUtils.getSession();
		UserMapper userMapper = session.getMapper(UserMapper.class);
		//省略设置值
		userMapper.updateUser(user);
		session.commit();
```



### 	11.4 @Select 查询

```java
public interface UserMapper {

	//查询
	@Select("select * from user")
	public List<User> findAll();
	
}
```

```
//测试
		SqlSession session = MybatisUtils.getSession();
		UserMapper userMapper = session.getMapper(UserMapper.class);
		//省略设置值
		userMapper.findAll().forEach(System.out::println);
```



### 11.5  @Alias 别名

```java
@Alias(value="u")
public class User{
    private int id;
    private String name;
    //省略getter，setter 
    
}
```

​	相当于在配置文件配置了一个别名为 u的User实体类





### 11.6 @Results和@Result

​			@Results相当于是映射文件里面的ResultMap，@Result相当于是配置里面的属性

```java
//Country 实体类与数据库表不一致
public class Country {

	private Integer id;
	private String name;

	//省略getter，setter
}

```

```java
//CountryMapper接口
public interface CountryMapper {

	@Select("select * from country")
	@Results({
   		/**  映射其它属性：
   					id：默认为false,表示非主键；true表示为主键属性
		 * 			property：实体类的属性
 		 * 			column：数据库中的列名
 		 */
		@Result(id = true,property = "id",column = "c_id"),
		@Result(property = "name",column = "c_name")
	})
	 List<Country> findCountryAll();
}

```

​		==注意：@Results与之前在xml中配置一个resultMap是一样的，用于配置实体类的属性与数据库列名不一致==



### 11.7  一对一

​		<font color="red">注解开发一对一需要分成两个sql语句来完成</font>

```
数据库设计：
			identity： id 主键
					  user_id  外键
					  identity_name
					  
			user：   u_id主键
					u_name
					u_sex
```

```java
public class Identity {
	private Integer id;
	private String identityName;
		//配置一对一
	private User user;
	
	//省略getter，settter
}
```

```java
//一个身份证属于一个用户
public interface IdentityMapper {
	
	@Select("select * from identity")
	@Results({
        //配置自己的属性
		@Result(id=true,column ="id",property = "id"),
		@Result(column ="identity_name",property = "identityName"),
        
        //配置一对一关联
        /*	 property：identity引用user的属性名
		*	column：Identity对应user的数据库外键名称，即当前表的外键
		*	one = @One(select = "对应的是UserMapper的findUserById查询单个用户的方法")
		*/
		@Result(column ="user_id",property = "user",one = @One(select = "com.eobard.anno.UserMapper.findUserById")),
	})
	public List<Identity> findIdentityWithUser();
}
```

```java
public interface UserMapper {

    //一定是根据上边的外键查询单个用户(column ="user_id")对应的是user的u_id
	@Select("select * from user u where u_id=#{userId}")
	public  User findUserById(int userId);
}
```

​		==注意：1.注解开发一对一的时候，配置one=@One一定是另外一个Mapper的某个方法，且里面是**包名.类名.方法名**==

​					==2.**最好是通过副表来查询一对一的主表，这样不会出现空指针**；**如果用主表来一对一查询副表要判断一次副表的数据不为空**，否则会出现主表有数据，副表没数据报空指针的情况==



### 11.8 一对多

```
数据库设计：
		dept： deptid 主键
			   dname  
			   
		emp：  empid 主键
			  de_id  外键
			  ename
              
	一个dept有多个emp
		
```

```java
public class Dept {

	private int deptId;
	private String dName;
    //多个emp
	private List<Emp> empList;
	
    //省略getter，setter
}

public class Emp {

	private int empid;
	private String ename;
    
    //省略getter，setter
}
```

```java
//DeptMapper
public interface DeptMapper {
	
	@Select("select * from dept")
	@Results({
		@Result(id = true,property = "deptId",column = "deptid"),
		@Result(property = "dName",column = "dname"),
        //配置一对多的关系
        /**
        * column：为Dept的主键名称
        * property：为Dept引用多方List的属性名
        * javaType：集合的class
        * many = @Many(select = "对应的是EmpMapper的findEmpById查询外键对应EmpMapper的方法")
        */
		@Result(property = "empList",column = "deptid",javaType = List.class,many = @Many(select="com.eobard.anno.EmpMapper.findEmpById"))
	})
	List<Dept> findDeptWithEmp();
}
```

```java
public interface EmpMapper {

    //一定是根据上边的主键查询单个Emp(column = "deptid")对应的是Emp的de_id
	@Select("select * from emp where de_id=#{id}")
	public Emp findEmpById(int deptID);
}

```



​	==不管是查询一对一，还是一对多的时候，**其中一个关联查询的column值**一定要在**另一个mapper查询条件为当前表所对应的外键**==



### 11.9 多对多

```
数据库设计：
		user： u_id 主键
			   u_name
               
         user_fun
         		id 主键
         		u_id  user表的外键
         		fun_Id func表的外键
         
         func   id 主键
         		function_name
```

```java
public class User {

	private Integer u_id;
	private String u_name;

    //一个用户有多个Func集合
	private List<Func> funcList;

    //省略getter，setter
}



public class Func {

	private Integer id;
	private String functionName;
    
      //省略getter，setter
}
	
```

```java
//UserMapper
public interface UserMapper {
    
    @Select("select u_id,u_name from user")
	@Results({
		@Result(id=true,column ="u_id",property = "u_id"),
		@Result(column ="u_name",property = "u_name"),
        
         //配置多对多的关系：相当于也是一对多
        /**
        * column：为User的主键名称
        * property：为user引用多方List的属性名
        * javaType：集合的class
        * many = @Many(select = "对应的是FuncMapper的findFuncById查询外键对应FuncMapper的方法")
        */
		@Result(property = "funcList",column = "u_id",javaType = List.class,many = 										@Many(select="com.eobard.anno.FuncMapper.findFuncById"))
	})
	List<User> findUserWithFunc();
}
```

```java
public interface FuncMapper {

     //一定是根据上边的主键查询单个Func(column = "u_id")对应的是user_fun中间表的u_id，且这里需要连中间表查询
	@Select("select f.function_Name from  func f inner join user_fun uf on uf.fun_Id=f.id and uf.u_Id=#{id}")
	@Results({@Result(property = "functionName",column = "function_Name")})
	public List<Func> findFuncById(int id);
}

```

​		==注意：在多对多查询时，第二个Mapper需要连表查询中间表这样才能让多对多的关系联立起来，所以需要在第二个Mapper的SQL语句查询两张表==



---



## 十二 . Spring整合Mybatis

### 12.1 整合准备

​			a. 导入相应jar包

​			b. 在src创建相应的配置文件： applicationContext.xml   db.properties   Log4j.properties  mybatis-config.xml



==注意：1. mybatis-config.xml文件中不用配置数据库连接，也不用配置SQL映射文件统统交给Spring管理==



### 12.2  通过SqlSessionTemplate 方式(了解)

```java
public class UserMapper{

	//引用SqlSessionTemplate模板并提供setter方便spring注入
	private SqlSessionTemplate  sqlSessionTemplate;

	public void  setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
		this.sqlSessionTemplate=sqlSessionTemplate;
	}
	
	
	public List<User> findAll(){
		return this.sqlSessionTemplate.selectList("UserMapper.selectUser" );
	}
}
```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="UserMapper">
	<select id="selectUser" resultType="User">
		select * from user
	</select>
</mapper>
```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns="http://www.springframework.org/schema/beans" 
	xmlns:aop="http://www.springframework.org/schema/aop" 
	xmlns:context="http://www.springframework.org/schema/context" 
	xmlns:tx="http://www.springframework.org/schema/tx" 
	xmlns:util="http://www.springframework.org/schema/util" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd 
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd 
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd ">
	
	<!-- 读取db.properties -->
	<context:property-placeholder location="db.properties"/>
	
	<!-- 配置c3p0连接池 -->
	<bean name="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.class}"/>
		<property name="jdbcUrl" value="${jdbc.url}"/>
		<property name="user" value="${jdbc.user}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>
	
	<!-- 配置mybatis sqlSessionFactory -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
			<!-- 配置数据源  -->
		<property name="dataSource" ref="dataSource"/>
			<!-- 告诉spring mybatis的核心配置文件 -->
		<property name="configLocation" value="classpath:mybatis-config.xml"/>
			<!-- 加载SQL映射文件 :  **/*.xml表示加载mappers的文件夹及其子文件夹的任意xml文件   -->
		<property name="mapperLocations" value="classpath:com/eobard/mappers/**/*.xml" />
			<!--通过整个包起别名-->
		<property name="typeAliasesPackage" value="com.eobard.entity"></property>
	</bean>
	
	<!--注入sqlSessionTemplate  -->
	<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory" />
	</bean>
	
		<!--注入UserMapper-->
	<bean id="userMapper" class="com.eobard.dao.UserMapper">
		<property name="sqlSessionTemplate" ref="sqlSessionTemplate" />
	</bean>
	
</beans>

```

```
//测试即可
		ApplicationContext ac=new ClassPathXmlApplicationContext("applicationContext.xml");
		UserMapper userMapper = (UserMapper) ac.getBean("userMapper");
		userMapper.findAll().forEach(System.out::println);
```

==通过SqlSessionTemplate模板来完成Mybatis的相关操作==



### 12.3 通过SqlSessionDaoSupport方式(了解)

```java
public class UserDaoImp extends SqlSessionDaoSupport{

	public List<User> findAll(){
		SqlSession session=this.getSqlSession();
		List<User> userList = session.selectList("UserMapper.selectUser");
		return userList;
	}
}

```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="UserMapper">
	<select id="selectUser" resultType="User">
		select * from user
	</select>
</mapper>
```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns="http://www.springframework.org/schema/beans" 
	xmlns:aop="http://www.springframework.org/schema/aop" 
	xmlns:context="http://www.springframework.org/schema/context" 
	xmlns:tx="http://www.springframework.org/schema/tx" 
	xmlns:util="http://www.springframework.org/schema/util" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd 
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd 
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd ">
	
	<!-- 读取db.properties -->
	<context:property-placeholder location="db.properties"/>
	
	<!-- 配置c3p0连接池 -->
	<bean name="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.class}"/>
		<property name="jdbcUrl" value="${jdbc.url}"/>
		<property name="user" value="${jdbc.user}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>
	
	<!-- 配置mybatis sqlSessionFactory -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
			<!-- 配置数据源  -->
		<property name="dataSource" ref="dataSource"/>
			<!-- 告诉spring mybatis的核心配置文件 -->
		<property name="configLocation" value="classpath:mybatis-config.xml"/>
			<!-- 加载SQL映射文件 :  **/*.xml表示加载mappers的文件夹及其子文件夹的任意xml文件   -->
		<property name="mapperLocations" value="classpath:com/eobard/mappers/**/*.xml" />
			<!--通过整个包起别名-->
		<property name="typeAliasesPackage" value="com.eobard.entity"></property>
			
	</bean>
	
		<!-- 1.Dao式开发 ：注入userDao -->
 	<bean id="userDao" class="com.eobard.dao.UserDaoImp">
			<!--注入dao层的实体必须要注入sqlSessionFactory同Hibernate-->
		<property name="sqlSessionFactory" ref="sqlSessionFactory"/>
	</bean> 
	
	
</beans>

```

==普通的Dao式开发==



### 12.4 通过MapperFactoryBean方式(了解)

```java
public interface UserMapper2 {

	public User getUserById(int userId);
}

```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mappers.UserMapper2">
	<select id="getUserById" resultType="User">
		select * from user where u_id=#{id}
	</select>
</mapper>
```



```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns="http://www.springframework.org/schema/beans" 
	xmlns:aop="http://www.springframework.org/schema/aop" 
	xmlns:context="http://www.springframework.org/schema/context" 
	xmlns:tx="http://www.springframework.org/schema/tx" 
	xmlns:util="http://www.springframework.org/schema/util" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd 
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd 
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd ">
	
	<!-- 读取db.properties -->
	<context:property-placeholder location="db.properties"/>
	
	<!-- 配置c3p0连接池 -->
	<bean name="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.class}"/>
		<property name="jdbcUrl" value="${jdbc.url}"/>
		<property name="user" value="${jdbc.user}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>
	
	<!-- 配置mybatis sqlSessionFactory -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
			<!-- 配置数据源  -->
		<property name="dataSource" ref="dataSource"/>
			<!-- 告诉spring mybatis的核心配置文件 -->
		<property name="configLocation" value="classpath:mybatis-config.xml"/>
			<!-- 加载SQL映射文件 :  **/*.xml表示加载mappers的文件夹及其子文件夹的任意xml文件   -->
		<property name="mapperLocations" value="classpath:com/eobard/mappers/**/*.xml" />
				<!--通过整个包起别名-->
		<property name="typeAliasesPackage" value="com.eobard.entity"></property>	
	</bean>
	
	
	<!-- 2. mapper动态代理开发 -->
	 <bean id="countryMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
			<!-- 注入 sqlSessionFactory -->
		<property name="sqlSessionFactory" ref="sqlSessionFactory"/>
			<!-- 配置接口：需要一个一个的注入动态mapper接口(缺点) -->
		<property name="mapperInterface" value="com.eobard.mappers.CountryMapper"/>
	</bean> 
	
	
</beans>

```

==通过Mybatis动态代理的方式来整合，动态代理的Mapper有多少个，就需要在spring容器中注入多少个==



### 12.5  通过MapperScannerConfigurer方式(重点掌握)

```java
public interface CountryMapper {

	List<Country> findAll();
}
```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.eobard.mappers.CountryMapper">
	<select id="findAll" resultType="Country">
		select * from country
	</select>
</mapper> 
```

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns="http://www.springframework.org/schema/beans" 
	xmlns:aop="http://www.springframework.org/schema/aop" 
	xmlns:context="http://www.springframework.org/schema/context" 
	xmlns:tx="http://www.springframework.org/schema/tx" 
	xmlns:util="http://www.springframework.org/schema/util" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd 
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd 
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd ">
	
	<!-- 读取db.properties -->
	<context:property-placeholder location="db.properties"/>
	
	<!-- 配置c3p0连接池 -->
	<bean name="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="driverClass" value="${jdbc.class}"/>
		<property name="jdbcUrl" value="${jdbc.url}"/>
		<property name="user" value="${jdbc.user}"/>
		<property name="password" value="${jdbc.password}"/>
	</bean>
	
	<!-- 配置mybatis sqlSessionFactory -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
			<!-- 配置数据源  -->
		<property name="dataSource" ref="dataSource"/>
			<!-- 告诉spring mybatis的核心配置文件 -->
		<property name="configLocation" value="classpath:mybatis-config.xml"/>
			<!-- 加载SQL映射文件 :  **/*.xml表示加载mappers的文件夹及其子文件夹的任意xml文件   -->
		<property name="mapperLocations" value="classpath:com/eobard/mappers/**/*.xml" />
			<!--通过整个包起别名-->
		<property name="typeAliasesPackage" value="com.eobard.entity"></property>
	</bean>
	
	
	<!-- 3.mapper动态扫描开发(☆☆☆☆推荐使用)
		  获取bean的时候就不能通过id来获取了，通过接口类型获取即可
	 -->
 	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
 			<!--以包的形式直接扫描全部动态mapper接口  -->
		<property name="basePackage" value="com.eobard.mappers"/>
	</bean> 
	
</beans>

```

```
//测试
		ApplicationContext ac=new ClassPathXmlApplicationContext("applicationContext.xml");
		CountryMapper countryMapper = ac.getBean(CountryMapper.class);
		countryMapper.findAll().forEach(System.out::println);
```

==通过MapperScannerConfigurer注入，注入就不需要给bean的id，测试使用的时候也不用用id来获取Bean，直接用class字节码来获取==



---



## 十三. 注解式声明事务

### 	1. 在IOC容易中添加注解式声明事务

```xml-dtd
<!--只需要在原有的applicationContext.xml中添加开启注解式声明式务即可-->

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<tx:annotation-driven transaction-manager="transactionManager"/>
```

### 	2.在相应的service层下加上注解@Transactional 即可完成**增删改**操作

​	

---



## 十四. Mybatis缓存使用

### 14.1 缓存常见问题

​			**Q：<font color="red">为什么使用缓存？</font>** 

​			          	减少和数据库的交互次数，减少系统开销，提高系统效率

​			**Q：<font color="red">什么样的数据需要缓存？</font>**

​						经常查询但是不经常改变的数据

​			Q：<font color="red">**缓存的顺序？**</font>

​							先从二级缓存中查询有无需要的数据，然后从一级缓存中查询有无需要的数据，然后从数据库中查询

### 14.2 一级缓存

​				**Mybatis默认是存在一级缓存的，即SqlSession级别的缓存**, 在同一个SqlSession会话中多次查询同样的SqlMapper会将第一次查询数据库的数据放入到一级缓存中，后续查询会从缓存中查询

```JAVA
	//省略其他获取资源等操作
	@Test
	public void test() {
		User user1 = userMapper.getUserById(1);
		User user2 = userMapper.getUserById(1);
		System.out.println(user1==user2);
	}
```

```
DEBUG [com.eobard.cache.UserMapper] - Cache Hit Ratio [com.eobard.cache.UserMapper]: 0.0
DEBUG [com.eobard.cache.UserMapper.getUserById] - ==>  Preparing: select * from user where u_id=? 
DEBUG [com.eobard.cache.UserMapper.getUserById] - ==> Parameters: 1(Integer)
DEBUG [com.eobard.cache.UserMapper.getUserById] - <==      Total: 1
DEBUG [com.eobard.cache.UserMapper] - Cache Hit Ratio [com.eobard.cache.UserMapper]: 0.0
true
```

==在当前会话中查询多次只会出现一个sql语句，这就是Mybatis的默认一级缓存，commit( 如增删改 )操作会清空一级缓存的数据==



### 14.3  二级缓存

​		 	●   二级缓存又称全局缓存，一级缓冲作用域太低了(只在相同会话中)，**作用域：在同一个namespace生成的mapper对象**

​			 ●  工作机制： 一个SqlSession会话查询的数据，这个对应的数据就会保存到一级缓存中，如果当前SqlSession会话被关闭了，对应的一级缓存中的数据就会消失；**这时，当前一个SqlSession会话关闭了，上一次SqlSession查询的一级缓存中的数据就会保存到二级缓存中；再开启新的SqlSession会话就会从二级缓存查询**



​	<font color="green">配置二级缓存</font>

```xml-dtd
<!--mybatis-config.xml文件-->
<settings>	
	<!--显示的开启二级缓存配置-->
	<setting name="cacheEnabled" value="true"/>
</settings>
```

```xml-dtd
<!--相应的mapper.xml文件中设置-->
<mapper namespace="com.eobard.cache.UserMapper">

		<!--开启二级缓存-->
	<cache />
		<!--useCache="true" 表示在当前的查询语句使用二级缓存-->
	<select id="getUserById" resultType="com.eobard.domain.User" useCache="true">
		select * from user where u_id=#{id}
	</select>
</mapper>
```

​	<font color="green">测试</font>

```java
public class UserCacheTest {
	private SqlSession session1;
	private SqlSession session2;
	private com.eobard.cache.UserMapper userMapper1;
	private com.eobard.cache.UserMapper userMapper2;
	
    @Before
	public void before() {
        //开启两个不同的session会话
		session1 = MybatisUtils.getSession();
		session2=MybatisUtils.getSession();
		userMapper1=session1.getMapper(UserMapper.class);
		userMapper2=session2.getMapper(UserMapper.class);
	}
	
	@Test
	public void test() {
        //第一个session获取
		User user1 = userMapper1.getUserById(1);
        //关闭第一个session
		MybatisUtils.closeSession(session1);
        //第二个session获取
		User user2 = userMapper2.getUserById(1);
		
			
	}
}
```

```
DEBUG [com.eobard.cache.UserMapper] - Cache Hit Ratio [com.eobard.cache.UserMapper]: 0.0
DEBUG [com.eobard.cache.UserMapper.getUserById] - ==>  Preparing: select * from user where u_id=? 
DEBUG [com.eobard.cache.UserMapper.getUserById] - ==> Parameters: 1(Integer)
DEBUG [com.eobard.cache.UserMapper.getUserById] - <==      Total: 1
DEBUG [com.eobard.cache.UserMapper] - Cache Hit Ratio [com.eobard.cache.UserMapper]: 0.5
```



​	==三点注意：1. 需要二级缓存的实体类最好实现序列化接口==

​						==2.只有当会话提交或者关闭的时候，才会提交到二级缓存中==

​						==3.不管是一级缓存还是二级缓存，在会话中若使用了增删改操作(commit() )都会让缓存失效,即清理缓存==



​			**<font color="red">结论：只要产生的xxxMapper对象来自于同一个namespace，则这些对象共享二级缓存</font>**



### 14.4 使用ehcache缓存   

​				1.首先导入ehcache的jar包

​    			2.在src根路径创建ehcache.xml配置文件

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
	<!-- 磁盘保存缓存路径 -->
	<diskStore path="D:\\ehcache" />

	<defaultCache 
		maxElementsInMemory="10000"
		eternal="false" 
		overflowToDisk="true"
		timeToIdleSeconds="120" 
		timeToLiveSeconds="120"
		memoryStoreEvictionPolicy="LRU">
	</defaultCache>
</ehcache>

<!-- 
属性说明：
		 diskStore：当内存中不够存储时，存储到指定数据在磁盘中的存储位置。
以下属性是必须的：
 		maxElementsInMemory - 在内存中缓存的element的最大数目 
 		eternal - 设定缓存的elements是否永远不过期。如果为true，则缓存的数据始终有效，如果为false那么还要根据		timeToIdleSeconds，timeToLiveSeconds判断
 		overflowToDisk - 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上
 
以下属性是可选的：
 		timeToIdleSeconds - 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时，这些数据便会删除，默认值是0,也就是可闲置时间无穷大
 		timeToLiveSeconds - 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大
 		memoryStoreEvictionPolicy - 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）
 -->

```

​		<font color="green">配置二级缓存</font>

```xml-dtd
<!--mybatis-config.xml文件-->
<settings>	
	<!--显示的开启二级缓存配置-->
	<setting name="cacheEnabled" value="true"/>
</settings>
```

```xml-dtd
<!--相应的mapper.xml文件中设置-->
<mapper namespace="com.eobard.cache.UserMapper">

		<!--开启ehcache二级缓存-->
	<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
		<!--useCache="true" 表示在当前的查询语句使用二级缓存-->
	<select id="getUserById" resultType="com.eobard.domain.User" useCache="true">
		select * from user where u_id=#{id}
	</select>
</mapper>
```



​	<font color="green">测试</font>

```java
public class UserCacheTest {
	private SqlSession session1;
	private SqlSession session2;
	private com.eobard.cache.UserMapper userMapper1;
	private com.eobard.cache.UserMapper userMapper2;
	
    @Before
	public void before() {
        //开启两个不同的session会话
		session1 = MybatisUtils.getSession();
		session2=MybatisUtils.getSession();
		userMapper1=session1.getMapper(UserMapper.class);
		userMapper2=session2.getMapper(UserMapper.class);
	}
	
	@Test
	public void test() {
        //第一个session获取
		User user1 = userMapper1.getUserById(1);
        //关闭第一个session
		MybatisUtils.closeSession(session1);
        //第二个session获取
		User user2 = userMapper2.getUserById(1);
			
	}
}
```

```
DEBUG [net.sf.ehcache.CacheManager] - Creating new CacheManager with default config
DEBUG [net.sf.ehcache.CacheManager] - Configuring ehcache from classpath.
DEBUG [net.sf.ehcache.config.ConfigurationFactory] - Configuring ehcache from ehcache.xml found in the classpath: file:/D:/Java/eclipse_workspace/MybatisDemo/bin/ehcache.xml
DEBUG [net.sf.ehcache.config.ConfigurationFactory] - Configuring ehcache from URL: 		        file:/D:/Java/eclipse_workspace/MybatisDemo/bin/ehcache.xml
DEBUG [net.sf.ehcache.config.ConfigurationFactory] - Configuring ehcache from InputStream
DEBUG [net.sf.ehcache.config.BeanHandler] - Ignoring ehcache attribute xmlns:xsi
DEBUG [net.sf.ehcache.config.BeanHandler] - Ignoring ehcache attribute xsi:noNamespaceSchemaLocation
DEBUG [net.sf.ehcache.config.DiskStoreConfiguration] - Disk Store Path: D:\\ehcache
DEBUG [net.sf.ehcache.store.DiskStore] - Deleting data file com.eobard.cache.UserMapper.data
DEBUG [net.sf.ehcache.store.MemoryStore] - Initialized net.sf.ehcache.store.LruMemoryStore for com.eobard.cache.UserMapper
DEBUG [net.sf.ehcache.store.LruMemoryStore] - com.eobard.cache.UserMapper Cache: Using SpoolingLinkedHashMap implementation
DEBUG [net.sf.ehcache.Cache] - Initialised cache: com.eobard.cache.UserMapper
DEBUG [com.eobard.cache.UserMapper] - Cache Hit Ratio [com.eobard.cache.UserMapper]: 0.0
DEBUG [com.eobard.cache.UserMapper.getUserById] - ==>  Preparing: select * from user where u_id=? 
DEBUG [com.eobard.cache.UserMapper.getUserById] - ==> Parameters: 1(Integer)
DEBUG [com.eobard.cache.UserMapper.getUserById] - <==      Total: 1
DEBUG [com.eobard.cache.UserMapper] - Cache Hit Ratio [com.eobard.cache.UserMapper]: 0.5

```



## 十五. MybatisX插件使用

​		该插件适合于Idea开发工具，在plugin中搜索**MybatisX**插件，可以在你创建接口的时候给你自动创建XML文件的结构


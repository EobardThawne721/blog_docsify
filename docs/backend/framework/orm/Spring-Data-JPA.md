# Spring Data JPA

## 一.入门环境

`1.导入依赖`

```xml-dtd
 <!-- MySQL -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!-- Spring Data JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--Spring 官方提供的热部署插件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
  	 <!--druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.5</version>
        </dependency>
```



`2.编写全局配置文件`

```properties
#数据库配置:
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/jpa?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=123456
#整合数据连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

#JPA配置:
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

> **spring.jpa.hibernate.ddl-auto该参数的几种配置如下：**
>
> *  **create ：每次加载hibernate时都会删除上一次的生成的表，然后根据你的实体类再重新来生成新表。**
> * **create-drop ：每次加载hibernate时根据model类生成表，但是sessionFactory一关闭，表就自动删除**
> * **update(常用) ：第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。**
> * **validate ：每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入 新值。**
> * **none：不自动修改数据库表结构，手动创建数据库表结构**



`3.实体类`

```JAVA
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity                 //标识是一个实体类
@Table(name="t_user")   //指定生产数据库的表名
public class User {

    @Id                 //主键
    @GeneratedValue(strategy=GenerationType.IDENTITY)   //主键类型:自增类型
    private Integer id;
    private String userName;	
    private String address;
    //当属性名与列名一致时，可以省略@Column
    @Column
    private Integer age;

}
```

> **注意：** 
>
> 1. **属性名若是驼峰命名法，数据库表生成的字段会有下划线_，如属性名为userName，列名则是user_name**
> 2. **属性与列名一致时，可以省略@Column**



`4.dao层接口`

```JAVA
/**
 *  接口推荐命名方式：实体类名称+Repository
 *  需要继承Spring Data JPA提供的接口:
 *          其中T为实体类,ID为实体类主键类型
 */
public interface UserCrudRepository extends CrudRepository<User,Integer> {

}
```



`5.测试`

```JAVA
 	@Resource
    private UserCrudRepository userCrudRepository;

    @Test
    void contextLoads() {
        userCrudRepository.save(new User(1,"eobard","九龙坡",21));
    }
```



`6.结果`

```
Hibernate: select user0_.id as id1_0_0_, user0_.address as address2_0_0_, user0_.age as age3_0_0_, user0_.user_name as user_nam4_0_0_ from t_user user0_ where user0_.id=?
Hibernate: insert into t_user (address, age, user_name) values (?, ?, ?)
```



## 二.JPA内置接口API

### 2.1 接口关系（重点）

```tex
CrudRepository(只有增删改查)
       ↑ 继承
PagingAndSortingRepository(支持分页、排序) 
       ↑ 继承
JpaRepository(功能最全,基本都继承它) 
       ↑ 继承			 实现
你自己写的 Repository	➡  JpaSpecificationExecutor(复杂查询,动态条件拼接,组合使用) 
```







### 2.2 CRUD基础接口

```JAVA
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    /** 保存或修改
     *			若数据库存在实体类指定的主键,则为修改
     *			若数据库不存在实体类指定的主键,则为新增
     */
    <S extends T> S save(S var1);
    
    //批量保存
    <S extends T> Iterable<S> saveAll(Iterable<S> var1);
    
    //根据主键查询实体
    Optional<T> findById(ID var1);
    
    //根据主键判断实体是否存在
    boolean existsById(ID var1);
    
    //查询实体的所有列表
    Iterable<T> findAll()
    
    //根据主键列表查询实体列表
    Iterable<T> findAllById(Iterable<ID> var1);
    
    //查询总数
    long count();
    
    //根据主键删除
    void deleteById(ID var1);
    
    //根据实体对象删除:实体类中必须指定id
    void delete(T var1);
    
    //根据实体对象批量删除
    void deleteAll(Iterable<? extends T> var1);
    
    //删除所有数据
    void deleteAll();
}
```

==**注意：该接口了解即可！！**==



### 2.3 分页和排序接口

#### 2.3.1 内置API

```JAVA
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
    //查询列表，支持排序
    Iterable<T> findAll(Sort var1);
    
    //分页查询
    Page<T> findAll(Pageable var1);
}
```

> 注意：从该接口继承关系可以知道，该**接口拥有CrudRepository的所有API，并且还增加了排序和分页的新方法**



#### 2.3.2 排序

`1.接口`

```JAVA
public interface UserPagingAndSortingRepository extends PagingAndSortingRepository<User,Integer> {

}
```



`2.测试类`

```JAVA
 	@Resource
    private UserPagingAndSortingRepository userPagingAndSortingRepository;

    @Test
    public void test1() throws Exception{
        //创建排序对象（参数1：升序或降序，参数2：排序属性名）
        Sort sort = Sort.by(Sort.Direction.ASC, "age")
                .and(Sort.by(Sort.Direction.DESC,"id"));
        userPagingAndSortingRepository.findAll(sort).forEach(user ->System.out.println(user) );
    }
```

> 该实例表示：首先按`age`升序排序，若出现相同年龄，则再根据`id`降序排序



#### 2.3.3 分页

> **所有的分页方法不需要关注具体的分页逻辑，只需要在方法的形参传入分页对象、方法的返回值为`Page<T>`即可**
>
> eg：
>
> ```java
> Page<Teaching> list(Pageable pageable);
> Page<TeachingCourseVO> listCoursesByTeacherId(Pageable pageable, Long teacherId);
> ```



`分页`

> PageRequest.of( 参数1代表第几页开始读(`下标从0开始，不用乘读取条数`)，参数2读取条数)

```java
  	@Resource
    private UserPagingAndSortingRepository userPagingAndSortingRepository;

	@Test
    public void test2() throws Exception{
        userPagingAndSortingRepository.findAll(PageRequest.of(0,1)).forEach(user -> System.out.println("user = " + user));
    }
```



`分页+排序`

>   PageRequest.of(参数1代表第几页开始读(`下标从0开始,不用乘读取条数`)，参数2读取条数，参数3代表排序条件，参数4代表排序字段)

```JAVA
	@Resource
    private UserPagingAndSortingRepository userPagingAndSortingRepository;

	 @Test
    public void test3() throws Exception{        		      userPagingAndSortingRepository.findAll(PageRequest.of(0,1,Sort.Direction.DESC,"id")).forEach(user -> System.out.println("user = " + user));
    }
```



==**注意：该接口了解即可！！**==



### 2.4 JpaRepository接口(重点)

```JAVA
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>,
QueryByExampleExecutor<T> {
    List<T> findAll();
    List<T> findAll(Sort var1);
    List<T> findAllById(Iterable<ID> var1);
    <S extends T> List<S> saveAll(Iterable<S> var1);
    void flush();
    <S extends T> S saveAndFlush(S var1);
    void deleteInBatch(Iterable<T> var1);
    void deleteAllInBatch();
    T getOne(ID var1);
    <S extends T> List<S> findAll(Example<S> var1);
    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
}

```

> 该接口继承关系：继承了CrudRepository接口API，优化了PagingAndSortingRepository接口API，**`实际开发中，普遍继承该接口`**，**如果需要分页或者排序直接将对应对象放在形参里即可**
>
> eg：
>
> ```java
> Page<Teaching> list(Pageable pageable);
> List<Teaching> findAll(Sort sort);
> ```



`1.接口`

```JAVA
public interface UserJpaRepository extends JpaRepository<User,Integer> {
	
}
```

`2.测试类`

```JAVA
  	@Resource
    private UserJpaRepository userJpaRepository;

    @Test
    public void test4() throws Exception{
        System.out.println("user：" + userJpaRepository.findById(1));
    }
```

==**注意：该接口重点掌握！！**==



### 2.5 动态查询接口

#### 2.5.1 内置API

```JAVA
public interface JpaSpecificationExecutor<T> {
    Optional<T> findOne(@Nullable Specification<T> var1);
    
    List<T> findAll(@Nullable Specification<T> var1);
    
    //分页查询（支持动态查询+分页查询+排序）
    Page<T> findAll(@Nullable Specification<T> var1, Pageable var2);
    
    List<T> findAll(@Nullable Specification<T> var1, Sort var2);
    
    long count(@Nullable Specification<T> var1);
}
```

> 注意：**该接口可以支持动态条件查询！**重点关注`findAll`此方法



#### 2.5.2 动态条件查询

`1.接口`

```JAVA
public interface UserJpaSpecificationExecutor  extends JpaRepository<User,Integer>, JpaSpecificationExecutor<User> {
    
}
```



`2.测试`

```JAVA
 	@Resource
    private UserJpaSpecificationExecutor userJpaSpecificationExecutor;

    @Test
    public void test6() throws Exception{
        //模仿传入的实体类
        User user=new User();
        user.setUserName("eo");
        user.setAge(21);
        
        //采用lambda表达式替换匿名内部类
        List<User> userList = userJpaSpecificationExecutor.findAll((root, criteriaQuery, criteriaBuilder) -> {
            //获取条件参数对象
            Predicate predicate = criteriaBuilder.conjunction();
            //动态拼接条件:用户名
            if (StringUtils.hasText(user.getUserName())) {
                predicate.getExpressions()
                        .add(criteriaBuilder.like(root.get("userName"), "%" + user.getUserName() + "%"));
            }
            //动态拼接条件:年龄
            if (!ObjectUtils.isEmpty(user.getAge())) {
                predicate.getExpressions()
                        .add(criteriaBuilder.lt(root.get("age"), user.getAge()));
            }
            return predicate;
        });

      userList.forEach(user1 -> System.out.println(user1));
    }
```



#### 2.5.3 动态条件+分页

`测试`

```JAVA
   	@Test
    public void test7() throws Exception{
        //模仿传入的实体类
        User user=new User();
        user.setUserName("eo");
        
        //创建分页对象,读取第一页,每次读取一条,按照id降序
        PageRequest pageRequest= PageRequest.of(0, 1, Sort.Direction.DESC, "id");

        //采用lambda表达式替换匿名内部类
        Page<User> userPage = userJpaSpecificationExecutor.findAll((root, criteriaQuery, criteriaBuilder) -> {
            //获取条件参数对象
            Predicate predicate = criteriaBuilder.conjunction();
            //动态拼接条件:用户名
            if (StringUtils.hasText(user.getUserName())) {
                predicate.getExpressions()
                        .add(criteriaBuilder.like(root.get("userName"), "%" + user.getUserName() + "%"));
            }
            //年龄
            if (!ObjectUtils.isEmpty(user.getAge())) {
                predicate.getExpressions()
                        .add(criteriaBuilder.lt(root.get("age"), user.getAge()));
            }
            return predicate;
        }, pageRequest);

        //获取用户列表
        userPage.getContent().forEach(user1 -> System.out.println(user1));
        
        System.out.println("总页数："+userPage.getTotalPages());
        System.out.println("总记录数："+userPage.getTotalElements());
        System.out.println("当前页码："+userPage.getNumber());
        System.out.println("每页显示数量："+userPage.getSize());
    }
```



## 三. JPA查询方法

### 3.1 方法命名查询

#### 3.1.1 命名查询规则

> **按照Spring Data JPA提供的方法命名规则定义方法名称，Spring Data JPA在程序执行的时候会根据方法名称进行解析，并`自动生成查询语句进行查询`**

| 关键词              | 例子                                        | JPQL                                          |
| ------------------- | :------------------------------------------ | --------------------------------------------- |
| `And `              | `findByUserNameAndAge`                      | `… where x.userName= ?1 and x.age = ?2`       |
| `Or`                | `findByUserNameOrAge`                       | `… where x.userName= ?1 or x.age = ?2`        |
| `Is,Equals`         | `findByFirstnameIs`,`findByFirstnameEquals` | `… where x.firstname = ?1`                    |
| `Between`           | `findByStartDateBetween`                    | `… where x.startDate between ?1 and ?2`       |
| `LessThan`          | `findByAgeLessThan`                         | `… where x.age < ?1`                          |
| `LessThanEqual`     | `findByAgeLessThanEqual`                    | `… where x.age <= ?1`                         |
| `GreaterThan`       | `findByAgeGreaterThan`                      | `… where x.age > ?1`                          |
| `GreaterThanEqual`  | `findByAgeGreaterThanEqual`                 | `… where x.age >= ?1`                         |
| `After`             | `findByStartDateAfter`                      | `… where x.startDate > ?1`                    |
| `Before`            | `findByStartDateBefore`                     | `… where x.startDate < ?1`                    |
| `IsNull`            | `findByAgeIsNull`                           | `… where x.age is null`                       |
| `IsNotNull,NotNull` | `findByAge(Is)NotNull`                      | `… where x.age not null`                      |
| `Like`              | `findByFirstnameLike`                       | `… where x.firstname like ?1`                 |
| `NotLike`           | `findByFirstnameNotLike`                    | `… where x.firstname not like ?1`             |
| `StartingWith`      | `findByFirstnameStartingWith`               | `… where x.firstname like ?1` (`value%`)      |
| `EndingWith`        | `findByFirstnameEndingWith`                 | `… where x.firstname like ?1` (`value%`)      |
| `OrderBy`           | `findByAgeOrderByLastnameDesc`              | `… where x.age = ?1 order by x.lastname desc` |
| `Not`               | `findByLastnameNot`                         | `… where x.lastname <> ?1`                    |
| `In`                | `findByAgeIn(Collection<Age> ages)`         | `… where x.age in ?1`                         |
| `NotIn`             | `findByAgeNotIn(Collection<Age> ages)`      | `… where x.age not in ?1`                     |
| `True`              | `findByActiveTrue()`                        | `… where x.active = true`                     |
| `False`             | `findByActiveFalse()`                       | `… where x.active = false`                    |
| `IgnoreCase`        | `findByFirstnameIgnoreCase`                 | `… where UPPER(x.firstame) = UPPER(?1)`       |



#### 3.1.2 方法命名查询例子

`1.接口`

```JAVA
public interface UserJpaRepository extends JpaRepository<User,Integer> {
    
    List<User> findByUserNameLike(String userName);

    List<User> findByIdIn(List<Integer> ids);

}

```



`2.测试`

```JAVA
 	@Resource
    private UserJpaRepository userJpaRepository;

	@Test
    public void test8() throws Exception{
        userJpaRepository.findByUserNameLike("%eo%").forEach(user -> System.out.println(user));
        
        List<Integer> ids=new ArrayList();
        ids.add(1);
        ids.add(3);
        userJpaRepository.findByIdIn(ids).forEach(user -> System.out.println(user));
    }
```



`3.结果`

```
Hibernate: select user0_.id as id1_0_, user0_.address as address2_0_, user0_.age as age3_0_, user0_.user_name as user_nam4_0_ from t_user user0_ where user0_.user_name like ? escape ?

Hibernate: select user0_.id as id1_0_, user0_.address as address2_0_, user0_.age as age3_0_, user0_.user_name as user_nam4_0_ from t_user user0_ where user0_.id in (? , ?)
```



#### 3.1.3 注意

> 方法名称必须要遵循驼峰式命名规则：xxxBy+ 属性名称(首字母大写)+查询条件+(首字母大写)+查询条件....

```JAVA
public class PartTree implements Streamable<PartTree.OrPart> {
   //从PartTree实体类可以看出,xxx关键词必须是以下10个之一
    private static final String QUERY_PATTERN = "find|read|get|query|search|stream";
    private static final String COUNT_PATTERN = "count";
    private static final String EXISTS_PATTERN = "exists";
    private static final String DELETE_PATTERN = "delete|remove";
}
```



### 3.2 JPQL查询

​			JPQL语句：使用JPQL语句，类似之前的HQL语句，语句里面全包含与实体类相关的信息，完全的**面向对象查询语句**

> **格式：** **(select 实体别名.属性名1，....  )** `from 实体名 `**(as 实体别名)** `where 实体别名.实体属性 = 值`

`1.接口`

```JAVA
public interface UserJpaRepository extends JpaRepository<User,Integer> {

   //按命名参数查询,通过:参数名与形参相对应 
   @Query("from User where userName=:name")
   User findUserByHQL(String name);

    //按位置参数查询,参数下标默认从1开始
    @Query("from User where id = ?1")
    User findUserById(Integer id);

    //指定命名参数查询的形参名
    @Query("from User where userName like %:Name% ")
    List<User> findUserNameLike(@Param("Name")String userName);
}
```

==**注意：1.使用按位置参数查询，下标从1开始，要和形参的类型和位置对应起来**==

​				==**<font color=red>2.按命名参数查询，命名参数要和形参名一致或通过@Param注解指定！</font>**==



`2.测试`

```JAVA
    @Resource
    private UserJpaRepository jpaRepository;

    @Test
    public void test() throws Exception{
        User user = jpaRepository.findUserByHQL("eobard");
        System.out.println("user = " + user);
        
        User userById = jpaRepository.findUserById(1);
        System.out.println("userById = " + userById);
        
        List<User> userList = jpaRepository.findUserNameLike("o");
        System.out.println("userList = " + userList);
    }
```



### 3.3 原生SQL查询

#### 3.3.1 返回JPA注解类

`1.接口`

```JAVA
public interface UserJpaRepository extends JpaRepository<User,Integer> {
  
	//使用nativeQuery=true指定为原生SQL,默认为JPQL语句查询
    @Query(value = "select * from t_user where user_name like %:userName% and address like %:address% " ,nativeQuery = true)
    List<User> findUserNameAndAddressLike(String userName,String address);

}

```



`2.测试`

```JAVA
   	@Resource
    private UserJpaRepository jpaRepository;

	@Test
    public void test1() throws Exception{
        System.out.println(jpaRepository.findUserNameAndAddressLike("o", "九龙坡"));
    }
```

> 注意：实际开发能用JPQL就别用原生SQL；因为使用原生SQL，JPA会将原生SQL再次转换为JPQL语句去执行



#### 3.3.2 countQuery属性

```java
public @interface Query {
    String value() default "";

    String countQuery() default "";
    
     boolean nativeQuery() default false;
}
```

* 原生SQL单表查询不用配置countQuery，因为会自动统计总数
* **原生SQL复杂多表查询，JPA不能统计总数，需要手动写统计总数的sql**



#### 3.3.3 返回自定义类（手动映射）

> **JPA返回结果单/多列是Object/Object[]的数组需要手动封装为自定义实体类**
>
> eg：根据老师id分页查询它教的所有课程

1. controller层

```java
public Result<List<TeachingListVO>> listCoursesByTeacherId(
        @RequestParam(defaultValue = "0", required = false) Integer page,
        @RequestParam(defaultValue = "10", required = false) Integer size,
        @RequestParam(required = true) Long teacherId) {
    Pageable pageable = PageRequest.of(page, size);
    Page<TeachingListVO> result = teachingService.listCoursesByTeacherId(pageable, teacherId);
    return Result.success(result.getContent());
}
```



2. service层

```java
@Data
@ApiModel("教师授课VO")
public class TeachingListVO {
    @ApiModelProperty("教师授课表主键ID")
    private Long teachingId;

    @ApiModelProperty("老师名称")
    private String teacherName;

    @ApiModelProperty("老师编号")
    private String teacherNo;

    @ApiModelProperty("课程名称")
    private String courseName;

    @ApiModelProperty("课程编号")
    private String courseNo;

    @ApiModelProperty("课程学分")
    private Integer credit;

    @ApiModelProperty("课程描述信息")
    private String description;
}
```

`````java
Page<TeachingListVO> listCoursesByTeacherId(Pageable pageable, Long teacherId);

@Override
public Page<TeachingListVO> listCoursesByTeacherId(Pageable pageable, Long teacherId) {
    Page<Object[]> result = teachingRepository.listCoursesByTeacherId(pageable, teacherId);
    //将查询数据转换为VO类
    return result.map(objects -> {
        TeachingListVO vo = new TeachingListVO();
        vo.setTeachingId(((Number) objects[0]).longValue());
        vo.setTeacherName((String) objects[1]);
        vo.setTeacherNo((String) objects[2]);
        vo.setCourseName((String) objects[3]);
        vo.setCourseNo((String) objects[4]);
        vo.setCredit((Integer) objects[5]);
        vo.setDescription((String) objects[6]);
        return vo;
    });
}
`````



3. dao层

```java
@Query(
    value = "SELECT " +
    "te.id as teachingId, " +
    "t.name as teacherName, " +
    "t.teacher_no as teacherNo, " +
    "c.name AS courseName, " +
    "c.course_no as courseNo, " +
    "c.credit, " +
    "c.description " +
    "FROM " +
    "t_teacher t " +
    "INNER JOIN t_teaching te ON t.id = te.teacher_id " +
    "INNER JOIN t_course c ON te.course_id = c.id " +
    "WHERE t.id = :teacherId",
    countQuery = "SELECT COUNT(*) " +
    "FROM " +
    "t_teacher t " +
    "INNER JOIN t_teaching te ON t.id = te.teacher_id " +
    "INNER JOIN t_course c ON te.course_id = c.id " +
    "WHERE t.id = :teacherId",
    nativeQuery = true)
Page<Object[]> listCoursesByTeacherId(Pageable pageable, Long teacherId);
```





#### 3.3.4 返回自定义类（投影接口，重点）

> **JPA底层使用动态代理生成实现类，把查询结果设置进去然后返回**
>
> eg：根据老师id分页查询它教的所有课程

1. controller层

`````java
public Result<List<TeachingCourseVO>> listCoursesByTeacherId(
        @RequestParam(defaultValue = "0", required = false) Integer page,
        @RequestParam(defaultValue = "10", required = false) Integer size,
        @RequestParam(required = true) Long teacherId) {
    Pageable pageable = PageRequest.of(page, size);
    Page<TeachingCourseVO> result = teachingService.listCoursesByTeacherId(pageable, teacherId);
    return Result.success(result.getContent());
}
`````



2. 投影接口

```java
//一一对应查询的返回列,且必须是get开头
public interface TeachingCourseVO {
    Long getTeachingId(); //对应返回列的teachingId

    String getTeacherName();//对应返回列的teacherName

    String getTeacherNo();//对应返回列的teacherNo

    String getCourseName();//对应返回列的courseName

    String getCourseNo();//对应返回列的courseNo

    Integer getCredit();//对应返回列的credit

    String getDescription();//对应返回列的description
}
```



3. service层 

```java
Page<TeachingCourseVO> listCoursesByTeacherId(Pageable pageable, Long teacherId);

@Override
public Page<TeachingCourseVO> listCoursesByTeacherId(Pageable pageable, Long teacherId) {
    //这里可以把这个接口转换为对应的实体类返回，不用返回TeachingCourseVO这个接口
    //Page<TeachingCourseVO> teachingCourseVOS = teachingRepository.listCoursesByTeacherId(pageable, teacherId);
    //List<TeachingListVO> teachingListVOS = teachingCourseVOS.getContent()
    //.stream()
    //.map(teachingCourseVO -> {
    //    TeachingListVO teachingListVO = new TeachingListVO();
    //    BeanUtils.copyProperties(teachingCourseVO, teachingListVO);
    //    return teachingListVO;
    //})
    //.collect(Collectors.toList());
    return teachingRepository.listCoursesByTeacherId(pageable, teacherId);
}
```



4. dao层

```java
@Query(
    value = "SELECT " +
    "te.id as teachingId, " +
    "t.name as teacherName, " +
    "t.teacher_no as teacherNo, " +
    "c.name AS courseName, " +
    "c.course_no as courseNo, " +
    "c.credit, " +
    "c.description " +
    "FROM " +
    "t_teacher t " +
    "INNER JOIN t_teaching te ON t.id = te.teacher_id " +
    "INNER JOIN t_course c ON te.course_id = c.id " +
    "WHERE t.id = :teacherId",
    countQuery = "SELECT COUNT(*) " +
    "FROM " +
    "t_teacher t " +
    "INNER JOIN t_teaching te ON t.id = te.teacher_id " +
    "INNER JOIN t_course c ON te.course_id = c.id " +
    "WHERE t.id = :teacherId",
    nativeQuery = true)
Page<TeachingCourseVO> listCoursesByTeacherId(Pageable pageable, Long teacherId);
```





### 3.4 @Modifying

#### 3.4.1 更新

`1.接口`

```JAVA
public interface UserJpaRepository extends JpaRepository<User,Integer> {
   
    @Transactional  //需要手动开启事务,后期在BLL层开事务
    @Modifying		//标记为更新
    @Query("update User set userName=:#{#u.userName},address=:#{#u.address},age=:#{#u.age} where id=:#{#u.id}")	//通过SPEL表达式获取类类型参数值
    int updateUser(@Param("u") User user);

}
```

> 注意：这里使用了类类型作为参数，JPQL语句的占位符要写成`:#{#别称.属性名}`



`2.测试`

```JAVA
	@Resource
    private UserJpaRepository jpaRepository;
  
	@Test
    public void test2() throws Exception{
        //先查
        User user = jpaRepository.findUserById(8);
        System.out.println(user);
        user.setUserName("陈冠希123");
        //后改
        System.out.println(jpaRepository.updateUser(user));
    }
```



#### 3.4.2 删除

`1.接口`

```JAVA
public interface UserJpaRepository extends JpaRepository<User,Integer> {
   
    @Transactional  //需要手动开启事务,后期在BLL层开事务
    @Modifying      //标记为删除
    @Query(value = "delete from t_user where id=:id",nativeQuery = true)//通过原生SQL语句
    int deleteByID(Integer id);

    
    @Transactional
    @Modifying 
    //通过JPQL语句
    @Query("update Person set email = :email where lastName =:lastName") 
	void updateEmailByLastName(@Param("lastName")String lastName,@Param("email")String email);
}
```



`2.测试`

```JAVA
	@Resource
    private UserJpaRepository jpaRepository;
  
	@Test
    public void test3() throws Exception{
      System.out.println(jpaRepository.deleteByID(8));
    }
```



## 四. 注解使用

### 4.1 @Entity

​				用于定义对象将会成为被 JPA 管理的实体，将字段映射到指定的数据库表中。`使用在实体类上`



### 4.2 @Table

​				用于指定实体类对应数据库的表名，`使用在实体类上`

```java
public @interface Table {
    //指定表的名字; 若不指定,表名默认为实体的名字一样
    String name() default "";
}
```



### 4.3 @Id

​				定义属性为数据库的主键，一个实体里面必须有一个，并且必须和 @GeneratedValue 配合使用和成对出现



### 4.4 @GeneratedValue

​				主键生成策略

```JAVA
public @interface GeneratedValue {
    //Id的生成策略:SEQUENCE(序列)、IDENTITY(自增)
    GenerationType strategy() default AUTO;
    
    //自定义扩展id策略
    String generator() default "";
}
```



#### 4.4.1 扩展主键生成策略

​				除了JPA自带的主键生成策略，还可以使用扩展主键生成策略

* **uuid**：采用128位的uuid算法生成主键，uuid被编码为一个32位16进制数字的字符串。占用空间大(字符串类型)。
* **assigned**：在插入数据的时候主键由程序处理(很常用)
* **guid**：采用数据库底层的guid算法机制，对应MYSQL的uuid()函数，SQL Server的newid()函数，ORACLE的rawtohex(sys_guid())函数等。

> **eg：自己指定主键输入**

```JAVA
@Entity
@Table
public class Address {
    @Id
    //这里需要注意generator的值和name的值需要对应,strategy为具体的策略，除了上诉几个外还可以自定义自己的生成策略,填写完整的类名称即可
    @GeneratedValue(generator = "myAssigned")
    @GenericGenerator(name = "myAssigned",strategy = "assigned")	//指定主键为自己指定输入
    private Integer id;

    private String position;
}
```



> **eg：使用雪花算法生成主键**

1. 自定义主键生成器

```java
public class SnowflakeIdGenerator implements IdentifierGenerator {

    @Override
    public Serializable generate(SharedSessionContractImplementor session, Object object) throws HibernateException {
        return generateId();
    }

    private Long generateId() {
        return SnowflakeUtil.nextId();
    }
}
```

````java
import java.net.NetworkInterface;

//雪花算法工具类
public class SnowflakeUtil {

    private static final long START_TIMESTAMP = 1609459200000L;

    private static final long NODE_BITS = 5L;

    private static final long SEQUENCE_BITS = 12L;

    private static final long NODE_MAX = -1L ^ (-1L << NODE_BITS);

    private static final long SEQUENCE_MASK = -1L ^ (-1L << SEQUENCE_BITS);

    private static long nodeId;

    private static long sequence = 0L;

    private static long lastTimestamp = -1L;

    static {
        nodeId = getNodeid();
    }

    public static long nextId() {
        long timestamp = System.currentTimeMillis();

        if (timestamp < lastTimestamp) {
            throw new RuntimeException("时钟回拨");
        }

        if (timestamp == lastTimestamp) {
            sequence = (sequence + 1) & SEQUENCE_MASK;
            if (sequence == 0) {
                timestamp = tilNextMillis(lastTimestamp);
            }
        } else {
            sequence = 0L;
        }

        lastTimestamp = timestamp;

        return (timestamp - START_TIMESTAMP) << (NODE_BITS + SEQUENCE_BITS)
                | (nodeId << SEQUENCE_BITS)
                | sequence;
    }

    private static long getNodeid() {
        try {
            byte[] mac = NetworkInterface.getByInetAddress(java.net.InetAddress.getLocalHost()).getHardwareAddress();
            if (mac == null) {
                return 1L;
            }
            long value = 0L;
            for (int i = 0; i < Math.min(mac.length, 2); i++) {
                value <<= 8;
                value |= mac[i] & 0xff;
            }
            return value & NODE_MAX;
        } catch (Exception e) {
            return 1L;
        }
    }

    private static long tilNextMillis(long lastTimestamp) {
        long timestamp = System.currentTimeMillis();
        while (timestamp <= lastTimestamp) {
            timestamp = System.currentTimeMillis();
        }
        return timestamp;
    }
}
````



2. 配置主键

```java
@GeneratedValue(generator = "snowflake-generator")
@GenericGenerator(name = "snowflake-generator", strategy = "com.eobard.util.SnowflakeIdGenerator")
@Column(name = "id")
private Long id;
```





### 4.5 @Basic

​				表示属性是到数据库表的字段的映射。如果实体的字段上没有任何注解，默认即为 @Basic

```JAVA
public @interface Basic {
    //可选，EAGER（默认）：立即加载；LAZY：延迟加载。（LAZY主要应用在大字段上面）
    FetchType fetch() default EAGER;
    
    //可选。这个字段是否可以为null，默认是true。
    boolean optional() default true;
}

```



### 4.6 @Transient

​				表示该属性并非一个到数据库表的字段的映射，表示非持久化属性。JPA 映射数据库的时候忽略它，与 @Basic 相反的作用



### 4.7 @Column

​				定义该属性对应数据库中的列名

```java
public @interface Column {
    //数据库中的表的列名；可选，如果不填写认为字段名和实体属性名一样。
    String name() default "";
    
    //是否唯一。默认flase，可选。
    boolean unique() default false;
    
    //数据字段是否允许空。可选，默认true。
    boolean nullable() default true;
    
    //执行insert操作的时候是否包含此字段，默认，true，可选。
    boolean insertable() default true;
    
    //执行update的时候是否包含此字段，默认，true，可选。
    boolean updatable() default true;
    
    //表示该字段在数据库中的实际类型。
    String columnDefinition() default "";
    
    //数据库字段的长度，可选，默认255
    int length() default 255;
}

```



### 4.8 @Temporal

​				用来设置 Date 类型的属性映射到对应精度的字段。

* @Temporal(TemporalType.DATE)	// date 映射为日期 （只有日期）
*  @Temporal(TemporalType.TIME)   // time 映射为日期 （只有时间）
*  @Temporal(TemporalType.TIMESTAMP)   // date time  映射为日期  （日期+时间）



### 4.9 审计功能

​			对于一些数据有创建时间和修改时间严格监控的表，JPA提供了注解为我们自动赋值

* @CreatedBy ：创建人
* @LastModifiedBy：修改人
* `@CreatedDate：创建时间`
* `@LastModifiedDate ：修改时间`

> 其中创建人和修改人功能需要实现具体接口，可自行搜索



`1.SpringBoot开启审计功能`

```JAVA
@SpringBootApplication
@EnableJpaAuditing	//开启审计监听
public class SpringdatajpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringdatajpaApplication.class, args);
    }

}
```

`````properties
# 使用spring管理Session上下文
spring.jpa.properties.hibernate.current_session_context_class=org.springframework.orm.hibernate5.SpringSessionContext
`````





`2.实体类开启监听+声明创建和修改时间字段`

```JAVA
@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity
@Table
@EntityListeners(AuditingEntityListener.class)	//开启实体类监听
public class Address {
    @Id
    @GeneratedValue(generator = "myAssigned")
    @GenericGenerator(strategy = "assigned",name = "myAssigned")
    private Integer id;

    private String position;

    @CreatedDate    //创建时间
    @Temporal(TemporalType.TIMESTAMP)  //注解只能用于java.util.Date或java.util.Calendar 类型
    private Date createTime;

    @LastModifiedDate   //修改时间
    private LocalDateTime updateTime;	//LocalDateTime类型就不需要加Temporal注解
}

```



`3.测试`

​			在执行增加、修改操作的时候，数据库的createTime和updateTime两个字段会被程序自动赋值



### 4.10 乐观锁

​			通过使用@Version来实现乐观锁配置，防止高并发

```JAVA
@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity
@Table
@EntityListeners(AuditingEntityListener.class)
public class Address {
    @Id
    @GeneratedValue(generator = "myAssigned")
    @GenericGenerator(strategy = "assigned",name = "myAssigned")
    private Integer id;

    private String position;

    @CreatedDate    
    @Temporal(TemporalType.TIMESTAMP)  
    private Date createTime;

    @LastModifiedDate   
    @Temporal(TemporalType.TIMESTAMP)
    private Date updateTime;


    @Version	//配置乐观锁,用long类型
    private Long version;
}
```



### 4.11 @OneToOne

```JAVA
public @interface OneToOne {
        //关系目标实体，非必填，默认该字段的类型。
        Class targetEntity() default void.class;
        
    	//cascade 级联操作策略
        /*
        1. CascadeType.PERSIST 级联新建
        2. CascadeType.REMOVE 级联删除
        3. CascadeType.REFRESH 级联刷新
        4. CascadeType.MERGE 级联更新
        5. CascadeType.ALL 四项全选
        6. 默认，关系表不会产生任何影响
        */
        CascadeType[] cascade() default {};
    
        //数据获取方式EAGER(立即加载)/LAZY(延迟加载)
        FetchType fetch() default EAGER;
    
       
    
        //关联关系被谁维护的。 非必填，一般不需要特别指定。
        //注意：只有关系维护方才能操作两者的关系。被维护方即使设置了维护方属性进行存储也不会更新外键关联。1）mappedBy不能与@JoinColumn或者@JoinTable同时使用。2）mappedBy的值是指另一方的实体里面属性的字段，而不是数据库字段，也不是实体的对象的名字。既是另一方配置了@JoinColumn或者@JoinTable注解的属性的字段名称。
        String mappedBy() default "";
    
    
        //是否级联删除。和CascadeType.REMOVE的效果一样。两种配置了一个就会自动级联删除
        boolean orphanRemoval() default false;
}

```



### 4.12 @OneToMany

```JAVA
public @interface OneToMany {
        Class targetEntity() default void.class;
    
      //cascade 级联操作策略
        /*
        1. CascadeType.PERSIST 级联新建
        2. CascadeType.REMOVE 级联删除
        3. CascadeType.REFRESH 级联刷新
        4. CascadeType.MERGE 级联更新
        5. CascadeType.ALL 四项全选
        6. 默认，关系表不会产生任何影响
        */
        CascadeType[] cascade() default {};
    
        //数据获取方式EAGER(立即加载)/LAZY(延迟加载)
        FetchType fetch() default LAZY;
    
        //关系被谁维护，单项的。注意：只有关系维护方才能操作两者的关系。
        String mappedBy() default "";
    
        //是否级联删除。和CascadeType.REMOVE的效果一样。两种配置了一个就会自动级联删除
        boolean orphanRemoval() default false;
}

```



### 4.13 @ManyToOne

```JAVA
public @interface ManyToOne {
        Class targetEntity() default void.class;
    
        CascadeType[] cascade() default {};
    
        FetchType fetch() default EAGER;
    
        boolean optional() default true;
}

```



### 4.14 @ManyToMany

```JAVA
public @interface ManyToMany {
        Class targetEntity() default void.class;
    
        CascadeType[] cascade() default {};
    
        FetchType fetch() default LAZY;
    
        String mappedBy() default "";
}
```





## 五.关联关系

### 5.1 一对一

> 一个学生属于一个班级

1. 数据库表设计

```SQL
t_student: 
			id   主键
			grade_id	外键
			student_name
            
 t_grade
 		   id	主键
 		   grade_name
```



2. 实体类

```JAVA
@Data
@Entity
@Table(name = "t_student")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String studentName;

	//一对一关联
    @OneToOne(cascade = CascadeType.PERSIST)
    //关联的外键字段:  学生表中引用年级表的外键名称
    @JoinColumn(name = "grade_id")
    private Grade grade;

}

==========================================================================================

@Data
@Entity
@Table(name = "t_grade")
public class Grade {

    @Id
    @GeneratedValue( strategy = GenerationType.IDENTITY)
    private Integer id;

    private String gradeName;
}
```





3. dao层

```JAVA
public interface StudentRepository extends JpaRepository<Student,Integer> {
    
}
```



4. 测试类

```JAVA
	@Resource
    private StudentRepository studentRepository;

    @Test
    void contextLoads() {
        //一對一添加
        Grade g=new Grade();
        g.setGradeName("s1");
        Student s=new Student();
        s.setStudentName("eobard");
        s.setGrade(g);
        studentRepository.save(s);

        //一对一查询
        Student student = studentRepository.findById(1).get();
        System.out.println("student = " + student);
        System.out.println(student.getGrade().getGradeName());

    }
```





### 5.2 一对多

> 一个角色有多个用户

1. 数据库表设计

```SQL
t_users: 
			id   主键
			role_id	外键
			user_name
            
 t_role
 		   id	主键
 		   role_name
```



2. 实体类设计

   ```java
   @Entity
   @Table(name = "t_users")
   public class Users {
   
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Integer id;
   
       private String userName;
   
       //多个用户对应一个角色
       @ManyToOne
       //关联的外键字段:用户表引用角色表的外键名称
       @JoinColumn(name = "role_id")
       private Role role;
   
   	//省略getter、settter
   }
   
   ======================================================================================
   
   @Entity
   @Table(name = "t_role")
   public class Role {
   
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Integer id;
   
       private String roleName;
   
       //一个角色有多个用户,关系由用户来维护
       //mappedBy:多方引用一方的属性名,表示让对方维护关系
       @OneToMany(mappedBy = "role",cascade = CascadeType.ALL)
       private Set<Users> users=new HashSet<>();
       
       //省略getter、settter
   
   }
   ```

   

3. dao层

```JAVA
public interface RoleRepository extends JpaRepository<Role,Integer> {
}
```



4. 测试类

```JAVA
  	@Resource
    private RoleRepository roleRepository;

    @Test
    public void testAdd(){
        Role role=new Role();
        role.setRoleName("销售部门");

        Users u=new Users();
        u.setUserName("周杰伦");
        u.setRole(role);

        Users u1=new Users();
        u1.setUserName("陈冠希");
        u1.setRole(role);

        role.getUsers().add(u);
        role.getUsers().add(u1);

        roleRepository.save(role);

    }


	@Test
	 /**
     * 这里需要加上事务或者将Role实体类一对多中的属性设置为立即加载,不然会出现懒加载异常
     * 因为在单元测试中,出现了两次的dao层操作是处于不同的事务中的
     */
    @Transactional
    public void testQuery(){
        Role role = roleRepository.findById(1).get();
        System.out.println(role);
        role.getUsers().forEach(users -> System.out.println("users = " + users));
    }
```



### 5.3 多对多

1. 数据库表设计

```SQL
project
		id 
		project_name
		
employee
		id
		employee_name
		
		
t_employee_project		中间表
		project_id  	外键
		employee_id		外键
```



2. 实体类设计

```JAVA
@Getter
@Setter
@Entity
public class Project {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String projectName;

    /**
     * 配置项目到员工的多对多关系
     * 1.声明表关系的配置
     *          @ManyToMany(targetEntity = Employee.class)      //多对多
     *              targetEntity：代表对方的实体类字节码
     *
     * 2.配置中间表（包含两个外键）
     *          @JoinTable
     *              name : 中间表的名称
     *              joinColumns：配置当前对象在中间表的外键
     *          @JoinColumn的数组
     *              name：外键名
     *              referencedColumnName：参照的主表的主键名
     *              inverseJoinColumns：配置对方对象在中间表的外键
     */
    @ManyToMany(targetEntity = Employee.class,cascade = CascadeType.ALL)
    //配置第三张表（中间表）
    //name属性：第三张表的表名称
    @JoinTable(name = "t_employee_project",
            //joinColumns：当前对象在中间表中的外键,referencedColumnName:当前对象的主键名称
            joinColumns = @JoinColumn(name = "project_id",referencedColumnName = "id"),
            //inverseJoinColumns:对方对象在中间表的外键,referencedColumnName:另一方对象的主键名称
       		inverseJoinColumns = @JoinColumn(name = "employee_id",referencedColumnName = "id"))
    private Set<Employee> employees=new HashSet<>();


}
===========================================================================================
    
    
    

@Getter
@Setter
@Entity
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String employeeName;

    @ManyToMany(mappedBy = "employees")//表示让对方维护关系,当前方放弃关系的维护
    private Set<Project> projects=new HashSet<>();
}
```



3. dao层

```JAVA
public interface ProjectRepository extends JpaRepository<Project,Integer> {
}
```



4. 测试

> 注意：必须在事务环境中运行，否则会出现 `detached entity passed to persist `错误

```java
 	@Resource
    private ProjectRepository projectRepository;

    @Test
    @Transactional
    @Commit
    public void test(){
        Project project1=new Project();
        project1.setProjectName("cloud项目");

        Project project2=new Project();
        project2.setProjectName("boot项目");

        Employee e1=new Employee();
        e1.setEmployeeName("张三");

        Employee e2=new Employee();
        e2.setEmployeeName("李四");

        Employee e3=new Employee();
        e3.setEmployeeName("王五");

        project1.getEmployees().add(e1);
        project1.getEmployees().add(e2);

        project2.getEmployees().add(e2);
        project2.getEmployees().add(e3);

        projectRepository.save(project1);
        projectRepository.save(project2);
    }
```





### 5.4 注意

1. 在关联关系的实体类中，若存在toString()方法，则一定不要把关联的关系属性输出，会存在问题！如果要获取关联关系的属性，则直接通过getter去获取对应的关联关系。

> eg： 如用户和角色之间

```JAVA
@Entity
@Table(name = "t_role")
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String roleName;

    @OneToMany(mappedBy = "role",cascade = CascadeType.ALL)
    private Set<Users> users=new HashSet<>();

    
    //只输出当前类的属性
    @Override
    public String toString() {
        return "Role{" +
                "id=" + id +
                ", roleName='" + roleName + '\'' +
                '}';
    }
```



2. 级联cascade属性：`某方`save的同时要去save或操作`其它方`，就在`某方设置级联`

> eg：添加班级的同时将学生添加进去，则在班级类设置级联操作



3. mappedBy属性：`某方`主动维护关系，`另一方`放弃关系维护(`另一方`的实体类中填`某方`引用当前方的属性值)

> eg：一个项目有多个员工，项目类来添加多个员工类，表示项目类主动维护关系。`即被动的一方放弃维护权`

```JAVA
public class Employee {//代表另一方
  	//省略其它代码
    
    @ManyToMany(mappedBy = "employees")//表示让对方维护关系,当前方放弃关系的维护
    private Set<Project> projects=new HashSet<>();
}
```

```JAVA
public class Project {//代表某方,项目类主动维护关系
	//省略其它代码
  
    @ManyToMany(targetEntity = Employee.class,cascade = CascadeType.ALL)
    private Set<Employee> employees=new HashSet<>();

}
```








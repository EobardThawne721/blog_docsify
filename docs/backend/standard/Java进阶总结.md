# Java进阶总结

## Effective Java总结

### 1. 建造者模式

#### 1.1 前言

> 问题：
>
> ​					问题1：对于一个类有多个构造函数或者想要在不同阶段设置不同属性值时,客户端会很难编写,并且通过JavaBean的setter方式会存在一定线程安全的问题。
>
> ​					问题2：有些对象的属性必须按照一定的顺序赋值,由各种对象数据组合而来
>
> 
>
> 解决：通过设计模式之建造者(Builder)模式
>
> 概念：将一个对象的内部的属性赋值分离开来,可以保证线程的安全



#### 1.2 传统方式

```JAVA
	//传统方式:	
public class Person{
			private String name;
			private Character sex;
			private int age;
			private String info;
		//getter,setter
		//多个重载Person构造函数
	}
```

#### 1.3 建造者模式1:

```JAVA
public class Person{
        private String name;
        private Character sex;
        private int age;
        private String info;
			
		private Person(PersonBuilder personBuilder) {
            name = personBuilder.name;
            sex = personBuilder.sex;
            age=personBuilder.age;
            info=personBuilder.info;
		}

		//静态内部类
		public static class PersonBuilder {
			private String name;
			private Character sex;
			private int age;
			private String info;

		//默认需要的构造函数
		public PersonBuilder(String name, Character sex) {
			this.name = name;
			this.sex = sex;
		}

		//下面是可选参数
		public PersonBuilder setAge(int val) {
			this.age = val;
			return this;
		}

		public PersonBuilder setInfo(String val) {
			this.info = val;
			return this;
		}
		
		//通过当前builder返回真正对象
		public Person build() {
			return new Person(this);
		}
	     }
	}
		

    //测试:  流式的API
    Person p=new Person.PersonBuilder("陈冠希",'男').setAge(30).build();
    //输出	Person [name=陈冠希, sex=男, age=30, info=nul]
```

#### 1.4 建造者模式2: 

```JAVA
public class Car{
				private String carName;
				private String wheel;
				private Double price;
				}

			public interface Builder{
				Builder buildCarName(String name);
				Builder buildCarWheel(String wheel);			
				Builder buildCarPrice(Double price);
				Car build();
				}

			public class CarBuilder{
				private Car car;

				public CarBuilder() {
					car = new Car();
					}
				
				@Override
				public CarBuilder buildCarName(String name) {
				car.setCarName(name);
					return this;
				}


				@Override
				public CarBuilder buildWheel(String wheel) {
				car.setWheel(wheel);
				return this;
				}

				@Override
				public CarBuilder buildPrice(double price) {
				car.setPrice(price);
				return this;
				}	

				@Override
				public Car build() {
				return this.car;
				}
				}
			


			public class DirectorCar {

				public static Car constructCar(ICarBuilder builder) {
					builder.buildCarName("toyota");
					builder.buildWheel("丰田");
					builder.buildPrice(1000d);
					return builder.build();
					}   
				}

		//测试:   指导者只需要传入数据让具体建造者来创建对象
			Car car = DirectorCar.constructCar(new CarBuilder());
		//输出	Car [carName=toyota, wheel=丰田, price=1000.0]	

```

#### 1.5建造者模式3(推荐使用): 

```JAVA
public class Car{
				private String carName;
				private String wheel;
				private Double price;
				}

			public interface Builder{
				Builder buildCarName(String name);
				Builder buildCarWheel(String wheel);			
				Builder buildCarPrice(Double price);
				Car build();
				}

			public class concreteBuilder{
				private Car car;

				public Car Builder() {
					car = new Car();
					}
				
				@Override
				public CarBuilder buildCarName(String name) {
				car.setCarName(name);
					return this;
				}


				@Override
				public CarBuilder buildWheel(String wheel) {
				car.setWheel(wheel);
				return this;
				}

				@Override
				public CarBuilder buildPrice(double price) {
				car.setPrice(price);
				return this;
				}	

				@Override
				public Car build() {
				return this.car;
				}
				}

		//省略了指导者的工作,综合了建造者1的方式,直接调用
		Car car=new concreteBuilder().buildCarName("奥迪Q5")
				               .buildPrice(88800d)
					.buildWheel("兰博基尼的轮胎").build();

		//输出   Car [carName=奥迪Q5, wheel=兰博基尼的轮胎, price=88800.0]
	
```



### 2. 实例化问题

> 问题：某些工具类不希望被实例化,在缺少显示构造器的情况下，编译器还是回自动提供无参构造
>
> 解决方法:  给工具类添加私有的无参构造方法



### 3. 不必要的对象

> 问题4：避免创建不必要的对象
>
> 解决方法：要优先使用基本类型而不是装箱类型，要当心无意识的自动装箱, 使用基本类型可以提高效率在实体类中的方法，尽量将公共的实例放在类中的静态或不可变属性中缓存起来可以提高性能



### 4. 内存泄漏

> 问题5：避免内存泄漏的问题
>
> 解决方法：只要类是自己管理内存，就应该警惕内存泄漏问题，一旦某个元素被释放掉，就应该把该元素包含的任何对象引用都清空



### 5. LSP原则

> 里氏替换原则:一个类型的任何重要属性也将适用于它的子类,该类的任何方法,在它的子类也应该同样运行的很好



### 6. 越过泛型检查

```JAVA
	//反射可以越来泛型的检查
		ArrayList<Integer> list=new ArrayList<Integer>();
		list.add(1);
		list.add(1);
		list.add(1);
		Class clazz = list.getClass();
		Method method = clazz.getMethod("add", Object.class);
		//通过本身实例化的对象来继续添加新的东西，如果是clazz.newInstance()则是一个新的对象,将不会添加到 list中
		method.invoke(list, "zs");

		//输出list [1, 1, 1, zs]	
```



### 7.  通过反射仿Spring

​			通过配置文件来动态执行某个类的某个方法（类似于spring,运行时才知道具体运行哪个类和方法）

```JAVA
//在同一个test包下
		public  class A{
			public void say(){
					System.out.println("one say");
				}
		}	


		public  class B{
			public void want(){
					System.out.println("two want");
				}
		}
```

```properties
##配置文件(类似于spring)   LikeSpring.txt
className=com.eobard.test.A 
method=want
```

```JAVA
//测试
			Properties properties =new Properties();
			properties.load(new FileReader("LikeSpring.txt"));
				//读取配置文件的方法名
			String clazz = properties.getProperty("className");
			String method = properties.getProperty("method");
				//读取配置文件的类名
			Class<?> class1 = Class.forName(clazz);
				//动态执行方法
			Method m = class1.getMethod(method);
				//通过反射实例化类
			m.invoke(class1.newInstance());
```



### 8. Java8接口新特性

​			在jdk8之前，实现了接口就必须重写方法，**但在jdk8之后如果实现了接口可以不用重写方法，jdk8之后的接口可以写出具体的实现方法，但前提是必须要用default修饰**



<font color="green">不重写接口方法</font>

```JAVA
public interface UserService {

    default boolean isExist(String account){
        return  true;
    }
}
```

```JAVA
public class UserServiceImpl implements UserService {

   //不实现任何方法
}
```

```JAVA
 public static void main(String[] args) {
        UserService userService=new UserServiceImpl();
        System.out.println(userService.isExist("root"));
    }

//默认返回接口的值:true
```



<font color="green">重写接口方法</font>

```JAVA
public class UserServiceImpl implements UserService {

    @Override
    public boolean isExist(String account) {
        return false;
    }
}
```

```JAVA
 public static void main(String[] args) {
        UserService userService=new UserServiceImpl();
        System.out.println(userService.isExist("root"));
    }

//返回实现类的值:false
```



## 阿里巴巴开发手册

### 1.常量命名规则

​		常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长。

```JAVA
public final int MAX_STOCK_COUNT=10;
```



### 2.其它类名命名规则	

* 抽象类命名使用 Abstract 或 Base 开头；异常类命名使用 Exception 结尾；测试类 命名以它要测试的类的名称开始，以 Test 结尾。
* 如果模块、接口、类、方法使用了设计模式，在命名时需体现出具体模式。将设计模式体现在名字中，有利于阅读者快速理解架构设计理念。

```JAVA
public class OrderFactory;	//工厂模式

public class LoginProxy;	//代理模式

public class ResourceObserver;//观察者模式
```

* 工具类不允许有 public 或 default 构造方法。

  

  

  




### 3.布尔类型变量命名规则

​		POJO 类中布尔类型的变量，都不要加 is 前缀，否则部分框架解析会引起序列化错误。

> ​	定义为基本数据类型 Boolean isDeleted 的属性，它的方法也是 isDeleted()，RPC框架在反向解析的时候，“误以为”对应的属性名称是 deleted，导致属性获取不到，进而抛出异常。



### 4. 接口属性和方法命名规则

​		接口类中的方法和属性不要加任何修饰符号（public 也不要加），保持代码的简洁 性，并加上有效的 Javadoc 注释。



### 5. Long形赋值

​		在 long 或者 Long 赋值时，数值后使用大写的 L，不能是小写的 l，小写容易跟数字 1 混淆，造成误解。



### 6.equals方法使用

​		Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals。

> ​	正例："test".equals(object);  		√
>
> ​	反例：object.equals("test");		   ×



### 7. 类中属性命名

* 【强制】所有的 POJO 类属性必须使用包装数据类型。

  > 数据库的查询结果可能是 null，因为自动拆箱，用基本数据类型接收有 NPE 风险。



### 8. 集合处理

* 【强制】不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator 方式，如果并发操作，需要对 Iterator 对象加锁。

  ```java
  //错误案例    
  ArrayList<String> list = new ArrayList<>();
          list.add("1");
          list.add("2");
          list.add("3");
         for (String i : list) {
              if("3".equals(i)){
                  list.remove(i);
             }
          }
  //这段代码会出现java.util.ConcurrentModificationException异常
  
  
  
  //正确案例
    Iterator<String> iterator = list.iterator();
          while (iterator.hasNext()) {
              String item = iterator.next();
              if ("1".equals(item)) {
                  iterator.remove();
            }
         }
  ```

  

### 9. 并发问题

​		在高并发场景中，避免使用”等于”判断作为中断或退出的条件。

> ​	判断剩余奖品数量等于 0 时，终止发放奖品，但因为并发处理错误导致奖品数量瞬间变成了负数，这样的话，活动无法终止。

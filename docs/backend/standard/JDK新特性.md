# JDK8

## 一.lambda表达式

### 1.1 语法

​				左侧是实现接口方法中的参数列表,右侧是表达式中所执行的功能

> 两点注意：
>
> 1. 表达式需要"函数式接口"的支持:  即接口中只有一个抽象方法的接口称为函数式接口,可以在接口用注解@FunctionalInterface声明,表示该接口只能有一个抽象方法
> 2. 是一个匿名函数，可以让代码更加简洁明了



### 1.2 四大内置函数式接口


- `Consumer<T>`: `void accept(T t);`  // 消费型接口,无返回值

- `Supplier<T>`: `T get();`  // 供给型接口,返回T类型

> 供给型接口使用注意: 首先在方法中确定大致的功能，然后在接口中具体实现细节  
> (比如：实例人，首先方法中肯定先声明类对象，返回人这个对象，然后具体是什么样的人就在接口中用 new 一个类然后用构造方法就完成了)

- `Function<T,R>`: `R apply(T t);`  // 函数型接口,传入T类型,返回一个R类型

- `Predicate<T>`: `boolean test(T t);`  // 预判型接口,返回一个 boolean 值

  - `BiPredicate<T, R>`: `boolean test(T t, R r);`  // 子类与预判型接口
  

`使用方式:
	方法1:直接用另一个方法让这个接口作为参数类型,然后调用就用lambda来完成接口的功能
	方法2:直接声明接口然后实例lambda表达式,最后用声明接口的变量来调用抽象方法,根据返回值来接收结`



### 1.3 其他用法

```
当有接口并且里面有一个抽象方法时:
	new 接口名(){ 
		//实现抽象方法
	}				//这就是匿名内部类可以用lambda表达式替换
```



## 二.Stream流

### 2.1 创建流(三种方式)


1. 通过实例化 Collection 系列集合，用实例调用 `stream()` 返回一个 `Stream<T>` 对象       ==> 获取一个集合流
2. 通过 `Arrays.stream(T[] t)` 返回一个 `Stream<T>` 对象                                                   ==> 获取一个数组流
3. 通过 `Stream.of(T... values)` 返回一个 `Stream<T>` 对象，这里的可变参数只写值，不用声明  ==> 获取一个自定义值流




### 2.2 常用中间操作

​				 中间的操作不会输出任何结果，只有在最后的终止操作才会输出新结果

#### 2.2.1 过滤

​			排除元素(只接收lambda形式)，首先让数据变成流对象,然后链式调用**filter( (e)->e.条件表达式 )**,返回一个新流对象接收。		

> 里面的lambda表达式是Function接口	



#### 2.2.2 限制

​			限制获取流中前L个元素，首先让数据变成流对象,然后链式调用**limit(long L)**



#### 2.2.3 跳过

​			跳过前K个元素，首先让数据变成流对象,然后链式调用**skip(long K)**



#### 2.2.4 映射(★★★★)

​			将元素转换成其他形式或者提取信息(只接受lambda)，首先让数据变成流对象,然后链式调用**map((e)->e.表达式 )** 			

> 里面的lambda表达式是Function接口



#### 2.2.5 排序(★★)

​			将元素按照字典型排序，首先让数据变成流对象,然后链式调用 **sorted()**		//自然排序

>  	**sorted(Comparator 接口)			//要么lambda表示自定义排序,要么匿名内部类形式自定义排序**

`Comparator比较接口表达`

```JAVA
//正数表示升序,负数表示降序(就是后面条件返回的值,可以加上+,-号)	
Comparator<Integer> com=(x,y)->{return Integer.compare(x,y);}//比较两个数的大小
Comparator<Integer> com=(x,y)->{return String str1.compareTo(str2);} //按照字典来比较字符串的顺序
```

eg：

```java
//排序(根据长度从小到大排序)
List<String> fruit = Arrays.asList("banana", "apple", "cherry");

//或者使用lambda表达式：Comparator.comparingInt(String::length)
fruit.sort((a, b) -> Integer.compare(a.length(), b.length()));

System.out.println(fruit);	//[apple, banana, cherry]
```







#### 2.2.6 归约(★★★★)

​			将集合中的所有元素经过指定运算，折叠成一个元素输出，如：求最值、平均数等，首先让数据变成流对象,然后链式调用**reduce(初始值,BiFunction<T t1,T t2>接口的lambda表达式 )**

```JAVA
//eg1:
	XXX.stream().reduce(0,(x,y)->x+y);		//计算初始值为0+x+y值
//eg2:
	XXX.stream().reduce(0,Integer::sum) 	//计算所有Integer的总值
```



#### 2.2.7 收集结果(★★)

​				转换为一个新的集合，首先让数据变成流对象,然后链式调用**map(筛选想要的).collect(XXX)**

> **XXX可以是下面这几种**
>
> * **Collectors.toList()																 //返回一个List集合,可以重复元素**	
> * **Collectors.Set()      													//返回一个set集合,不能重复元素**
> * **`Collectors.toCollection(Supplier<T>接口类型)`	//返回自定义的类型,可在供给型接口具体写lambda表达式**
> * **Collectors.counting()                                     //返回一个Long**





#### 2.2.8 是否匹配条件

​			首先让数据变成流对象，然后链式调用 **`anyMatch( Predicate<T>接口的lambda表达式)`**   //返回boolean值



#### 2.2.9 去重

​				去掉重复的元素(必须让数据重写hashcode()和equals()才能去重)，首先让数据变成流对象,然后链式调用**distinct()**





#### 2.2.10 分组操作

> **分组操作分为二分类和多分类，其中二分类是多分类的特例，能用二分类的地方，可以用多分类实现**



* **二分操作：Collectors.partitioningBy(Function<? super T, ? extends K> classifier, Collector<? super T, A, D> downstream)，按照classifier分类后按downstream操作进行后续处理**

  ```java
  //      二分场景
  List<Integer> age = Arrays.asList(20, 17, 19, 21);
  
  
  //  eg:按是否成年分为两组
  Map<Boolean, List<Integer>> res1 = age.stream()
      								.collect(Collectors.partitioningBy(i -> i > 18));
  
  //      false=>[17] , number:1
  //      true=>[20, 19, 21] , number:3
  res1.forEach((flag, res) -> System.out.println(flag + "=>" + res + " , number:" + res.size()));
  ```

  

* **多分操作：Collectors.groupingBy(Function<? super T, ? extends K> classifier, Collector<? super T, A, D> downstream)：按照classifier分类后按downstream操作进行后续处理**

  ```java
  //       按条件多分组
  List<String> name = Arrays.asList("Alice", "Bob", "Ana", "Charlie", "Brown");
  
  //  eg:按开头字母分组
  Map<Character, List<String>> result = name.stream()
      									.collect(
      							Collectors.groupingBy(str ->str.charAt(0)));
  
  //      A=>[Alice, Ana]
  //      B=>[Bob, Brown]
  //      C=>[Charlie]
  result.forEach((c, res) -> System.out.println(c + "=>" + res));
  ```

  ```java
  //  eg:按字母长度>=5的分组,并统计每个组的数量
  Map<Boolean, Long> result2 = name.stream()
      						.collect(Collectors.groupingBy(str -> str.length() >= 5, Collectors.counting()));
  //      false=>2
  //      true=>3
  result2.forEach((c, res) -> System.out.println(c + "=>" + res));
  ```

  







### 2.3 常用终止操作

​					也称之为"惰性求值"，**一次性执行中间操作的全部结果**



#### 2.3.1 遍历(★★★★ )

```JAVA
//简单遍历
list.forEach(e->System.out.println(e));	
//判断条件遍历
list.forEach(e->{
    	if(condition){
    			System.out.println(e);
        }  
});
//链式调用
list.stream()
			 .filter((e)->e.条件表达式)
			 .limit(X)								//取前面X条数据,可省略不写
			 .forEach(System.out::println);			//让foreach循环遍历有条件的List集合
```



#### 2.3.2 获取第一个值

​						中间操作接收的新流对象.findFirst();  

> **返回一个`Optional<T>`的容器对象，取得里面的值直接调用get()方法即可**



#### 2.3.3 流中总个数

​						中间操作接收的新流对象.count();

> 求得流中的总个数,返回一个long类型



#### 2.3.4 流中的最大值

​					中间操作接收的新流对象.max( Comparator<T t,T t>的lambda表达式 );

> 返回一个`Optional<T>`的容器对象，取得里面的值直接调用get()方法即可



#### 2.3.5 流中的最小值

​					中间操作接收的新流对象.min( Comparator<T t,T t>的lambda表达式 );

> **返回一个`Optional<T>`的容器对象，取得里面的值直接调用get()方法即可**



==注意：若涉及到的计算量很大,或者任务多的时候,最好使用并行流 **parallelStream()**或者**parallel()**方法 ，多线程同时进行操作，可以让cpu其他核全部利用起来,而不同stream流是一个单核单线程运行，效率没有并行流高==



### 2.4指定范围内的整数流

> **可以替代for循环**

```java
//左闭右开区间:[0,3)
IntStream.range(0,3).forEach(System.out::print);
System.out.println();
//左闭右闭区间:[0,3]
IntStream.rangeClosed(0,3).forEach(System.out::print);
```





## 三. 接口的改动

### 3.1 区别

​				**java8之前**：接口只有静态常量和抽象方法

​				**java8之后**：接口可以有默认方法和静态方法,即可以有实现的方法



### 3.2 用法

 ```JAVA
 	default XXX xxx(){ //具体实现代码块}
 	//接口名.xxx()调用该方法
 ```

> 若一个类实现某个接口,类中的方法和接口中的默认方法重名时,调用方法会优先调用类中的方法,即"类优先原则"



## 四. 新时间API

​					**全是不可变对象,线程安全,使用ISO-8601日期标准**



### 4.1 日期类

* LocalDate:本地日期类
* LocalTime:本地时间类
* LocalDateTime:本地日期时间类(★)	

> 三者会一即可,方法一样的,推荐使用本地日期时间类



`创建当前时间实例(静态方法)`

```JAVA
LocalDateTime ldt1=LocalDateTime.now();									//获取当前时间
LocalDateTime ldt2=LocalDateTime.of(year,month,day,hour,minute,second);//获取自定义时间
```



`改变时间`

```JAVA
LocalDateTime ldt=LocalDateTime.now();	
ldt.plusXXX(long n);	//增加 年/月/日/时/分/秒; XXX:years/Months/....
ldt.minusXXX(long n);//减少
```



`获取时间`

```JAVA
LocalDateTime ldt=LocalDateTime.now();	
ldt.getYear();			//获取年
ldt.getMonthValue();	//获取月
ldt.getDayOfMonth();	//获取日
ldt.getHour();			//获取时
ldt.getMinute();		//获取分
```



### 4.2 间隔

* Duration:计算两个"时间"的间隔
* Period:计算两个"日期"的间隔



`"时间"间隔`

```JAVA
Duration  du=Duration.between(LocalTime时间实例1,LocalTime时间实例2);
du.getXXX;				//获取秒和纳秒
du.toMills/...();      //获取其他的类型
```



`"日期"间隔`

```JAVA
LocalDateTime ldt1=LocalDateTime.now();		
LocalDateTime ldt2=LocalDateTime.now();		
Period p=Period.between(ldt1,ldt2);
p.getXXX;//获取想要的间隔
```



### 4.3 自定义格式化日期

```java
LocalDateTime ldt=LocalDateTime.now();	
DateTimeFormatter dtf=DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String str=dtf.format(ldt);
```



## 五. Optional类

### 5.1 常用方法

* **T get()：如果Optional中不为空则返回值，否则抛出异常：NoSuchElementException**
* **void ifPresent(Consumer<? super T> consumer)：如果值存在则执行Consumer函数型接口，否则不做任何事情**
* **boolean isPresent()：如果值存在则方法会返回true，否则返回 false**
* **static  Optional  ofNullable(T value)：如果值存在则返回 Optional 包装的指定值，否则返回空的 Optional**
* **T orElse(T other)：如果存在值则返回值， 否则返回 other**
* **T orElseGet(Supplier<? extends T> other)：如果存在值则返回值， 否则返回 Supplier函数型接口的结果**
* **orElseThrow(Supplier<? extends X> exceptionSupplier)：如果存在值则返回值，否则抛出指定异常**



### 5.2 取值操作

> **orElseXxx方法: 取值操作，一定有返回结果**

```JAVA
Object obj = null;

//如果obj不为空则返回本身;否则返回默认值
Object returned1 = Optional.ofNullable(obj).orElse("default value");//default value

//如果obj不为空则返回本身;否则执行Supplier函数型接口
Object returned2 = Optional.ofNullable(obj).orElseGet(() -> {		//2
    return 1 + 1;
});

//如果obj不为空则返回本身;否则抛出异常
Object returned3 = Optional.ofNullable(obj)
    			.orElseThrow(() -> new RuntimeException("obj不能为空！"));//抛出异常
```





### 5.3 执行后续操作

> **ifPresentXxx方法: 执行副作用操作，没有返回值**

```JAVA
Object object=new Object();

//如果object存在则执行Consumer函数型接口，这里不返回值
Optional.ofNullable(object).ifPresent(System.out::println);	//打印object的内存地址

//如果object存在则执行Consumer函数型接口，需要返回值
int address=Optional.ofNullable(object).map(obj->return obj.hashCode()).orElse(-1);// 返回hashCode地址

Object obj = null;
//如果obj存在则执行Consumer函数型接口;否则执行Runnable函数型接口
Optional.ofNullable(obj).ifPresentOrElse(System.out::println,
                                         ()-> System.out.println("null obj"));//打印null obj
```







<HR />



# JDK10

## 一.局部变量类型自动推断

```JAVA
var s=10;
var s="10";
var s = List.of(1, 2, 3);
```

> **`var`只能用于局部变量，不能用于成员变量和方法参数**





<HR />





# JDK12

## 一.switch语法糖 

```JAVA
var id=10;
switch (id){
    case 1,2,3-> System.out.println(method.id);
    case 4,5,6-> System.out.println(method.name);
    case 7,8,9-> System.out.println(method.notes);
    default -> System.out.println("null");
}
```

> `-> ` 可以省略`break`关键字



<HR />





# JDK13

## 一.字符串文本块

```java
var s = """
    Elly said,"Maybe I was a bird in another life."
    Noah said,"If you're a bird , I'm a bird."
    <html>
    <body>
    <p>Hello, world</p>
    </body>
    </html>
    """;
```





## 二.switch返回值

> **jdk13之前想要拿到switch的结果，需要定义一个变量,然后为其赋值,现在可以使用`yield`关键字返回**

```java
 var res = switch (id) {
    case 1, 2, 3 -> {
        var i = 100;
        yield i + id;
    }
    default -> {
        yield id;
    }
};
```



<HR />



# JDK14

## 一.Record

> 本质是一个`final`类，所有属性都会用`final`修饰，会自动编译出get、hashCode 、equals、toString 等方法，==**Record只会根据形参生成一个全参构造但可以使用紧凑构造器进行容错处理**==。使用Record可以更方便的创建一个常量类。==可用于Dto（接收前端json数据）或Vo类（返回前端对象值）==

```JAVA
 public record UserInfo(Integer id, String name, String otherInfo) {
   
   	//    public UserInfo(Integer id, String name, String otherInfo) {
    //        if (id == null) {
    //            this.id = 1;
    //        } else {
    //            this.id = id;
    //        }
    //        this.name = name;
    //        this.otherInfo = otherInfo;
    //    }
   
   		//紧凑构造器（等同于上面的写法），进行容错处理：当变量里面的id是空的时候默认赋值
   		public UserInfo {		
        if (id == null) {
            id = 1;
        }
    	}
   
      static final String SEX = "man";
      static Integer age = 20;

      public void showInfo() {
          System.out.println(id + ":" + name() + ":" + otherInfo + ":" + SEX);
      }
      public static void showAge() {
          System.out.println(age);
      }
  }


var userInfo = new UserInfo(1, "eobard", "something else");//构造方法赋值
System.out.println(userInfo.id);    //直接使用属性值输出
System.out.println(userInfo.name());//使用属性生成的方法输出
System.out.println(UserInfo.SEX);   //调用静态final常量值
userInfo.showInfo();                //调用成员方法
UserInfo.showAge();                 //调用静态方法
```


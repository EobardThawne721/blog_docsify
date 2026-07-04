# Redis

## 一. 开始

### 1.1 下载

​	[Release 3.2.100 · microsoftarchive/redis · GitHub](https://github.com/microsoftarchive/redis/releases/tag/win-3.2.100)

​	打开网址选择 **Redis-x64-3.2.100.zip**下载即可，然后解压 ，可以选择将解压后的word文件删除



### 1.2 介绍

* **redis.windows.conf :**     redis核心配置文件
* **redis-benchmark.exe:**   性能测试工具
* **redis-check-aof.exe:**  AOF文件修复工具
* **redis-cli.exe：** **命令行客户端工具**
* **redis-server.exe：** 服务器启动命令



**Redis单线程具有原子性，无需考虑并发的问题**



### 1.3  启动

* 双击 redis-server.exe 启动Redis，**端口默认6379** (别关闭，让它最小化）
* 再双击 redis-cli.exe 就可以输入命令行



### 1.4 Redis可视化工具

​		解压 **RedisDesktopManager_jb51.rar**，一直下一步即可





### 1.5 本机连接远程Redis（重要）

```bash
# -h 远程ip地址  -p 端口号  -a 密码  -n 索引库(0开始)
redis-cli -h 127.0.0.1  -p 6379 
```





## 二. 操作Redis

### 2.1 基本操作

#### a. 切换库

​		redis有16个库(下标0-15)，默认是0，即第一个库

```
select 0
```



#### b. 清除屏幕信息

```
clear
```



#### c. 查询所有键

```
keys * 
```



#### d. 设置时效性

```
setex key seconds value
```



#### e. 补充设置时效

```
expire key seconds
```

==注意：补充设置时效是指之前已经设置好了值，但是没有设置时间==



#### f. 清空当前库所有数据

```
flushdb
```



#### g. 清空所有库数据

```
flushall
```



#### h.删除键

```
del key1 key2
```



#### i. 查看键类型

```
type key
```





### 2.2 String类型操作

#### 2.2.1 基本语法

| 含义                         | 语法                          |
| :--------------------------- | :---------------------------- |
| 添加/修改数据                | set key value                 |
| 获取数据                     | get key                       |
| 添加/修改多个数据            | mset key1 value1 key2  value2 |
| 获取多个数据                 | mget key1 key2                |
| 获取长度                     | strlen key                    |
| 追加(存在则追加，否则新增)   | append key value              |
| 设置单个数据过期时间(单位秒) | setex key seconds value       |

==注意：String类型适用于经常查询且简单的数据，如果没有K对应的V，会返回(nil)表示为空==



#### 2.2.2 场景一:数据库分表ID主键一致性

​													

**问题**

​			在大型的项目中，分表操作都是基本操作，一张表的数据过于庞大，会影响查询性能，可以用多张表来存储相同类型的数据(比如商品表将会分成多个表来存放商品)，但是对应的主键ID必须保证唯一性，不能重复。Oracle数据库提供了Sequence序列的解决方法，但是Mysql却不支持。



**开发经验**

* redis用于控制数据库ID生成策略，保证主键的唯一性，**单线程具有原子性，无需考虑并发的问题**
* 适用于所有数据库，且支持数据库集群



**操作**

```
语法:
incr key				//设置值,默认为1
incrby key increment	//让值每次掉用incr key后每次自增1

用法：
incr goods_id			//初始化goods_id 为1
incrby goods_id 1		//每次自增1
incr goods_id			//goods_id 为2
```

==注意：在设置了自增后，再次调用incr key才能自增，然后将自增的数据保存在分表的数据里即可==



#### 2.2.3 场景二:数据时效性(重点)

​													

**问题**

​			对于类似于热搜榜的数据，热销的商品，验证码，投票之类的数据在一定时间内有效



**开发经验**

* redis控制数据的生命周期，通过是否失效来控制业务行为，适用于所有具有时效性控制的操作，**可以代替之前存放于session中的判断业务(如验证码)**



**操作**

```
语法:
setex key seconds value		//设置k-v的生命周期，单位秒
psetex key millseconds value //设置k-v的生命周期，单位毫秒


用法：
setex code 60 198625	//设置code为198625的生命周期为60秒
psetex code 2000 198625	//设置code为198625的生命周期为2000毫秒(2秒)
```

==注意：生命周期过了之后再次调用会返回空(nil)==



### 2.3 Hash类型

#### 2.3.1 概述

> **对应java的类型为`Map<String, Map<String,Object> >`双层Map类型**

* 对一系列存储的数据进行分组，方便管理，典型应用存储对象信息
* 一个K存储空间保存多个field-value数据
* 底层采用哈希表结构存储数据	
* 存储结构优化
  * 1.如果field较少，存储结构优化为类数组结构
  * 2.如果field很多，存储结构使用HashMap结构

==注意：适用于经常添加或修改的数据==



#### 2.3.2 基本语法

| 含义                             | 语法                                  |
| -------------------------------- | ------------------------------------- |
| **添加/追加单个field-value数据** | hset key field value                  |
| **获取单个field的value数据**     | hget key field                        |
| **获取所有field-value数据**      | hgetall  key                          |
| **添加多个field-value数据**      | hmset key field1 value1 field2 value2 |
| 获取多个field的value数据         | hmget key field1 field2               |
| **删除指定field的数据**          | hdel key field1 field2                |
| 查询键的所有field                | hkeys key                             |
| 查询键所有的value                | hvals key                             |
| **单个field自增**                | hincrby key field increment           |





#### 2.3.3 场景一:购物车

​												

**案例需求**

1. 张三买了100台电脑，200个键盘
2. 李四买了200个键盘，400个鼠标
3. 李四又追加了100个鼠标垫，并查看了自己的购物车
4. 张三将电脑数量增加了100台，并查看了自己的购物车
5. 张三将键盘商品清除了，并查看了自己的购物车
6. 李四清空了购物车



**实现**

```properties
#1.张三买了100台电脑，200个键盘
hmset zs pc 100 keyboard 200

#2.李四买了200个键盘，400个鼠标
hmset ls keyboard 200 mouse 400

#3.李四又追加了100个鼠标垫，并查看了自己的购物车
hset ls mousecover 100
hgetall ls

#4.张三将电脑数量增加了100台，并查看了自己的购物车
hincrby zs pc 100
hgetall zs

#5.张三将键盘商品清除了，并查看了自己的购物车
hdel key keyboard
hgetall zs

#6.李四清空了购物车
del ls
```





#### 2.3.4 场景二:抢购限购功能(重点)

​																	

**开发经验**

​				redis应用于抢购、限购、限量发放优惠码等业务设计，当抢购数量小于0就让业务做出相应的操作



**操作**

```
hset iphone num 5	//设置iPhone数量为5
	
hincrby iphone num -1	//每次操作就减少一个，当数量小于0的时候，让业务代码做出操作
```



### 2.4 List类型

#### 2.4.1 概述

* 存储需求：存储多个数据，并对数据按顺序进行区分
* 存储结构：一个存储空间保存多个数据
* list类型：保存多个数据，底层使用双向链表结构



#### 2.4.2 基本语法

| 含义                                         | 语法                           |
| -------------------------------------------- | ------------------------------ |
| 添加/追加数据                                | lpush key value1 value2 value3 |
| 获取指定索引数据(从0开始，-1结束)            | lindex key index               |
| 获取所有数据(从0开始，-1结束)                | lrange key  start stop         |
| 获取长度                                     | llen key                       |
| 移除数据                                     | lpop key                       |
| 移除指定数据(从左到右删除指定value的count次) | lrem key count value           |

> ​	可以将List类型的数据添加想成栈(先进后出)，所以查询时会首先查询到最后一个数据

==注意：lpush代表数据从左边首先进入，rpush代表数据从右边首先进入==

​			==lpop代表删除左边的第一个元素，rpop代表删除右边的第一个元素==





#### 2.4.3 场景一:抖音点赞数量，微信点赞顺序

​																

**业务场景**

​				各种微博关注列表，各种新闻按时间的顺序展示，微信的点赞顺序，抖音的点赞数量等等



**开发经验**

* redis应用于具有操作先后顺序的数据控制

* redis应用于数量控制，定时的向数据库更新缓存的数据提高性能优化





### 2.5 Set类型

#### 2.5.1 概述

* 存储大量的数据，查询方面提供更高的效率
* 能够保存大量的数据，高效的内部存储机制，不能存储空值且值不能重复且无序



#### 2.5.2 基本语法

| 含义               | 语法                             |
| ------------------ | -------------------------------- |
| 添加/追加数据      | sadd key member1 member2 member3 |
| 获取全部数据       | smembers key                     |
| 删除数据           | srem key member1  member2        |
| 获取数据总数       | scard key                        |
| 判断数据是否存在   | sismember key                    |
| 移动集合的某个数据 | smove  fromkey  tokey  value     |



#### 2.5.3场景一:网站首页随机信息推荐(重点)

​											

**业务场景**

​			redis应用于各种随机推荐信息，例如歌单推荐，热点新闻推荐等等



**操作**

```
语法：
sadd key  member1 member2		//添加数据
srandmember key  count			//随机取出count个数据

用法：
sadd  movie a b c d e f g		//模拟增加各种影片信息
srandmember moive 4				//随机取出4种影片
```





#### 2.5.4 场景二:筛选共同好友，推荐好友

​											

**业务场景**

​			redis应用于同类信息的关联搜索，二度关联搜索，深度关联搜索，显示共同关注，共同好友



**解决方案**

​				Set类型可以操作两个集合的交，并，差集

* 交集：两个集合共同的
* 并集：两个结合总共的
* 差集：A-B，集合A-集合B 剩下A的数据



**操作**

```
语法：
sinter key1  key2	//交集
sunion key1  key2 	//并集
sdiff  key1  key2	//差集

sinterstore newkey  key1 key2	//将交集产生的数据放入新key中
sunionstore newkey  key1 key2	//将并集产生的数据放入新key中
sdiffstore	newkey  key1 key2	//将差集产生的数据放入新key中
	
用法：
sadd set1 a b c d e f g
sadd set2 a b c d h i j

sinter set1 set2 	//返回a b c d
sunion set1 set2	//返回a b c d e f g h i j
sdiff set1 set2		//返回e f g
```



### 2.6 ZSet类型

#### 2.6.1 概述

* 可以保存可排序的数据
* 在set的存储结构上添加可排序字段，因此数据不能重复



#### 2.6.2 基本语法

| 含义                                     | 语法                                    |
| ---------------------------------------- | --------------------------------------- |
| 添加/追加数据                            | zadd key  score member   score2 member2 |
| 获取数据(从小到大,从0开始，-1结束)       | zranage key start  stop                 |
| 获取数据(从大到小,从0开始，-1结束)       | zrevrange key start stop                |
| 获取数据带有分数(可以升序降序，默认升序) | zrange key start stop withscores        |
| 移除数据                                 | zrem key member                         |
| 区间查询(可以升序降序，默认升序)         | zrangebyscore key  min max              |
| 获取排名                                 | zrank key member                        |
| 获取score值                              | zscore key member                       |

==注意：ZSet类型其它操作和Set类型一致，只需要更改前缀Z==



#### 2.6.3 场景一:排行榜(重点)

​																						

**业务场景**

​				投票选出排行榜前几，各类网站的Top系列，聊天室活跃度设计，游戏好友亲密度



**操作**

```
zadd list 5 zs 8 ls 9 ww  1 zl	//添加数据
zrange list 0 2					//从小到大筛选出前3名
```



## 三. 持久化

### 3.1 概述

​			Redis是一个内存数据库，当每次关机或重启后数据会丢失，我们可以将Redis内存中的数据持久化保存在本地磁盘中。**RDB:Redis默认方式，在一定时间内，检测key的变化情况，然后持久化数据**。如果从数据库查询10000条数据并放入内存中，用RDB的方式就可能会出现宕机的情况，就应该用另一种方法:**AOF**

* RDB：注重性能，忽略数学备份
* AOF：注重备份，忽略性能



## 四. Jedis

### 4.1 开始

`a.导入依赖`

```xml-dtd
 <!--jedis-->
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>3.2.0</version>
    </dependency>
<!--fastjson-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.78</version>
    </dependency>
```

`b.测试`

```JAVA
 @Test
    public void test(){
        //连接Redis
        Jedis jedis=new Jedis("localhost",6379);
        //设置字符串
        jedis.set("userName","eobard");
        //设置有效期
        jedis.setex("code",60,"7895");
        //获取键
        String userName = jedis.get("userName");
        System.out.println("userName = " + userName);
        //关闭连接
        jedis.close();
    }
}
```

> ​	注意：后期获取数据列表的时候，可以将数据列表放入缓冲中，然后从缓存中获取数据

```JAVA
   		List<User> users = new ArrayList<>();
        users.add(new User(1,"tom","hk"));
        users.add(new User(2,"jack","cq"));
        users.add(new User(3,"herry","cd"));
        jedis.set("userList", JSON.toJSONString(users));
        System.out.println(jedis.get("userList"));
```



### 4.2 Jedis使用

> Jedis的API和redis的命令一致，可以根据redis的命令来调用API，**相应的API都有对应的返回值，后期可以根据返回值来判断相应的命令是否成功**



### 4.3 Jedis连接池

`a.创建jedis.properties配置文件`

```properties
#ip地址
host=127.0.0.1
#端口号
port=6379
#最大连接数
maxTotal=50
#最大空闲数
maxIdle=10
```



`b.创建工具类`

```JAVA
public class JedisPoolUtils {
    private JedisPoolUtils(){}

    private static JedisPool pool=null;

    static {
        //读取配置文件
        InputStream in = JedisPoolUtils.class.getClassLoader().getResourceAsStream("jedis.properties");
        Properties properties=new Properties();
        try {
            properties.load(in);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //设置最大连接数和最大空闲数
        JedisPoolConfig config=new JedisPoolConfig();
        config.setMaxTotal(Integer.valueOf(properties.getProperty("maxTotal")));
        config.setMaxIdle(Integer.valueOf(properties.getProperty("maxIdle")));
        //初始化连接池
        pool=new JedisPool(config,properties.getProperty("host"),Integer.valueOf(properties.getProperty("port")));
    }

    //获取连接
    public static Jedis getJedis(){
        return pool.getResource();
    }
}
```



`c.测试`

```JAVA
 @Test
    public void  test2(){
        Jedis jedis = JedisPoolUtils.getJedis();
        jedis.setex("code",10,"12345");
        jedis.close();
    }
```



## 五. SSM整合Redis

`1.导入依赖`

```xml-dtd
 <!-- redis-->
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>2.4.2</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-redis</artifactId>
      <version>1.3.0.RELEASE</version>
    </dependency>
```



`2.resources创建redis.properties`

```properties
#连接主机地址
redis.host=127.0.0.1
#连接端口
redis.port=6379
#最大空闲数
redis.maxIdle=300
#最大连接数
redis.maxTotal=500
#超时时间
redis.timeOut=100000
```



`3.applicationContext.xml修改代码`

```xml-dtd
 	 <!-- 扫描注解所在的包 -->
    <context:component-scan base-package="com.eobard.dao,com.eobard.service,com.eobard.utils"/>

    <!--加载redis的配置文件信息-->
    <context:property-placeholder ignore-unresolvable="true" location="classpath:redis.properties"/>

```



`4.创建RedisUtils工具类操作API`

```JAVA
package com.eobard.utils;

public class RedisUtils {
    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 指定缓存失效时间
     * @param key  键
     * @param time 时间(秒)
     * @return
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 判断key是否存在
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除缓存
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }

    /**
     * 普通缓存获取
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        if(!StringUtils.isEmpty(key)){
            return redisTemplate.opsForValue().get(key);
        }
        return null;
    }

    /**
     * 普通缓存放入
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }

    /**
     * 普通缓存放入并设置时间
     * @param key   键
     * @param value 值
     * @param time  时间(分钟) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.MINUTES);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 递增
     * @param key   键
     * @param delta 要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     * @param key   键
     * @param delta 要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    // ================================Map=================================

    /**
     * HashGet
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除hash表中的值
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }

    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }

    // ============================set=============================

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    // ===============================list=================================

    /**
     * 获取list缓存的内容
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     *
     * @param key 键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 移除N个值为value
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

}
```



`5.SSM通过注解注入Redis`

```JAVA
package com.eobard.utils;

@Repository     //这里不用@Configuration
public class RedisConfig {

    @Value("${redis.maxIdle}")
    private Integer maxIdle;//最大空闲数
    @Value("${redis.maxTotal}")
    private Integer maxTotal;//最大连接数
    @Value("${redis.host}")
    private String hostName;//主机地址
    @Value("${redis.port}")
    private Integer port;//主机端口号
    @Value("${redis.timeOut}")
    private Integer timeOut;//超时时间


    /*设置Jedis连接池*/
    @Bean
    public JedisPoolConfig poolConfig() {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxIdle(maxIdle);
        poolConfig.setMaxTotal(maxTotal);
        return poolConfig;
    }

    /*Spring整合Jedis,设置连接属性*/
    @Bean
    public JedisConnectionFactory connectionFactory(JedisPoolConfig poolConfig){
        JedisConnectionFactory connectionFactory = new JedisConnectionFactory();
        connectionFactory.setHostName(hostName);
        connectionFactory.setPort(port);
        connectionFactory.setPoolConfig(poolConfig);
        connectionFactory.setTimeout(timeOut);
        return connectionFactory;
    }

    /*注入RedisTemplate工具类*/
    @Bean("redisTemplate")
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
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

    /*注入自定义Redis操作工具类*/
    @Bean
    public RedisUtils redisUtils() {
        return new RedisUtils();
    }
}
```



`6.测试`

```JAVA
@RunWith(SpringJUnit4ClassRunner.class)		   
@ContextConfiguration("classpath:applicationContext.xml")  
public class TestRedis {
    
    @Autowired
    private RedisUtils redisUtils;


    @Test
    public void test() {
        if(!redisUtils.hasKey("name")){
            redisUtils.set("name","喜喜123");
        }
        System.out.println(redisUtils.get("name"));
    }
}
```





## 六. Redisson使用指南

### 6.1 Spring Boot 集成

#### 6.1.1 Maven 依赖配置

```xml
<!--Spring Boot 3.0.2-->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.25.0</version>
</dependency>
```



#### 6.1.2 配置文件

```properties
# Spring Boot 3 Redis 配置
spring.data.redis.host=127.0.0.1
spring.data.redis.port=6379
spring.data.redis.password=your_password
spring.data.redis.database=0

# Spring Boot 2 Redis 配置
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=your_password
spring.redis.database=0
```



#### 6.1.3 Redisson配置类

- **单机模式**：非必须配置，可以直接注入`RedissonClient`客户端使用即可，若不手动注入`redisson-spring-boot-starter`会自动读取配置文件并创建`RedissonClient`实例

  ```java
  // eg:非必须配置,这里演示自定义注入,设置相应配置
  @Configuration
  public class RedissonConfig {
  
      // Spring容器关闭时自动调用RedissonClient的shutdown()方法释放资源
      @Bean(destroyMethod = "shutdown")
      public RedissonClient redissonClient() {
          Config config = new Config();
          // 使用单机Redis
          config.useSingleServer()
                  .setAddress("redis://127.0.0.1:6379")
                  .setPassword("your_password")
                  .setDatabase(0)
                  .setPingConnectionInterval(30000);  // 每30s心跳检测
          return Redisson.create(config);
      }
  }



* **集群模式**

  ```java
  @Configuration
  public class RedissonConfig {
  
      // Spring容器关闭时自动调用RedissonClient的shutdown()方法释放资源
      @Bean(destroyMethod = "shutdown")
      public RedissonClient redissonClusterClient() {
          Config config = new Config();
          // 使用Redis集群模式
          config.useClusterServers()
                  .addNodeAddress("redis://192.168.1.1:6379", "redis://192.168.1.2:6379", "redis://192.168.1.3:6379")
                  .setPassword("your_password")
                  .setConnectTimeout(10000)               // 连接超时：10s
                  .setTimeout(3000)                      // 命令等待超时：3s
                  .setRetryAttempts(3)                   // 命令重试次数
                  .setRetryInterval(1000)                // 重试间隔
                  .setMasterConnectionPoolSize(64)       // 主节点连接池（生产 64~128）
                  .setSlaveConnectionPoolSize(64)         // 从节点连接池
                  .setMasterConnectionMinimumIdleSize(8) // 主节点最小空闲连接
                  .setSlaveConnectionMinimumIdleSize(8)  // 从节点最小空闲连接
                  // 心跳检测（必须开，防止连接假死）
                  .setPingConnectionInterval(30000)      // 30秒心跳一次
                  // 空闲连接超时自动释放
                  .setIdleConnectionTimeout(60000);       // 60秒空闲释放
          return Redisson.create(config);
      }
  }
  ```





### 6.2 分布式锁

#### 6.2.1 基础分布式锁

> **场景：部分成功、部分失败，抢锁成功则执行，失败则允许在一定时间内重试，不排队、不阻塞。适用于防重复提交、限流、接口幂等、秒杀、争抢式任务**



1. **优惠券领取控制**
   
   - **业务需求**：用户领取优惠券时，需要保证每人限领一张，防止并发情况下同一用户重复领取
   - **实现方式**：使用用户ID作为锁Key，确保同一用户的领取请求串行处理
   - **代码示例**：
   ```java
   public boolean claimCoupon(Long userId, Long couponId) {
       String lockKey = "coupon:claim:" + userId;
       RLock lock = redissonClient.getLock(lockKey);
       boolean isLocked = false;
       try {
           isLocked = lock.tryLock(10, 30, TimeUnit.SECONDS);
           if (isLocked) {
               // 检查用户是否已领取
               if (userCouponRepository.exists(userId, couponId)) {
                   return false;
               }
               // 扣减库存并记录领取
               couponRepository.decreaseStock(couponId);
               userCouponRepository.save(userId, couponId);
               return true;
           } else {
               throw new RuntimeException("获取锁失败");
           }
       } catch (InterruptedException e) {
           Thread.currentThread().interrupt();
           throw new RuntimeException("锁获取被中断", e);
       } finally {
           if (isLocked && lock.isHeldByCurrentThread()) {
               lock.unlock();
           }
       }
   }
   ```





1. **订单号生成**

   - **业务需求**：分布式环境下生成全局唯一订单号，避免订单号重复
   - **实现方式**：使用固定锁 Key 控制订单号生成器的并发访问
   - **代码示例**：
   ```java
   public String generateOrderNo() {
       String lockKey = "order:no:generator";
       RLock lock = redissonClient.getLock(lockKey);
       boolean isLocked = false;
       try {
           isLocked = lock.tryLock(10, 30, TimeUnit.SECONDS);
           if (isLocked) {
               String date = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd"));
               RAtomicLong atomicLong = redissonClient.getAtomicLong("order:seq:" + date);
               Long sequence = atomicLong.incrementAndGet();
               return date + String.format("%06d", sequence);
           } else {
               throw new RuntimeException("获取锁失败");
           }
       } catch (InterruptedException e) {
           Thread.currentThread().interrupt();
           throw new RuntimeException("锁获取被中断", e);
       } finally {
           if (isLocked && lock.isHeldByCurrentThread()) {
               lock.unlock();
           }
       }
   }
   ```

   

2. **库存扣减（简单场景）**

   - **业务需求**：普通商品下单时扣减库存，保证不超卖
   - **实现方式**：使用商品ID作为锁Key，控制库存扣减的并发
   - **代码示例**：
   ```java
   public boolean deductStock(Long productId, Integer quantity) {
       String lockKey = "stock:deduct:" + productId;
       RLock lock = redissonClient.getLock(lockKey);
       boolean isLocked = false;
       try {
           isLocked = lock.tryLock(10, 30, TimeUnit.SECONDS);
           if (isLocked) {
               Integer currentStock = productRepository.getStock(productId);
               if (currentStock < quantity) {
                   throw new InsufficientStockException("库存不足");
               }
               productRepository.decreaseStock(productId, quantity);
               return true;
           } else {
               throw new RuntimeException("获取锁失败");
           }
       } catch (InterruptedException e) {
           Thread.currentThread().interrupt();
           throw new RuntimeException("锁获取被中断", e);
       } finally {
           if (isLocked && lock.isHeldByCurrentThread()) {
               lock.unlock();
           }
       }
   }
   ```







### 2.2 公平锁

**场景说明**：需要保证锁的获取顺序，先请求的线程先获得锁，适用于需要严格顺序执行的场景。

**典型业务场景**：

1. **消息队列顺序消费**
   - **业务需求**：订单状态变更消息需要按顺序处理（创建→支付→发货），不能乱序执行
   - **实现方式**：使用订单ID作为公平锁Key，确保同一订单的消息按请求顺序处理
   - **代码示例**：
   ```java
   @KafkaListener(topics = "order-status-topic")
   public void consumeOrderMessage(OrderMessage message) {
       String lockKey = "order:fair:" + message.getOrderId();
       RLock fairLock = redissonClient.getFairLock(lockKey);
       try {
           fairLock.lock(60, TimeUnit.SECONDS);
           processOrderStatusChange(message);
       } finally {
           if (fairLock.isHeldByCurrentThread()) {
               fairLock.unlock();
           }
       }
   }
   ```

2. **银行转账顺序处理**
   - **业务需求**：同一账户的多笔转账请求需要按发起顺序处理，避免余额计算错误
   - **实现方式**：使用账户ID作为公平锁Key，保证转账请求FIFO执行
   - **代码示例**：
   ```java
   public void processTransfer(Long accountId, TransferRequest request) {
       String lockKey = "transfer:fair:" + accountId;
       RLock fairLock = redissonClient.getFairLock(lockKey);
       try {
           fairLock.lock(30, TimeUnit.SECONDS);
           BigDecimal balance = accountService.getBalance(accountId);
           if (balance.compareTo(request.getAmount()) >= 0) {
               accountService.decreaseBalance(accountId, request.getAmount());
               accountService.increaseBalance(request.getTargetAccountId(), request.getAmount());
           }
       } finally {
           if (fairLock.isHeldByCurrentThread()) {
               fairLock.unlock();
           }
       }
   }
   ```

3. **限流抢购排队**
   - **业务需求**：热门商品抢购时，用户请求需要按到达顺序排队处理，先到先得
   - **实现方式**：使用商品ID作为公平锁Key，配合队列实现公平抢购
   - **代码示例**：
   ```java
   public boolean fairSeckill(Long productId, Long userId) {
       String lockKey = "seckill:fair:" + productId;
       RLock fairLock = redissonClient.getFairLock(lockKey);
       try {
           boolean acquired = fairLock.tryLock(30, 10, TimeUnit.SECONDS);
           if (!acquired) {
               return false;
           }
           Integer stock = productRepository.getStock(productId);
           if (stock <= 0) {
               return false;
           }
           productRepository.decreaseStock(productId);
           orderService.createOrder(productId, userId);
           return true;
       } catch (InterruptedException e) {
           Thread.currentThread().interrupt();
           return false;
       } finally {
           if (fairLock.isHeldByCurrentThread()) {
               fairLock.unlock();
           }
       }
   }
   ```



### 2.3 读写锁

**场景说明**：读多写少场景，读锁共享，写锁独占，可以显著提升并发性能。

**典型业务场景**：

1. **商品详情页缓存**
   - **业务需求**：商品详情页访问量巨大，需要频繁读取商品信息，但商品信息更新频率较低
   - **实现方式**：读锁允许多个线程并发读取缓存，写锁独占更新缓存数据
   - **代码示例**：
   ```java
   @Service
   public class ProductCacheService {
   
       @Autowired
       private RedissonClient redissonClient;
   
       @Autowired
       private ProductRepository productRepository;
   
       public Product getProduct(Long productId) {
           String lockKey = "product:cache:" + productId;
           RReadWriteLock rwLock = redissonClient.getReadWriteLock(lockKey);
           RLock readLock = rwLock.readLock();
   
           try {
               readLock.lock(10, TimeUnit.SECONDS);
   
               RBucket<Product> productBucket = redissonClient.getBucket("product:" + productId);
               Product product = productBucket.get();
               if (product != null) {
                   return product;
               }
   
               readLock.unlock();
               RLock writeLock = rwLock.writeLock();
               try {
                   writeLock.lock(10, TimeUnit.SECONDS);
   
                   productBucket = redissonClient.getBucket("product:" + productId);
                   product = productBucket.get();
                   if (product != null) {
                       return product;
                   }
   
                   product = productRepository.findById(productId).orElse(null);
                   if (product != null) {
                       productBucket.set(product);
                       productBucket.expire(30, TimeUnit.MINUTES);
                   }
                   return product;
               } finally {
                   if (writeLock.isHeldByCurrentThread()) {
                       writeLock.unlock();
                   }
               }
           } finally {
               if (readLock.isHeldByCurrentThread()) {
                   readLock.unlock();
               }
           }
       }
   
       public void updateProduct(Product product) {
           String lockKey = "product:cache:" + product.getId();
           RReadWriteLock rwLock = redissonClient.getReadWriteLock(lockKey);
           RLock writeLock = rwLock.writeLock();
   
           try {
               writeLock.lock(30, TimeUnit.SECONDS);
   
               productRepository.save(product);
               RBucket<Product> productBucket = redissonClient.getBucket("product:" + product.getId());
               productBucket.set(product);
               productBucket.expire(30, TimeUnit.MINUTES);
           } finally {
               if (writeLock.isHeldByCurrentThread()) {
                   writeLock.unlock();
               }
           }
       }
   }
   ```

2. **配置中心热更新**
   - **业务需求**：系统配置需要被多个服务节点频繁读取，但配置更新操作较少
   - **实现方式**：读锁支持高并发读取配置，写锁独占更新配置
   - **代码示例**：
   ```java
   @Service
   public class ConfigService {
   
       @Autowired
       private RedissonClient redissonClient;
   
       private Map<String, String> localConfigCache = new ConcurrentHashMap<>();
   
       public String getConfig(String configKey) {
           String lockKey = "config:" + configKey;
           RReadWriteLock rwLock = redissonClient.getReadWriteLock(lockKey);
           RLock readLock = rwLock.readLock();
   
           try {
               readLock.lock(5, TimeUnit.SECONDS);
   
               if (localConfigCache.containsKey(configKey)) {
                   return localConfigCache.get(configKey);
               }
   
               String value = configRepository.getValue(configKey);
               localConfigCache.put(configKey, value);
               return value;
           } finally {
               if (readLock.isHeldByCurrentThread()) {
                   readLock.unlock();
               }
           }
       }
   
       public void updateConfig(String configKey, String newValue) {
           String lockKey = "config:" + configKey;
           RReadWriteLock rwLock = redissonClient.getReadWriteLock(lockKey);
           RLock writeLock = rwLock.writeLock();
   
           try {
               writeLock.lock(10, TimeUnit.SECONDS);
   
               configRepository.updateValue(configKey, newValue);
               localConfigCache.put(configKey, newValue);
   
               notifyConfigChange(configKey, newValue);
           } finally {
               if (writeLock.isHeldByCurrentThread()) {
                   writeLock.unlock();
               }
           }
       }
   }
   ```

3. **用户会话状态管理**
   - **业务需求**：用户登录状态需要频繁读取验证，但登录登出操作相对较少
   - **实现方式**：读锁支持高并发会话验证，写锁处理登录登出状态变更
   - **代码示例**：
   ```java
   @Service
   public class SessionService {
   
       @Autowired
       private RedissonClient redissonClient;
   
       public UserSession getSession(String sessionId) {
           String lockKey = "session:" + sessionId;
           RReadWriteLock rwLock = redissonClient.getReadWriteLock(lockKey);
           RLock readLock = rwLock.readLock();
   
           try {
               readLock.lock(5, TimeUnit.SECONDS);
               RBucket<UserSession> sessionBucket = redissonClient.getBucket("session:" + sessionId);
               return sessionBucket.get();
           } finally {
               if (readLock.isHeldByCurrentThread()) {
                   readLock.unlock();
               }
           }
       }
   
       public void createSession(String sessionId, UserSession session) {
           String lockKey = "session:" + sessionId;
           RReadWriteLock rwLock = redissonClient.getReadWriteLock(lockKey);
           RLock writeLock = rwLock.writeLock();
   
           try {
               writeLock.lock(10, TimeUnit.SECONDS);
               RBucket<UserSession> sessionBucket = redissonClient.getBucket("session:" + sessionId);
               sessionBucket.set(session);
               sessionBucket.expire(30, TimeUnit.MINUTES);
           } finally {
               if (writeLock.isHeldByCurrentThread()) {
                   writeLock.unlock();
               }
           }
       }
   
       public void invalidateSession(String sessionId) {
           String lockKey = "session:" + sessionId;
           RReadWriteLock rwLock = redissonClient.getReadWriteLock(lockKey);
           RLock writeLock = rwLock.writeLock();
   
           try {
               writeLock.lock(10, TimeUnit.SECONDS);
               RBucket<UserSession> sessionBucket = redissonClient.getBucket("session:" + sessionId);
               sessionBucket.delete();
           } finally {
               if (writeLock.isHeldByCurrentThread()) {
                   writeLock.unlock();
               }
           }
       }
   }
   ```

### 2.4 联锁（MultiLock）

**场景说明**：需要同时锁定多个资源，只有当所有锁都获取成功时才认为获取成功。

**典型业务场景**：

1. **跨账户转账操作**
   - **业务需求**：用户A向用户B转账，需要同时锁定两个账户，保证转账操作的原子性
   - **实现方式**：使用联锁同时锁定转出账户和转入账户，按账户ID排序避免死锁
   - **代码示例**：
   ```java
   @Service
   public class TransferService {
   
       @Autowired
       private RedissonClient redissonClient;
   
       @Autowired
       private AccountRepository accountRepository;
   
       public boolean transfer(Long fromAccountId, Long toAccountId, BigDecimal amount) {
           // 按账户ID排序，确保所有节点获取锁的顺序一致，避免死锁
           Long firstAccountId = Math.min(fromAccountId, toAccountId);
           Long secondAccountId = Math.max(fromAccountId, toAccountId);
   
           String lockKey1 = "account:" + firstAccountId;
           String lockKey2 = "account:" + secondAccountId;
   
           RLock lock1 = redissonClient.getLock(lockKey1);
           RLock lock2 = redissonClient.getLock(lockKey2);
   
           RedissonMultiLock multiLock = new RedissonMultiLock(lock1, lock2);
   
           try {
               boolean isLocked = multiLock.tryLock(10, 30, TimeUnit.SECONDS);
               if (!isLocked) {
                   throw new RuntimeException("获取账户锁失败，请稍后重试");
               }
   
               Account fromAccount = accountRepository.findById(fromAccountId);
               Account toAccount = accountRepository.findById(toAccountId);
   
               if (fromAccount.getBalance().compareTo(amount) < 0) {
                   throw new InsufficientBalanceException("余额不足");
               }
   
               fromAccount.setBalance(fromAccount.getBalance().subtract(amount));
               toAccount.setBalance(toAccount.getBalance().add(amount));
   
               accountRepository.save(fromAccount);
               accountRepository.save(toAccount);
   
               // 记录转账流水
               transactionRepository.save(new Transaction(fromAccountId, toAccountId, amount));
   
               return true;
   
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
               throw new RuntimeException("转账操作被中断", e);
           } finally {
               multiLock.unlock();
           }
       }
   }
   ```

2. **库存扣减与积分扣减组合操作**
   - **业务需求**：用户购买虚拟商品时，需要同时扣减库存和用户积分，两者必须同时成功或失败
   - **实现方式**：使用联锁同时锁定商品库存和用户积分，保证组合操作的原子性
   - **代码示例**：
   ```java
   @Service
   public class VirtualGoodsPurchaseService {
   
       @Autowired
       private RedissonClient redissonClient;
   
       public boolean purchaseVirtualGoods(Long userId, Long goodsId, Integer quantity) {
           String stockLockKey = "virtual:stock:" + goodsId;
           String pointsLockKey = "user:points:" + userId;
   
           RLock stockLock = redissonClient.getLock(stockLockKey);
           RLock pointsLock = redissonClient.getLock(pointsLockKey);
   
           RedissonMultiLock multiLock = new RedissonMultiLock(stockLock, pointsLock);
   
           try {
               boolean isLocked = multiLock.tryLock(5, 20, TimeUnit.SECONDS);
               if (!isLocked) {
                   throw new RuntimeException("系统繁忙，请稍后重试");
               }
   
               VirtualGoods goods = goodsRepository.findById(goodsId);
               if (goods.getStock() < quantity) {
                   throw new InsufficientStockException("商品库存不足");
               }
   
               User user = userRepository.findById(userId);
               Integer requiredPoints = goods.getPointsPrice() * quantity;
               if (user.getPoints() < requiredPoints) {
                   throw new InsufficientPointsException("积分不足");
               }
   
               // 扣减库存
               goods.setStock(goods.getStock() - quantity);
               goodsRepository.save(goods);
   
               // 扣减积分
               user.setPoints(user.getPoints() - requiredPoints);
               userRepository.save(user);
   
               // 创建订单
               orderRepository.createOrder(userId, goodsId, quantity, requiredPoints);
   
               return true;
   
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
               throw new RuntimeException("购买操作被中断", e);
           } finally {
               multiLock.unlock();
           }
       }
   }
   ```

3. **多仓库库存调拨**
   - **业务需求**：从一个仓库调拨商品到另一个仓库，需要同时锁定调出仓库和调入仓库的库存
   - **实现方式**：使用联锁同时锁定两个仓库的库存记录，确保调拨操作的原子性
   - **代码示例**：
   ```java
   @Service
   public class WarehouseTransferService {
   
       @Autowired
       private RedissonClient redissonClient;
   
       public boolean transferStock(Long fromWarehouseId, Long toWarehouseId,
                                    Long productId, Integer quantity) {
           // 按仓库ID排序，避免死锁
           Long firstWarehouseId = Math.min(fromWarehouseId, toWarehouseId);
           Long secondWarehouseId = Math.max(fromWarehouseId, toWarehouseId);
   
           String lockKey1 = "warehouse:stock:" + firstWarehouseId + ":" + productId;
           String lockKey2 = "warehouse:stock:" + secondWarehouseId + ":" + productId;
   
           RLock lock1 = redissonClient.getLock(lockKey1);
           RLock lock2 = redissonClient.getLock(lockKey2);
   
           RedissonMultiLock multiLock = new RedissonMultiLock(lock1, lock2);
   
           try {
               boolean isLocked = multiLock.tryLock(10, 60, TimeUnit.SECONDS);
               if (!isLocked) {
                   throw new RuntimeException("获取仓库锁失败");
               }
   
               // 检查调出仓库库存
               WarehouseStock fromStock = warehouseStockRepository
                   .findByWarehouseIdAndProductId(fromWarehouseId, productId);
               if (fromStock.getQuantity() < quantity) {
                   throw new InsufficientStockException("调出仓库库存不足");
               }
   
               // 执行调拨
               fromStock.setQuantity(fromStock.getQuantity() - quantity);
               warehouseStockRepository.save(fromStock);
   
               WarehouseStock toStock = warehouseStockRepository
                   .findByWarehouseIdAndProductId(toWarehouseId, productId);
               toStock.setQuantity(toStock.getQuantity() + quantity);
               warehouseStockRepository.save(toStock);
   
               // 记录调拨单
               transferOrderRepository.save(new TransferOrder(
                   fromWarehouseId, toWarehouseId, productId, quantity));
   
               return true;
   
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
               throw new RuntimeException("调拨操作被中断", e);
           } finally {
               multiLock.unlock();
           }
       }
   }
   ```



### 2.5 典型业务应用场景

#### 场景一：秒杀系统中的库存并发控制

**业务背景**：电商平台秒杀活动中，大量用户同时抢购限量商品，需要保证库存扣减的准确性，防止超卖。

**面临的并发问题**：
- 同一时刻多个用户同时下单，并发读取库存导致数据不一致
- 库存扣减操作非原子性，可能出现库存为负数的情况
- 分布式部署下，多台服务器同时处理请求，本地锁无法生效

**使用分布式锁的必要性**：
- 保证库存扣减的原子性，确保同一时刻只有一个请求能修改库存
- 防止超卖现象，保护商家利益
- 支持高并发场景下的数据一致性

**实现方式概述**：
```java
@Service
public class SeckillService {

    @Autowired
    private RedissonClient redissonClient;

    @Autowired
    private ProductRepository productRepository;

    public boolean seckillProduct(Long productId, Long userId) {
        String lockKey = "seckill:product:" + productId;
        RLock lock = redissonClient.getLock(lockKey);

        try {
            boolean isLocked = lock.tryLock(5, 10, TimeUnit.SECONDS);
            if (!isLocked) {
                return false;
            }

            Integer stock = productRepository.getStock(productId);
            if (stock <= 0) {
                return false;
            }

            productRepository.decreaseStock(productId);
            createOrder(productId, userId);
            return true;

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return false;
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```

---

#### 场景二：分布式任务调度中的任务执行唯一性保证

**业务背景**：分布式系统中，定时任务需要在多个节点中只执行一次，避免重复处理导致的数据错误或资源浪费。

**面临的并发问题**：
- 多个服务实例同时启动，定时任务被重复执行
- 任务执行时间较长，下一个调度周期开始时上一个任务还未完成
- 任务失败后重试机制可能导致同一任务被多次执行

**使用分布式锁的必要性**：
- 确保同一时刻只有一个节点执行特定任务
- 防止任务重复执行导致的数据不一致或资源浪费
- 支持任务执行失败后的安全重试

**实现方式概述**：
```java
@Component
public class DistributedJob {

    @Autowired
    private RedissonClient redissonClient;

    @Scheduled(cron = "0 0 1 * * ?")
    public void dailyDataSync() {
        String lockKey = "job:dailyDataSync";
        RLock lock = redissonClient.getLock(lockKey);

        try {
            boolean isLocked = lock.tryLock(0, 30, TimeUnit.MINUTES);
            if (!isLocked) {
                System.out.println("任务已在其他节点执行，跳过本次调度");
                return;
            }

            System.out.println("开始执行数据同步任务");
            syncData();
            System.out.println("数据同步任务完成");

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```

---

#### 场景三：分布式事务中的资源竞争控制

**业务背景**：微服务架构下，跨服务的业务操作需要保证数据一致性，涉及多个服务的资源竞争控制。

**面临的并发问题**：
- 多个服务同时操作共享资源（如账户余额、积分等）
- 分布式事务中，部分服务成功部分失败导致的数据不一致
- 并发操作导致的死锁或资源饥饿问题

**使用分布式锁的必要性**：
- 控制对共享资源的并发访问，保证操作的原子性
- 配合分布式事务框架（如 Seata）实现最终一致性
- 防止资源竞争导致的数据不一致

**实现方式概述**：
```java
@Service
public class AccountService {

    @Autowired
    private RedissonClient redissonClient;

    @Transactional
    public boolean transfer(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        String lockKey1 = "account:" + Math.min(fromAccountId, toAccountId);
        String lockKey2 = "account:" + Math.max(fromAccountId, toAccountId);

        RLock lock1 = redissonClient.getLock(lockKey1);
        RLock lock2 = redissonClient.getLock(lockKey2);

        RedissonMultiLock multiLock = new RedissonMultiLock(lock1, lock2);

        try {
            boolean isLocked = multiLock.tryLock(10, 30, TimeUnit.SECONDS);
            if (!isLocked) {
                throw new RuntimeException("获取账户锁失败");
            }

            BigDecimal fromBalance = getBalance(fromAccountId);
            if (fromBalance.compareTo(amount) < 0) {
                throw new RuntimeException("余额不足");
            }

            decreaseBalance(fromAccountId, amount);
            increaseBalance(toAccountId, amount);

            return true;

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("转账被中断", e);
        } finally {
            multiLock.unlock();
        }
    }
}
```

### 2.6 注意事项

- **锁续期**：Redisson 支持 watchdog 机制，默认每 10 秒自动续期锁的过期时间，避免业务未执行完锁就过期
- **锁粒度**：尽量减少锁的粒度，锁范围过大会影响并发性能
- **异常处理**：确保在 finally 块中释放锁，且要判断锁是否被当前线程持有
- **超时设置**：合理设置锁的等待时间和持有时间，避免死锁或长时间阻塞

### 2.6 性能优化建议

- 使用 Redisson 的异步 API 减少阻塞
- 合理配置连接池大小，避免连接耗尽
- 监控锁的等待时间和获取成功率
- 对于高频锁场景，考虑使用本地缓存 + 分布式锁的二级缓存模式
- **秒杀场景优化**：使用分段锁或库存预热策略，将热点数据分散到多个锁，降低锁竞争
- **任务调度优化**：设置合理的锁超时时间，避免任务执行时间过长导致锁自动释放
- **转账场景优化**：使用联锁时统一锁的获取顺序，防止死锁发生

---

## 3. 分布式集合

### 3.1 RMap - 分布式 Map

**场景说明**：跨 JVM 实例共享缓存数据，需要高性能的分布式哈希表。

**代码实现**：

```java
import org.redisson.api.RMap;
import org.redisson.api.RedissonClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class DistributedMapService {

    @Autowired
    private RedissonClient redissonClient;

    public void mapOperations(String mapName) {
        RMap<String, String> map = redissonClient.getMap(mapName);

        map.put("key1", "value1");
        map.putIfAbsent("key1", "value2");
        map.putIfAbsent("key2", "value2");

        String value = map.get("key1");
        System.out.println("获取值: " + value);

        map.putIfAbsent("key3", "value3", 30, TimeUnit.SECONDS);

        map.getAndPut("key1", "newValue");

        map.remove("key2");

        boolean contains = map.containsKey("key1");
        System.out.println("是否包含key1: " + contains);

        map.addAndGet("counter", 1);

        System.out.println("Map大小: " + map.size());
    }

    public void mapCacheExample(String cacheName) {
        RMapCache<String, String> mapCache = redissonClient.getMapCache(cacheName);

        mapCache.put("key1", "value1", 30, TimeUnit.SECONDS);

        String value = mapCache.get("key1");
        System.out.println("缓存值: " + value);

        mapCache.remove("key1");
    }
}
```

### 3.2 RList - 分布式 List

**场景说明**：需要跨节点共享列表数据，如任务队列、排行榜等。

**代码实现**：

```java
import org.redisson.api.RList;

@Service
public class DistributedListService {

    @Autowired
    private RedissonClient redissonClient;

    public void listOperations(String listName) {
        RList<String> list = redissonClient.getList(listName);

        list.add("item1");
        list.add("item2");
        list.add("item3");

        list.add(1, "item1_5");

        String first = list.get(0);
        String last = list.get(list.size() - 1);

        list.set(0, "newItem1");

        list.remove(1);
        list.remove("item3");

        System.out.println("列表大小: " + list.size());
        System.out.println("是否包含item2: " + list.contains("item2"));

        list.readLock().lock(10, TimeUnit.SECONDS);
        try {
            System.out.println("读取列表: " + list);
        } finally {
            list.readLock().unlock();
        }

        list.writeLock().lock(10, TimeUnit.SECONDS);
        try {
            list.clear();
        } finally {
            list.writeLock().unlock();
        }
    }
}
```

### 3.3 RSet - 分布式 Set

**场景说明**：需要存储不重复的元素集合，如去重、标签集合等。

**代码实现**：

```java
import org.redisson.api.RSet;
import org.redisson.api.RSortedSet;

@Service
public class DistributedSetService {

    @Autowired
    private RedissonClient redissonClient;

    public void setOperations(String setName) {
        RSet<String> set = redissonClient.getSet(setName);

        set.add("item1");
        set.add("item2");
        set.add("item1");
        System.out.println("Set大小(去重后): " + set.size());

        set.remove("item1");

        System.out.println("是否包含item2: " + set.contains("item2"));

        RSet<String> otherSet = redissonClient.getSet("otherSet");
        otherSet.add("item1");
        otherSet.add("item3");

        set.union(otherSet.getName());
        System.out.println("合并后Set: " + set.readAll());

        set.intersection(otherSet.getName());
        System.out.println("交集: " + set.readAll());
    }

    public void sortedSetExample(String sortedSetName) {
        RSortedSet<Integer> sortedSet = redissonClient.getSortedSet(sortedSetName);
        sortedSet.add(3);
        sortedSet.add(1);
        sortedSet.add(2);

        System.out.println("排序后: " + sortedSet.readAll());

        Integer first = sortedSet.first();
        Integer last = sortedSet.last();
        System.out.println("最小值: " + first + ", 最大值: " + last);
    }
}
```

### 3.4 注意事项

- **内存占用**：RMap 的数据存储在 Redis 内存中，大量数据时注意 Redis 内存容量
- **操作原子性**：单个操作是原子的，但复合操作（如 get-then-set）需要加锁
- **序列化**：Redisson 使用 JDK 序列化，可配置其他序列化方式如 JSON、Kryo

### 3.5 性能优化建议

- 对于只读场景，使用本地缓存减少 Redis 访问
- 合理设置 TTL，避免数据无限增长
- 使用管道（Pipeline）批量操作减少网络往返
- 监控 Redis 内存使用，及时清理过期数据

---

## 4. 分布式对象

### 4.1 RBucket - 分布式通用对象桶

**场景说明**：存储单个对象到 Redis，适用于缓存简单对象、分布式配置等场景。

**代码实现**：

```java
import org.redisson.api.RBucket;
import org.redisson.api.RedissonClient;

@Service
public class DistributedBucketService {

    @Autowired
    private RedissonClient redissonClient;

    public void bucketOperations(String bucketName) {
        RBucket<Object> bucket = redissonClient.getBucket(bucketName);

        bucket.set("simpleString");

        bucket.set(new User(1L, "张三", 25), 30, TimeUnit.SECONDS);

        Object value = bucket.get();
        System.out.println("获取值: " + value);

        RBucket<User> typedBucket = redissonClient.getBucket(bucketName, new UserCodec());
        typedBucket.set(new User(1L, "李四", 30));

        User user = typedBucket.get();
        System.out.println("获取用户: " + user);

        boolean isExists = bucket.isExists();
        System.out.println("bucket是否存在: " + isExists);

        bucket.delete();

        bucket.setIfAbsent("lockedValue");
        System.out.println("设置成功: " + bucket.get());

        bucket.setIfExists("newValue");
    }

    public void trySetAndGet(String bucketName) {
        RBucket<String> bucket = redissonClient.getBucket(bucketName);

        String oldValue = bucket.getAndSet("newValue");
        System.out.println("旧值: " + oldValue);

        String value = bucket.getAndDelete();
        System.out.println("删除的值: " + value);
    }
}

@Data
class User implements Serializable {
    private Long id;
    private String name;
    private Integer age;

    public User() {}
    public User(Long id, String name, Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
}
```

### 4.2 RAtomicLong - 分布式原子Long

**场景说明**：计数器、统计、分布式 ID 生成等需要原子操作的场景。

**代码实现**：

```java
import org.redisson.api.RAtomicLong;

@Service
public class DistributedAtomicService {

    @Autowired
    private RedissonClient redissonClient;

    public void atomicLongOperations(String counterName) {
        RAtomicLong atomicLong = redissonClient.getAtomicLong(counterName);

        atomicLong.set(0L);

        long incremented = atomicLong.incrementAndGet();
        System.out.println("递增后: " + incremented);

        long decremented = atomicLong.decrementAndGet();
        System.out.println("递减后: " + decremented);

        long added = atomicLong.addAndGet(10);
        System.out.println("加10后: " + added);

        long current = atomicLong.get();
        System.out.println("当前值: " + current);

        atomicLong.delete();
    }

    public void compareAndSet(String counterName) {
        RAtomicLong atomicLong = redissonClient.getAtomicLong(counterName);
        atomicLong.set(100);

        boolean success = atomicLong.compareAndSet(100, 200);
        System.out.println("CAS成功: " + success + ", 值: " + atomicLong.get());

        success = atomicLong.compareAndSet(100, 300);
        System.out.println("CAS失败: " + success + ", 值: " + atomicLong.get());
    }
}
```

### 4.3 RCountDownLatch - 分布式倒计时锁

**场景说明**：等待一组任务完成后再执行主任务，类似 JDK 的 CountDownLatch。

**代码实现**：

```java
import org.redisson.api.RCountDownLatch;

@Service
public class DistributedCountDownLatchService {

    @Autowired
    private RedissonClient redissonClient;

    public void countDownLatchExample(String latchName, int count) throws InterruptedException {
        RCountDownLatch latch = redissonClient.getCountDownLatch(latchName);
        latch.trySetCount(count);

        System.out.println("等待 " + count + " 个任务完成...");

        new Thread(() -> {
            for (int i = 0; i < count; i++) {
                try {
                    Thread.sleep(1000);
                    System.out.println("任务 " + (i + 1) + " 完成");
                    latch.countDown();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }).start();

        latch.await();
        System.out.println("所有任务已完成，继续执行主流程");
    }

    public void taskCompletion(String latchName) {
        RCountDownLatch latch = redissonClient.getCountDownLatch(latchName);
        latch.countDown();
        System.out.println("当前剩余: " + latch.getCount());
    }
}
```

### 4.4 RSemaphore - 分布式信号量

**场景说明**：控制同时访问某个资源的线程数量，如连接池限流、并发数限制。

**代码实现**：

```java
import org.redisson.api.RSemaphore;

@Service
public class DistributedSemaphoreService {

    @Autowired
    private RedissonClient redissonClient;

    public void semaphoreExample(String semaphoreName, int permits) throws InterruptedException {
        RSemaphore semaphore = redissonClient.getSemaphore(semaphoreName);
        semaphore.trySetPermits(permits);

        for (int i = 0; i < 5; i++) {
            final int threadId = i;
            new Thread(() -> {
                try {
                    System.out.println("线程 " + threadId + " 等待获取信号量");
                    semaphore.acquire();
                    System.out.println("线程 " + threadId + " 获取到信号量，开始执行");
                    Thread.sleep(2000);
                    System.out.println("线程 " + threadId + " 释放信号量");
                    semaphore.release();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }).start();
        }
    }

    public boolean tryAcquire(String semaphoreName) {
        RSemaphore semaphore = redissonClient.getSemaphore(semaphoreName);
        return semaphore.tryAcquire();
    }

    public void release(String semaphoreName) {
        RSemaphore semaphore = redissonClient.getSemaphore(semaphoreName);
        semaphore.release();
    }
}
```

### 4.5 注意事项

- **对象大小**：RBucket 存储对象不宜过大，影响序列化/反序列化性能
- **TTL 管理**：为避免数据永不过期导致内存溢出，建议总是设置合理的 TTL
- **类型安全**：推荐使用泛型版本的 Bucket，避免类型转换异常

### 4.6 性能优化建议

- 对于频繁读取的数据，考虑使用本地缓存
- 使用合适的编解码器减少序列化开销
- 避免存储过大的对象，可以分片存储

---

## 5. 分布式限流方案

### 5.1 基于 RRateLimiter 的令牌桶限流

**场景说明**：限制 API 接口调用频率、防止 DDoS 攻击、控制并发访问。

**代码实现**：

```java
import org.redisson.api.RRateLimiter;
import org.redisson.api.RateIntervalUnit;
import org.redisson.api.RateType;

@Service
public class RateLimiterService {

    @Autowired
    private RedissonClient redissonClient;

    public void initRateLimiter(String limiterName) {
        RRateLimiter limiter = redissonClient.getRateLimiter(limiterName);

        limiter.trySetRate(RateType.OVERALL, 100, 1, RateIntervalUnit.SECONDS);
        System.out.println("限流器已初始化: 每秒100个请求");
    }

    public boolean tryAcquire(String limiterName, int permits) {
        RRateLimiter limiter = redissonClient.getRateLimiter(limiterName);
        return limiter.tryAcquire(permits);
    }

    public boolean tryAcquireWithTimeout(String limiterName, int permits, long timeout, TimeUnit unit) {
        RRateLimiter limiter = redissonClient.getRateLimiter(limiterName);
        return limiter.tryAcquire(permits, timeout, unit);
    }

    public void acquire(String limiterName, int permits) {
        RRateLimiter limiter = redissonClient.getRateLimiter(limiterName);
        limiter.acquire(permits);
    }

    public boolean executeWithRateLimit(String limiterName, int permits, Runnable task) {
        RRateLimiter limiter = redissonClient.getRateLimiter(limiterName);
        if (limiter.tryAcquire(permits)) {
            try {
                task.run();
                return true;
            } finally {
            }
        }
        return false;
    }
}
```

### 5.2 分布式滑动窗口限流

**场景说明**：需要更精确的限流控制，基于时间窗口的算法。

**代码实现**：

```java
import org.redisson.api.RMap;

@Service
public class SlidingWindowRateLimiter {

    @Autowired
    private RedissonClient redissonClient;

    private static final long WINDOW_SIZE_MS = 1000;
    private static final int MAX_REQUESTS = 100;

    public boolean isAllowed(String userId) {
        String key = "rate_limit:" + userId;
        RMap<Long, Long> map = redissonClient.getMap(key);

        long now = System.currentTimeMillis();
        long windowStart = now - WINDOW_SIZE_MS;

        map.keySet().removeIf(timestamp -> timestamp < windowStart);

        Long currentCount = map.get(now);
        if (currentCount == null) {
            currentCount = 0L;
        }

        if (currentCount >= MAX_REQUESTS) {
            return false;
        }

        map.put(now, currentCount + 1);
        map.expire(WINDOW_SIZE_MS * 2, TimeUnit.MILLISECONDS);

        return true;
    }

    public boolean isAllowedWithLua(String userId) {
        String key = "rate_limit:lua:" + userId;

        RMap<Long, Long> map = redissonClient.getMap(key);

        long now = System.currentTimeMillis();
        long windowStart = now - WINDOW_SIZE_MS;

        Long count = map.values().stream()
                .filter(timestamp -> timestamp >= windowStart)
                .count();

        return count < MAX_REQUESTS;
    }
}
```

### 5.3 注解方式限流

**代码实现**：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimit {
    String key();
    int permits() default 1;
    long timeout() default 0;
    String unit() default "SECONDS";
}

@Aspect
@Component
public class RateLimitAspect {

    @Autowired
    private RedissonClient redissonClient;

    @Around("@annotation(rateLimit)")
    public Object around(ProceedingJoinPoint joinPoint, RateLimit rateLimit) throws Throwable {
        RRateLimiter limiter = redissonClient.getRateLimiter(rateLimit.key());
        limiter.trySetRate(RateType.OVERALL, rateLimit.permits(), 1,
                RateIntervalUnit.valueOf(rateLimit.unit()));

        if (limiter.tryAcquire(rateLimit.permits(), rateLimit.timeout(), TimeUnit.SECONDS)) {
            return joinPoint.proceed();
        }
        throw new RuntimeException("请求过于频繁，请稍后再试");
    }
}

@Service
public class RateLimitedService {

    @RateLimit(key = "api:user:list", permits = 100, timeout = 5, unit = "SECONDS")
    public List<User> getUserList() {
        return userRepository.findAll();
    }
}
```

### 5.4 注意事项

- **限流维度**：根据实际业务选择限流维度（用户 ID、IP、接口等）
- **突发流量**：令牌桶算法允许一定程度的突发流量，滑动窗口算法更平滑
- **分布式一致性**：注意多节点部署时限流的准确性

### 5.5 性能优化建议

- 限流 key 避免过于精细导致 Redis 内存压力
- 可以结合本地限流（如 Guava RateLimiter）做二级限流
- 使用 Redis Cluster 分散热点 key 的压力

---

## 6. 消息队列

### 6.1 RQueue - 分布式队列

**场景说明**：异步任务处理、任务分发、消息传递等场景。

**代码实现**：

```java
import org.redisson.api.RQueue;

@Service
public class DistributedQueueService {

    @Autowired
    private RedissonClient redissonClient;

    public void queueOperations(String queueName) {
        RQueue<String> queue = redissonClient.getQueue(queueName);

        queue.offer("message1");
        queue.offer("message2");
        queue.offer("message3");

        String first = queue.peek();
        System.out.println("查看队首: " + first);

        String removed = queue.poll();
        System.out.println("取出队首: " + removed);

        int size = queue.size();
        System.out.println("队列大小: " + size);

        boolean isEmpty = queue.isEmpty();
        System.out.println("是否为空: " + isEmpty);
    }

    public void fifoTaskProcessing(String taskQueueName) {
        RQueue<Task> taskQueue = redissonClient.getQueue(taskQueueName);

        taskQueue.offer(new Task(1L, "任务1"));
        taskQueue.offer(new Task(2L, "任务2"));

        while (!taskQueue.isEmpty()) {
            Task task = taskQueue.poll();
            if (task != null) {
                processTask(task);
            }
        }
    }

    private void processTask(Task task) {
        System.out.println("处理任务: " + task.getName());
    }
}

@Data
class Task implements Serializable {
    private Long id;
    private String name;

    public Task() {}
    public Task(Long id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

### 6.2 RBlockingQueue - 阻塞队列

**场景说明**：消费者需要等待队列中有元素时才处理，适合生产者-消费者模式。

**代码实现**：

```java
import org.redisson.api.RBlockingQueue;

@Service
public class BlockingQueueService {

    @Autowired
    private RedissonClient redissonClient;

    public void blockingQueueConsumer(String queueName) throws InterruptedException {
        RBlockingQueue<String> blockingQueue = redissonClient.getBlockingQueue(queueName);

        while (true) {
            String message = blockingQueue.poll(10, TimeUnit.SECONDS);
            if (message != null) {
                System.out.println("收到消息: " + message);
                processMessage(message);
            }
        }
    }

    public void blockingQueueWithTimeout(String queueName) throws InterruptedException {
        RBlockingQueue<String> blockingQueue = redissonClient.getBlockingQueue(queueName);

        String message = blockingQueue.poll(30, TimeUnit.SECONDS);
        if (message != null) {
            System.out.println("收到消息: " + message);
        } else {
            System.out.println("等待超时，无新消息");
        }
    }

    private void processMessage(String message) {
        System.out.println("处理消息: " + message);
    }
}
```

### 6.3 RDelayedQueue - 延迟队列

**场景说明**：订单超时取消、延迟任务调度、缓存预热等需要延迟处理的场景。

**代码实现**：

```java
import org.redisson.api.RDelayedQueue;

@Service
public class DelayedQueueService {

    @Autowired
    private RedissonClient redissonClient;

    public void delayedQueueExample(String targetQueueName) {
        RQueue<String> targetQueue = redissonClient.getQueue(targetQueueName);

        RDelayedQueue<String> delayedQueue = redissonClient.getDelayedQueue(targetQueue);

        delayedQueue.offer("消息1", 5, TimeUnit.SECONDS);
        delayedQueue.offer("消息2", 10, TimeUnit.SECONDS);
        delayedQueue.offer("消息3", 30, TimeUnit.SECONDS);

        System.out.println("延迟消息已放入，将分别在5秒、10秒、30秒后到达");
    }

    public void orderTimeoutCancel(String orderQueueName, long orderId) throws InterruptedException {
        RBlockingQueue<Order> orderQueue = redissonClient.getBlockingQueue(orderQueueName);
        RDelayedQueue<Order> delayedQueue = redissonClient.getDelayedQueue(orderQueue);

        Order order = new Order(orderId, "待支付", System.currentTimeMillis());
        delayedQueue.offer(order, 30, TimeUnit.MINUTES);

        System.out.println("订单已加入延迟队列，30分钟后检查支付状态");
    }

    public void consumeDelayedMessages(String queueName) throws InterruptedException {
        RBlockingQueue<String> queue = redissonClient.getBlockingQueue(queueName);

        while (true) {
            String message = queue.take();
            System.out.println("收到延迟消息: " + message);
        }
    }
}

@Data
class Order implements Serializable {
    private Long id;
    private String status;
    private Long createTime;

    public Order() {}
    public Order(Long id, String status, Long createTime) {
        this.id = id;
        this.status = status;
        this.createTime = createTime;
    }
}
```

### 6.4 RTopic - 发布订阅

**场景说明**：广播消息、事件通知、多消费者场景。

**代码实现**：

```java
import org.redisson.api.RTopic;
import org.redisson.api.listener.MessageListener;

@Service
public class PubSubService {

    @Autowired
    private RedissonClient redissonClient;

    public void publishMessage(String topicName, String message) {
        RTopic topic = redissonClient.getTopic(topicName);

        long subscribers = topic.getPlayers().size();
        System.out.println("当前订阅者数量: " + subscribers);

        topic.publish(message);
        System.out.println("消息已发布: " + message);
    }

    public void publishOrderEvent(String topicName, OrderEvent event) {
        RTopic<OrderEvent> topic = redissonClient.getTopic(topicName);
        topic.publish(event);
        System.out.println("订单事件已发布: " + event.getType());
    }

    public int subscribe(String topicName) {
        RTopic<String> topic = redissonClient.getTopic(topicName);

        int listenerId = topic.addListener(new MessageListener<String>() {
            @Override
            public void onMessage(String channel, String msg) {
                System.out.println("收到消息: " + msg + ", 来自频道: " + channel);
            }
        });

        return listenerId;
    }

    public void subscribeOrderEvents(String topicName) {
        RTopic<OrderEvent> topic = redissonClient.getTopic(topicName);

        topic.addListener(new MessageListener<OrderEvent>() {
            @Override
            public void onMessage(String channel, OrderEvent event) {
                System.out.println("收到订单事件: " + event.getType() + ", 订单ID: " + event.getOrderId());
                handleOrderEvent(event);
            }
        });
    }

    private void handleOrderEvent(OrderEvent event) {
        switch (event.getType()) {
            case "CREATED":
                System.out.println("处理订单创建");
                break;
            case "PAID":
                System.out.println("处理订单支付");
                break;
            case "CANCELLED":
                System.out.println("处理订单取消");
                break;
        }
    }
}

@Data
class OrderEvent implements Serializable {
    private String type;
    private Long orderId;
    private Long timestamp;

    public OrderEvent() {}
    public OrderEvent(String type, Long orderId) {
        this.type = type;
        this.orderId = orderId;
        this.timestamp = System.currentTimeMillis();
    }
}
```

### 6.5 注意事项

- **消息持久化**：RQueue 基于 Redis List 实现，消息持久化在内存中，重启会丢失
- **消费确认**：需要自行实现消费确认机制，避免消息丢失
- **顺序性**：Redis 队列不保证消息严格顺序，如需顺序需额外处理
- **监控**：监控队列长度，避免消息积压

### 6.6 性能优化建议

- 对于高可靠性场景，建议使用专业的消息队列（RabbitMQ、RocketMQ）
- 设置合理的队列大小限制
- 消费者异常处理使用 try-catch，避免消息丢失
- 考虑消息压缩减少网络传输


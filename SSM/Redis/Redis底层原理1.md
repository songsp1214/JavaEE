# Redis原理解析
---
你可能不知道自己的潜力有多大，尽情的来吧。        --[RuiDer](https://ruider.github.io)

---
![](https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=1492620793,3977100373&fm=27&gp=0.jpg)
### Before
- Java基础
- Spring
- Mabatis
- Redis
- Belief

### Redis概念介绍
```java 
Redis:一款NoSql技术，面向Java Web。它就是一个简单的基于内存的数据库，
并提供持久化服务。Redis和MongoDB是当前使用最广泛的NoSql，主要应对每秒
几十万的读写操作，其性能远远超过数据库。并且在高并发下保证数据的一致性和
安全性。

```
### Redis优势
- 使用ANSI C语言完成，接近汇编语言，运行速度快
- 基于内存读/写，读写速度相当快。
- 支持6种数据结构，分别是字符串String，链表List，集合Set，散列hash，和基数HyperLogLog，以及有序集合Zset，规则相对较少，处理快。

综上优势，Redis的读写速度更快。。

### Redis使用
```Java
Redis的使用有两种方式：
	- 与Spring结合Spring-Data-Redis(RedisTemplate对象操作)
	- Jedis操作Redis的数据读写
两种方式的使用截然不同。

```

### 缓存
```Java
持久化数据库的缺点与优点：
	-数据从数据库读取也就是索引磁盘速度慢
	-数据写如数据库速度还说的过去
在Java Web领域，对于数据库的操作读的操作远超过写的操作，一般是1:9到3:7 的比例。对于
数据库中的数据，读取操作会直接进行整个磁盘的索引，而索引整个磁盘的速度是非常慢的，但是
读取内存的速度就不一样了，会非常速度。

缓存的缺点：
	磁盘的容量一般是TGB级别，不同于内存的小容量一般是GB级别，容量有限，而且磁盘十分
	廉价，内存价格也比磁盘昂贵。
```
### 何时使用缓存Redis
```Java
因为缓存基于内存，内存空间有限，价格高贵，所以应该有条件限制
使用Redis的条件：
	- 数据使用频繁，读取操作多于写操作
	- 业务数据经常被使用吗？命中率如何？
	- 业务数据大小如何？
一般而言，缓存的作用主要用于存储一些常用的数据，
比如用户登录信息，银行卡信息等。。

```

###  Redis缓存简单原理
```Java
使用缓存是因为缓存的读取速度快，而考虑到写操作时，数据库的写操作与其速度差不到那去，
所以直接考虑缓存的读取操作
- 当第一次读取数据时，首先检测Redis中是否含有该数据
- 如果有直接读取
- 如果没有，从数据库读取，写入Redis
- 第二遍读取数据时，直接从缓存中读取了

```
### 高速读/写场合
```Java
在互联网行业，对于高并发业务，比如秒杀和抢红包这些业务，
需要高质量的缓存机制相应千万级别的访问量。如果使用数据库，
数据库每秒处理万级别的sql语句，简直是异想天开，有时会使数据库瘫痪，甚至有时
会使服务器奔溃。
	在这样的场合一般考虑使用异步写入数据库。
```

### 异步写入数据库
```Java
	对于高速读/写场合中单单使用Redis的场景，把这些需要高速读/写的数据，
缓存到Redis中，而在满足一定条件时，触发这些缓存的数据写入数据库中去。
	细细解释一下这个问题，当一个请求发送数据当服务器时，一开始存储数据是
Redis缓存，没有进行任何关于数据库的操作，但是缓存不能持久化，因此需要
需要把这些数据存入数据库，因此会在一定条件时判断一个请求是否结束，一旦结束，
就会将缓存中的数据更新到数据库。而这个条件就像秒杀或者抢红包业务中红包数量
是否为0，如果为空，一次性写入数据库，完成持久化工作。
```
## Redis的原理解析
此部分需要从两个方面为读者解读，不管是配置也好还是代码堆积也好，他们之间紧紧地联系在一起，读者可以参考阅读。

- Spring环境下的Redis配置细节
- Java环境下的Jedis操作

### Spring IOC原理简单回忆
SpringIoc控制反转技术简单的理解就是管理我们日常所用的JavaBean类或者对象，因为一个Bean对象拥有属性和它们各自对应的getter和setter方法等，（只是简单的举JavaBean对象，其他对象也可以），SpringIoc容器干了什么呢？通过我们人为的配置，SpringIOC会在容器中保存对应的Bean对象。下面代码介绍：

```Java
package com.ruider;
public class Person{
	private String name;
	private int age;
	/** getter and setter **/
}


这是一个Person类，拥有属性name和age
初始化一个对象：

Person a=new  Person();
a.setName("a");
a.setAge(20);

再看看我们的Spring中的配置

<Bean id="person" class="com.ruider.Person">
	<property name="name" value="a"/>
	<property name="age" value="20"/>
</Bean>

使用配置获取Person对象

@Autowired
Person a=null;

不管是Java代码式创建对象还是使用SpringIOC容器获取对象，他们都是等价的。
理解这一点会有助于后面的Redis原理解析。
```
###Redis属性以及连接池的属性
Redis：（connectionFactory常用的）

```
- jedisCOnfig连接池配置信息
- port端口，一般是6379
- host或者localhost
- password 密码
- TIMEOUT 超时
```
Redis连接池：（JedisPoolConfig常用）
```
- 最大空闲数MaxIdle
- 最大连接数MaxTotal
- 最大等待时间MaxWaitingTimes
```
Redis使用对象RedisTemplate属性
```
- connectionFactory连接工厂（相当于连接池）
- keySerializer Redis键序列化转换类型
- valueSerializer Redis值序列化转换类型
```
### Redis配置式redis-config.xml
根据上述的三种属性介绍进行Redis的属性配置
```Java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx" xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.2.xsd
     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
     http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context-3.2.xsd"
    default-autowire="byName" default-lazy-init="true">

    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="${redis.maxIdle}" />
        <property name="maxWaitMillis" value="${redis.maxWait}" />
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
    </bean>
    <!-- redis服务器中心 -->
    <bean id="connectionFactory"
        class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="poolConfig" ref="poolConfig" />
        <property name="port" value="${redis.port}" />
        <property name="hostName" value="${redis.host}" />
        <property name="password" value="${redis.password}" />
        <property name="timeout" value="${redis.timeout}"></property>
    </bean>
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="keySerializer">
            <bean
                class="org.springframework.data.redis.serializer.StringRedisSerializer" />
        </property>
        <property name="valueSerializer">
            <bean
                class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
        </property>
    </bean>

```
（1）其中对于JedisPoolConfig连接池的配置
```
<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="${redis.maxIdle}" />
        <property name="maxWaitMillis" value="${redis.maxWait}" />
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
    </bean>
```
至此，JedisPoolConfig的对象已经存储在SpringIOC容器中，在程序应用中可以通过`@Autowired`获取JedisPoolConfig的对象。

上面JedisPoolConfig的配置用Java语句实现如下：
```java
public class JedisPoolDemo{
	public static void main(String[] args){
		JedisPoolConfig jedisPoolConfig=new JedisPoolConfig();
		jedisPoolConfig.setMaxIdle(20);   //连接池最大空闲数
		jedisPoolConfig.setMaxTotal(100);  //连接池最大连接数
		jedisPoolConfig.setMaxWaitingMillis(2000);//最大等待时间为2秒
	}
}
```
（2）其中ConnectionFactory的配置如下
```
<bean id="connectionFactory"
        class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="poolConfig" ref="poolConfig" />
        <property name="port" value="${redis.port}" />
        <property name="hostName" value="${redis.host}" />
        <property name="password" value="${redis.password}" />
        <property name="timeout" value="${redis.timeout}"></property>
    </bean>
```
至此ConnectionFactory的对象已经存储在SpringIOC容器中，在程序应用中可以通过`@Autowired`获取JedisPoolConfig的对象。

上面ConnectionFactory的配置用Java语句实现如下：
```java
	//JedisPoolConfig设置
	JedisPoolConfig jedisPoolConfig=new JedisPoolConfig();
	jedisPoolConfig.setMaxIdle(20);   //连接池最大空闲数
	jedisPoolConfig.setMaxTotal(100);  //连接池最大连接数
	jedisPoolConfig.setMaxWaitingMillis(2000);//最大等待时间为2秒
	 
	 //ConnectionFactory的设置
	ConnectionFactory connectionFactory=new ConnectionFactory(jedisPoolConfig);
	connectionFactory.setHost("localhost");   
	connectionFactory.setPort("6379");  
	connectionFactory.setTimeOut(2000);
}
```
（3）其中RedisTemplate的配置如下：
```
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="keySerializer">
            <bean
                class="org.springframework.data.redis.serializer.StringRedisSerializer" />
        </property>
        <property name="valueSerializer">
            <bean
                class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
        </property>
    </bean>
```
至此RedisTemplate的对象已经存储在SpringIOC容器中，在程序应用中可以通过`@Autowired`获取RedisTemplate的对象。

上面ConnectionFactory的配置用Java语句实现如下：
```java
public class RedisTempalteDemo{
	public static void main(String[] args){
		//JedisPoolConfig设置
		JedisPoolConfig jedisPoolConfig=new JedisPoolConfig();
		jedisPoolConfig.setMaxIdle(20);   //连接池最大空闲数
		jedisPoolConfig.setMaxTotal(100);  //连接池最大连接数
		jedisPoolConfig.setMaxWaitingMillis(2000);//最大等待时间为2秒
		 
		 //ConnectionFactory的设置
		ConnectionFactory connectionFactory=new ConnectionFactory(jedisPoolConfig);
		connectionFactory.setHost("localhost");   
		connectionFactory.setPort("6379");  
		connectionFactory.setTimeOut(2000);
		
		//RedisTemplate设置
		RedisTemplate redisTemplate=new RedisTemplate();
		redisTemplate.setConnectionFactory(connectionFactory);
		//Seriallizer设置
		StringSerializer stringSerializer=new StringSerializer();
		redisTemplate.setKeySerializer(stringSerializer);
		redisTemplate.setValueSerializer(stringSerializer);
	}
}
```

### 补充上面redis-config.xml用到的属性配置
新建redis.properties
```
#redis setting  
redis.host=127.0.0.1
redis.port=6379
redis.password=123456
redis.maxIdle=100
redis.maxActive=300
redis.maxWait=1000
redis.testOnBorrow=true
redis.timeout=100000

fep.local.cache.capacity =10000
```
### Redis底层原理
篇幅太长，详情在[Redis的底层](https://blog.csdn.net/qq_40910541/article/details/81530607)有介绍，主要介绍一些Redis的命令以及这些命令被Java封装的使用方式。
### About Me
[我的个人博客](https://ruider.github.io)
[我的Github](https://github.com/RuiDer)

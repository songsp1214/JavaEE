# <center>Mybatis缓存</center>
---
因为热爱，所以拼搏。        --[RuiDer](ruider.github.io)

---
### 前导必备
	- 数据库
	- Mybatis
### Mybatis缓存
> Mybatis提供缓存支持，志在提升开发的性能。数据库数据的索引一般是基于磁盘的，而内存和高速缓存相对磁盘的读写速度相当快，基本是磁盘读写数据的十倍。尤其在互联网开发中，追求速度，缓存为性能提供捷径。Mybatis缓存包括一级缓存和二级缓存，一级缓存基于SqlSession层面，二级缓存基于SqlSessionFactory层面。

### 一级缓存和二级缓存
```java
缓存：将sql语句select查询出来对应的POJO对象进行缓存，供后续SqlSession对象使用。

一级缓存
	条件：
	- 同一个SqlSession对象
	- 同一个sql查询语句
	当同一个SqlSession对象进行两次相同select语句查询时，第一次会执行sql语句进行数据库的索引，
之后将所查询数据对应的POJO对象进行缓存；第二次执行相同参数的sql语句查询前，不会像第一次执行sql语句，
而是直接从缓存中索引数据对象。所以说，一级缓存是基于SqlSession的。不同的SqlSession对象的缓存不可以
共享，不能相互访问。为了解决相互共享问题，Mybatis提供二级缓存解决该问题。注意，一级缓存对每个SqlSession
对象要求他们在所有sql语句后必须执行commit语句，否则，不存在一级缓存。

	二级缓存
	条件：
	- 同一个SqlSessionFactory对象
	- 不同的SqlSession对象
	- 共享POJO对象、
	二级缓存是相对SqlSessionFactory对象而言的，同一个SqlSessionFactory创建的不同的SqlSession对象，
可以共享该SqlSessionFactory下的所有POJO对象，也就是数据库数据的共享。但是二级缓存要求对应的POJO类需要实现
序列化，也就是实现java.io.Serializable接口。
```
### 一级缓存 和 二级缓存 的配置
```Java
一级缓存：Mybatis默认一级缓存，只要每个SqlSession对象执行commit语句，都会存在一级缓存。
二级缓存：
	<cache
	...sql语句...
	/>
	加入上述语句，就会开启二级缓存。二级缓存会进行序列化和反序列化的过程，所以说POJO
必须要实现Seriaizable接口。二级缓存cache元素会将select出来的POJO对象进行缓存，对于
insert,update,delete sql语句二级缓存会自动刷新。
```
### 二级缓存配置cache元素配置项
```Java
//cache接口源码
public interface Cache{
	//获取缓存id
	public int getId();
	
	//保存POJO对象,key为键，value为值
	public void putObject(Object key,Object value);

	//获取POJO对象
	public Object getObject(Object key);
	
	//清除缓存
	void clear();

	//获取缓存大小
	public int getSize();
	
	//获取读写锁,主要是多线程时使用
	ReadWriteLock getReadWriteLock();
}
```
### 配置Redis或者MongoDB缓存处理
```java
<cache type="com....RedisCache">
	<property name="Host" value="localHost">
</cache>
```

[我的CSDN](https://blog.csdn.net/qq_40910541)

[我的博客](ruider.github.io)

# <center>Mybatis N+1问题解析</center>
---
因为热爱，所以拼搏。           --[RuiDer](ruider.github.io)

---
### 前导必备
- Mybatis
- 数据库
- 级联
### N+1问题？？
> N+1问题来源于数据库中常见的级联技术，即N个数据库表形成关联关系，当再增加一个关联表时，也就是N+1个级联关系，由于某些时候，我们并不需要加载数据库的所有数据，而是某一个数据库表中数据，这时Mybatis会自动加载所有表的数据，多执行几条无关sql语句，会造成数据库资源的浪费以及系统性能的下降，这就是级联表的缺点。
### 如何解决N+1问题
> Mybatis本身给出解决方案，就是延迟加载。
### 延迟加载
> 延迟加载会解决上述的N+1问题，也就是在N+1个级联表的情况下，只加载需求的数据库表数据。这是互联网发展的需求，性能提升的途径。
### 如何配置Mybatis完成延迟加载
```Java
全局配置：
	- lazyLoadingEnabled        true/false
	- aggressiveLazyLoading      true/false
	
	lazyLoadingEnabled:延迟加载的全局开关，当开启时，所有关联都会延迟加载。在特定的关联中，
使用fetchType属性覆盖该内容的功能。fetchType将在后面介绍。

	aggressiveLazyLoading：是层级延迟加载开关，什么意思呢？就是处于同一个层级的关联表会同
时延迟加载，或者同时被加载。
	
	配置：
		在Mybatis的全局配置中的setting标签中加入设置
		<setting>
			<setting name="lazyLoadingEnabled" value="true"/>
			<setting name="aggressiveLazyLoading" value="true"/>
		</setting>
```
### 全局配置的缺点
上面的配置属于全局配置，会出现一个问题，同一个层级的级联表也存在需求性的差异，同一级的某个数据库表的数据或许不是我们经常使用的，那么上述的配置也会影响系统的性能。

### 全局配置的优化 --- fetchType属性
> Mybatis使用fetchType属性解决全局配置的缺点。fetchType出现在级联元素association,collection中，它存在两个值
```Java
	- eager:获得当前POJO后立即加载对应的数据。
	- lazy:获得当前POJO后延迟加载对应的数据。
	
配置：
		<collection properties=".." column=".." fetchType="eager"
			select="映射接口"
		/>
```
[我的github](https://github.com/RuiDer)
[我的博客](ruider.github.io)

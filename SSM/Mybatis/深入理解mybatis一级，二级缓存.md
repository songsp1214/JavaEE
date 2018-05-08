# 深入理解Mybatis一二级缓存
> 因为热爱，所以拼搏。。          --来自大二的艺术家     RuiDer

---------------------------------
## 缓存
`合理使用缓存是优化中最常见的，将从数据库中查询出来的数据放入缓存中，下次使用时不必从数据库查询，
而是直接从缓存中读取，避免频繁操作数据库，减轻数据库的压力，同时提高系统性能。`
 
#### 一级缓存
`一级缓存是SqlSession级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构用于存储缓存数据。
不同的sqlSession之间的缓存数据区域是互相不影响的。也就是他只能作用在同一个sqlSession中，不同的sqlSession中的缓
存是互相不能读取的。`
 
一级缓存的工作原理：
![](http://img.blog.csdn.net/20170613200330598)
`
用户发起查询请求，查找某条数据，sqlSession先去缓存中查找，是否有该数据，如果有，读取；
如果没有，从数据库中查询，并将查询到的数据放入一级缓存区域，供下次查找使用。
但sqlSession执行commit，即增删改操作时会清空缓存。这么做的目的是避免脏读。
如果commit不清空缓存，会有以下场景：A查询了某商品库存为10件，并将10件库存的数据存入缓存中，
之后被客户买走了10件，数据被delete了，但是下次查询这件商品时，并不从数据库中查询，而是从缓存中查询，就会出现错误。
`
#### 二级缓存
`
二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。
二级缓存的作用范围更大。
还有一个原因，实际开发中，MyBatis通常和Spring进行整合开发。Spring将事务放到Service中管理，对于每一个service中的sqlsession是不同的，
这是通过mybatis-spring中的org.mybatis.spring.mapper.MapperScannerConfigurer创建sqlsession自动注入到service中的。 每次查询之后都要
进行关闭sqlSession，关闭之后数据被清空。所以spring整合之后，如果没有事务，一级缓存是没有意义的。
`
二级缓存原理：
![](http://img.blog.csdn.net/20170613200342848)
`
二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。
UserMapper有一个二级缓存区域（按namespace分），其它mapper也有自己的二级缓存区域（按namespace分）。每一个namespace的mapper都有一个
二级缓存区域，两个mapper的namespace如果相同，这两个mapper执行sql查询到数据将存在相同的二级缓存区域中。
`
## 开启二级缓存
1，打开总开关
在MyBatis的配置文件中加入：
```
<span style="font-size:18px;"><settings>    
   <!--开启二级缓存-->    
    <setting name="cacheEnabled" value="true"/>    
</settings> </span>  
```
------------------------------
2，在需要开启二级缓存的mapper.xml中加入caceh标签
```
<span style="font-size:18px;"><cache/></span> 
```
# resultType和resultMap的区别

---
Go ahead!!                                       --[RuiDer](https:ruider.github.io)

---
> 这几天初次上手公司的项目，以前没有注意过mybatis-config.xml中的resultType(一直在用这个，不咋用resultMap)，近几日撸完一个项目，测试时发现有些问题，Mybatis-config.xml一直出错，检查后，发现原因在于resultType和resultMap的混用。在此来反思一波，希望读者慎重。
### before 
- Java
- Mybatis
- Mysql
- Spring
### 对比学习，读者自己揣摩区别
先给出数据库表user
```mysql
ID     user_name   age
1       张三        22
2       李四        23
```
给出实体类User
```java
public class User{
	private int userId;
	private String userName;
	private int age;
	/**
	setter and getter
	**/
}
```
给出dao层
```java 
import java.util.List;
public interface UserMapper{
	List<User>  findAll();
}
```
#### 区别：（两种mybatis-config.xml配置）
@1 使用resultType
```java
<mapper namespace="UserMapper">
	<select id="findAllUser" resultType="User">
	select 
	ID as 'userId',
	user_name as 'userName',
	age
	from User
	</select>
</mapper>
```
@2 使用resultMap
```java
<mapper namespace="UserMapper">
	<resultMap id="user" type="User">
		<result property="userId" column="ID" javaType="int">
		<result property="userName" column="user_name" javaType="String">
		<result property="age" column="age" javaType="int">
	</resultMap>
	<select id="findAllUser" resultMap="user">
	select 
	ID,
	user_name,
	age
	from User
	</select>
</mapper>
```

### 对比即可得出结论，作者不予总结


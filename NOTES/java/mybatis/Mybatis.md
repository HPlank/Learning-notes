Mybatis

### mybatis中的参数绑定

#{ } 解析一个JDBC预编译语句{PreparedStatement}的参数标记占位符？。使用该方式可以避免SQL注入。一般用于参数绑定。

${ } 仅仅是一个纯粹的String替换，在mybatis的动态sql解析阶段进行变量替换（表名或列名）。存在sql注入问题。

### ThreadLocal:

threadLocal提供了线程内存储变量的能力，这些变量的不同之处在于每一个线程读取的变量是对应的相互独立的。通过get和set方法就可以得到当前线程对应的值

#### 使用ThreadLocal存储SqlSession

若多个DML错做属于一个事务，因为commit（）和rollback（）都是由SqlSession完成的，所以必须要保证使用一个SqlSession。到那时多个不同的DML操作可能在不同类的不同方法中，每个方法要单独的获取SqlSession。

所以使用ThreadLocal来存储，保证线程中的操作都是使用一个SqlSession

### mybatis事务提交方式

mybatis事务提交反射光hi默认为手动提交，jdbc中默认是自动提交的

手动：

```java
SqlSession sqlSession = sqlSessionFactory.openSession();
```

自动：

```java
SqlSession sqlSession = sqlSessionFactory.openSession(true);
```

### Mybatis动态代理

Mapper 动态代理（或称为接口 绑定）的操作方式。

这种方式下程序员只需要写 Dao 接口，不需要创建 Dao 的接口实现类， 

Mybatis 会自动生成接口实现类的代理对象。

在 Dao 层我们只要创建接口与映射配置文件即 可。

这种方式可以大大简化 Dao 层代码结构。

Mapper 动态代理规范

  接口名称需要与映射配置文件名称相同 

 映射配置文件中 namespace 必须是接口的全名。 

 接口中的方法名和映射配置文件中的标签的 id 一致。

 接口中的返回值类型和映射配置文件中的 resultType 的指定的类型一致

### Mapper 动态代理模式下的多参数处理

#### 1.顺序传值法

在映射文件中，SQL 语句中的参数需要使用 arg0，arg1...或者 param1，param2...表示参 数的顺序。

此方法可读性低，且要求参数的顺序不能出错，在开发中不建议使用。

```
<!--根据用户姓名与性别查询用户,使用顺序传参法-->
<select id="selectUsersOrderParam" resultType="users">
	select * from users where username = #{arg0} and usersex= #{arg1}
</select>

<!--根据用户姓名与性别查询用户,使用顺序传参法-->
<select id="selectUsersOrderParam" resultType="users">
	select * from users where username = #{arg0} and usersex= #{arg1}
</select>

<!--根据用户姓名与性别查询用户,使用顺序传参法-->
<select id="selectUsersOrderParam" resultType="users">
	select * from users where username = #{param1} and usersex = #{param2}
</select>
```



#### 2.@Param 注解传参法

在接口方法的参数列表中通过@Param 注解来定义参数名称，在 SQL 语句中通过注解中 所定义的参数名称完成参数位置的指定。

```
!--根据用户姓名与性别查询用户,使用@Param 注解传参法-->
<select id="selectUsersAnnParam" resultType="users">
	select * from users where username = #{name} and usersex= #{sex}
</select>
selectUsersAnnParam( @Param("name") String username, @Param("sex") String usersex );
```



#### 3.POJO 传参法

在 Mapper 动态代理中也可以使用 POJO 作为传递参数的载体，在 SQL 语句中绑定参数 时使用 POJO 的属性名作为参数名即可。

此方式推荐使用。

```
<!--根据用户姓名与性别查询用户,使用 POJO 传参法-->
<select id="selectUsersPOJOParam" resultType="users">
	select * from users where username = #{username} and usersex = #{usersex}
</select>
```

```
List<Users> selectUsersPOJOParam(Users users);
```

#### 4.Map 传参

在 SQL 语句中绑定参数 时使用 Map 的 Key 作为参数名即可。

此方法适合在传递多参数时，如果没有 POJO 能与参数 匹配，可以使用该方式传递参数。推荐使用

MyBatis 传递 map 参数时，如果传递参数中没有对应的 key 值，在执行 sql 语句时默认 取的是 null

```
<!--根据用户姓名与性别查询用户,使用 Map 传参法-->
<select id="selectUsersMapParam" resultType="users">
	select * from users where username = #{keyname} and usersex= #{keysex}
</select>

List<Users> selectUsersMapParam(Map<String,String> map);
```

### 映射配置文件中的特殊字符处理

在 Mybatis 的映射配置文件中不可以使用一些特殊字符，如：<，>。

#### 使用符号实体

![image-20220116132839139](C:\Users\郝康将\AppData\Roaming\Typora\typora-user-images\image-20220116132839139.png)

```
<select id="selectUsers" resultType="users">
	select * from users where userid &gt; #{userid}
</select>
```

#### 使用 CDA

CDATA：全称为 Character Data,以""，CDATA 中的内容不会被解析程 序解析。

```
<select id="selectUsers" resultType="users">
	select * from users where userid <![CDATA[ > ]]> #{userid}
</select>
```

### 开启自动获取自增主键值

#### 局部配置

```
<!--添加用户获取主键值[自增]-->
<insert id="insertUsersGetKey" useGeneratedKeys="true" keyProperty="userid">
	insert into users values(default ,#{username},#{usersex})
</insert>
```

#### 全局配置

```
<settings>
	<setting name="useGeneratedKeys" value="true"/>
</settings>

<!--添加用户获取主键值[自增]-->
<insert id="insertUsersGetKey" keyProperty="userid">
	insert into users values(default ,#{username},#{usersex})
</insert>
```

### 动态sql

if标签

```
<!-- 根据用户给定的条件进行查询-->
<select id="selectUsersByProperty" resultType="users">
	select * from users where 1=1
	<if test="userid != 0">
		and userid = #{userid}
	</if>
	<if test="username != null and username != ''">
		and username = #{username}
	</if>
	<if test="usersex != null and usersex != ''">
		and usersex = #{usersex}
	</if>
</select>
```

choose、when、otherwise 标签		从多个条件中选择一个使用

```
<!--多选一条件-->
<select id="selectUsersByChoose" resultType="users">
	select * from users where 1=1
	<choose>
		<when test="username != null and username != ''">
			and username = #{username}
		</when>
		<when test="usersex != null and usersex != ''">
			and usersex = #{usersex}
		</when>
		<otherwise>
			and userid = 1	
		</otherwise>
	</choose>
</select>
```

where 标使用 where 标签，就不需要提供 where 1=1 这样的条件了。

如果判断条件不为空则自 动添加 where 关键字，并且会自动去掉第一个条件前面的 and 或 or

```
<!-- 根据用户给定的条件进行查询使用 where 标签实现-->
<select id="selectUsersByPropertyWhere" resultType="users">
	select * from users
	<where>
		<if test="userid != 0">
			and userid = #{userid}
		</if>
		<if test="username != null and username != ''">
			and username = #{username}
		</if>
		<if test="usersex != null and usersex != ''">
			and usersex = #{usersex}
		</if>
	</where>
</select>

```

bind 标bind 标签允许我们在 OGNL 表达式以外创建一个变量，并可以将其绑定到当前的 SQL 语句中。

一般应用于模糊查询，通过 bind 绑定通配符和查询

```
<!--根据用户姓名模糊查询-->
<select id="selectUsersByLikeName" resultType="users">
	<bind name="likeName" value="'%'+name+'%'"/>
	select * from users where username like #{likeName}
</select>
```

set 标set 标签用在 update 语句中。借助 if 标签，可以只对有具体值的字段进行更新。

set 标 签会自动添加 set 关键字，自动去掉最后一个 if 语句的多余的逗号

```
<!-- 选择更新-->
<update id="usersUpdate">
	update users
	<set>
        <if test="username != null and username != ''">
			username = #{username},
		</if>
		<if test="usersex != null and usersex != ''">
			usersex = #{usersex},
		</if>
	</set>
	where userid = #{userid}
</update>

```

foreach 标foreach 标签的功能非常强大，我们可以将任何可迭代对象如 List、Set 、Map 或者数 组对象作为集合参数传递给 foreach 标签进行遍历。

它也允许我们指定开头与结尾的字符串 以及集合项迭代之间的分隔符。

![image-20220116170632469](C:\Users\郝康将\AppData\Roaming\Typora\typora-user-images\image-20220116170632469.png)

 迭代 List、Set

类型：

```
collection=”collection”
```

名称

```
void example(@Param(“suibian”) List list)
void example(@Param(“suibian”) Set set)

collection=”suibian”

<!--查询用户 ID 为 1 或者 2 的用户-->
<select id="selectUsersByIdUseCollection" resultType="users">
	select * from users where userid in
	<foreach collection="collection" item="userid" open="("separator="," close=")">
    	#{userid}
	</foreach>
</select>
```

迭代数组

类型：

```
collection=”array”
```

名称：

```
void example(@Param(“suibian”) int[] arr)
collection=”suibian”

<!--查询用户 ID 为 1 或者 2 的用户使用数组传递参数-->
<select id="selectUsersByIdUseArray" resultType="users">
	select * from users where userid in
	<foreach collection="array" item="userid" open="(" separator=","close=")">
		#{userid}
	</foreach>
</select>
```

迭代 Map：

名称：

```
void example(@Param(“suibian”) Map map)
collection=”suibian”或者 collection=”suibian.entry

<!--根据给定的条件做计数处理-->
<!-- select count(*) from users where username = 'itbz-sxt4' and usersex = 'male'-->
<select id="selectUsersCount" resultType="int">
	select count(*) from users where
	<foreach collection="suibian" separator="and" item="value" index="key">
		${key} = #{value}
	</foreach>
</select>
```

使用 foreach 标签完成批量添加

```
<!--批量添加用户-->
<!--insert into users values(DEFAULT,'itbz-sxt5','male'),(DEFAULT,'itbz-sxt6','male')-->
<insert id="insertUsersBatch" >
	insert into users values
	<foreach collection="collection" item="user" separator=","> 
		(default ,#{user.username},#{user.usersex})
	</foreach>
</insert>
```


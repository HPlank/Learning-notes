Mybatis

### mybatis中的参数绑定

#{ } 解析一个JDBC预编译语句{PreparedStatement}的参数标记占位符？。使用该方式可以避免SQL注入。一般用于参数绑定。

${ } 仅仅是一个纯粹的String替换，在mybatis的动态sql解析阶段进行变量替换（表名或列名）。存在sql注入问题。

MyBatis的#{}之所以能够预防SQL注入是因为底层使用了PreparedStatement类的setString()方法来设置参数，此方法会获取传递进来的参数的每个字符，然后进行循环对比，如果发现有敏感字符（如：单引号、双引号等），则会在前面加上一个'/'代表转义此符号，让其变为一个普通的字符串，不参与SQL语句的生成，达到防止SQL注入的效果。

其次${}本身设计的初衷就是为了参与SQL语句的语法生成，自然而然会导致SQL注入的问题（不会考虑字符过滤问题）。

1.6.2 #{}和${}用法总结 1）#{}在使用时，会根据传递进来的值来选择是否加上双引号，因此我们传递参数的时候一般都是直接传递，不用加双引号，${}则不会，我们需要手动加

2）在传递一个参数时，我们说了#{}中可以写任意的值，${}则必须使用value；即：${value}

3）#{}针对SQL注入进行了字符过滤，${}则只是作为普通传值，并没有考虑到这些问题

4）#{}的应用场景是为给SQL语句的where字句传递条件值，${}的应用场景是为了传递一些需要参与SQL语句语法生成的值。

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

## Mybatis缓存

提升查询的效率和减少数据库的压 力

Mybatis 会将相同查询条件的 SQL 语句的查询结果存储在 内存或者某种缓存介质当中，

当下次遇到相同的查询 SQL 时候不在执行该 SQL，

而是直接从 缓存中获取结果，减少服务器的压力，

尤其是在查询越多、缓存命中率越高的情况下，使用 缓存对性能的提高更明显

Mybatis 会将相同查询条件的 SQL 语句的查询结果存储在 内存或者某种缓存介质当中，当下次遇到相同的查询 SQL 时候不在执行该 SQL，而是直接从 缓存中获取结果，减少服务器的压力，尤其是在查询越多、缓存命中率越高的情况下，使用 缓存对性能的提高更明显

###  一级缓存

一级缓存也叫本地缓存，MyBatis 的一级缓存是在会话（SqlSession）层面进行缓存的。 MyBatis 的一级缓存是默认开启的，不需要任何的配置。默认开启。

![image-20220117081626076](C:\Users\郝康将\AppData\Roaming\Typora\typora-user-images\image-20220117081626076.png)

一级缓存的生命周期：

Mybatis开启一个数据库会话是，会创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象。Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及内部的Executor对象还有PerpetualCache对象也一并释放掉。（随SqlSession对象的产生和消亡）

如果SqlSession调用了close方法，会释放掉以及缓存PerPetualCache对象，一级缓存将不可用。（SqlSession消亡）

如果调用了clearCache（）会清空PerpetualCache对象中的数据，但时该对象仍可用。

SqlSession中执行了任何一个update操作（update（），delete(),insert()），都会清空PerpetualCache对象的数据，但该对象可以继续使用。

### 二级缓存

二级缓存是 SqlSessionFactory 上的缓存，可以是由一个 SqlSessionFactory 创 建的不同的 SqlSession 之间共享缓存数据。

默认并不开启。

SqlSession 在执行 commit()或者 close()的时候将数据放入到二级缓存。

![image-20220117085104671](C:\Users\郝康将\AppData\Roaming\Typora\typora-user-images\image-20220117085104671.png)

![image-20220117085117607](C:\Users\郝康将\AppData\Roaming\Typora\typora-user-images\image-20220117085117607.png)

二级缓存的配置方式：

实现二级缓存的时候，MyBatis 要求缓存的 POJO 必须 是可序列化的， 也就是要求实现 Serializable 接口。

在映射配置文件中配置就可以 开启缓存了。

二级缓存特点：

映射语句文件中的所有 select 语句将会被缓存。 

映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。 

二级缓存是以 namespace 为单位的，不同 namespace 下的操作互不影响 

如果在加入标签的前提下让个别 select 元素不使用缓存，可以使用 useCache 属性，设置为 false。

 缓存会使用默认的 Least Recently Used（LRU，最近最少使用的）算法来收回。 

根据时间表，比如 No Flush Interval,（CNFI 没有刷新间隔），缓存不会以任何时间顺序 来刷新。 （只有DML才会刷新）

缓存会存储列表集合或对象(无论查询方法返回什么)的 1024 个引用 

缓存会被视为是 read/write(可读/可写)的缓存，意味着对象检索不是共享的，而且可以 安全的被调用者修改，不干扰其他调用者或线程所做的潜在修改。

cache 标签的可选属性：

![image-20220117085351117](C:\Users\郝康将\AppData\Roaming\Typora\typora-user-images\image-20220117085351117.png)

测试二级缓存：

在 mybatis-config.xml 文件中的标签配置开启二级缓存。cacheEnabled 的默认 值就是 true，所以这步的设置可省略

```
<settings>
<setting name="cacheEnabled" value="true"/>
</settings>

 在映射配置文件中添加<cache/>
<mapper namespace="com.bjsxt.mapper.UsersMapper">
	<cache/>
</mapper>
```

## Mybatis 的多表关联查询

resultMap 的基础使用场景：

当查询到的结果集的列名与 POJO 的属性名不匹配时， Mybatis 是无法完成影射处理的。

解决方案：

1.定义列别名

2.通过resultMap标签中定义映射关系解决改问题

### 通过resultMap标签解决实体与结果集的映射

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
	PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.bjsxt.mapper.UsersMapper">
	<resultMap id="usersMapper" type="com.bjsxt.pojo.Users">
		<id property="userid" column="id"/>
		<result property="username" column="name"/>
		<result property="usersex" column="sex"/>
	</resultMap>
	<select id="selectUsersAll" resultMap="usersMapper">
		select userid as id ,username as name ,usersex as sex from users
	</select>
</mapper>
```

### 一对一的关联查询

#### < association>标签

< association>标签是处理单一的关联对象(处理单一属性的关联关系)。 

property：指定关联对象的属性 

javaType：关联对象的类型（可以省略） 

select：执行一个新的查询 

column：在新的查询中用哪个列的值作为查询条件

#### < collection>标签

< collection>标签是处理所关联对象是多个的(处理关联属性是集合时的关联关系)。 

property：指定关联对象的属性 

javaType：关联对象的类型（可以省略。默认为 List 类型，如果集合是 Set类型时需要配置并给定Set的全名）

ofType：指定集合里存放的对象类型 

select：执行一个新的查询 

column：在新的查询中用哪个列的值作为查询条件

###  Mybatis 多表查询中的数据加载方式

#### 连接查询

使用内连接或者外连接的方式查询数据。 

优点：在一次查询中完成数据的查询操作。降低查询次数提高查询效率。

 缺点：如果查询返回的结果集较多会消耗内存空间。

#### N+1 次查询 

分解式查询，将查询分解成多个 SQL 语句。 

优点：配和着延迟加载可实现结果集分步获取，节省内存空间。 

缺点：由于需要执行多次查询，相比连接查询效率低。

N+1 查询的加载方式：

立即加载：再一次查询中执行所有的sql

延迟加载：在一次查询中执行部分sql语句，根据操作映射的关联触发其他查询

### PageHelperf分页

```
PageHelper.startPage(int pageNum,int pageSize);

给定分页参数，该方法需要在执行查询之前调用
pageNum：起始的页数，从 1 开始计算。
pageSize：每页显示的条数。
PageInfo 对象
存放分页结果对象
pageInfo.getList() 获取分页查询结果。
pageInfo.getTotal() 获取查询总条数。
pageInfo.getPages() 获取总页数。
pageInfo.getPageNum() 获取当前页。
pageInfo.getSize() 获取每页显示的条数。
```

### Mybatis 与 Servlet整合

Open Session In View：模式

Open Session in view 是将一个数据库会话绑定到当前请求线程中，在请求期间一直保持数据库会话对象处于 Open，使数据库会话对象在i请求的整个期间都可以使用。

直达有DML操作产生响应后关闭当前数据库会话对象

![image-20220117235919766](C:\Users\郝康将\AppData\Roaming\Typora\typora-user-images\image-20220117235919766.png)


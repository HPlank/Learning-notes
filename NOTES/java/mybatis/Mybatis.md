## Mybatis

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

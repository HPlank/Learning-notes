## Mybatis

### mybatis中的参数绑定

#{ } 解析一个JDBC预编译语句{PreparedStatement}的参数标记占位符？。使用该方式可以避免SQL注入。一般用于参数绑定。

${ } 仅仅是一个纯粹的String替换，在mybatis的动态sql解析阶段进行变量替换（表名或列名）。存在sql注入问题。




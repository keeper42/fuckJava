Mybatis的概述
ORM(Object Relational Mapping)持久层框架的佼佼者
真正实现了SQL语句与Java代码的分离
优秀的功能：动态SQL、缓存、插件-pageHelper（物理分页）等

Mybatis入参处理
参数处理
传递单个参数的形式( mybatis会自动进行参数的赋值)
传递多个参数( mybatis会自动封装在Map集合中)
Collection、List、Array作为参数,封装为Map，但是有一定的规则

参数处理详解
单参数Mybatis不做特殊处理，直接取出参数值赋给xml文件如︰#{id}
多参数∶1)JavaBean传递参数2 )Map接口3)注解@param

Mybatis入参处理
集合类型参数处理
当参数为Collection接口，转换为Map ,Map的Key为collection
如果参数类型为List接口的，除了collection的值外，list作为key
如果参数为数组，也会转换为Map ，Map的key为array

参数处理总结
◆使用map传递参数,业务可读性差
◆@param ,受到参数个数( n )的影响,建议n< 5时,为最佳的传参方式
◆以上参数的处理各有利弊，参数> 5,建议用Javabean方法

重要知识点:
◆传统JDBC进行批量数据插入 的回顾
◆Mybatis借助MySQL 数据库对批量插入的支持
◆MyBatis基 于SqlSession的ExecutorType进行批量添加

使用传统JDBC进行数据的插入
◆1、传统的利用for循环进行插入,种方式存在严重效率问题,需要频
繁获取Session ,获取连接
◆2、使用批处理,代码和SQL的耦合, 代码量较大
◆解决：使用MyBatis支持批量插入的配置和语法

理论及代码总结:
◆1 ) MySQL下批量保存的两种方式,建议使用第一种
◆2 )可借助Executor的Batch批量添加,可与Spring框架整合

```
SqlSession sqlSession = getSqlSessionFactory().openSession(ExecutorType.BATCH);
```

Mybatis拦截器

基于Mybatis拦截器的分页
重要知识点
◆Mybatis的四大对象、插件原理及接口
◆Mybatis的插件开发过程
◆使用PageHelper插件实现分页功能（代理模式？）

Mybatis核心对象
◆ParameterHandler:处理SQL 的参数对象
◆ResultSetHandler :处理SQL的返回结果集
◆StatementHandler:数据库的处理对象,用于执行SQL语句
◆Executor:Mybatis的执行器 ,用于执行增删改查操作

基于动态代理。可以通过插件的方式去修改持久层操作的行为。

Mybatis插件原理
◆Mybatis的插件借助于责任链的模式进行对拦截的处理
◆使用动态代理对目标对象进行包装,达到拦截的目的
◆作用于Mybatis的作用域对象之上



分页原理
◆分 页的分类:内存分页和物理分页
◆MySQL中实现物理分页的关键字

分页原理-三个参数
◆当前页
◆>每页的条数
◆总记录数

总结:
◆遵循插件尽量不使用的原则,因为会修改底层设计
◆插件是生 成的层层代理对象的责任链模式,使用反射机制实现
◆插件的编写要考虑全面,特别是多个插件层层代理的时候
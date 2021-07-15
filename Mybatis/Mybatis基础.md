[一、容器Configuration](#hconfiguration)[二、动态SQL模板](#hsql)[1、MappedStatement（映射器）](#h1mappedstatement)[2、解析过程](#h2)[三、SqlSession](#hsqlsession)[1.基本介绍](#h1)[2.分类](#h2-1)[3.Executor](#h3executor)[四、Mapper(殊途同归)](#hmapper)[1.存在的意义](#h1-1)[2.工作原理](#h2-2)[五、缓存](#h)[1.一级缓存](#h1-2)[2.二级缓存](#h2-3)[2.1基本信息](#h21)[2.2如何工作](#h22)[六、插件](#h-1)[七、结果映射](#h-2)[八、总结](#h-3)

> 看过Mybatis后，我觉得Mybatis虽然小，但是五脏俱全，而且设计精湛。

这个黑盒背后是怎样一个设计，下面讲讲我的理解

### 一、容器Configuration 

`Configuration` 像是Mybatis的总管，Mybatis的所有配置信息都存放在这里，此外，它还提供了设置这些配置信息的方法。Configuration可以从配置文件里获取属性值，也可以通过程序直接设置。

用一句话概述Configuration，他`类似Spring中的容器概念`，而且是中央容器级别，存储的Mybatis运行所需要的大部分东西。

### 二、动态SQL模板 

使用mybatis，我们大部分时间都在干嘛？在XML写SQL模板，或者在接口里写SQL模板

```xml
<?xml version="1.0" encoding="UTF-8" ?><!DOCTYPE mapper  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  "http://mybatis.org/dtd/mybatis-3-mapper.dtd"><!-- mapper:根标签，namespace：命名空间，命名空间唯一 --><mapper namespace="UserMapper">   <select id="selectUser" resultType="com.wqd.model.User">      select * from user where id= #{id}   </select></mapper>复制代码
```

或者

```java
@Mapperpublic interface UserMapper {    @Insert("insert into user( name, age) " +            "values(#{user.name}, #{user.age})")    void save(@Param("user") User user);    @Select("select * from user where id=#{id}")    User getById(@Param("id")String id);}复制代码
```

> 这对于Mybatis框架内部意味着什么？

#### 1、MappedStatement（映射器）

- 就像使用Spring，我们写的`Controller类对于Spring 框架来说`是在定义`BeanDefinition`一样。
- 当我们在XML配置，在接口里配置SQL模板，都是在定义Mybatis的域值`MappedStatement`

一个SQL模板对应`MappedStatement`

mybatis 在启动时，就是把你定义的SQL模板，解析为统一的`MappedStatement`对象，放入到容器`Configuration`中。每个`MappedStatement`对象有一个ID属性。这个id同我们平时mysql库里的id差不多意思，都是唯一定位一条SQL模板,这个id 的命名规则：命名空间+方法名

> Spring的BeanDefinition，Mybatis的MappedStatement

#### 2、解析过程

> 同Spring一样，我们可以在xml定义Bean,也可以java类里配置。涉及到两种加载方式。

这里简单提一下两种方法解析的入口：

**1.xml方式的解析**：

提供了`XMLConfigBuilder`组件，解析XML文件，这个过程既是Configuration容器创建的过程，也是`MappedStatement`解析过程。

```java
XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);Configuration config = parser.parse()复制代码
```

**2.与Spring使用时**：
会注册一个`MapperFactoryBean`，在MapperFactoryBean在实例化，执行到`afterPropertiesSet()`时，触发`MappedStatement`的解析

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/5/26/17251955c3e27c1c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)在这里插入图片描述

最终会调用Mybatis提供的一`MapperAnnotationBuilder` 组件，从其名字也可以看出，这个是处理注解形式的`MappedStatement`

`殊途同归`形容这两种方式很形象，感兴趣的可以看看源码

### 三、SqlSession 

#### 1.基本介绍

> 有了SQL模板，传入参数，从数据库获取数据，这就是SqlSession干的工作。

SqlSession代表了我们通过Mybatis与数据库进行的一次会话。使用Mybatis，我们就是使用`SqlSession`与数据库交互的。

我们把SQL模板的id，即`MappedStatement` 的id 与 参数告诉SqlSession，SqlSession会根据模板id找到对应`MappedStatement` ，然后与数据交互，返回交互结果

```java
User user = sqlSession.selectOne("com.wqd.dao.UserMapper.selectUser", 1);复制代码
```

#### 2.分类

- DefaultSqlSession：最基础的sqlsession实现，所有的执行最终都会落在这个`DefaultSqlSession`上，线程不安全
- SqlSessionManager ： 线程安全的Sqlsession，通过`ThreadLocal`实现线程安全。

#### 3.Executor

Sqlsession有点像`门面模式`，SqlSession是一个门面接口，其内部工作是委托`Executor`完成的。

```java
public class DefaultSqlSession implements SqlSession {  private Configuration configuration;  private Executor executor;//就是他 }复制代码
```

我们调用`SqlSession`的方法，都是由`Executor`完成的。

```java
public void select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler) {    try {      MappedStatement ms = configuration.getMappedStatement(statement);      ----交给Executor      executor.query(ms, wrapCollection(parameter), rowBounds, handler);    } catch (Exception e) {      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);    } finally {      ErrorContext.instance().reset();    }  }复制代码
```

### 四、Mapper(殊途同归) 

#### 1.存在的意义

```java
UserMapper userMapper = sqlsession.getMapper(UserMapper.class);User user = userMapper.getById("51");复制代码
```

Mapper的意义在于，让使用者可以像调用方法一样执行SQL。
区别于，需要显示传入SQL模板的id，执行SQL的方式。

```java
User user = sqlSession.selectOne("com.wqd.dao.UserMapper.getById", 1);复制代码
```

#### 2.工作原理

代理！！！代理！！！ 代理！！！Mapper通过代理机制，实现了这个过程。

`1、MapperProxyFactory`: 为我们的Mapper接口创建代理。

- 单独使用Mybatis时，`Mybatis会调用MapperRegistry.addMapper()`方法，为UserDao接口，创建`new MapperProxyFactory(type)`
- 当和Spring一起使用时，`MapperScannerRegistrar组件触发ClassPathMapperScanner组件的doScan方法`将UserDao的BeanDefinition 的BeanClass设置为MapperProxyFactory， 在走SpringBean实例化时，就从MapperProxyFactory里获取UserDao的实例对象（即代理对象）。

```java
public T newInstance(SqlSession sqlSession) {    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);    return newInstance(mapperProxy);}protected T newInstance(MapperProxy<T> mapperProxy) {    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);}复制代码
```

MapperProxyFactory通过JDK动态代理技术，在内存中帮我们创建一个代理类出来。（虽然你看不到，但他确实存在）

`2、MapperProxy`：就是上面创建代理时的增强

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {    try {      if (Object.class.equals(method.getDeclaringClass())) {        return method.invoke(this, args);      } else if (isDefaultMethod(method)) {        return invokeDefaultMethod(proxy, method, args);      }    } catch (Throwable t) {      throw ExceptionUtil.unwrapThrowable(t);    }   --------------------------    final MapperMethod mapperMethod = cachedMapperMethod(method);    return mapperMethod.execute(sqlSession, args);}复制代码
```

针对非默认，非Object方法(也就是我们的业务方法)，会封装成一个`MapperMethod`, 调用的是`MapperMethod.execute`

```
3、MapperMethod`
一个业务方法在执行时，会被封装成`MapperMethod`, MapperMethod 执行时，又会去调用了`Sqlsession
public Object execute(SqlSession sqlSession, Object[] args) {    Object result;    switch (command.getType()) {      case SELECT:        ...        result = sqlSession.selectOne(command.getName(), param);        ...        break;      ....}复制代码
```

绕了一周，终究回到了最基本的调用方式上。

```java
result = sqlSession.selectOne(command.getName(), param);User user = sqlSession.selectOne("com.wqd.dao.UserMapper.getById", 1);复制代码
```

总结下：

- 最基本用法=sqlsession.selectOne(statement.id，参数)
- **Mapper**=User代理类getById`---》`MapperProxy.invoke方法`---》`MapperMethod.execute()`---》`sqlsession.selectOne(statement.id，参数)

> 显然这一绕，方便了开发人员，但是对于系统来说带来的是多余开销。

### 五、缓存 

Mybatis 还加入了缓存的设计。

分为一级缓存和二级缓存

#### 1.一级缓存

先看长什么样子？原来就是HashMap的封装

```java
public class PerpetualCache implements Cache {  private String id;  private Map<Object, Object> cache = new HashMap<Object, Object>();  public PerpetualCache(String id) {    this.id = id;  }}复制代码
```

在什么位置？作为`BaseExecutor`的一个属性存在。

```java
public abstract class BaseExecutor implements Executor { protected BaseExecutor(Configuration configuration, Transaction transaction) {    this.localCache = new PerpetualCache("LocalCache"); }}复制代码
```

`Executor`上面说过，Sqlsession的能力其实是委托`Executor`完成的.Executor作为Sqlsession的一个属性存在。

所以：**MyBatis一级缓存的生命周期和SqlSession一致**。

#### 2.二级缓存

##### 2.1基本信息

二级缓存在设计上相对与一级缓存就比较复杂了。

以xml配置为例，二级缓存需要配置开启，并配置到需要用到的`namespace`中。

```xml
<setting name="cacheEnabled" value="true"/>复制代码
<mapper namespace="mapper.StudentMapper">    <cache/></mapper>复制代码
复制代码
```

同一个`namespace`下的所有`MappedStatement`共用同一个二级缓存。二级缓存的生命周期跟随整个应用的生命周期，同时二级缓存也实现了同`namespace`下`SqlSession`数据的共享。

二级缓存配置开启后，其数据结构默认也是`PerpetualCache`。这个和一级缓存的一样。

但是在构建二级缓存时，mybatis使用了一个典型的设计模式`装饰模式`，对`PerpetualCache`进行了一层层的增强，使得二级缓存成为一个被层层装饰过的`PerpetualCache`，每装饰一层，就有不同的能力，这样一来，二级缓存就比一级缓存丰富多了。

装饰类有：

- LoggingCache：日志功能，装饰类，用于记录缓存的命中率，如果开启了DEBUG模式，则会输出命中率日志
- LruCache：采用了Lru算法的Cache实现，移除最近最少使用的Key/Value
- ScheduledCache: 使其具有定时清除能力
- BlockingCache： 使其具有阻塞能力

```java
层层装饰private Cache setStandardDecorators(Cache cache) {    try {      MetaObject metaCache = SystemMetaObject.forObject(cache);      if (size != null && metaCache.hasSetter("size")) {        metaCache.setValue("size", size);      }      if (clearInterval != null) {        cache = new ScheduledCache(cache);        ((ScheduledCache) cache).setClearInterval(clearInterval);      }      if (readWrite) {        cache = new SerializedCache(cache);      }      cache = new LoggingCache(cache);      cache = new SynchronizedCache(cache);      if (blocking) {        cache = new BlockingCache(cache);      }      return cache;    } catch (Exception e) {      throw new CacheException("Error building standard cache decorators.  Cause: " + e, e);    }  }复制代码
```

##### 2.2如何工作

二级缓存的工作原理，还是用到`装饰模式`，不过这次装饰的`Executor`。使用`CachingExecutor`去装饰执行SQL的`Executor`

```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {    executorType = executorType == null ? defaultExecutorType : executorType;    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;    Executor executor;    if (ExecutorType.BATCH == executorType) {      executor = new BatchExecutor(this, transaction);    } else if (ExecutorType.REUSE == executorType) {      executor = new ReuseExecutor(this, transaction);    } else {      executor = new SimpleExecutor(this, transaction);    }    if (cacheEnabled) {      executor = new CachingExecutor(executor);//装饰    }    executor = (Executor) interceptorChain.pluginAll(executor);    return executor;  }复制代码
```

当执行查询时，先从二级缓存中查询,二级缓存没有时才去走`Executor`的查询

```java
private Executor delegate;private TransactionalCacheManager tcm = new TransactionalCacheManager();public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)      throws SQLException {        Cache cache = ms.getCache();        ....        List<E> list = (List<E>) tcm.getObject(cache, key);        if (list == null) {          list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);          tcm.putObject(cache, key, list); // issue #578 and #116        }        return list;      }    }    return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);  }复制代码
```

其中`TransactionalCacheManager` 属性为二级缓存提供了事务能力。

```java
public void commit(boolean required) throws SQLException {    delegate.commit(required);    tcm.commit();也就是事务提交时才会将数据放入到二级缓存中去}复制代码
```

总结下二级缓存

- 二级缓存是层层装饰
- 二级缓存工作原理是装饰普通执行器
- 装饰执行器使用`TransactionalCacheManager`为二级缓存提供事务能力

### 六、插件 

> 一句话总结mybaits插件：代理，代理，代理，还是代理。

Mybatis的插件原理也是动态代理技术。

```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {    ..      executor = new SimpleExecutor(this, transaction);    ....    if (cacheEnabled) {      executor = new CachingExecutor(executor);    }    插件的入口    executor = (Executor) interceptorChain.pluginAll(executor);    return executor;  }InterceptorChain public Object pluginAll(Object target) {    for (Interceptor interceptor : interceptors) {      target = interceptor.plugin(target);    }    return target;  }复制代码
```

以分页插件为例，
创建完Executor后，会执行插件的`plugn`方法，插件的`plugn`会调用`Plugin.wrap`方法，在此方法中我们看到了我们属性的JDK动态代理技术。创建`Executor`的代理类，以Plugin为增强。

```java
QueryInterceptorpublic Object plugin(Object target) {        return Plugin.wrap(target, this);}public class Plugin implements InvocationHandler {public static Object wrap(Object target, Interceptor interceptor) {    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);    Class<?> type = target.getClass();    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);    if (interfaces.length > 0) {      return Proxy.newProxyInstance(          type.getClassLoader(),          interfaces,          new Plugin(target, interceptor, signatureMap));    }    return target;  }}复制代码
```

最终的执行链：Executor代理类方法--》Plugin.invoke方法--》插件.intercept方法--》Executor类方法

### 七、结果映射 

> 介于结果映射比较复杂，再开一篇来细节吧

### 八、总结 

- mybatis可以说将装饰器模式，动态代理用到了极致。非常值得我们学习。
- 框架留给应用者的应该是框架运行的基本单位，也就是域值的概念，应用者只需要定义原料，然后就是黑盒运行。

例如：

- Spring的`BeanDefinition`
- Mybatis的`MappedStatement`

> Mybatis是一个非常值得阅读的框架，相比于Spring的重，将Mybatis作为第一个源码学习的框架，非常非常的合适。


作者：享学源码链接：https://juejin.im/post/5ecd3493e51d45786973be27来源：掘金著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
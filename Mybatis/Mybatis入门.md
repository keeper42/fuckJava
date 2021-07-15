### 1-1 什么是MyBatis

```
a.MyBatis是对JDBC的封装
b.将SQL语句放在映射文件中(Mapper.XML)
c.自动将输入参数映射到SQL语句的动态参数中
d.自动将SQL语句执行的结果映射成Java对象
```

### 1-2 MyBatis的应用

```
(0)准备工作，创建学生表，添加数据
    
    CREATE TABLE tbl_student(id VARCHAR(32),NAME VARCHAR(32),age INT);
 
    INSERT INTO tbl_student VALUE("A0001","wyf",23);
    INSERT INTO tbl_student VALUE("A0002","lh",24);
    INSERT INTO tbl_student VALUE("A0003","hzt",25);
    INSERT INTO tbl_student VALUE("A0004","zyx",26);
    INSERT INTO tbl_student VALUE("A0005","lif",27);
    
(1)创建项目，搭建包结构
    
(2)导入jar包
    a.导入MyBatis相关的jar包
    b.导入MySQL的驱动包
    c.导入log4j相关的jar包
    
(3)在src的根目录下创建MyBatis主配置文件mybatis-config.xml,搭建配置文件结构
 
(4)创建mapper包结构,创建SQL映射文件XxxMapper.xml
 
(5)在src根目录下引入log4j文件
 
(6)搭建测试环境,测试根据ID查询数据库中所有记录
```

### 1-3 MyBatis的简单入门

```
public static void main(String[] args) {
 
    String resource = "mybatis-config.xml";
    //输出流
    InputStream inputStream = null;
    try {
        //通过加载MyBatis的主配置文件mybatis-config.xml,创建输入流对象
        inputStream = Resources.getResourceAsStream(resource);
    } catch (IOException e) {
        e.printStackTrace();
    }
    /*
        SqlSessionFactoryBuilder：SqlSessionFactory的建造者
            通过调用建造者对象建造方法，为我们创建一个SqlSession对象
     */
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
 
    // 我们未来所有的操作，使用的都是SqlSession对象来完成
    // 列如增删改查，处理事务等等，都是使用session对象来完成
    SqlSession session = sqlSessionFactory.openSession();
 
    /*
        需求:根据ID查单条
            如果取得的是单条记录，我们调用selectOne方法
            参数1：根据命名空间.sqlId的形式找到我们需要使用的sql语句
            参数2：我们要为sql语句中传递的参数
     */
    Student student = session.selectOne("test1.getById", "A0001");
 
    System.out.println("student = " + student);
 
    session.close();
}
```

### 2-1 在Mapper.xml文件中写sql语句需要注意的事项

```
namespace:命名空间          不同的mapper映射文件使用namespace来区分          不同的mapper映射文件使用的namespace的命名不允许出现重复使用命名空间,sqlId的形式来找到我们想要执行的sql语句      sql语句必须要写在相应的标签中     <insert>在标签中写insert开头的sql语句 处理添加操作    <update>在标签中写update开头的sql语句 处理修改操作    <delete>在标签中写delete开头的sql语句 处理删除操作    <select>在标签中写select开头的sql语句 处理查询操作     parameterType:为sql语句传递的参数的类型    resultType：如果返回值是多条记录，那么resultType的返回值类型，应该写为集合的泛型     注意：        在未来开发中            a.所有的标签都必须要写ID属性            b.<select>标签parameterType属性可以省略                        resultType必须写            c.对于<insert><update><delete>这3个标签,通常我们只写ID属性，其他的一概不写
```

### 2-2 MyBatis解决JDBC存在的问题

```
(1)获取连接，得到statement、处理rs、关闭资源非常繁琐。
    解决：使用SqlSession搞定一切。
(2)将sql语句写死在java代码中，如果修改sql语句，需要修改java代码，需要重新编译。程序的维护性不高。
    解决：将sql语句写在Mapper.xml文件中与java代码分离。
(3)向PrepareStatement对占位符的位置设置参数时，非常繁琐。
    解决：MyBatis自动将java队形映射至sql语句中，通过statement中的parameterType定义输入参数的类型。
(4)解析结果集时需要把字段的值设置到相应的实体类属性名中。
    解决：MyBatis自动将sql执行结果映射到java对象，通过statement中的resultType定义输出结果的类型。
```

### 2-3 MyBatis的主配置文件

```
(1)直接引入mybatis-config.xml文件，并且加载数据库驱动的方式为
        <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/db1"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
        </environment>
    
(2)直接引入mybatis-config.xml文件，但是加载数据驱动的方式为引入properties文件的形式
   
    db.propertise文件放到src根目录下，配置文件里的内容大概为
        jdbc.driver=com.mysql.jdbc.Driver
        jdbc.url=jdbc:mysql://localhost:3306/db1
        jdbc.username=root
        jdbc.password=root
        
    mybatis-config.xml配置文件里的内容变为
        <properties resource="db.properties"/>
 
        <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
        </environments>
```
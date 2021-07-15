1. \${}: Properties的变量占位符，#{}: sql中的参数占位符。

2. 创建mapper，只需创建EmployeeMapper接口，并继承BaseMapper接口。

3. @Target(ElementType.TYPE)

   @TableId(value="id", type=IdType.AUTO)  // value: 指定表中主键列的列名

   @TableName(value="xxx")

   @TableField(value="xx_xx", exist="true")

4. Insert: Integer result = employeeMapper.insert(employee);

5. 通用CRUD_MP全局策略配置

   ```xml
   <bean id="sqlSessionFactoryBean" class="com.baomidou.mybatisplus.spring.MybatisSqlSessionFactoryBean">
   			<!-- 数据源 -->
     		<property name="dataSource" ref="dataSource"></property>
   			<property name="configLocation" value="classpath:mybatis-config.xml"></property>
   			<!-- 别名处理 -->	
     		<property name="typeAliasesPackage" value="com.atguigu.mp.beans"></property>
         <!-- 注入全局MP策略配置 -->
     		<property name="globalCofig" ref="globalConfiguration"></property>
   </bean>
   
   <bean id="globalConfiguration" class="com.baomidou.mybatisplus.entity.GlobalConfiguration">
         <!-- 下划线与驼峰命名的映射，默认是true -->
         <property name="dbColumnUnderLine" value="true"></property>
         <!-- 全局的主键策略 -->
   			<property name="idType" value ="0"></property>
         <!-- 全局的表前缀策略配置 -->
         <property name="tablePrefix" value="me_"></property>
   </bean>
   ```

6. insertAllColumn, selectByMap, selectPage, deleteBatchIds

7. MP启动注入SQL原理分析

   （1）问题：xxxMapper继承了BaseMapper<T>，其中提供了通用的CRUD方法，最终还是通过SQL操作数据。

   ​          前置知识：Mybatis源码中比较重要的对象（Configuration, MappedStatement）， Mybatis框架执行流程。。。

   （2）通过现象看本质：

   ​          A. employeeMapper的本质 org.apache.ibatis.binding.MapperProxy

   ​          B. MapperProxy中sqlSession->sqlSessionFactory

   ​          C. sqlSession中有Configuration->mappedStatements, 每一个mappedStatements都表示Mapper接口中的一个方法与Mapper映射文件中的一个SQL。MP在启动就会挨个分析xxxMapper中的方法，并且将对应的SQL语句处理好，保存到configuration中的mappedStatements对象中。

   ​          D. 本质：将每一个方法都构造成 mappedStatement

   ![image-20210318131124617](/Users/mac/Library/Application Support/typora-user-images/image-20210318131124617.png), 

   sqlMethod：枚举对象 ，MP支持的SQL方法

   TableInfo：数据表反射对象，可以获取到数据表的相关信息

   sqlSource：SQL语句处理对象

   MapperBuilderAssistant：用于缓存SQL参数、查询方法结果集处理等，通过MapperBuilderAssistant将每一个mappedStatement添加到Configuration的mappedStatements中

   仅需要继承一个mapper即可实现大部分单表CRUD操作，BaseMapper提供的方法可以实现增删查改、单一、批量、分页等操作。

   

   EntityWrapper Condition 条件构造器


















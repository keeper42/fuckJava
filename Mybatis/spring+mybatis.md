Spring+Mybatis
1. pom. xml
里面都是项目的依赖配置
spring框架配置，Mybatis的配置， 整合配置，
数据源的依赖配置，数据库驱动的配置等等。
2. web. xml文件中，需要配置Spring的核心监听器

```xml
<beans>
    <!-- 声明用了annotation注解bean
         开启组件扫描：它会到基础包下扫描@Service @Repository @Controller @Component这四种注解声明的bean，扫描后会将这些bean交由Spring容器管理
    -->
    <context:component-scan base-package="com.imooc.shop"></context:component-scan>

    <!-- 配置数据源 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close"/>

    <!-- 配置sqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean"
          p:dataSource-ref="dataSource">
        <!-- 配置类型别名：采用包扫描的方式到基础包下扫描所有的类，作为MyBatis2能够转换的类型，多个包之间用;分隔 -->
        <property name="typeAliasesPackage">
            <value>
                com.imooc.shop.bean
            </value>
        </property>
    </bean>

    <!-- 配置数据访问接口 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer"
          p:sqlSessionFactoryBeanName="sqlSessionFactory">
        <!-- 配置数据访问接口：采用包扫描的方式到基础包下扫描所有的类，作为MyBatis2的数据访问接口，
             并创建这些类的代理对象，创建出来后会把这些代理对象交给Spring容器管理，bean的id名默认为接口的类名前面首字母小写,多个包之间用;分隔 -->
        <property name="basePackage">
            <value>
                com.imooc.shop.repository
            </value>
        </property>
    </bean>

    <!-- 配置DataSourceTransactionManager -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
          p:dataSource-ref="dataSource"/>

    <!-- 开启annotation注解事务 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>

</beans>

```

Spring与Mybatis框架的整合
1. IntelliJ IDEA集成MAVEN环境
2. 使用IntelliJ IDEA创建MAVEN的WEB项目
3. 导入项目的配置依赖到pom.xm|文件中
4. 项目分层,创建实体类,创建Mybatis的持久层接口及映射文件
5. 配置Spring框架配置文件

架构串联测试
1.书写一个查询所有商量类型的**Servlet**类
2.在Servlet中注入业务层组件
3.调用业务层组件的方法
4.在业务层组件中注入Mybatis的持久层对象
5.在持久层对象中查询出所有的商品类型数据返回


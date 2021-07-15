

（注：2020.06.17—2020.06.19  三天时间学习DAO设计模式大概花了五六个小时）

一、程序的分层：显示层+控制层+业务层+数据层（DAO）+数据库

一个业务层的操作需要与多个数据层操作共同完成，数据层完成的是一个个原子性操作，所有的操作业务在业务层中完成。

数据层：即数据访问层（Data Access Object，DAO），是专门进行数据库院子操作的。

业务层：即业务对象（Business Object，BO），又称服务层（Service），业务层核心目的是调用多个数据层操作以完成整体项目的业务设计。



二、在实际工作中，针对简单java类开发给出如下要求：

（1）考虑到日后程序有可能出现分布式应用的问题，所以简单java类必须实现java.io.Serializable接口；

（2）简单java类名称必须与表名保持一致；

（3）类中的属性不允许使用基本数据类型，都必须使用基本数据类型的包装类；

（4）类中可以定义多个构造方法，但是必须要保留一个无参构造方法；

（5）[可选]重写equals()、toString()、hashCode()；

（6）类中的属性必须使用private对象，封装后的属性必须提供getter、 setter。



三、用户所提出的所有需求都应该划分到业务层，因为它指的是功能，而开发人员必须要根据业务层去划分数据层。

```
public class DatabaseConnection {
    private static final String DBDRIVER = "oracle. jdbc . driver . OracleDriver" ;
    private static final String DBURL = "jdbc: oracle: thin:@localhost :1521:mldn'
    private static final String DBUSER = 'scott" ;
    private static final String PASSWORD = "tiger" ;
    private Connection conn = null ;
    /**
    * 在构造方法里面为conn对象进行实例化,可以直接取得数据库的连接对象<br>
    * 由于所有的操作都是基于数据库完成的，如果数据库取得不到连接,那么也就意味着所有的操作都可以停止了
    */
    public DatabaseConnection() {
        try {
            Class.forName(DBDRIVER) ;
            this.conn = DriverManager.getConnection(DBURL, DBUSER, PASSWORD);
            } catch (Exception e) { //I虽然此处有异常，但是抛出的意义不大
            e. printStackTrace();
        }
    }
    /**
    *取得一个数据库的连接对象
    */
    @return Connection实例化对象
    public Connection getConnection() {
        return this.conn ;
    }
    /**
    *负责数据库的关闭
    */
    public void close() {
        if (this.conn != nu1l) {
        //表示现在存在有连接对象
        try {
            this.conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
        	}
        }
    }

}
```

整个的操作过程之中，DatabaseConnection只是无条件的提供有数据库连接，而至于说有多少个连接对象，它都不关心。

额外话题:从最早的DAO设计模式来讲实际上还会考虑到一个问题，多数据库间的移植问题。

此时就需要设置-一个专门的表示连接标准的接口，DatabaseConnectionFactory，dbc.properties: db=mysql]



四、开发Value Object

现在的程序严格来讲已经给出了四个层次。不同层次之间一定要进行数据的传递，但是既然要操作的是指定的数据表，所以数据的结构必须要与表的结构一一对应， 那么自然就可以想到简单Java类（po、to、pojo、vo、dto）。

不管有多少张表，只要是实体表，那么一定要写简单java类，而且不要试图一次性将所有表都转换到位。

```
import java. io. Serializable;
import java.util.Date;
@SuppressWarnings("seria1")
public class Emp implements Serializable {
    private Integer empno ;
    private String ename ;
    private String job ;
    private Date hiredate ;
    private Double sal ;
    private Double comm ;
    // setter、 getterl
}
```



五、开发数据层

数据层最终是交给业务层进行调用的，所以业务层必须知道数据层的执行标准，即:业务的操作方法，但是不需要知道它的具体实现。。

业务层--数据层标准--数据层--JDBC--数据库

开发数据层操作标准：不同层之间如果要进行访问，那么必须要提供有接口，以定义操作标注，那么对于数据层也是一样的，因为数据层的最终是要交给业务层执行，所以需要先定义出数据层的接口。

对于数据层的接口给出如下的开发要求:。

* 数据层既然是进行数据操作的，那么就将其保存在dao包下;

* 既然不同的数据表的操作有可能使用不同的数据层开发，那么就针对数据表进行命名；

  ——如emp表，那么数据层的接口就应该定义为IEmpDAO；

* 对于整个数据层的开发严格来讲就只有两类功能:。
  ——数据更新:建议它的操作方法以doXxx0的形式命名，例如: doCreate()、 doUpdate()、 doRemove()
  ——数据查询。。。

```
public interface IEmpDAO {
	/**
    * 实现数据的增加操作
    * @param Vo包含了要增加数据的VO对象
    * @return 数据保存成功返回true,否则返回false
    * @throws Exception SQL 执行异常
    */
 	public boolean doCreate(Emp vo) throws Exception ;
    /**
    * 实现数据的修改操作,本次修改是根据id进行全部字段数据的修改
    * @param Vo包含了要修改数据的信息，-定要提供有ID内容
    * @return数据修改成功返回true,否则返回false
    * @throws Exception SQL执行异常
    */
    public boolean doUpdate(Emp vo) throws Exception ;
    /**
    * 执行数据的批量删除操作，所有要删除的数据以Set集合的形式保存
    * @paramids包含了所有要删除的数据ID,不包含有重复内容
    * @return删除成功返回true ( 删除的数据个数与要删除的数据个数相同)，否则返回false。
    * @throws Exception SQL执行异常
    */
    public boolean doRemoveBatch(Set<Integer> ids) throws Exception ;
    /**
    * 根据雇员编号查询指定的雇员信息
    * @param id要查询的雇 员编号
    * @return如果雇员信息存在，则将数据以V0类对象的形式返回，如果雇员数据不存在，则返回nu11
    */
    @throws Exception
    public Emp findById(Integer id) throws Exception ;
    
    /**
    * 根据雇员编号查询指定的雇员信息
    * @param id要查询的雇员编号
    * @return如果雇员信息存在，则将数据以V0类对象的形式返回，如果雇员数据不存在，则返回nu11
    * @throws Exception SQL执行异常
    public Emp findById(Integer id) throws Exception ;
    /**
    * 查询指定数据表的全部记录,并且以集合的形式返回
    * @retuFn如果表中有数据，则所有的数据会封装为VO对象而后利用List集合返回，<br>
    * 如果没有数据，那么集合的长度为日(size() ==日,不是nu1l)
    * @throws Exception SQL执行异常
    */
    public List<Emp> findA11() throws Exception ;
    /**
    *分页进行数据的模糊查询，查询结果以集合的形式返回
    * @param currentPage 当前所在的页
    * @param lineSize 每页显示数据行数
    * @param column 要进行模糊查询的数据列
    * @param keyWord 模糊查询的关键字
    * @return 如果表中有数据，则所有的数据会封装为VO对象而后利用List集合返回，<br>
    * @throws Exception SQL执行异常
    */
    public List<Emp> findAllSp1it(Integer currentPage, Integer lineSize, String...
	
    /**
    * 进行模糊查询数据量的统计，如果表中没有记录统计的结果就是θ
    * @paramcolumn要进行模糊查询的数据列
    * @param keyWord 模糊查询的关键字
    * @return返回表中的数据量，如果没有数据返回0
    * @throws Exception SQL执行异常
    */
    public Integer getAllCount (String column, String keyWord) throws Exception ;

    
}
```

+ doCreate (Emp vo)
bool ean
+ doUpdate (Emp vo)
bool ean
+ doRemoveBatch (Set<Integer> ids) : boolean
+ findById() : Emp|
+ findA1l() : List<Emp>
+ findAllSplit(...) : List<Emp>
+ getAllCount(. ..) : Integer



五、数据层实现类

数据层需要被业务层调用，数据层需要进行数据库的执行(PreparedStatement)，由于一个业务层需要执行多个数据层的调用，所以数据库的打开和关闭操作应该由业务层控制较为合理。

所有的数据层实现类要求保存在dao.impl子包下。范例: EmpDAOImpl 子类



六、数据层工厂类——DAOFactory

业务层要想进行数据层的调用，那么必须要取得IEmpDAO接口对象，但是不同层之间要想取得接口对象实例，则需要使用工厂设计模式。


```
DAOFactory 工厂设计类
package cn. mldn. factory;
import java. sq1. Connection;
import cn . mldn . dao. IEmpDAO;
import cn. mldn. dao. imp1. EmpDAOImp1;
public class DAOFactory {
    public static IEmpDAO getIEmpDAOInstance (Connection conn) {
    	return new EmpDAOImp1(conn) ;
    }
}
```

使用工厂的特征就是外层不需要知道具体的字子类。




七、开发业务层

业务层是真正留给外部调用的，可能是控制层，或者是直接调用，既然业务层也是有不同层进行调用，所以业务层开发的首要任务就是定义业务层的操作标准。对于业务层方法的定义要写得有意义。

```
import cn. mldn. vo. Emp ;

/**
 * 定义emp表的业务层的执行标准，此类一定要负责数据库的打开与关闭操作
 * 此类可以通过DAOFactory类取得IEmpDAO接口对象
 * @author m1dn
 */
public interface IEmpService {
    /**
    * 实现雇员数据的增加操作，本次操作要调用IEmpDAO接口的如下方法: <br>
    * <li> 需要调用IEmpDAO . findById()方法,判断要增加数据的id是否已经存在;
    * <li>如果现在要增加的数据编号不存在则调用IEmpDAO . doCreate( )方法，返回操作的结果
    * @param vo包含了要增加数据的V0对象
    *
    * @return如果增加数据的ID重复或者保存失败返回false,否则返回true
    *
    * @throws Exception SQL执行异常
    * /
    public boolean insert(Emp vo) throws Exception ;
    /** 
    * 实现雇员数据的修改操作，本次要调用I EmpDAO . doUpdate( )方法，本次修改属于全部内容的修改;
    * @param Vo包含了要修改数据的V0对象
    * @return修改成功返回true,否则返回false
    * @throws Exception SQL执行异常
    */
    public boolean update(Emp vo) throws Exception;
    /**
    *
    * 执行雇员数据的删除操作，可以删除多个雇员信息,调用I EmpDAO . doRemoveBatch( )方法
    *
    * @param ids包含了全部要删除数据的集合，没有重复数据
    * @return删除成功返回true,否则返回false
    * @throws Exception SQL执行异常
    */
    public boolean delete(Set<Integer> ids) throws Exception ;
    /**
    * 根据雇员编号查找雇员的完整信息，调用IEmpDAO. findById()方法
    *
    * @param ids 要查找的雇员编号:
    * @return 如果找到了则雇员信息以VO对象返回，否则返回nu1l
    * @throws Exception SQL执行异常
    */
    public Emp get(int ids) throws Exception ;
	/**
    * 查询全部雇员信息，调用IEmpDAO . findA1l( )方法
    * @return查询结果以List集合的形式返回，如果没有数据则集合的长度为日
    * @throws Exception SQL 执行异常
    */
	public List<Emp> list() throws Exception ;
	
	/**
	* 实现数据的模糊查询与数据统计，要调用I EmpDAO接口的两个方法: <br>
    * <li>调用IEmpDAO. findA1lSplit( )方法，查询出所有的表数据,返回的List<Emp>;
    * <1i>调用IEmpDAO. getAllCount()方法，查询所有的数据量,返回的Integer;
    * @param currentPage 当前所在页.
    * @param lineSize每页显示的记录数
    * @param column 模糊查询的数据列
    * @param keyWord模糊查询关键字
    * @return本方法由于需要返回多种数据类型，所以使用Map集合返回,由于类型不统一，所以所有value的类型设置为Object，返回内容如下：
    * <li>key = allEmps, value = IEmpDAO.findA1lSplit()返回结果，List<Emp>;
	* <li>key = empCount, value = IEmpDAO.getAllCount()返回结果，Integer;
    * @throws Exception SQL执行异常
    */
    public Map<String, object> list(int currentPage, int lineSize, String column, String keyWord) throws Exception;

	
	
}

```



八、业务层实现类

业务层实现类的核心功能：

* 负责控制数据库的打开和关闭，当存在了业务层对象后其目的就是为了操作数据库，即业务层对象实例化之后就必须准备好数据库连接；
* 根据DAOFactory调用getIEmpDAOInstance0方法而后取得IEmpDAO接口对象。业务层实现类保存在dao.impl子包中。
* 

```
package cn. mldn. service . impl;
import java. util. List;
public class EmpServiceImpl implements IEmpService {
    //在这个类的对象内部就提供有一个数据库连接类的实例化对象
    private DatabaseConnection dbc = new DatabaseConnection() ;
    @Override
    public boolean insert(Emp vo) throws Exception {
        try {
        	if DAOFactory.getIEmpDAOInstance(this.dbc.getConnection()).findById(vo. getEmpno()) == null) {
        		return DAOFactory.getIEmpDAOInstance(this.dbc.getConnection()).doCreate(vo);
        	}
			return false;
        } catch (Exception e) {
        	throw e ;
        } finally {
        	this. dbc.close() ;
        }
    }
    @Override
    public boolean update(Emp vo) throws Exception {
        // TODO Auto-generated method stub
        return false;
    }
}

```

不同层之间的访问依靠的就是工厂类和接口进行操作。




九、定义业务层工厂类——ServiceFactory

业务层最终依然需要被其它的层所使用，所以需要为其定义工厂类，该类也同样保存在factory子包下，如果从实际的开发来讲，业务层应该分为两种：

* 前台业务逻辑：可以将其保存在service.front包中，工厂类：ServiceFrontFactory；
* 后台业务逻辑：可以将其保存在service.back包中，工厂类：ServiceBackFactory。

```
package cn.mldn.factory;
import cn.mldn.service.IEmpService;+
import cn.mldn.service.impl.EmpService Imp1;

public class ServiceFactory {
    public static IEmpService getIEmpServiceInstance(){
    	return new EmpServiceImp1();
    }
}

```


在实际的编写之中，子类永远都是不可见的，同时在整个操作里面，控制层完全看不到数据库的任何操作，即没有任何的JDBC代码。



十、代码测试

​		因为最终的业务层是需要由用户去调用的，所以测试分为两种。

1. 调用测试

   按照传统方式产生对象，而后调用里面的方式进行操作。保存在test子包内。调用业务层方法：

```
package cn . mldn. test;
import java . util . Date;
import cn. mldn. factory . ServiceFactory; 
import cn. mldn. vo. Emp;
public class TestEmpInsert {
    public static void main(String[] args) {
    Emp Vo = new Emp() ;
    vo. setEmpno(8889);
    vo. setEname( "陈冠祐" );
    vo. setJob("摄影师");
    vo. setHiredate(new Date());
    vo. setSal(10.0);
    vo. setComm(0.5);
    try {
        System.out.println(ServiceFactory.getIEmpServiceInstance().insert(vo);
    } catch (Exception e) {|
        e.printStackTrace();
    }
}

```

分页查询：

```
package cn. mldn. test;
import java. util . Iterator;
public class TestEmpSplit {
    @Suppres sWarnings ("unchecked" )
    public static void main(String[] args) {
        try {
            Map<String, object> map = ServiceFactory.getIEmpServiceInstance().list(2,5,"ename", "");
            int count = (Integer) map. get( " empCount"); 
            System. out. println("数据量: " + count) ;
            List<Emp> a1l = (List<Emp>) map. get("al1Emps");
            Iterator<Emp> iter = all. iterator();
            while (iter .hasNext()) {
            Emp Vo = iter. next();
            System. out . print1n(vo.getEname() + "," + vo.getJob());
        } catch (Exception e) {
       		e. printStackTrace();
        }
    }
}

```

整个的操作流程客户端的调用非常的容易，不需要涉及到具体的数据存储细节。



2. 利用junit进行测试

对于这种业务的测试，使用junit是最好的选择。
首先要选择测试的类或接口，现在选择好IEmpService接口进行测试。



十一、总结

1、理解分层的操作；
2、理解单表的CRUD的完整操作；
3、如果觉得需要进行提高，那么就把关系都编写熟练了。

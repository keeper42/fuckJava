PO：persistent object 持久对象

1 ．有时也被称为Data对象，对应数据库中的entity，可以简单认为一个PO对应数据库中的一条记录。

2 ．在hibernate持久化框架中与insert/delet操作密切相关。

3 ．PO中不应该包含任何对数据库的操作。

------

POJO ：plain ordinary java object 无规则简单java对象

一个中间对象，可以转化为PO、DTO、VO。

1 ．POJO持久化之后==〉PO

（在运行期，由Hibernate中的cglib动态把POJO转换为PO，PO相对于POJO会增加一些用来管理数据库entity状态的属性和方法。PO对于programmer来说完全透明，由于是运行期生成PO，所以可以支持增量编译，增量调试。）

2 ．POJO传输过程中==〉DTO

3 ．POJO用作表示层==〉VO

PO 和VO都应该属于它。

------

BO：business object 业务对象

业务对象主要作用是把业务逻辑封装为一个对象。这个对象可以包括一个或多个其它的对象。

比如一个简历，有教育经历、工作经历、社会关系等等。我们可以把教育经历对应一个PO，工作经历对应一个PO，社会关系对应一个PO。

建立一个对应简历的BO对象处理简历，每个BO包含这些PO。

这样处理业务逻辑时，我们就可以针对BO去处理。

封装业务逻辑为一个对象（可以包括多个PO，通常需要将BO转化成PO，才能进行数据的持久化，反之，从DB中得到的PO，需要转化成BO才能在业务层使用）。

关于BO主要有三种概念

1 、只包含业务对象的属性；

2 、只包含业务方法；

3 、两者都包含。

在实际使用中，认为哪一种概念正确并不重要，关键是实际应用中适合自己项目的需要。

------

VO：value object 值对象 / view object 表现层对象

1 ．主要对应页面显示（web页面/swt、swing界面）的数据对象。

2 ．可以和表对应，也可以不，这根据业务的需要。

------

DO：Domain Object，领域对象

从现实世界中抽象出来的有形或无形的业务实体 。

------

DTO：Data Transfer Object，数据传输对象

用于跨进程或远程传输时，不应该包含业务逻辑。

------

DAO：Data Access Object，数据访问对象

1 ．用来封装对数据库的访问（CRUD）。

2 ．通过接收Business层的数据，将POJO持久化为PO。

------

PO：Persistent Object，持久对象

1 ．Data对象，对应数据库中的entity，可以简单地认为一个PO对应数据库中的一条记录。

2 ．PO中不应该包含任何对数据库的操作。

------





简易的关系图：

![这里写图片描述](https://img-blog.csdn.net/20180717104224284?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE4NzA1NDc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
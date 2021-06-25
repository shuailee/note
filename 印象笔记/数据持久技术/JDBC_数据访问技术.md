# JDBC 数据访问技术

### JDBC 数据访问技术

操作数据库技术的发展大致可以分为几个阶段：

* 首先是 JDBC 阶段，初学 JDBC 可能会使用原生的 JDBC 的 API
* 再然后可能会使用数据库连接池，比如：c3p0、dbcp，HikariCP,、druid
* 然后就是一些第三方工具类数据库访问技术，比如 dbutils ，spring jdbc等,此阶段还是自己手写 SQL 语句
* 再往后就是数据持久层的实现orm框架:遵从JPA规范的框架（ Hibernate，OpenJPA .toplink.） mybatis
JDBC 是贯穿始终的，即使到了框架部分，也会对 JDBC 进行整合

#### 一 JDBC API操作数据库

JDBC（Java Data Base Connectivity，Java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。

##### （1） JDBC 常用对象

**1 java.sql.Driver（接口）**：数据库驱动接口，允许连接不同的数据库，由java提供接口，由各大数据库供应商实现。DriverManager类会试着加载尽可能多的已注册的的驱动程序，然后它会让每个驱动程序依次试着连接到目标URL。通过Class.forName(“foo.bah.Driver”)方法可以显式加载和注册一个驱动程序。

**注册原理：**

每个数据库厂商在实现java驱动程序的时候会实现java.sql.Driver接口，在这个接口的实现内部有一个静态方法块，在类加载时会首先执行静态代码块。静态代码块会将自己主动注册到DriverManager中。**现在一般会在项目启动时动态加载所有配置的驱动，然后根据需要自动匹配驱动使用**

例如： Class.forName(“com.mysql.jdbc.Driver”);这段代码会将一个mysql驱动类的Class加载到JVM中,在执行com.mysql.jdbc.Driver驱动类加载的过程中会执行 静态代码块，静态代码块内部实现将自己注册到DriverManager中；

**2 java.sql.DriverManager（类）**：管理一组 JDBC 驱动程序的基本服务,作为初始化的一部分，DriverManager 类会尝试加载在 “jdbc.drivers” 系统属性中引用的驱动程序类。这允许用户定制由他们的应用程序使用的 JDBC Driver。应用程序也可继续使用 Class.forName() 显式地加载 JDBC 驱动程序。

`conn = DriverManager.getConnection(DB_URL, USER, PASSWORD);`

在调用getConnection方法时，DriverManager会试着从初始化时加载的那些驱动程序以及使用与当前应用程序相同的类加载器显式加载的那些驱动程序中查找合适的驱动程序。

**3 javax.sql.DataSource（接口）**： 提供了连接到数据源的另一种更优的方法，支持数据库连接池,作为 DriverManager工具的替代项，DataSource对象是获取连接的首选方法

通过DataSource对象访问的驱动程序本身不会向DriverManager注册。通过查找操作获取DataSource对象，然后使用该对象创建Connection对象。通过DataSource对象获取的连接与通过DriverManager设施获取的连接相同

使用JNDI方式注册DataSource，然后在使用时根据名字将DataSource取出。JNDI(Java Naming and Directory Interface，Java命名和目录接口)是一组在Java应用中访问命名和目录服务的API。它将名称和对象联系起来，使得我们可以用名称访问对象。

DataSource接口由驱动程序供应商实现。共有三种类型的实现：

1. 基本实现 - 生成标准的Connection对象
2. 连接池实现 - 生成自动参与连接池的Connection对象，此实现与中间层连接池管理器一起使用。例如DBCP实现，它是Apache Commons提供的一种数据库连接池组件，也是tomcat的数据库连接池组件。
3. 分布式事务实现 - 生成一个Connection对象，该对象可用于分布式事务，大多数情况下总是参与连接池。此实现与中间层事务管理器一起使用，大多数情况下总是与连接池管理器一起使用。
**4 java.sql.Connection（接口）**：与特定数据库建立连接（会话），在连接上下文中执行 SQL 语句并返回结果。

> 注：在配置 Connection 时，JDBC 应用程序应该使用适当的Connection方法，比如setAutoCommit或setTransactionIsolation。默认情况下，Connection对象处于自动提交模式下，这意味着它在执行每个语句后都会自动提交更改。如果禁用了自动提交模式，那么要提交更改就必须显式调用commit方法；否则无法保存数据库更改。

**5 java.sql.Statement（接口）**：提供执行sql语句的API,由Connection对象创建：`Statement stmt=conn.createStatement();`

> **Statement、PreparedStatement和CallableStatement的区别:**
> 
> 继承关系：CallableStatement<PreparedStatement<Statement<Wrapper
> 
> 
> **Statement:**
> 
> 1. Statement接口提供了执行语句和获取结果的基本方法，
> 2. Statement每次执行sql语句，数据库都要执行sql语句的编译 ，最好用于仅执行一次查询并返回结果的情形，效率高于PreparedStatement。
> 3. 用于执行普通的不带参的查询SQL；
> 4. 支持批量更新,批量删除;
> **PreparedStatement:**
> 
> 1. PreparedStatement接口添加了处理 IN 参数的方法；
> 2. PreparedStatement是预编译的,在执行可变参数的一条SQL时，PreparedStatement比Statement的效率高，因为DBMS预编译一条SQL当然会比多次编译一条SQL的效率要高
> 3. 安全性好，有效防止Sql注入等问题。prepareStatement对象防止sql注入的方式是把用户非法输入的单引号用\反斜杠做了转义，从而达到了防止sql注入的目的
> **CallableStatement接口** 继承自PreparedStatement,支持带参数的SQL操作; 添加了处理 OUT 参数的方法。 支持调用存储过程,提供了对输出和输入/输出参数(INOUT)的支持;

**6 java.sql.ResultSet（接口）**：数据库查询的结果集，有一个指向当前行的光标，默认在第一行之前，通过next方法后移光标来迭代访问结果集中的数据，每列只能读取一次，也可以通过索引取值。

结果集默认是不可更新的，但是可以通过特定方法来实现更新结果集并保存至数据库。

```
//可以通过在创建Statement命令时指定额外参数来实现。
Statement stmt=conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE,ResultSet.CONCUR_UPDATABLE);
//执行查询
ResultSet rs = stmt.executeQuery(sql);
//在查询结果返回的基础上，执行插入一条数据到数据库
rs.moveToInsertRow(); //光标移动到新插入行的地方
rs.updateInt("user_id", 15);
rs.updateString("user_name", "牛栏山");
rs.updateString("user_address", "天方夜谭");
rs.insertRow();//添加一行到数据库

```

**一个jdbc的例子：**

```
<!-- 链接mysql数据库必须的包 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.16</version>
</dependency>
```

```
public class JdbcDemo {
/**
 * 后缀参数不加时区会报错
 * */
public static final  String DB_URL= "jdbc:mysql://localhost:3306/shuai?useUnicode=true&characterEncoding=UTF-8&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=UTC";
public static final  String USER= "root";
public static final  String PASSWORD= "1qaz@WSX";
public static final  String DRIVER= "com.mysql.jdbc.Driver";

    public static void queryUser() {
        Connection conn=null;
        Statement stmt=null;
        ResultSet rs=null;
        try {
            // 1 注册驱动            
            // jdbc4之前加载注册驱动类的方式，在执行forname动态类加载进jvm时会执行com.mysql.jdbc.Driver服务实现类的静态代码块 //该com.mysql.jdbc.Driver类的静态代码块会将其注册进DriverManager中，源码中静态代码块是通过调用loadInitialDrivers()方法来完成引入和查找数据库驱动的 //如果是spi 则使用 ServiceLoader.load(Driver.class); 的方式扫描加载
            Class.forName(DRIVER);
            // 2 获取连接
            conn = DriverManager.getConnection(DB_URL, USER, PASSWORD);
            // 3 们通过Connection 创建一个Statement 对象。
            stmt = conn.createStatement();
            // 4 通过Statement 的execute()方法执行SQL。当然Statement 上面定义了非常多的方法。execute()方法返回一个ResultSet 对象，
            String sql="SELECT * FROM tb_user";
             rs= stmt.executeQuery(sql);
            // 5 我们通过ResultSet 获取数据。转换成一个POJO 对象。
            List<User> users=new ArrayList<>();
            while(rs.next()){
                //可以根据标识读取，也可以根据列索引
                int id=rs.getInt("user_id");
                String username=rs.getString("user_name");
                String user_address=rs.getString("user_address");
                User user=new User();
                user.setUser_id(id);
                user.setUser_name(username);
                user.setUser_address(user_address);
                users.add(user);
            }
            System.out.println("result size:"+users.size());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        finally {
            //6  最后，我们要关闭数据库相关的资源，包括ResultSet、Statement、Connection，它们的关闭顺序和打开的顺序正好是相反的
            try {
                rs.close();
                stmt.close();
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

```

##### （2）事务

事务基本概念一组要么同时执行成功，要么同时执行失败的SQL语句，是数据库操作的一个执行单元。需要满足ACID特性

事务开始于：**连接到数据库上，并执行一条DML语句insert、update或delete**

事务结束于：**显示/隐式执行了commit或rollback语句**

1. 执行一条DDL语句，例如create table语句，在这种情况下，会自动执行commit语句。
2. 执行了一条DML语句，该语句却失败了，在这种情况中，会为这个无效的DML语句执行rollback语句。
3. 断开与数据库的连接
**1 事物需要满足ACID特性：**

> * 原子性：所谓原子性是指本次数据处理要么都提交、要么都不提交，即不能先提交一部分，然后处理其他的程序，然后接着提交未完成提交的剩余部分。概念类似于编程语言的原子操作。
> * 一致性：所谓一致性是指数据库数据由一个一致的状态在提交事务后变为另外一个一致的状态。例如，用户确认到货操作：确认前，订单状态为待签收、客户积分为原始积分，此状态为一致的状态；在客户确认到后后，订单状态为已完成、客户积分增加本次消费的积分，这两个状态为一致状态。不能出现，订单状态为待签收，客户积分增加或者订单状态为已完成，客户积分未增加的状态，这两种均为不一致的情况。一致性与原子性息息相关。
> * 隔离性：所谓隔离性是指事物与事务之间的隔离，即在事务提交完成前，其他事务与未完成事务的数据中间状态访问权限，具体可通过设置隔离级别来控制。
> * 持久性：所谓持久性是指本次事务提交完成或者回滚完成均为持久的修改，除非其他事务进行操作否则数据库数据不能发生改变

```
Connection conn = getConnByDatasource();
/**
 * 修改jack 为tom
 * 修改tom4 为peter
 * */
String sql1 = "update tb_user set user_name='tom' where user_name='牛栏山'";
//这是一条错误的语句
String sql2 = "update tb_user settttt username='peter' where user_name='牛栏山'";
try {
    //事物第1步：设置事物是自动提交还是手动提交，默认为true 自动提交，false 为非自动提交
    conn.setAutoCommit(false);
    //设置数据库隔离级别 mysql默认就是REPEATABLE_READ 可重读 它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行
    conn.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
    
    Statement stmt = conn.createStatement();
    stmt.executeLargeUpdate(sql1);
    //这是一条错误的语句
    stmt.executeLargeUpdate(sql2);
    //事物第2步:提交事物
    conn.commit();
} catch (SQLException e) {
    // TODO 自动生成的 catch 块
    e.printStackTrace();

    //事物第3步：抛出异常的时候，执行事物回滚
    try {
        conn.rollback();
    } catch (SQLException e1) {
        // TODO 自动生成的 catch 块
        e1.printStackTrace();
    }

} finally {
    closeConn(conn);
}
```

**2 数据库并发事务可能出现的几种状态:**

* **脏读(Drity Read)：** 一个事务更新数据过程中该数据被另外一个事务读取到，然后前一个事务最后回滚了对数据的更改，导致后一个数据读取到了不准确的数据。
> 一个事务读取另外一个事务尚未提交的数据。如下图，线程thread1在事务中在time1时刻向库表中新增一条数据‘test’并在time3时刻回滚数据；线程thread2在time2时刻读取，若thread2读取到‘test’，则为读脏。
> 
> 
> ![67b9520253b562a2f9cdbb06dd4cb9d0.png](image/67b9520253b562a2f9cdbb06dd4cb9d0.png)

* **不可重复读(Non-repeatable read):** 在一个事务的两次查询之中，相同一条数据查询结果不一样，这可能是两次查询过程中间插入了一个事务更新的原有的数据。
> 如下图，线程thread1在事务中time1时刻将数据库中‘test’更新为‘00’，并在time3时刻提交；thread2在一个事务中分别在time2和time4两个时刻读取这条记录，若两次读取结果不同则为不可重读。（注意：1.不可重读针对已经提交的数据。2.两次或多次读取同一条数据。）
> 
> 
> ![55139d6e853a099f3b3134f8ec2c7887.png](image/55139d6e853a099f3b3134f8ec2c7887.png)

* **幻读(Phantom Read):** 其他事务的数据操作导致某个事务两次读取数据数量不一致，在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就会发现有几列数据是它先前所没有的。
> 线程thread1在事务中time1时刻向数据库中新增‘00’，并在time3时刻提交；thread2在一个事务中分别在time2和time4两个时刻扫描库表，若两次读取结果不同则为幻读。（注意：1.幻读针对已经提交的数据。2.两次或多次读取不同行数据，数量上新增或减少。）
> 
> 
> ![77ec0e5072323eb946366de7afc4e6bc.png](image/77ec0e5072323eb946366de7afc4e6bc.png)

**3 数据库事物隔离级别：**

  针对上面3种事物并发情况，SQL标准定义了4类隔离级别，包括了一些具体规则，用来限定事务内外的哪些改变是可见的，哪些是不可见的。主要描述了多个事物并行执行（并发事物）造成的脏读，重复读，幻读的问题。例如在事务提交前后相同的sql执行结果可能不一样造成的问题。

1. **Read Uncommitted（读取未提交内容）**：在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读（Dirty Read）。例如在改隔离级别下，当前事物A在第一次查询后得到结果，1，2，3 此时另外一个事物B将1更新程10，未提交事物。然后再在事物A中执行查询得到的结果就是10，2，3，尽管B中没提交对数据记录的更新，但是事物A还是读取到了事物B未提交的更新；
2. **Read Committed（读取提交内容）**：这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这种隔离级别 也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的commit，所以同一select可能返回不同结果。
3. **Repeatable Read（可重读，不可读脏）**：这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读 （Phantom Read）。简单的说，幻读指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行。InnoDB和Falcon存储引擎通过多版本并发控制（MVCC，Multiversion Concurrency Control）机制解决了该事物级别下的幻读问题，也就是说在mysql中不存在这个问题了。RR级别，mysql也解决了写的幻读问题.通过间隙锁和行锁。
4. **Serializable（可串行化**） ：这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争
**4 JDBC事务隔离级别**

* TRANSACTION_NONE 无事务
* TRANSACTION_READ_UNCOMMITTED 读未提交，允许读脏，不可重读，幻读。
* TRANSACTION_READ_COMMITTED 读提交，即不能读脏，但是可能发生不可重读和幻读。
* TRANSACTION_REPEATABLE_READ 可重复读，innodb的隔离级别。可能发生幻读。
* TRANSACTION_SERIALIZABLE 直译为串行事务，保证不读脏，可重复读，不可幻读，事务隔离级别最高。

![clipboard.png](image/clipboard.png)
> 低级别的隔离级一般支持更高的并发处理，并拥有更低的系统开销。
> 
> 
> 隔离级别对当前事务有效，例如若当前事务设置为TRANSACTION_READ_UNCOMMITTED，则允许当前事务对其他事务未提交的数据进行读脏，而非其他事务可对当前事务未提交的数据读脏。
> 
> 
> 若未显示设置隔离级别，jdbc将采用数据库默认隔离级别
> 
> 
> 部分数据库不支持TRANSACTION_NONE，例如mysql

**5 事物的传播特性**

Spring它对JDBC的隔离级别作出了补充和扩展，其提供了7种事务传播行为。所谓事务传播行为就是多个事务方法相互调用时，事务如何在这些方法间传播。Spring 支持 7 种事务传播行为（Transaction Propagation Behavior）：

|Propagation_Required     |如果没有，就开启一个事务；如果有，就加入当前事务（方法B看到自己已经运行在 方法A的事务内部，就不再起新的事务，直接加入方法A）                    |
|Propagation_requires_new |如果没有，就开启一个事务；如果有，就将当前事务挂起。（方法A所在的事务就会挂起，方法B会起一个新的事务，等待方法B的事务完成以后，方法A才继续执行）|
|Propagation_nested       |如果没有，就开启一个事务；如果有，就在当前事务中嵌套其他事务                                                                                    |
|Propagation_supports     |如果没有，就以非事务方式执行；如果有，就加入当前事务（方法B看到自己已经运行在 方法A的事务内部，就不再起新的事务，直接加入方法A）                |
|Propagation_not_supported|没有以非事务运行,有就挂起当前事务A，B以非事务运行结束后再运行事务A                                                                              |
|Propagation_NEVER        |没有以非事务运行,有就抛出异常                                                                                                                   |
|Propagation_Mandatory    |没有就抛出异常，有就使用当前事务                                                                                                                |

**6 Java SPI 机制**

   SPI 是 Java 提供的一种服务加载方式，全名为 Service Provider Interface。根据 Java 的 SPI 规范，我们可以定义一个服务接口，具体的实现由对应的实现者去提供，即服务提供者。然后在使用的时候再根据 SPI 的规范去获取对应的服务提供者的服务实现,通过 SPI 服务加载机制进行服务的注册和发现，可以有效的避免在代码中将服务提供者写死。从而可以基于接口编程，实现模块间的解耦;

  简单来说，它就是一种动态替换发现机制。例如：有个接口想在运行时才发现具体的实现类，那么你只需要在程序运行前添加一个实现即可，并把新加的实现描述给JDK即可。此外，在程序的运行过程中，也可以随时对该描述进行修改，完成具体实现的替换。

**spi服务提供者（接口实现者）是如何完成类加载的？**

**主要使用：class.form("xxx")和ServiceLoader.load()**

  jdbc对外暴露drive数据库驱动注册接口，只有接口没有实现，实现由各大不同的数据库供应商来根据自己的情况通过实现jdbc的这个接口来实现自身的驱动，从而支持在java中访问数据库。但是当有多个服务商对接口实现时，我们如何将这些接口实现全部扫描进我的程序，然后根据情况判断使用哪一个实现呢呢？ jdbc4.0以前， 开发人员可以基于Class.forName("xxx服务实现的完全限定名")的方式来显示装载某个驱动实现。但是通过反射扫描全部的接口实现代价太大，jdk提供了一个使用配置文件的方式 ，jdbc4可以通过在METAINF/services/java.sql.Driver文件里指定所有接口实现类的方式来暴露驱动提供者， 应用程序可以通过JDK提供的ServiceLoader.load()来加载配置文件中的描述信息，完成类加载操作。

客户端使用jdbc时不需要去改变代码，直接根据需求引入不同的spi接口服务即可。例如Mysql的是com.mysql.jdbc.Drive,Oracle则是oracle.jdbc.driver.OracleDrive

**SPI机制的约定：**

1. 在META-INF/services/目录中创建以接口完全限定名命名的文件，该文件内容为Api具体实现类的全限定名
2. 使用ServiceLoader类动态加载META-INF中的实现类
3. 如SPI的实现类为Jar则需要放在主程序classPath中
4. Api具体实现类必须有一个不带参数的构造方法

![clipboard-1.png](image/clipboard-1.png)
在日常开发的时候我们一般会对问题就行抽象，创建接口然后进行不同的业务实现，这样子当我们有一个新的业务需要实现时，需要在原有程序中添加实现然后重新打包。而通过spi机制我们可以不用修改已经稳定的程序，而是新建一个新的jar包来完成接口规范的实现；例如一个缓存接口里面有查询，检查，删除缓存的功能。刚开始只支持本地缓存，后来支持redis缓存。通过spi就可以实现每加一个新的缓存工具就新建包单独实现其缓存功能，然后引入新的缓存包的方式。从而实现业务功能模块的解耦。

SPI 在Dubbo中的应用： http://dubbo.apache.org/zh-cn/docs/source_code_guide/dubbo-spi.html

#### 二 数据库连接池

   数据库链接的建立和关闭是极其耗费系统资源的操作。通过DriverManager获取的数据库连接，一个数据库连接对象均对应一个物理数据库连接，每次操作都打开一个物理连接，使用完后立即关闭连接。频繁的打开、关闭连接将造成系统性能的低下。

   为了解决数据库连接的频繁请求、释放，JDBC2.0引入了数据库连接池技术。数据库连接池，在系统启动时，就主动建立足够的数据库连接，并将这些连接组成一个连接池。每次应用程序请求数据库连接时，无须重新打开连接，而是从连接池中取出已有的链接使用，使用完毕后不再关闭数据库连接，而是直接将连接归还给连接池。数据库连接池是Connection对象的工厂。

**所有的池化技术原理上来说都是将每次使用才去申请的资源改为在系统启动时就先初始化一批放到池子里，在使用时直接从池子里取，从而达到节省系统开销的目的。**

跟系统远程连接相关的普遍采用池化技术，例如：数据库连接池，redis连接池，tomcat连接池，线程池等

**连接池的常用配置有：**

1. 连接池初始连接数/核心连接数
2. 连接池的最大连接数
3. 连接池的最小连接池
4. 连接池每次增加的容量
5. 失效时间
**JDBC连接池：**

JDBC的数据库连接池使用javax.sql.DataSource来表示，DataSource只是一个接口，该接口通常由驱动程序供应商提供实现，也有一些开源组织提供实现（DBCP、C3P0）。DataSource通常被称为数据源，它包含连接池和连接池管理两个部分，但习惯上也经常把DataSource称为连接池

数据源和数据库连接不同，数据源无须创建多个，它是产生数据库连接的工厂，因此整个应用只需要一个数据源即可

**DataSource接口共有三种类型的实现**：

1. 基本实现 - 生成标准的Connection对象
2. 连接池实现 - 生成自动参与连接池的Connection对象，此实现与中间层连接池管理器一起使用。例如DBCP实现，它是Apache Commons提供的一种数据库连接池组件，也是tomcat的数据库连接池组件。
3. 分布式事务实现 - 生成一个Connection对象，该对象可用于分布式事务，大多数情况下总是参与连接池。此实现与中间层事务管理器一起使用，大多数情况下总是与连接池管理器一起使用。
##### (1) DBCP数据源

tomcat的连接池正是采用该连接池实现

```
<!-- dpcp链接池需要引入 commons-pool2和commons-dbcp2两个包 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.7.0</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-dbcp2 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.7.0</version>
</dependency>

<!-- 链接mysql数据库必须的包 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.16</version>
</dependency>
```

```
public static Connection getConnByDatasource() {
    try {
        BasicDataSource ds = new BasicDataSource();
        ds.setDriverClassName(DRIVER);
        ds.setUrl(DB_URL);
        ds.setUsername(USER);
        ds.setPassword(PASSWORD);
        //设置连接池的初始连接数
        ds.setInitialSize(5);
        //设置连接池中最少有两个空闲的链接
        ds.setMinIdle(2);
        Connection conn = ds.getConnection();
        System.out.println(conn);
        return conn;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

##### (2) C3P0数据源

相比之下，c3p0 的数据源性能更胜一筹，Hibernate推荐使用该连接池。C3P0连接池不仅可以自动清理不使用的Connection，还可以自动清理Statement和ResultSet

```
//创建连接池实例
ComboPooledDataSource ds = new ComboPooledDataSource();
//设置连接池连接数据库所需的驱动
ds.setDriverClass("com.mysql.jdbc.Driver");
//设置链接数据库的URL
ds.setJdbcUrl("jdbc:mysql://localhost:3306/javaee");
//设置连接数据库的用户名
ds.setUser("root");
//设置连接数据库的密码
ds.setPassword("pass");
//设置连接池的最大连接数
ds.setMaxPoolSize(40);
//设置连接池的最小连接数
ds.setMinPoolSize(2);
//设置连接池的初始连接数
ds.setInitialPoolSize(5);
//设置连接池的缓存Statement的最大数
ds.setMaxStatements(180);
//获取数据库连接：
Connection conn = ds.getConnection();
```

![bbe8e8861f9fd09a3a375b45e023e646.png](image/bbe8e8861f9fd09a3a375b45e023e646.png)

##### （3）HikariCP 连接池

地址：https://github.com/brettwooldridge/HikariCP 

常用的连接池有C3P0,DBCP,它们都比较成熟稳定，但性能不是十分好，所以有了BoneCP这个连接池,它是一个高速、免费、开源的JAVA连接池，它的性能几乎是前者的25倍，十分强悍。但BoneCP这个连接池在2013年停止更新了，就是为了让步于HikariCP这个连接池。 

**HiKariCP是一个快速，简单，可靠，轻量级，高性能的一个jdbc数据库连接池，大小只有130KB。**

**HikariCP所做的一些优化:**

1. 字节码精简：优化代码，直到编译后的字节码最少，这样，CPU缓存可以加载更多的程序代码；
2. 优化代理和拦截器：减少代码，例如HikariCP的Statement proxy只有100行代码，只有BoneCP的十分之一；
3. 自定义数组类型（FastStatementList）代替ArrayList：避免每次get()调用都要进行range check，避免调用remove()时的从头到尾的扫描；
4. 自定义集合类型（ConcurrentBag）：提高并发读写的效率；
5. 其他针对BoneCP缺陷的优化，比如对于耗时超过一个CPU时间片的方法调用的研究（但没说具体怎么优化）。
##### （4）Druid 连接池

项目地址： https://github.com/alibaba/druid

Druid阿里开源的数据库连接池，能够提供强大的监控和扩展功能，Druid 是一个 JDBC 组件库，包含数据库连接池、SQL Parser 等组件, 被大量业务和技术产品使用或集成，经历过最严苛线上业务场景考验，是你值得信赖的技术产品。

**Druid可以做什么**

1. 可以监控数据库访问性能(需要开启开发账号的ddl权限)，Druid内置提供了一个功能强大的StatFilter插件，能够详细统计SQL的执行性能，这对于线上分析数据库访问性能有帮助。
2. 替换DBCP和C3P0。Druid提供了一个高效、功能强大、可扩展性好的数据库连接池。
3. 数据库密码加密。直接把数据库密码写在配置文件中，这是不好的行为，容易导致安全问题。DruidDruiver和DruidDataSource都支持PasswordCallback。
4. SQL执行日志，Druid提供了不同的LogFilter，能够支持Common-Logging、Log4j和JdkLog，你可以按需要选择相应的LogFilter，监控你应用的数据库访问情况。
5. 扩展JDBC，如果你要对JDBC层有编程的需求，可以通过Druid提供的Filter-Chain机制，很方便编写JDBC层的扩展插件。
6. 在使用druid的时候，要将hikaricp的依赖排除掉
#### 三 数据库CRUD操作框架

**一般的数据库操作技术、框架都会自动集成上面的连接池，我们无需手动创建连接池或者创建连接。只需要配置好连接池参数即可。**

##### （1）Apache DbUtils

1. 功能：DbUtils 解决的最核心的问题就是结果集的映射， 可以把ResultSet 封装成JavaBean,结果集到实体的映射是通过反射实现的；
2. QueryRunner 类：DbUtils 提供了一个QueryRunner 类，它对数据库的增删改查的方法进行了封装，那么我们操作数据库就可以直接使用它提供的方法。
3. 数据源：在QueryRunner 的构造函数里面，我们又可以传入一个数据源(数据库连接池的实现，例如dbcp,c3p0,druid,HikariCP等),这样我们就不需要再去写各种创建和释放连接的代码了。queryRunner = new QueryRunner(dataSource);
4. 地址： https://commons.apache.org/proper/commons-dbutils/
##### （2）Spring JDBC

https://docs.spring.io/spring-cloud-gcp/docs/1.0.0.BUILD-SNAPSHOT/reference/html/_spring_jdbc.html

https://www.cnblogs.com/wangyujun/p/10687780.html

1. Spring 也对原生的JDBC 进行了封装，并且给我们提供了一个模板方法JdbcTemplate，来简化我们对数据库的操作。
2. JdbcTemplate类通过模板设计模式帮助我们消除了冗长的代码，只做需要做的事情（即可变部分），并且帮我们做哪些固定部分，如连接的创建及关闭
3. dbcTemplate类对可变部分采用回调接口方式实现，如ConnectionCallback通过回调接口返回给用户一个连接，从而可以使用该连接做任何事情、StatementCallback通过回调接口返回给用户一个Statement，从而可以使用该Statement做任何事情等等，还有其他一些回调接口
4. Spring除了提供JdbcTemplate核心类，还提供了基于JdbcTemplate实现的NamedParameterJdbcTemplate类用于支持命名参数绑定、 SimpleJdbcTemplate类用于支持Java5+的可变参数及自动装箱拆箱等特性
5. 对于结果集的处理，Spring JDBC 也提供了一个RowMapper 接口，可以把结果集转换成Java 对象
6. **数据源：支持多种数据库连接池，例如dbcp,c3p0,druid,HikariCP等**
###### Spring JDBC的连接池配置：

**(1) jdbc连接池数据源配置：**

DriverManagerDataSource没有实现连接池化连接的机制，每次调用getConnection()获取新连接时，只是简单地创建一个新的连接。所以，一般这种方式常用于开发时测试，不用于生产

```
<bean id="dataSource"
    class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver">
    </property>
    <property name="url" value="jdbc:mysql://localhost:3307/lucene?characterEncoding=UTF8" />
    <property name="username" value="root"></property>
    <property name="password" value="guo941102"></property>
</bean>
```

**(2) c3p0连接池数据源配置**

```
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
        <property name="driverClass" value="com.mysql.jdbc.Driver" />
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3307/lucene?characterEncoding=UTF8" />
        <property name="user" value="root" />
        <property name="password" value="guo941102" />
</bean>
```

**3 dbcp连接池数据源配置**

```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
    destroy-method="close"><!--设置为close使Spring容器关闭同时数据源能够正常关闭，以免造成连接泄露  -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://localhost:3307/lucene?characterEncoding=UTF8" />
    <property name="username" value="root" />
    <property name="password" value="guo941102" />
    <property name="defaultReadOnly" value="false" /><!-- 设置为只读状态，配置读写分离时，读库可以设置为true -->
    <!-- 在连接池创建后，会初始化并维护一定数量的数据库安连接，当请求过多时，数据库会动态增加连接数，
    当请求过少时，连接池会减少连接数至一个最小空闲值 -->
    <property name="initialSize" value="5" /><!-- 在启动连接池初始创建的数据库连接，默认为0 -->
    <property name="maxActive" value="15" /><!-- 设置数据库同一时间的最大活跃连接默认为8，负数表示不闲置 -->
    <property name="maxIdle" value="10"/><!-- 在连接池空闲时的最大连接数，超过的会被释放，默认为8，负数表示不闲置 -->
    <property name="minIdle" value="2" /><!-- 空闲时的最小连接数，低于这个数量会创建新连接，默认为0 -->
    <property name="maxWait" value="10000" /><!-- 连接被用完时等待归还的最大等待时间，单位毫秒，超出时间抛异常，默认为无限等待 -->
</bean>
```

1. 为要映射的实体对象定义一个mapper对象，该mapper对象实现RowMapper接口的mapRow方法，我们在mapRow()方法里面完成对结果集的手动映射
2. 在DAO 层调用的时候就可以传入自定义的Mapper 类，最终返回映射的实体类型。
```
public class EmployeeRowMapper implements RowMapper {
@Override
public Object mapRow(ResultSet resultSet, int i) throws SQLException {
    Employee employee = new Employee();
    employee.setEmpId(resultSet.getInt("emp_id"));
    employee.setEmpName(resultSet.getString("emp_name"));
    employee.setEmail(resultSet.getString("emial"));
    return employee;
}
}

public List<Employee> query(String sql){
    //指定特定数据源 阿里的druid
    new JdbcTemplate( new DruidDataSource());
    return jdbcTemplate.query(sql,new EmployeeRowMapper());
}
```

**优点：** 通过这种方式，我们对于结果集的处理只需要写一次代码，然后在每一个需要映射的地方传入这个RowMapper 就可以了，减少了很多的重复代码。

**缺点：** 但是还是有问题：每一个实体类对象，都需要定义一个Mapper，然后要编写每个字段映射的getString()、getInt 这样的代码，还增加了类的数量；

**结果集自动映射：**

spring jdbc 需要手动映射结果集，如果要实现结果集的自动映射（数据库字段自动映射到实体对象上）需要解决两个问题：

1. 名称对应问题（数据库字段名称和实体字段名称不一定一致）
2. 数据类型对应问题（数据库的JDBC 类型和Java 对象的类型要匹配起来）
**办法：** 通过泛型和反射，我们可以创建一个BaseRowMapper<T>，通过反射的方式自动获取所有属性，把表字段全部赋值到属性。上面的方法就可以改成：`return jdbcTemplate.query(sql,new BaseRowMapper(Employee.class));`

这样，我们在使用的时候只要传入我们需要转换的类型就可以了，不用再单独创建一个RowMapper

jdbcTemplate有多种数据类型返回的api：https://www.cnblogs.com/sprinkle/p/6253663.html

##### DbUtils 和Spring JDBC对比：

1. 无论是QueryRunner 还是JdbcTemplate，都可以传入一个数据源进行初始化，也就是资源管理这一部分的事情，可以交给专门的数据源组件去做，不用我们手动创建和关闭；
2. 对操作数据的增删改查的方法进行了封装；
3. 可以帮助我们映射结果集，无论是映射成List、Map 还是实体类。

**共同缺点：**
4. SQL 语句都是写死在代码里面的，依旧存在硬编码的问题；
5. 参数只能按固定位置的顺序传入（数组），它是通过占位符去替换的，不能自动映射；
6. 在方法里面，可以把结果集映射成实体类，但是不能直接把实体类映射成数据库的记录（没有自动生成SQL 的功能）；
7. 查询没有缓存的功能。
8. 无论是QueryRunner 还是JdbcTemplate都没有实现结果集的自动映射，需要手动写数据字段到对象字段的映射。

---

##### (3) JPA - ORM框架

Java Persistence API（Java 持久层 API）：用于对象持久化的 API规范，使得应用程序以统一的方式访问持久层，是一种 ORM 规范。JPA的出现有两个原因：简化现有Java EE和Java SE应用的对象持久化的开发工作和整合ORM技术，实现持久化领域的统一.

 JPA提供的技术：

  (1)ORM映射元数据:  JPA支持XML和JDK 5.0注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持 久化到数据库表中；

  (2)JPA 的API:  用来操作实体对象，执行CRUD操作，框架在后台替我们完成所有的事情，开发者从繁琐的JDBC和SQL代码中解  脱出来。

  (3)查询语言:   通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合

目前比较成熟的 JPA 框架主要包括 Jboss 的 Hibernate EntityManager、Oracle 捐献给 Eclipse 社区的 EclipseLink、Apache 的 OpenJPA 等。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAArgAAAIVCAYAAAA6QowfAAAgAElEQVR4Aey9C3hc1Xnv/Z8ZSaO7JdvyVb7gC9gGgrENGJMQMARMKLdAIUkhJSkthC95Cv1KD0lPStOeND25FM7XNEDrU5IHWoID4daGm7FDEsAEGwgXG4MvAstX2ZYs2ZJmpJn9Pf81WqMtaUYajWc0e2b++3mkvWdf1l7r9+7Z89/vfte7fI7jONAkAiIgAiIgAiIgAiIgAgVCwF8g7VAzREAEREAEREAEREAERMAQkMDVhSACIiACIiACIiACIlBQBCRwC8qcaowIiIAIiIAIiIAIiIAErq4BERABERABERABERCBgiIggVtQ5lRjREAEREAEREAEREAEJHB1DYiACIiACIiACIiACBQUAQncgjKnGiMCIiACIiACIiACIiCBq2tABERABERABERABESgoAhI4BaUOdUYERABERABERABERABCVxdAyIgAiIgAiIgAiIgAgVFQAK3oMypxoiACIiACIiACIiACEjg6hoQAREQAREQAREQAREoKAISuAVlTjVGBERABERABERABERAAlfXgAiIgAiIgAiIgAiIQEERkMAtKHOqMSIgAiIgAiIgAiIgAhK4ugZEQAREQAREQAREQAQKioAEbkGZU40RAREQAREQAREQARGQwNU1IAIiIAIiIAIiIAIiUFAEJHALypxqjAiIgAiIgAiIgAiIgASurgEREAEREAEREAEREIGCIiCBW1DmVGNEQAREQAREQAREQAQkcHUNiIAIiIAIiIAIiIAIFBQBCdyCMqcaIwIiIAIiIAIiIAIiIIGra0AEREAEREAEREAERKCgCEjgFpQ51RgREAEREAEREAEREAEJXF0DIiACIiACIiACIiACBUVAAregzKnGiIAIiIAIiIAIiIAISODqGhABERABERABERABESgoAhK4BWVONUYEREAEREAEREAEREACV9eACIiACIiACIiACIhAQRGQwC0oc6oxIiACIiACIiACIiACEri6BkRABERABERABERABAqKgARuQZlTjREBERABERABERABEZDA1TUgAiIgAiIgAiIgAiJQUARKCqo1aowIiIAIjEAgGo3CcRz4fL4R9szdZtbNy/XLHRmdWQREQARSIyCBmxon7SUCIpDnBHp6etDb2wsKXP55efL7/QgE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7FBml4drl7aJgAiIgAgMJSCBO5SJ1oiACBQwAa97b70qvAv4klDTREAECpBAf56bAmycmiQCIiACgwlYr+jxzhnu8LnPfQ4vvfRS3NN6vGVK3A62lj6LgAiIQHoEBiZ+TK8MHSUCIiACRUfgiSeeAP/q6upw3nnnFV371WAREAER8DIBeXC9bB3VTQREwLMEmF9XkwiIgAiIgDcJSOB60y6qlQiIgAiIgAiIgAiIQJoEJHDTBKfDREAEREAEREAEREAEvElAAtebdlGtREAEREAEREAEREAE0iQggZsmOB0mAiIgAiIgAiIgAiLgTQISuN60i2olAiIgAiIgAiIgAiKQJgEJ3DTB6TAREAEREAEREAEREAFvEpDA9aZdVCsREAEREAEREAEREIE0CUjgpglOh4mACIiACIiACIiACHiTgASuN+2iWomACIiACIiACIiACKRJQAI3TXA6TAREQAREQAREQAREwJsEJHC9aRfVSgREQAREQAREQAREIE0CErhpgtNhIiACIiACIiACIiAC3iQggetNu6hWIiACIiACIiACIiACaRKQwE0TnA4TAREQAREQAREQARHwJgEJXG/aRbUSAREQAREQAREQARFIk4AEbprgdJgIiIAIiIAIiIAIiIA3CUjgetMuqpUIiIAIiIAIiIAIiECaBCRw0wSnw0RABERABERABERABLxJQALXm3ZRrURABERABERABERABNIkIIGbJjgdJgIiIAIiIAIiIAIi4E0CErjetItqJQIiIAIiIAIiIAIikCYBCdw0wekwERABERABERABERABbxKQwPWmXVQrERABERABERABERCBNAlI4KYJToeJgAiIgAiIgAiIgAh4k4AErjftolqJgAiIgAiIgAiIgAikSUACN01wOkwEREAEREAEREAERMCbBCRwvWkX1UoEREAEREAEREAERCBNAhK4aYLTYSIgAiIgAiIgAiIgAt4kIIHrTbuoViIgAiIgAiIgAiIgAmkSkMBNE5wOEwEREAEREAEREAER8CYBCVxv2kW1EgEREAEREAEREAERSJOABG6a4HSYCIiACIiACIiACIiANwlI4HrTLqqVCIiACIiACIiACIhAmgQkcNMEp8NEQAREQAREQAREQAS8SUAC15t2Ua1EQAREQAREQAREQATSJCCBmyY4HSYCIiACIiACIiACIuBNAhK43rSLaiUCIiACIiACIiACIpAmAQncNMHpMBEQAREQAREQAREQAW8SkMD1pl1UKxEQAREQAREQAREQgTQJSOCmCU6HiYAIiIAIiIAIiIAIeJOABK437aJaiYAIiIAIiIAIiIAIpElAAjdNcDpMBERABERABERABETAmwQkcL1pF9VKBERABERABERABEQgTQISuGmC02EiIAIiIAIiIAIiIALeJCCB6027qFYiIAIiIAIiIAIiIAJpEpDATROcDhMBERABERABERABEfAmAQlcb9pFtRIBERABERABERABEUiTgARumuB0mAiIgAiIgAiIgAiIgDcJSOB60y6qlQiIgAiIgAiIgAiIQJoEJHDTBKfDREAEREAEREAEREAEvElAAtebdlGtREAEREAEREAEREAE0iQggZsmOB0mAiIgAiIgAiIgAiLgTQISuN60i2olAiIgAiIgAiIgAiKQJgEJ3DTB6TAREAEREAEREAEREAFvEpDA9aZdVCsREAEREAEREAEREIE0CZSkeZwOEwEREIGiIRAKhdDZ2Yne3t54m8PhsFnu7u5GS0tLfD0XfD4fgsEgKioqUFKi2+wAOPogAiIgAmNAwOc4jjMG59EpREAERCAvCUQiETz00EP46U9/it27d8fbwFvnvn37UF9fj/Ly8vh6Lvj9fpx11ln4+te/jqVLlw7Ypg8iIAIiIALZJyDXQvYZ6wwiIAJ5TICe27fffhsHDx7E/PnzUVdXh0AggGg0aoSs9RHQa8t19PJ+9NFH2LJlC5qamrB48WKzfx4jUNVFQAREIO8ISODmnclUYREQgbEkQK/t3r17jUi97LLLsGTJEhN+QCFrhS7rYwVuR0cH/uM//gMvv/wydu3ahfb2duPlHcs661wiIAIiUOwEJHCL/QpQ+0VABJISoHf23XffNTG2EydOxAknnICTTjrJ7M8whERe3KNHj6KxsdGI4O3bt8fDGJKeRBtEQAREQAQyTkBZFDKOVAWKgAgUCoGenh5QpB45cgQzZszAhAkTjKeW4paTndN7yz8K4srKSkybNg21tbXGg3vgwAGzvlCYqB0iIAIikA8E5MHNByupjiIgAjkhQG8sO5IxUwLjb6dOnWqyIpSWloLilxkS6MXlZEMUGLZw4oknYtKkSaZT2uHDh8GOasqmkBMT6qQiIAJFSkACt0gNr2aLgAiM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)

1. Hibernate 实现了该规范，并对其进行了扩展.
2. Springdata JPA是在hibernate的基础上更上层的封装实现.
3. OpenJPA
##### （4）MyBatis

Hibernate 在业务复杂的项目中使用也存在一些问题：

1、比如使用get()、save() 、update()对象的这种方式，实际操作的是所有字段，没有办法指定部分字段，换句话说就是不够灵活。

2、这种自动生成SQL 的方式，如果我们要去做一些优化的话，是非常困难的，也就是说可能会出现性能比较差的问题。

3、不支持动态SQL（比如分表中的表名变化，以及条件、参数）。连表查询支持也不够友好

4、Hibernate 是全自动化的orm，操作对象就等于操作数据库，可移植性好，mybatis是半自动化的orm可以自定义sql等 比较灵活，可移植性差。一般传统公司使用Hibernate比较多，互联网公司使用mybatis比较多；

#### （3）Spring Boot数据源配置

  由于Spring Boot的自动化配置机制，大部分对于数据源的配置都可以通过配置参数的方式去改变。只有一些特殊情况，比如：更换默认数据源，多数据源共存等情况才需要去修改覆盖初始化的Bean内容。

**springboot 2.x默认使用hikari作为数据库连接池**，不需要添加额外依赖，只需在application.properties中配置即可。在Spring Boot自动化配置中，对于数据源的配置可以分为两类：

**（1）通用配置：**

以`spring.datasource.*`的形式存在，主要是对一些**即使使用不同数据源也都需要配置的一些常规内容**。比如：数据库链接地址、用户名、密码等。这里就不做过多说明了，通常就这些配置：

```
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

**(2) 数据源连接池配置：**

以`spring.datasource.<数据源名称>.*`的形式存在，比如：Hikari的配置参数就是`spring.datasource.hikari.*`形式。下面这个是我们最常用的几个配置项及对应说明：

```
spring.datasource.hikari.minimum-idle=10
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.idle-timeout=500000
spring.datasource.hikari.max-lifetime=540000
spring.datasource.hikari.connection-timeout=60000
spring.datasource.hikari.connection-test-query=SELECT 1
```

**这些配置的含义：**

* spring.datasource.hikari.minimum-idle: 最小空闲连接，默认值10，小于0或大于maximum-pool-size，都会重置为maximum-pool-size
* spring.datasource.hikari.maximum-pool-size: 最大连接数，小于等于0会被重置为默认值10；大于零小于1会被重置为minimum-idle的值
* spring.datasource.hikari.idle-timeout: 空闲连接超时时间，默认值600000（10分钟），大于等于max-lifetime且max-lifetime>0，会被重置为0；不等于0且小于10秒，会被重置为10秒。
* spring.datasource.hikari.max-lifetime: 连接最大存活时间，不等于0且小于30秒，会被重置为默认值30分钟.设置应该比mysql设置的超时时间短
* spring.datasource.hikari.connection-timeout: 连接超时时间：毫秒，小于250毫秒，否则被重置为默认值30秒
* spring.datasource.hikari.connection-test-query: 用于测试连接是否可用的查询语句


# Mybatis

* [一 MyBatis简介]()    * [1 什么是mybatis，有什么优点？]()
    * [2 MyBatis 与 Hibernate 有哪些不同？]()

* [二 SqlSession]()    * [1 sqlsession作用域和生命周期]()
    * [2对象生命周期和依赖注入框架]()
    * [3 普通 java项目中使用Mybatis]()
    * [3 Springboot项目中使用Mybatis]()

* [三 Mybatis详解]()    * [1 SQL映射]()
    * [2. 命名解析]()
    * [3. 映射器（IUserDao接口）【重要】]()
    * [4 动态sql]()    * [(1) 多条件查询判断 if]()
        * [(2) 多条件查询判断连续拼接 if+where]()
        * [(3) choose when标签,当我们值需要多个条件中的一个时使用]()
        * [(4) 多字段更新判断 update if]()
        * [(5) trim 格式化标记]()
        * [(6) include命令 定义sql片段在别的地方引用，可以增加代码的重用性]()
        * [(7) foreach 标记]()
        * [(8) 拼接OR查询条件]()
        * [(9) in查询]()
        * [(10) 批量更新：需要在连接字符串中加入 &allowMultiQueries=true 开启批量语句执行]()
        * [(11) MyBatis模糊查询]()
        * [(12) 批量插入去重]()

* [四 缓存]()    * [(1) 本地缓存]()    * [本地缓存实现原理：]()
    
    * [(2) 二级缓存]()

* [五 Mybatis插件]()
* [六 字段类型处理器（typeHandlers）]()
* [七 其他知识点]()    * [1. 语法]()
    * [2. #{} 和 ${}的区别]()
    * [3. 结果集映射（数据库字段和实体字段映射）]()    * [4 Mybatis日志]()

* [八 多数据源配置]()
* [九 Mybatis-Plus]()

#### 一 MyBatis简介

##### 1 什么是mybatis，有什么优点？

1. MyBatis 是一款优秀的ORM持久层框架，它封装了jdbc，避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。开发时只需要关注 SQL 语句本身，不需要花费精力去处理加载驱动、创建连接、创建statement 等繁杂的过程
2. MyBatis 支持定制化 SQL、存储过程以及高级映射。程序员可以直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高。
3. MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 pojo对象为数据库中的记录。避免了手动写jdbc代码，设置参数和结果集映射。
##### 2 MyBatis 与 Hibernate 有哪些不同？

1. Mybatis 和 hibernate 不同，它不完全是一个 ORM 框架，因为 MyBatis 需要程序员自己编写 Sql 语句，而hibernate是全自动化的，直接操作java对象无需和sql打交道
2. Mybatis 直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，因为这类软件需求变化频繁，一但需求变化要求迅速输出成果。但是mybatis 无法做到数据库无关性，如果需要实现支持多种数据库的软件，则需要自定义多套 sql 映射文件，工作量大
3. Hibernate 对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件，如果用 hibernate 开发可以节省很多代码，提高效率。
4. MyBatis 专注于 SQL 本身，是一个足够灵活的 DAO 层解决方案。MyBatis 适用于对性能的要求很高，或者需求变化较多的项目，如互联网项目
**Mybatis 从执行 sql 到返回 result 的过程:**

通过 xml 文件或注解的方式将要执行的各种 statement 配置起来，并通过java 对象和 statement 中 sql 的动态参数进行映射生成最终执行的 sql 语句，最后由 mybatis 框架执行 sql 并将结果自动映射为 java 对象并返回。

#### 二 SqlSession

  每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的,SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射绑定（bind）的 SQL 语句：`IUserDao userMapper = sqlSession.getMapper(IUserDao.class);`

  SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。既然有了 SqlSessionFactory，顾名思义，我们就可以从中获得 SqlSession 的实例了

**加载配置，创建SqlSessionFactory：**

```
String resource = "org/mybatis/builder/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(inputStream);
```

**SqlSession 可以执行如下命令：**

```
<T> T selectOne(String statement)
<E> List<E> selectList(String statement)
<T> Cursor<T> selectCursor(String statement)
<K,V> Map<K,V> selectMap(String statement, String mapKey)
int insert(String statement)
int update(String statement)
int delete(String statement)
```

##### 1 sqlsession作用域和生命周期

**（1）SqlSessionFactoryBuilder类： SqlSessionFactoryBuilder类用来创建SqlSessionFactory，只需要使用一次；**

这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在，以保证所有的 XML 解析资源可以被释放给更重要的事情。

**（2）SqlSessionFactory类: SqlSessionFactory应该只被创建一次，应用运行期间一直使用，使用单例；**

SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏味道（bad smell）”。因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

**（3）SqlSession类:每个线程都应该有它自己的 SqlSession 实例，使用完毕记得关闭**

  每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。

  绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，要考虑 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。 换句话说，每次收到的 HTTP 请求，就可以打开一个 SqlSession，返回一个响应，就关闭它。 这个关闭操作是很重要的，你应该把这个关闭操作放到 finally 块中以确保每次都能执行关闭。 下面的示例就是一个确保 SqlSession 关闭的标准模式：

```
SqlSession session = sqlSessionFactory.openSession();
try {
  // 你的应用逻辑代码
} finally {
  session.close();
}
```

![.png](image/.png)

##### 2对象生命周期和依赖注入框架

依赖注入框架可以创建线程安全的、基于事务的 SqlSession 和映射器，并将它们直接注入到你的 bean 中，因此可以直接忽略它们的生命周期；例如在spring中使用mybatis，你不需要关心sqlsession的创建，由框架自动完成。

##### 3 普通 java项目中使用Mybatis

（0）引入mybatis

```
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
</dependency>

```

(1) 定义mybatis-config.xml文件配置SqlSessionFactoryBuilder :

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!--
XML 配置文件中包含了对 MyBatis 系统的核心设置，包含获取数据库连接实例的数据源
（DataSource）和决定事务作用域和控制方式的事务管理器（TransactionManager）
-->
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/shuai"/>
                <property name="username" value="root"/>
                <property name="password" value="1qaz@WSX"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="mapper/user-mapper.xml"/>
    </mappers>
</configuration>

<!--
还有很多可以在 XML 文件中进行配置，上面的示例指出的则是最关键的部分。 要注意 XML 头部的声明，它用来验证 XML 文档正确性。
environment 元素体中包含了事务管理和连接池的配置。
mappers 元素则是包含一组映射器（mapper），这些映射器的 XML 映射文件包含了 SQL 代码和映射定义信息-->
```

（2）定义sql映射文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--命名空间应该是对应接口的包名+接口名 -->
<mapper namespace="com.shuailee.dao.IUserDao">
    <!--查询用户信息并分页 -->
    <select id="queryUser" resultType="com.shuailee.model.User">
        select * from tb_user
    </select>
</mapper>
```

（3）映射接口：

映射器接口不需要去实现任何接口或继承自任何类,只要方法可以被唯一标识对应的映射语句就可以了 :`com.shuailee.dao.IUserDao+queryUser`

```
// 外部直接使用该接口即可 
public interface IUserDao { 
List<User> queryUser(); 
}
```

（4）测试代码：

```
public class App 
{
    /**
     *从 XML 中构建 SqlSessionFactory, 执行查询和map
     * */
    public void queryUserTest() {
        /**
         * 每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder
         * 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例
         *
         *  从 XML 文件中构建 SqlSessionFactory 的实例非常简单，建议使用类路径下的资源文件进行配置
         * 但是也可以使用任意的输入流（InputStream）实例，包括字符串形式的文件路径或者 file:// 的 URL 形式的文件路径来配置
         * MyBatis 包含一个名叫 Resources 的工具类，它包含一些实用方法，可使从 classpath 或其他位置加载资源文件更加容易
         */
        String resource = "mybatis-config.xml";
        InputStream inputStream = null;
        try {
            inputStream = Resources.getResourceAsStream(resource);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //从 SqlSessionFactory 中获取 SqlSession
        //既然有了 SqlSessionFactory，顾名思义，我们就可以从中获得 SqlSession 的实例了。
        //SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句
        SqlSessionFactory sqlSessionFactory =   new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession= sqlSessionFactory.openSession();
       try {
            //执行查询和映射
            IUserDao userMapper = sqlSession.getMapper(IUserDao.class); //使用正确描述每个语句的参数和返回值的接口
            List<User> user = userMapper.queryUser();
            System.out.println(user.get(0).toString());
        }finally {
            sqlSession.close();
        }
    }

    public static void main( String[] args )
    {
        App app=new App();
        app.queryUserTest();
    }
}
```

##### 3 Springboot项目中使用Mybatis

**（0）引入mybatis**

```
<!--mybatis-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>

<!--druid-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.9</version>
</dependency>
<!--mysql driven-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.19</version>
</dependency>

<!--mybatis分页插件-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.3</version>
</dependency>
```

**（1）数据源配置：**

https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter

```
# jdbc基本配置，在连接字符串中加入 &allowMultiQueries=true  开启批量语句执行
spring.datasource.url = jdbc:mysql://aaa.com:3306/shuai?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true
spring.datasource.username = test
spring.datasource.password = 123456
spring.datasource.driver-class-name = com.mysql.cj.jdbc.Driver
spring.datasource.type = com.alibaba.druid.pool.DruidDataSource

# druid数据库连接池配置
# 初始连接数
spring.datasource.druid.initial-size=10
# 最大连接数
spring.datasource.druid.max-active=30
# 最小连接数
spring.datasource.druid.min-idle=5
spring.datasource.druid.max-wait=
# 获取连接等待超时时间
spring.datasource.druid.max-wait =60000
# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.datasource.druid.time-between-eviction-runs-millis = 60000
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.druid.min-evictable-idle-time-millis = 300000

# 配置监控统计拦截的filters，去掉后监控界面SQL无法进行统计，’wall’用于防火墙
# spring.datasource.druid.filters=config,stat,wall,log4j 

# mybatis 配置（可以不需要）
# mybatis.mapper-locations=classpath:mapper/*.xml 
# mybatis.type-aliases-package=com.wdbyte.domain
```

**（2）定义sql映射xml文件和映射接口**

```
<!--命名空间应该是对应接口的包名+接口名-->
<mapper namespace="com.shuailee.dao.IUserDao">
    <select id="queryUserByID" parameterType="int" resultType="hashmap" >
        SELECT * FROM tb_user WHERE user_id = #{id}
    </select>
</mapper>
```

```
// 外部直接使用该接口即可
public interface IUserDao {
     List<User> queryUser();
}
```

**（3）如果接口位于不同的模块，需要在项目启动类配置bean扫描路径**

`@MapperScan(basePackages = {"com.shuailee.repository.dao"})`

#### 三 Mybatis详解

##### 1 SQL映射

（1）. xml方式

```
<!--命名空间应该是对应接口的包名+接口名-->
<mapper namespace="com.shuailee.dao.IUserDao">
    <select id="queryUserByID" parameterType="int" resultType="hashmap" >
        SELECT * FROM tb_user WHERE user_id = #{id}
    </select>
</mapper>
```

这个语句被称作 queryUserByID，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的键是列名，值便是结果行中的对应值

> 1. parameterType 入参类型
> 2. resultType 返回数据类型，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身，resultType , resultMap，不能同时使用。
> 3. resultMap 外部 resultMap 的命名引用
> 4. flushCache 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。
> 5. useCache 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。

（2）. 注解方式

```

@Mapper
public interface IUserDao {
     @ResultType(User.class)
     @Select("select * from tb_user")
     List<User> queryUser();
}
```

##### 2. 命名解析

MyBatis 对所有的命名配置元素（包括语句，结果映射，缓存等）使用了如下的命名解析规则：

* **完全限定名（比如 “com.shuailee.dao.IUserDao.queryUser）** 将被直接用于查找及使用。利用更长的完全限定名来将不同的语句隔离开来，同时也实现了接口绑定
* **短名称（比如 “queryUser”）** 如果全局唯一也可以作为一个单独的引用。 如果不唯一，有两个或两个以上的相同名称（比如 “com.foo.selectAllThings” 和 “com.bar.selectAllThings”），那么使用时就会产生“短名称不唯一”的错误，这种情况下就必须使用完全限定名。
##### 3. 映射器（IUserDao接口）【重要】

是一些由你创建的、绑定你映射的语句的接口。映射器接口的实例是从 SqlSession 中获得的，它的生命周期可以同SqlSession 相同属于线程，虽然在整个请求作用域保持映射器实例不会有什么问题，但是这会让作用域管理太多资源，提升了复杂性。为了避免这种复杂性，最好把映射器放在方法作用域内，在方法执行时被创建，用过之后即可丢弃

**【重要】接口映射的原理：**

1. Dao接口即 Mapper映射接口。接口的完全限定名就是映射文件中的namespace的值。接口的方法名就是映射文件中 Mapper 的 Statement 的 id 值；映射接口内的Mapper接口是没有实现类的，当调用接口方法时接口完全限定名+方法名会拼接成一个 key 值，可唯一定位一个 MapperStatement。
2. 在 Mybatis 中，每一个<select>、<insert>、<update>、<delete> 标签，都会被解析为一个MapperStatement 对象。而参数就是传递给 sql 的参数。
3. 例如通过`com.shuailee.dao.IUserDao.queryUserByID`可以找到namespace为`com.shuailee.dao.IUserDao`的下面id为queryUserByID的MapperStatement
4. Mapper 接口的工作原理是 JDK 动态代理，Mybatis 运行时会使用 JDK动态代理为 Mapper 接口生成代理对象 proxy，代理对象会拦截接口方法，转而执行 MapperStatement 所代表的 sql，然后将 sql 执行结果返回。
##### 4 动态sql

动态sql就是支持逻辑语法的sql，主要用来解决不同条件下拼接sql语句的情况：例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号等。MyBatis 采用功能强大的基于 OGNL 的表达式来淘汰其它大部分元素。

`主要语法：if choose (when, otherwise) trim (where, set) foreach`

###### **(1) 多条件查询判断 if**

```
<select id="findActiveBlogWithTitleLike"   resultType="Blog">
  SELECT * FROM BLOG   WHERE state = ‘ACTIVE’
  <!--如果没有传入“title”则不拼接title查询条件：-->
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```

###### **(2) 多条件查询判断连续拼接 if+where**

“where”标签如果它包含的标签中有返回值的话，它就插入一个‘where’,如果标签返回的内容是以AND或OR开头的，则会剔除掉AND 或OR

```
<select id="queryUserinfo2" parameterType="java.util.Map"  resultType="com.shuailee.model.UserModel">
    SELECT * FROM tb_user
    <where>
        <if test="sex != null">
            AND user_sex = #{sex}
        </if>
        <if test="name != null">
            AND user_name like #{name}
        </if>
    </where>
</select>
```

###### **(3) choose when标签,当我们值需要多个条件中的一个时使用**

```
<!--
如果 sex 不为空，那么查询语句为：select * from user where user_sex=?
如果 sex 为空，name 是否为空，如果不为空，那么语句为 select * from user where user_name=?;
如果 sex 为空，name 为空，那么语句为 select * from user where user_email=?;

-->
<select id="queryUserinfo3" parameterType="java.util.Map" resultType="com.shuailee.model.UserModel">
    SELECT * FROM tb_user
    <where>
        <choose>
            <when test="sex != null">
                AND user_sex = #{sex}
            </when>
            <when test="name != null">
                AND user_name like #{name}
            </when>
            <otherwise>
                and user_email=#{email}
            </otherwise>
        </choose>
    </where>
</select>
```

###### **(4) 多字段更新判断 update if**

```
<update id="updateUserById" parameterType="com.shuailee.model.UserModel">
 update tb_user
     <set>
         <if test="userName!=null and userName!=''">
             user_name = #{userName},
         </if>
         <if test="userSex!=null and userSex!=''">
             user_sex = #{userSex}
         </if>
     </set>
        where user_id = #{userId}
</update>
```

###### **(5) trim 格式化标记**

**完成where标记的功能：**

* prefix：前缀追加
* prefixoverride：去掉第一个and或者是or
```
<select id="queryUserinfo4" parameterType="java.util.Map" resultType="com.shuailee.model.UserModel">
    SELECT * FROM tb_user
    <trim prefix="where" prefixOverrides="and | or">
        <if test="sex != null">
            AND user_sex = #{sex}
        </if>
        <if test="name != null">
            AND user_name like #{name}
        </if>
    </trim>
</select>
```

**完成set标记的功能**

* suffix：后缀追加
* suffixoverride：去掉最后一个逗号（也可以是其他的标记，就像是上面前缀中的and一样）
```
<update id="updateUserById2" parameterType="com.shuailee.model.UserModel">
    update tb_user
    <trim prefix="set" suffixOverrides=",">
        <if test="userName!=null and userName!=''">
            user_name = #{userName},
        </if>
        <if test="userSex!=null and userSex!=''">
            user_sex = #{userSex}
        </if>
    </trim>
    where user_id = #{userId}
</update>
```

###### **(6) include命令 定义sql片段在别的地方引用，可以增加代码的重用性**

```
<!--定义一个条件筛选的判断-->
<sql id="selectUserByPara">
    <if test="userName!=null and userName!=''">
        user_name = #{userName},
    </if>
    <if test="userSex!=null and userSex!=''">
        user_sex = #{userSex}
    </if>
</sql>

<!--修改引用-->
<update id="updateUserById3" parameterType="com.shuailee.model.UserModel">
    update tb_user
    <trim prefix="set" suffixOverrides=",">
        <!-- 引用 sql 片段，如果refid 指定的不在本文件中，
        那么需要在前面加上 namespace -->
        <include refid="selectUserByPara"></include>
    </trim>
    where user_id = #{userId}
</update>
```

###### **(7) foreach 标记**

```
<!--
collection:指定输入对象中的集合属性
item:每次遍历生成的对象
open:开始遍历时的拼接字符串
close:结束时拼接的字符串
separator:遍历对象之间需要拼接的字符串
-->
<!--1 多行插入-->
<insert id="batchInsertUser2" parameterType="com.shuailee.model.User"
        useGeneratedKeys="true" keyProperty="user_id">
    insert into tb_user (user_name,user_sex,user_birthday,user_address,user_email) 
    values
    <foreach item="item" collection="list" separator=",">
        (#{item.user_name}, #{item.user_sex}, #{item.user_birthday}, #{item.user_address}, #{item.user_email})
    </foreach>
</insert>

最终得到sql语句：

insert into redeem_code (batch_id, code, type, facevalue,create_user,create_time) values (?,?,?,?,?,? ),(?,?,?,?,?,? ),(?,?,?,?,?,? ),(?,?,?,?,?,? )

```

###### **(8) 拼接OR查询条件**

```
<!--2 我们用 foreach 来改写 select * from user where id=1 or id=2 or id=3
注意：
parameterType参数类型为List  collection="list"
parameterType参数类型为ArrayList  collection="values[]"
-->
<select id="queryUserbyids" resultType="com.shuailee.model.User"
 parameterType="java.util.List">
    SELECT * FROM tb_user
    <where>
        <foreach collection="list" item="id" open=" (" close=")"
         separator="or">
             user_id = #{id}
        </foreach>
    </where>
</select>
```

###### **(9) in查询**

**接口：**`int selectInformationAppid(@Param("appid") String appid,@Param("infoSourceList")List infoSourceList);`

```
<select id="selectInformationAppid" resultType="java.lang.Integer">
    select count(1) from info_source where app_id = #{appid} and information_source
    in
    <foreach item="item"  collection="infoSourceList" separator="," open="("  close=")">
        #{item.informationSource,jdbcType=VARCHAR}
    </foreach>
</select>
```

###### **(10) 批量更新：需要在连接字符串中加入 &allowMultiQueries=true 开启批量语句执行**

```
# Druid 数据源基本配置 spring.datasource.url = jdbc\:mysql\://192.168.5.15\:3306/middleware?useUnicode\=true&characterEncoding\=utf8&serverTimezone\=GMT%2B8&zeroDateTimeBehavior\=convertToNull&allowMultiQueries\=true

// 接口定义
int batchUpdate(List infoSourceList);
```

**sql映射：**

```
<update id="batchUpdate" parameterType="com.web.entity.InfoSource">

    <foreach item="item" collection="list" separator=";">
        update info_source
        <set>
            <if test="item.informationSource != null">
                information_source = #{item.informationSource,jdbcType=VARCHAR},
            </if>
            <if test="item.informationSourceName != null">
                information_source_name = #{item.informationSourceName,jdbcType=VARCHAR},
            </if>
            <if test="item.informationAppid != null">
                information_appid = #{item.informationAppid,jdbcType=VARCHAR},
            </if>
            <if test="item.informationAppkey != null">
                information_appkey = #{item.informationAppkey,jdbcType=VARCHAR},
            </if>
            <if test="item.updateUser != null">
                update_user = #{item.updateUser,jdbcType=VARCHAR},
            </if>
            <if test="item.delete != null">
                `delete` = #{item.delete,jdbcType=BIT},
            </if>
        </set>
        where id = #{item.id,jdbcType=INTEGER}
    </foreach>
</update>
```

###### **(11) MyBatis模糊查询**

**1. 直接传参**

```
public void selectBykeyWord(String keyword) {
     String roleName = "%" + keyword + "%";
     userDao.selectBykeyWord(roleName);
 }
// 接口映射定义
List<RoleEntity> selectBykeyWord(@Param("roleName") String roleName);

```

```
<select id="selectBykeyWord" parameterType="string" resultType="com.why.mybatis.entity.RoleEntity">
	SELECT * FROM t_role
	WHERE
	    role_name LIKE #{roleName}
</select>

```

**2. CONCAT（）函数**

MySQL的 CONCAT（）函数用于将多个字符串连接成一个字符串，是最重要的mysql函数之一

`List<RoleEntity> selectBykeyWord(@Param("keyword") String keyword);`

```
<select id="selectBykeyWord" parameterType="string" resultType="com.why.mybatis.entity.RoleEntity">
        SELECT  * FROM t_role
        WHERE
            role_name LIKE CONCAT('%',#{keyword},'%')
    </select>

```

###### **(12) 批量插入去重**

参考：https://www.cnblogs.com/better-farther-world2099/articles/11737376.html

对大量的数据进行写入数据库操作时，会出现这样的问题：完全不一样，一部分不一样。我们一般希望存在就更新，不存在就新增得效果。mybatis中得实现思路是通过给mysql数据表建立一个唯一索引来判断是否重复，如果重复则更新（或者返回错误），如果不重复则新增。同时再MySQL 中使用 REPLACE 和INSERT … ON DUPLICATE KEY UPDATE语法

1）需要表存在一个唯一索引作为条件，如果不希望有两条一摸一样的就建立复合唯一索引：

`alter table trading_sync_product add unique index sku_code_sync(sku_code,sync_time);`

2）INSERT … ON DUPLICATE KEY UPDATE

只需要在 INSERT 语句后面声明 ON DUPLICATE KEY UPDATE 子句，插入数据时 MySQL 就会根据唯一索引和主键进行判断，如果有唯一索引和主键有重复，则会跟新数据，否则会插入数据

**批量插入：有就更新，没有就新增**

```
<insert id="batchInsert" useGeneratedKeys="true" keyProperty="id" parameterType="com.ghub.middleware.repository.entity.TradingSyncProduct">
  insert into trading_sync_product (sync_time, sku_code,
    trade_type, stock_num_init, stock_num_new,
    remark, create_by, create_time, update_time, update_by,sync_status)
  values
  <foreach item="item" collection="list" separator=",">
    (#{item.syncTime,jdbcType=DATE}, #{item.skuCode,jdbcType=VARCHAR},
    #{item.tradeType,jdbcType=INTEGER}, #{item.stockNumInit,jdbcType=INTEGER}, #{item.stockNumNew,jdbcType=INTEGER},
    #{item.remark,jdbcType=VARCHAR}, #{item.createBy,jdbcType=VARCHAR}, #{item.createTime,jdbcType=TIMESTAMP},
    #{item.updateTime,jdbcType=TIMESTAMP}, #{item.updateBy,jdbcType=VARCHAR}, #{item.syncStatus,jdbcType=INTEGER})
  </foreach>
  ON DUPLICATE KEY UPDATE
  sync_time=VALUES(sync_time),
  sku_code=VALUES(sku_code),
  trade_type=VALUES(trade_type),
  stock_num_init=VALUES(stock_num_init),
  stock_num_new=VALUES(stock_num_new),
  remark=VALUES(remark),
  update_time=VALUES(update_time),
  update_by=VALUES(update_by),
  sync_status=VALUES(sync_status)
</insert>
```

**使用限制：**

  更新的内容中unique key或者primary key最好保证一个，不然不能保证语句执行正确(有任意一个unique key重复就会走更新,当然如果更新的语句中在表中也有重复校验的字段，那么也不会更新成功而导致报错,只有当该条语句没有任何一个unique key重复才会插入新记录)；

* 尽量不对存在多个唯一键的table使用该语句，避免可能导致数据错乱。
* 在有可能有并发事务执行的insert 语句情况下不使用该语句，可能导致产生death lock。
* 如果数据表id是自动递增的不建议使用该语句；id不连续，如果前面更新的比较多，新增的下一条会相应跳跃的更大。
* 该语句是mysql独有的语法，如果可能会设计到其他数据库语言跨库要谨慎使用。
* 如果不使用VALUES：（trade_type=trade_type）,则还是原来的数据（有就不更新，没有就新增），使用VALUES（trade_type=VALUES(trade_type)）则是（有就更新，没有就新增）
**产生death lock原理：**

insert ... on duplicate key 在执行时，innodb引擎会先判断插入的行是否产生重复key错误，如果存在，在对该现有的行加上S（共享锁）锁，如果返回该行数据给mysql,然后mysql执行完duplicate后的update操作，然后对该记录加上X（排他锁），最后进行update写入。

#### 四 缓存

Mybatis 使用到了两种缓存：本地缓存（local cache）和二级缓存（second level cache）。

##### (1) 本地缓存

* 每当一个新 session 被创建，MyBatis 就会创建一个与之相关联的本地缓存。任何在 session 执行过的查询语句本身都会被保存在本地缓存中，那么，相同的查询语句和相同的参数所产生的更改就不会二度影响数据库了。本地缓存会被增删改、提交事务、关闭事务以及关闭 session 所清空。
* 本地缓存随着session的生命周期结束而结束，也可以调用void clearCache()或session.close()都会清空缓存；
* 可以设置 localCacheScope来设置本地缓存的使用方式，STATEMENT 表示缓存仅在语句执行时有效，如果 localCacheScope 被设置为 SESSION，那么 MyBatis 所返回的引用将传递给保存在本地缓存里的相同对象。对返回的对象（例如 list）做出任何更新将会影响本地缓存的内容，进而影响存活在 session 生命周期中的缓存所返回的值。因此，不要对 MyBatis 所返回的对象作出更改，以防后患。（对返回对象的更改会影响缓存里的值）
**mybatis-config.xml 关于缓存的配置：**

```
<settings>
    <!--全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存。
    全局关闭二级缓存 -->
    <setting name="cacheEnabled" value="false"/>
    <!--设置本地缓存的作用域,默认SESSION，SESSION会缓存一个会话中的所有查询，STATEMENT，
    本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据-->
    <setting name="localCacheScope" value="SESSION"/>
</settings>
```

###### **本地缓存实现原理：**

源码中：PerpetualCache是本地缓存的实现类，实现了Cache接口；默认的本地缓存使用非线程安全集合类HashMap来实现，由于它同sqlsession相同的生命周期所以不会有并发问题；

mybatis的Executor模块是执行SQL的执行器，它有3个实现类如图：

![-1.png](image/-1.png)

1. SimpleExecutor是默认使用的执行器；
2. ReuseExecutor则是将Statement与SQL建立关联关系缓存起来，这样就不用每次都要重复创建新的Statement；
3. BatchExecutor则是批量执行器，通过封装底层JDBC的BATCH相关的API，来加速批量相关的操作。
**而我们关心的本地缓存PerpetualCache则以全局变量存在于BaseExecutor中**

![-2.png](image/-2.png)

**（1）Executor执行器的创建流程：**

![-3.png](image/-3.png)

当通过sqlsessionfactory调用opensession方法创建sqlsession的时候，总是会创建Executor执行器，sqlsession随着请求的线程被创建，随着请求结束而消亡，作用域在方法或线程的上下文。在并发场景下会创建多个sqlsession,Executor执行器也最是会被创建；本地缓存在BaseExecutor中，每次也都会被创建新的实例；所以不会存在并发问题；

**（2）查询流程：**

![-4.png](image/-4.png)

最终会调用BaseExecutor中的query方法，方法中会读取配置文件中的缓存配置来决定是否读取或清空缓存：

```
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    // 如果不是嵌套查询或者嵌套查询执行完毕，并且flushCache配置为true
    if (queryStack == 0 && ms.isFlushCacheRequired()) {
      // 清空一级缓存
      clearLocalCache();
    }
    List<E> list;
    try {
      // 本次查询执行查询次数+1
      queryStack++;
      //获取缓存
      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
      if (list != null) {
        // 处理储存过程，针对OUT的参数设定缓存值
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
        // 没有命中缓存则到数据库中查询
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
      }
    } finally {
      // 本次查询执行查询次数-1
      queryStack--;
    }
    // 查询执行完毕
    if (queryStack == 0) {
      for (DeferredLoad deferredLoad : deferredLoads) {
        deferredLoad.load();
      }
      deferredLoads.clear();
      // 如果本地缓存Scope被配置为STATEMENT，则每次执行的SQL的时候都要清空缓存。
      // 因为有queryStack == 0的判断，所以查询如果是嵌套查询，缓存还是起作用的。
      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
        clearLocalCache();
      }
    }
    return list;
  }
```

执行sql查询，并写入缓存的代码：

```
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  List<E> list;
  // 缓存预占位
  localCache.putObject(key, EXECUTION_PLACEHOLDER);
  try {
    // 执行DB查询操作
    list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
  } finally {
    // 释放预占位
    localCache.removeObject(key);
  }
  // 放入真正的缓存
  localCache.putObject(key, list);
  // 如果是存储过程的调用则将储存过程的执行的结果保存至localOutputParameterCache中
  if (ms.getStatementType() == StatementType.CALLABLE) {
    localOutputParameterCache.putObject(key, parameter);
  }
  return list;
}
```

**（3）更新流程：**

![-5.png](image/-5.png)

增删改不同的操作最终到执行器时仅对应到一个update的方法，只要在一个SqlSession声明周期内有执行增删改的任何一种操作，那么与SqlSession关联的所有的一级缓存内容将被清空

```
@Override
public int update(MappedStatement ms, Object parameter) throws SQLException {
  ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
  if (closed) {
    throw new ExecutorException("Executor was closed.");
  }
  // 清空一级缓存、清空存储过程缓存
  clearLocalCache();
  // 执行DB操作
  return doUpdate(ms, parameter);
}
```

![-6.png](image/-6.png)

##### (2) 二级缓存

**mapper 映射xml中的配置：**

```
<mapper namespace="com.shuailee.dao.IUserDao">
    <!--开启二级缓存-->
    <cache />
    <!--mapper语句
    1  SQL语句中的flushCache="true"，可以理解为每次执行SQL之前都会主动清空缓存，
    然后再从数据库中查询结果，在查询语句中默认值为false，其他情况下为true
    2 SQL语句中的useCache="true" 默认为true 执行语句时使用缓存
    -->
    <select id="queryUser" resultType="com.shuailee.model.User" flushCache="true">
    SELECT * FROM tb_user
    </select>
</mapper>
```

![-7.png](image/-7.png)

  二级缓存在Mybatis是默认关闭，我们通过配置首先启用它。启用它的配置很简单只要在Mapper文件中配置中加入<cache/>即可。

二级缓存属于某个mapper的，一级缓存属于某个sqlsession, 开启二级缓存会：

1. Mapper中所有的查询结果将会被缓存。
2. insert、update、delete语句会刷新缓存。
3. 默认使用LRU算法来刷新缓存。
4. 缓存不会定时进行刷新。
5. 缓存对象的上限默认设置为1024（不管返回的结果是对象和还是列表）
6. 缓存结果是线程安全的。
**全局配置**

* cacheEnabled，可选项为true或者false，默认为true。当设置为false时，SQL执行器由默认的CachingExecutor转变为SimpleExecutor，而二级缓存又依赖于CachingExecutor执行器。废话了这么其实就是设置为false时二级缓存就没了。
* localCacheScope，可选项为SESSION 或者STATEMENT，默认为SESSION。如果配置为STATEMENT本地会话仅用在语句执行上（嵌套查询可以使用到缓存），对相同 SqlSession 的不同调用将不会共享数据。
**Mapper中的配置**

* 可以通过cache定制二级缓存的行为，具体的选项可以参照上面详细描述。
* 可以通过<cache-ref namespace=""/>来引用其他Mapper的二级缓存
* SQL语句中的flushCache="true"，可以理解为每次执行SQL之前都会主动清空缓存，然后再从数据库中查询结果，在查询语句中默认值为false，其他情况下为true。
* SQL语句中的useCache="true"，每次执行这条SQL的时候都会主动的使用缓存，在查询语句中默认值为true,其他情况下为false。
**缓存使用流程**

![-8.png](image/-8.png)

#### 五 Mybatis插件

  MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

1. `Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)`，**Executor 扮演者执行器的角色，是所有的SQL执行的总入口；**
2. `ParameterHandler (getParameterObject, setParameters)`，**ParameterHandler负责对SQL语句的输入参数进行处理；**
3. `StatementHandler (prepare, parameterize, batch, update, query)`，**StatementHandler 负责SQL语句执行调用、Statement的初始化、调用ParameterHandler将输入参数绑定到Statement上；**
4. `ResultSetHandler (handleResultSets, handleOutputParameters)`，**ResultSetHandler 对执行的结果集和实体进行映射处理**。
  可以看出Mybatis在SQL执行前、输入参数处理、SQL执行、结果集处理等这些重要的步骤中都预留了丰富的拦截点。即便我们平时用到的分页插件PageHelper也是基于这些基本的拦截点开发的。需要注意的是如果在试图修改或重写已有方法的行为的时候，你很可能在破坏 MyBatis 的核心模块。 这些都是更低层的类和方法，所以使用插件的时候要特别当心。

**自定义插件：**

通过 MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可。

```
/**
 * @program: study-gupao
 * @description: 开发一个自己的监控的插件，用于监控每个SQL执行的真正时长
 * 使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可
 * @author: shuai.li
 * @create: 2019-06-18 21:09
 **/

//插件将会拦截在 Executor 实例中所有的 “update”,"query" 方法调用， 这里的 Executor 是负责执行低层映射语句的内部对象
@Intercepts({
@Signature(type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
@Signature(type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
@Signature(type = Executor.class,
        method = "update",
        args = {MappedStatement.class, Object.class})
})
@Component
public class MonitorSQLExecutionTimePlugin implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        //获取invocation中执行的sql
        Object[] args = invocation.getArgs();
        MappedStatement mappedStatement = (MappedStatement) args[0];
        Object param = args[1];
        BoundSql boundSql = mappedStatement.getBoundSql(param);
        String sql = boundSql.getSql();

        //计算时间并打印
        long beginTime = System.currentTimeMillis();
        Object result = invocation.proceed();
        long endTime = System.currentTimeMillis();
        System.out.println(MessageFormat.format("sql : {0}, 自定义插件elapsed time {1} ms", sql, endTime - beginTime));
        return result;
    }

    @Override
    public Object plugin(Object o) {
        return Plugin.wrap(o,this);
    }

    @Override
    public void setProperties(Properties properties) {

    }
}
```

上面的插件将会拦截在 Executor 实例中所有的 “update”，“select” 方法调用， 这里的 Executor 是负责执行低层映射语句的内部对象。在springboot项目中 自定义插件与mybatis集成只需要在插件类上打上@Component 即可

https://github.com/EvanDylan/article/blob/master/mybatis/plugin.md

#### 六 字段类型处理器（typeHandlers）

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型

![-9.png](image/-9.png)

你可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型, 具体做法为：实现 org.apache.ibatis.type.TypeHandler 接口， 或继承一个很便利的类 org.apache.ibatis.type.BaseTypeHandler， 然后可以选择性地将它映射到一个 JDBC 类型。

```
/**
 * @program: study-gupao
 * @description: 你可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型
 * 具体做法为：实现 org.apache.ibatis.type.TypeHandler 接口， 或继承一个很便利的类 org.apache.ibatis.type.BaseTypeHandler，
 * 然后可以选择性地将它映射到一个 JDBC 类型。
 * @author: shuai.li
 * @create: 2019-06-17 21:44
 **/
@MappedJdbcTypes(JdbcType.VARCHAR)
public class ExampleTypeHandler extends BaseTypeHandler<String> {
    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, String s, JdbcType jdbcType) throws SQLException {
        preparedStatement.setString(i, s);
    }

    @Override
    public String getNullableResult(ResultSet resultSet, String s) throws SQLException {
        return resultSet.getString(s);
    }

    @Override
    public String getNullableResult(ResultSet resultSet, int i) throws SQLException {
        return resultSet.getString(i);
    }

    @Override
    public String getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        return callableStatement.getString(i);
    }
}
```

使用上述的类型处理器将会覆盖已经存在的处理 Java 的 String 类型属性和 VARCHAR 参数及结果的类型处理器。 要注意 MyBatis 不会通过窥探数据库元信息来决定使用哪种类型，所以你必须在参数和结果映射中指明那是 VARCHAR 类型的字段， 以使其能够绑定到正确的类型处理器上。这是因为 MyBatis 直到语句被执行时才清楚数据类型。

#### 七 其他知识点

##### 1. <![CDATA[" ... "]]> 语法

XML 元素中，"<" 和 "&" 是非法的，"<" 会产生错误，因为xml解析器会把该字符解释为新元素的开始。"&" 也会产生错误，因为解析器会把该字符解释为字符实体的开始。而脚本代码中经常含有这些特殊字符，为了避免错误，可以将脚本代码定义在 CDATA标签中，CDATA 中的所有内容都会被解析器忽略。CDATA 由 "<![CDATA[" 开始，由 "]]>" 结束

例如一个批量更新，将时间作为比较条件的例子：

```
<update id="batchUpdate" parameterType="com.ghub.middleware.repository.entity.TradingSyncProduct">
  <foreach item="item" collection="list" separator=";">
    update trading_sync_product
    <set>
      <if test="item.syncStatus != null">
        sync_status = #{item.syncStatus,jdbcType=INTEGER},
      </if>
    </set>
    WHERE sku_code = #{item.skuCode,jdbcType=VARCHAR} AND
    <![CDATA[ sync_time <= #{item.syncTime,jdbcType=DATE} ]]>
  </foreach>
</update>
```

##### 2. #{} 和 ${}的区别

* Mybatis 在处理#{}时，会将 sql 中的#{}替换为?号，然后调用 PreparedStatement 的set 方法来赋值（会把？号替换为参数值的同时会加上转义符，从而避免了sql注入）
* ${}仅仅是字符串替换，无法避免sql注入
##### 3. 结果集映射（数据库字段和实体字段映射）

**1. sql语句中 as 定义别名

2. 通过 <resultMap> 来映射字段名和实体类属性名的一一对应的关系**

```
<mapper namespace="com.shuailee.dao.IUserDao">
    <select id="getOrder" parameterType="int" resultMap="orderresultmap">
        select * from orders where order_id=#{id}
    </select>
    <resultMap type=”me.gacl.domain.order” id=”orderresultmap”>
        <!–用 id 属性来映射主键字段–>
        <id property=”id” column=”order_id”>
        <!–用 result 属性来映射非主键字段，property 为实体类属性名，column
        为数据表中的属性–>
        <result property = “orderno” column =”order_no”/>
        <result property=”price” column=”order_price” />
    </reslutMap>
</mapper>
```

**3. 自动映射**

* 当自动映射查询结果时，MyBatis 会获取结果中返回的列名并在 Java 类中查找相同名字的属性（忽略大小写）。 这意味着如果发现了 ID 列和 id 属性，MyBatis 会将列 ID 的值赋给 id 属性。
* 通常数据库列使用大写字母组成的单词命名，单词间用下划线分隔；而 Java 属性一般遵循驼峰命名法约定。为了在这两种命名方式之间启用自动映射，需要将 mapUnderscoreToCamelCase 设置为 true
* 甚至在提供了结果映射后，自动映射也能工作。在这种情况下，对于每一个结果映射，在 ResultSet 出现的列，如果没有设置手动映射，将被自动映射。在自动映射处理完毕后，再处理手动映射。
###### 4 Mybatis日志

Mybatis 的内置日志工厂提供日志功能，内置日志工厂将日志交给以下其中一种工具作代理：`SLF4J ,Apache Commons Logging ， Log4j2 ，Log4j， JDK logging`它会使用第一个查找得到的工具（按上文列举的顺序查找）。如果一个都未找到，日志功能就会被禁用；如果你指定了日志但是没有找到，则会忽略继续按照顺序查找；

#### 八 多数据源配置

多数据源就是配置多个数据库连接，不同账号和密码，每个数据库使用不同的数据源。

还要手动配置Datasouce和Sqlsessionfactory。其他就和单数据库使用一样了

**Mybatis结合Druid数据连接池实现多数据源配置**

**1 配置文件配置多数据源**

```
# 基本配置
############## 数据库 1#############
spring.datasource.db1.url = jdbc:mysql://127.0.0.1:4000/db1?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&autoReconnectForPools=true&noAccessToProcedureBodies=true
spring.datasource.db1.username = test
spring.datasource.db1.password = 123
spring.datasource.db1.driver-class-name = com.mysql.jdbc.Driver
spring.datasource.db1.initialSize = 5
spring.datasource.db1.minIdle = 5
spring.datasource.db1.maxActive = 100
# 配置获取连接等待超时的时间
spring.datasource.db1.maxWait = 60000
# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.datasource.db1.timeBetweenEvictionRunsMillis = 60000
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.db1.minEvictableIdleTimeMillis = 300000
spring.datasource.db1.maxEvictableIdleTimeMillis = 1200000
spring.datasource.db1.validationQuery = SELECT 1 FROM DUAL
spring.datasource.db1.testWhileIdle = true
spring.datasource.db1.testOnBorrow = true
spring.datasource.db1.testOnReturn = false
spring.datasource.db1.keepAlive = true
# 打开PSCache，并且指定每个连接上PSCache的大小
spring.datasource.db1.poolPreparedStatements = true
spring.datasource.db1.maxPoolPreparedStatementPerConnectionSize = 20
# 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
spring.datasource.db1.filters = stat,wall
# 通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.db1.connectionProperties = druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000

spring.datasource.db1.mapperLocation = classpath*:db1/mapper/*.xml
spring.datasource.db1.typeAliasesPackage = com.shuailee.repository.model.db1

################数据库 2#########
#数据库 2 配置只需要修改数据库连接和账号密码即可
spring.datasource.db2.url = jdbc:mysql://127.0.0.1:4000/db2?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&autoReconnectForPools=true&noAccessToProcedureBodies=true
spring.datasource.db2.username = test2
spring.datasource.db2.password = 1234
...
spring.datasource.db2.mapperLocation = classpath*:db2/mapper/*.xml
spring.datasource.db2.typeAliasesPackage = com.shuailee.repository.model.db2
```

**2 为每个数据库配置独立的数据源bean**

**db1数据库**

```
@Data
@Configuration
@ConfigurationProperties(prefix = "spring.datasource.db1")
@MapperScan(basePackages = "com.shuailee.repository.mapper", sqlSessionFactoryRef = "db1SqlSessionFactory")
@Slf4j
public class DB1DataSource {

    private String url;
    private String username;
    private String password;
    private String driverClassName;
    private int initialSize;
    private int minIdle;
    private int maxActive;
    private long maxWait;
    private long timeBetweenEvictionRunsMillis;
    private long minEvictableIdleTimeMillis;
    private long maxEvictableIdleTimeMillis;
    private String validationQuery;
    private boolean testWhileIdle;
    private boolean testOnBorrow;
    private boolean testOnReturn;
    private boolean poolPreparedStatements;
    private int maxPoolPreparedStatementPerConnectionSize;
    private String connectionProperties;
    private String filters;
    private boolean keepAlive;

    private String mapperLocation;
    private String typeAliasesPackage;

    /**
     *  DB
     *
     * @return
     */
    @Bean(name = "db1Ds")
    public DataSource gsuperDataSourceConfig() throws Exception {
        log.info("db1Ds db build");
        DruidDataSource druid = new DruidDataSource();
        // 监控统计拦截的filters
        druid.setFilters(filters);

        // 配置基本属性
        druid.setDriverClassName(driverClassName);
        druid.setUsername(username);
        druid.setPassword(password);
        druid.setUrl(url);

        //初始化时建立物理连接的个数
        druid.setInitialSize(initialSize);
        //最大连接池数量
        druid.setMaxActive(maxActive);
        //最小连接池数量
        druid.setMinIdle(minIdle);
        //获取连接时最大等待时间，单位毫秒。
        druid.setMaxWait(maxWait);
        //间隔多久进行一次检测，检测需要关闭的空闲连接
        druid.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
        //一个连接在池中最小生存的时间
        druid.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
        druid.setMaxEvictableIdleTimeMillis(maxEvictableIdleTimeMillis);
        //用来检测连接是否有效的sql
        druid.setValidationQuery(validationQuery);
        //建议配置为true，不影响性能，并且保证安全性。
        druid.setTestWhileIdle(testWhileIdle);
        //申请连接时执行validationQuery检测连接是否有效
        druid.setTestOnBorrow(testOnBorrow);
        druid.setTestOnReturn(testOnReturn);
        druid.setKeepAlive(keepAlive);
        //是否缓存preparedStatement，也就是PSCache，oracle设为true，mysql设为false。分库分表较多推荐设置为false
        druid.setPoolPreparedStatements(poolPreparedStatements);
        // 打开PSCache时，指定每个连接上PSCache的大小
        druid.setMaxPoolPreparedStatementPerConnectionSize(maxPoolPreparedStatementPerConnectionSize);
        druid.setConnectionProperties(connectionProperties);
        return druid;
    }

    /**
     * @return
     * @throws Exception
     */
    @Bean(name = "db1SqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(gsuperDataSourceConfig());
        val resolver = new PathMatchingResourcePatternResolver();
        factoryBean.setMapperLocations(resolver.getResources(mapperLocation));
        factoryBean.setTypeAliasesPackage(typeAliasesPackage);
        PageInterceptor pageInterceptor = new PageInterceptor();
        Properties properties = new Properties();
        properties.setProperty("helperDialect", "mysql");
        properties.setProperty("reasonable", "true");
        properties.setProperty("supportMethodsArguments", "true");
        properties.setProperty("autoRuntimeDialect", "true");
        pageInterceptor.setProperties(properties);
        PageInterceptor[] pageInterceptors = {pageInterceptor};
        factoryBean.setPlugins(pageInterceptors);
        return factoryBean.getObject();
    }

    /**
     * @return
     * @throws Exception
     */
    @Bean(name = "db1SessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate() throws Exception {
        SqlSessionTemplate template = new SqlSessionTemplate(sqlSessionFactory());
        return template;

    }

    /**
     * @return
     * @throws Exception
     */
    @Bean(name = "db1Transaction")
    public DataSourceTransactionManager transactionManager() throws Exception {
        return new DataSourceTransactionManager(gsuperDataSourceConfig());
    }

}
```

**db2数据库**DB2数据库配置只需要修改对应bean名称和路径即可。

#### 九 Mybatis-Plus

**文档:** https://mp.baomidou.com/guide/

**mybatis-plus和mybatis的不同：**

* mybatis-plus是对mybatis的增强，未对mybatis做修改，所以mybatis的所有配置和使用方式都保持不变。使用时只需要引入mybatis-plus的maven依赖即可。
* 在不需要自定义sql的情况下mybatis-plus不需要配置xml，只需要定义mapper接口继承BaseMapper 即可实现大部分CRUD操作
* 关于分页，mybatis分页需要使用PageHelper 插件完成分页，而mybatis-plus内置了分页拦截器PaginationInnerInterceptor实现，需要配置在MybatisPlusInterceptor 类
* 条件构造器：通过EntityWrapper<T>（实体包装类），可以用于拼接SQL 语句，并且支持排序、分组查询等复杂的SQL。

pom:
```
<!--druid连接池-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.9</version>
</dependency>
<!--mybatis-plus-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.2</version>
</dependency>
<!--mysql-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>${mysql-connecto.version}</version>
</dependency>
```

**分页拦截器**

```
/**
 * @package: com.klein.config
 * @description: 分页拦截器配置  3.4之前使用PaginationInterceptor  3.4之后使用MybatisPlusInterceptor
 * @author: klein
 * @date: 2021-06-10 10:36
 **/
@Configuration
public class MyBatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor paginationInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
        paginationInnerInterceptor.setOverflow(false);
        paginationInnerInterceptor.setMaxLimit(500L);
        interceptor.addInnerInterceptor(paginationInnerInterceptor);
        return interceptor;
    }
}
```

**接口映射器**

```
/**
 * @package: com.klein.dao.mapper
 * @description: mybatisplus mapper接口定义
 * @author: klein
 * @date: 2021-06-09 18:13
 **/
public interface UserMapper extends BaseMapper<User> {

}
```

**实体对象**

```
@Data
@TableName(value = "User")//指定表名
public class User {
    /**
     * value与数据库主键列名一致，若实体类属性名与表主键列名一致可省略value
     * 指定自增策略
     * */
    @TableId(value = "id",type = IdType.AUTO)
    private Long id;
    /**
     * 若没有开启驼峰命名，或者表中列名不符合驼峰规则，可通过该注解指定数据库表中的列名，exist标明数据表中有没有对应列
     * */
    @TableField(value = "name",exist = true)
    private String name;
    private Integer age;
    private String email;
}
```

**分页查询**

```
 @Test
    public void testSelectPage() {

        System.out.println(("----- selectList method process ------"));
        List<User> userList = userMapper.selectList(null);
        userList.forEach(System.out::println);

        System.out.println(("----- selectPage method test ------"));

        // 分页需要配置 MybatisPlusInterceptor 分页拦截器
        // 使用QueryWrapper条件构造器进行分页，物理分页
        Page<User> userPage = userMapper.selectPage(new Page<User>(1, 2),
                new QueryWrapper<User>()
                        .between("age", 18, 50)
                        .in("id", Arrays.asList(1, 2, 3, 4, 5))
        );

        List<User> userPageList = userPage.getRecords();
        userPageList.forEach(System.out::println);
    }
```

**git源码：[lab-01-springboot-hub]()**

%5Btoc%5D%0A%0A%23%23%23%23%20%E4%B8%80%20MyBatis%E7%AE%80%E4%BB%8B%0A%23%23%23%23%23%201%20%E4%BB%80%E4%B9%88%E6%98%AFmybatis%EF%BC%8C%E6%9C%89%E4%BB%80%E4%B9%88%E4%BC%98%E7%82%B9%EF%BC%9F%0A%0A1.%20MyBatis%20%E6%98%AF%E4%B8%80%E6%AC%BE%E4%BC%98%E7%A7%80%E7%9A%84ORM%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6%EF%BC%8C%E5%AE%83%E5%B0%81%E8%A3%85%E4%BA%86jdbc%EF%BC%8C%E9%81%BF%E5%85%8D%E4%BA%86%E5%87%A0%E4%B9%8E%E6%89%80%E6%9C%89%E7%9A%84%20JDBC%20%E4%BB%A3%E7%A0%81%E5%92%8C%E6%89%8B%E5%8A%A8%E8%AE%BE%E7%BD%AE%E5%8F%82%E6%95%B0%E4%BB%A5%E5%8F%8A%E8%8E%B7%E5%8F%96%E7%BB%93%E6%9E%9C%E9%9B%86%E3%80%82%E5%BC%80%E5%8F%91%E6%97%B6%E5%8F%AA%E9%9C%80%E8%A6%81%E5%85%B3%E6%B3%A8%20SQL%20%E8%AF%AD%E5%8F%A5%E6%9C%AC%E8%BA%AB%EF%BC%8C%E4%B8%8D%E9%9C%80%E8%A6%81%E8%8A%B1%E8%B4%B9%E7%B2%BE%E5%8A%9B%E5%8E%BB%E5%A4%84%E7%90%86%E5%8A%A0%E8%BD%BD%E9%A9%B1%E5%8A%A8%E3%80%81%E5%88%9B%E5%BB%BA%E8%BF%9E%E6%8E%A5%E3%80%81%E5%88%9B%E5%BB%BAstatement%20%E7%AD%89%E7%B9%81%E6%9D%82%E7%9A%84%E8%BF%87%E7%A8%8B%0A2.%20MyBatis%20%E6%94%AF%E6%8C%81%E5%AE%9A%E5%88%B6%E5%8C%96%20SQL%E3%80%81%E5%AD%98%E5%82%A8%E8%BF%87%E7%A8%8B%E4%BB%A5%E5%8F%8A%E9%AB%98%E7%BA%A7%E6%98%A0%E5%B0%84%E3%80%82%E7%A8%8B%E5%BA%8F%E5%91%98%E5%8F%AF%E4%BB%A5%E7%9B%B4%E6%8E%A5%E7%BC%96%E5%86%99%E5%8E%9F%E7%94%9F%E6%80%81%20sql%EF%BC%8C%E5%8F%AF%E4%BB%A5%E4%B8%A5%E6%A0%BC%E6%8E%A7%E5%88%B6%20sql%20%E6%89%A7%E8%A1%8C%E6%80%A7%E8%83%BD%EF%BC%8C%E7%81%B5%E6%B4%BB%E5%BA%A6%E9%AB%98%E3%80%82%0A3.%20MyBatis%20%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E7%AE%80%E5%8D%95%E7%9A%84%20XML%20%E6%88%96%E6%B3%A8%E8%A7%A3%E6%9D%A5%E9%85%8D%E7%BD%AE%E5%92%8C%E6%98%A0%E5%B0%84%E5%8E%9F%E7%94%9F%E7%B1%BB%E5%9E%8B%E3%80%81%E6%8E%A5%E5%8F%A3%E5%92%8C%20Java%20%E7%9A%84%20pojo%E5%AF%B9%E8%B1%A1%E4%B8%BA%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%AD%E7%9A%84%E8%AE%B0%E5%BD%95%E3%80%82%E9%81%BF%E5%85%8D%E4%BA%86%E6%89%8B%E5%8A%A8%E5%86%99jdbc%E4%BB%A3%E7%A0%81%EF%BC%8C%E8%AE%BE%E7%BD%AE%E5%8F%82%E6%95%B0%E5%92%8C%E7%BB%93%E6%9E%9C%E9%9B%86%E6%98%A0%E5%B0%84%E3%80%82%0A%0A%23%23%23%23%23%202%20MyBatis%20%E4%B8%8E%20Hibernate%20%E6%9C%89%E5%93%AA%E4%BA%9B%E4%B8%8D%E5%90%8C%EF%BC%9F%0A%0A1.%20Mybatis%20%E5%92%8C%20hibernate%20%E4%B8%8D%E5%90%8C%EF%BC%8C%E5%AE%83%E4%B8%8D%E5%AE%8C%E5%85%A8%E6%98%AF%E4%B8%80%E4%B8%AA%20ORM%20%E6%A1%86%E6%9E%B6%EF%BC%8C%E5%9B%A0%E4%B8%BA%20MyBatis%20%E9%9C%80%E8%A6%81%E7%A8%8B%E5%BA%8F%E5%91%98%E8%87%AA%E5%B7%B1%E7%BC%96%E5%86%99%20Sql%20%E8%AF%AD%E5%8F%A5%EF%BC%8C%E8%80%8Chibernate%E6%98%AF%E5%85%A8%E8%87%AA%E5%8A%A8%E5%8C%96%E7%9A%84%EF%BC%8C%E7%9B%B4%E6%8E%A5%E6%93%8D%E4%BD%9Cjava%E5%AF%B9%E8%B1%A1%E6%97%A0%E9%9C%80%E5%92%8Csql%E6%89%93%E4%BA%A4%E9%81%93%0A2.%20Mybatis%20%E7%9B%B4%E6%8E%A5%E7%BC%96%E5%86%99%E5%8E%9F%E7%94%9F%E6%80%81%20sql%EF%BC%8C%E5%8F%AF%E4%BB%A5%E4%B8%A5%E6%A0%BC%E6%8E%A7%E5%88%B6%20sql%20%E6%89%A7%E8%A1%8C%E6%80%A7%E8%83%BD%EF%BC%8C%E7%81%B5%E6%B4%BB%E5%BA%A6%E9%AB%98%EF%BC%8C%E9%9D%9E%E5%B8%B8%E9%80%82%E5%90%88%E5%AF%B9%E5%85%B3%E7%B3%BB%E6%95%B0%E6%8D%AE%E6%A8%A1%E5%9E%8B%E8%A6%81%E6%B1%82%E4%B8%8D%E9%AB%98%E7%9A%84%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%EF%BC%8C%E5%9B%A0%E4%B8%BA%E8%BF%99%E7%B1%BB%E8%BD%AF%E4%BB%B6%E9%9C%80%E6%B1%82%E5%8F%98%E5%8C%96%E9%A2%91%E7%B9%81%EF%BC%8C%E4%B8%80%E4%BD%86%E9%9C%80%E6%B1%82%E5%8F%98%E5%8C%96%E8%A6%81%E6%B1%82%E8%BF%85%E9%80%9F%E8%BE%93%E5%87%BA%E6%88%90%E6%9E%9C%E3%80%82%E4%BD%86%E6%98%AFmybatis%20%E6%97%A0%E6%B3%95%E5%81%9A%E5%88%B0%E6%95%B0%E6%8D%AE%E5%BA%93%E6%97%A0%E5%85%B3%E6%80%A7%EF%BC%8C%E5%A6%82%E6%9E%9C%E9%9C%80%E8%A6%81%E5%AE%9E%E7%8E%B0%E6%94%AF%E6%8C%81%E5%A4%9A%E7%A7%8D%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E8%BD%AF%E4%BB%B6%EF%BC%8C%E5%88%99%E9%9C%80%E8%A6%81%E8%87%AA%E5%AE%9A%E4%B9%89%E5%A4%9A%E5%A5%97%20sql%20%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6%EF%BC%8C%E5%B7%A5%E4%BD%9C%E9%87%8F%E5%A4%A7%0A3.%20Hibernate%20%E5%AF%B9%E8%B1%A1%2F%E5%85%B3%E7%B3%BB%E6%98%A0%E5%B0%84%E8%83%BD%E5%8A%9B%E5%BC%BA%EF%BC%8C%E6%95%B0%E6%8D%AE%E5%BA%93%E6%97%A0%E5%85%B3%E6%80%A7%E5%A5%BD%EF%BC%8C%E5%AF%B9%E4%BA%8E%E5%85%B3%E7%B3%BB%E6%A8%A1%E5%9E%8B%E8%A6%81%E6%B1%82%E9%AB%98%E7%9A%84%E8%BD%AF%E4%BB%B6%EF%BC%8C%E5%A6%82%E6%9E%9C%E7%94%A8%20hibernate%20%E5%BC%80%E5%8F%91%E5%8F%AF%E4%BB%A5%E8%8A%82%E7%9C%81%E5%BE%88%E5%A4%9A%E4%BB%A3%E7%A0%81%EF%BC%8C%E6%8F%90%E9%AB%98%E6%95%88%E7%8E%87%E3%80%82%0A4.%20MyBatis%20%E4%B8%93%E6%B3%A8%E4%BA%8E%20SQL%20%E6%9C%AC%E8%BA%AB%EF%BC%8C%E6%98%AF%E4%B8%80%E4%B8%AA%E8%B6%B3%E5%A4%9F%E7%81%B5%E6%B4%BB%E7%9A%84%20DAO%20%E5%B1%82%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E3%80%82MyBatis%20%E9%80%82%E7%94%A8%E4%BA%8E%E5%AF%B9%E6%80%A7%E8%83%BD%E7%9A%84%E8%A6%81%E6%B1%82%E5%BE%88%E9%AB%98%EF%BC%8C%E6%88%96%E8%80%85%E9%9C%80%E6%B1%82%E5%8F%98%E5%8C%96%E8%BE%83%E5%A4%9A%E7%9A%84%E9%A1%B9%E7%9B%AE%EF%BC%8C%E5%A6%82%E4%BA%92%E8%81%94%E7%BD%91%E9%A1%B9%E7%9B%AE%0A%0A%0A**Mybatis%20%E4%BB%8E%E6%89%A7%E8%A1%8C%20sql%20%E5%88%B0%E8%BF%94%E5%9B%9E%20result%20%E7%9A%84%E8%BF%87%E7%A8%8B%3A**%0A%E9%80%9A%E8%BF%87%20xml%20%E6%96%87%E4%BB%B6%E6%88%96%E6%B3%A8%E8%A7%A3%E7%9A%84%E6%96%B9%E5%BC%8F%E5%B0%86%E8%A6%81%E6%89%A7%E8%A1%8C%E7%9A%84%E5%90%84%E7%A7%8D%20statement%20%E9%85%8D%E7%BD%AE%E8%B5%B7%E6%9D%A5%EF%BC%8C%E5%B9%B6%E9%80%9A%E8%BF%87java%20%E5%AF%B9%E8%B1%A1%E5%92%8C%20statement%20%E4%B8%AD%20sql%20%E7%9A%84%E5%8A%A8%E6%80%81%E5%8F%82%E6%95%B0%E8%BF%9B%E8%A1%8C%E6%98%A0%E5%B0%84%E7%94%9F%E6%88%90%E6%9C%80%E7%BB%88%E6%89%A7%E8%A1%8C%E7%9A%84%20sql%20%E8%AF%AD%E5%8F%A5%EF%BC%8C%E6%9C%80%E5%90%8E%E7%94%B1%20mybatis%20%E6%A1%86%E6%9E%B6%E6%89%A7%E8%A1%8C%20sql%20%E5%B9%B6%E5%B0%86%E7%BB%93%E6%9E%9C%E8%87%AA%E5%8A%A8%E6%98%A0%E5%B0%84%E4%B8%BA%20java%20%E5%AF%B9%E8%B1%A1%E5%B9%B6%E8%BF%94%E5%9B%9E%E3%80%82%0A%0A%23%23%23%23%20%E4%BA%8C%20SqlSession%0A%0A%26emsp%3B%26emsp%3B%E6%AF%8F%E4%B8%AA%E5%9F%BA%E4%BA%8E%20MyBatis%20%E7%9A%84%E5%BA%94%E7%94%A8%E9%83%BD%E6%98%AF%E4%BB%A5%E4%B8%80%E4%B8%AA%20SqlSessionFactory%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E4%B8%BA%E6%A0%B8%E5%BF%83%E7%9A%84%2CSqlSession%20%E5%AE%8C%E5%85%A8%E5%8C%85%E5%90%AB%E4%BA%86%E9%9D%A2%E5%90%91%E6%95%B0%E6%8D%AE%E5%BA%93%E6%89%A7%E8%A1%8C%20SQL%20%E5%91%BD%E4%BB%A4%E6%89%80%E9%9C%80%E7%9A%84%E6%89%80%E6%9C%89%E6%96%B9%E6%B3%95%E3%80%82%E4%BD%A0%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%20SqlSession%20%E5%AE%9E%E4%BE%8B%E6%9D%A5%E7%9B%B4%E6%8E%A5%E6%89%A7%E8%A1%8C%E5%B7%B2%E6%98%A0%E5%B0%84%E7%BB%91%E5%AE%9A%EF%BC%88bind%EF%BC%89%E7%9A%84%20SQL%20%E8%AF%AD%E5%8F%A5%EF%BC%9A%60%20IUserDao%20userMapper%20%3D%20sqlSession.getMapper(IUserDao.class)%3B%60%0A%0A%26emsp%3B%26emsp%3BSqlSessionFactory%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%20SqlSessionFactoryBuilder%20%E8%8E%B7%E5%BE%97%E3%80%82%E8%80%8C%20SqlSessionFactoryBuilder%20%E5%88%99%E5%8F%AF%E4%BB%A5%E4%BB%8E%20XML%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E6%88%96%E4%B8%80%E4%B8%AA%E9%A2%84%E5%85%88%E5%AE%9A%E5%88%B6%E7%9A%84%20Configuration%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E6%9E%84%E5%BB%BA%E5%87%BA%20SqlSessionFactory%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E3%80%82%E6%97%A2%E7%84%B6%E6%9C%89%E4%BA%86%20SqlSessionFactory%EF%BC%8C%E9%A1%BE%E5%90%8D%E6%80%9D%E4%B9%89%EF%BC%8C%E6%88%91%E4%BB%AC%E5%B0%B1%E5%8F%AF%E4%BB%A5%E4%BB%8E%E4%B8%AD%E8%8E%B7%E5%BE%97%20SqlSession%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E4%BA%86%0A%0A**%E5%8A%A0%E8%BD%BD%E9%85%8D%E7%BD%AE%EF%BC%8C%E5%88%9B%E5%BB%BASqlSessionFactory%EF%BC%9A**%0A%0A%60%60%60java%0AString%20resource%20%3D%20%22org%2Fmybatis%2Fbuilder%2Fmybatis-config.xml%22%3B%0AInputStream%20inputStream%20%3D%20Resources.getResourceAsStream(resource)%3B%0ASqlSessionFactoryBuilder%20builder%20%3D%20new%20SqlSessionFactoryBuilder()%3B%0ASqlSessionFactory%20factory%20%3D%20builder.build(inputStream)%3B%0A%60%60%60%0A**SqlSession%20%E5%8F%AF%E4%BB%A5%E6%89%A7%E8%A1%8C%E5%A6%82%E4%B8%8B%E5%91%BD%E4%BB%A4%EF%BC%9A**%0A%60%60%60java%0A%3CT%3E%20T%20selectOne(String%20statement)%0A%3CE%3E%20List%3CE%3E%20selectList(String%20statement)%0A%3CT%3E%20Cursor%3CT%3E%20selectCursor(String%20statement)%0A%3CK%2CV%3E%20Map%3CK%2CV%3E%20selectMap(String%20statement%2C%20String%20mapKey)%0Aint%20insert(String%20statement)%0Aint%20update(String%20statement)%0Aint%20delete(String%20statement)%0A%60%60%60%0A%0A%23%23%23%23%23%201%20sqlsession%E4%BD%9C%E7%94%A8%E5%9F%9F%E5%92%8C%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%0A%0A**%EF%BC%881%EF%BC%89SqlSessionFactoryBuilder%E7%B1%BB%EF%BC%9A%20SqlSessionFactoryBuilder%E7%B1%BB%E7%94%A8%E6%9D%A5%E5%88%9B%E5%BB%BASqlSessionFactory%EF%BC%8C%E5%8F%AA%E9%9C%80%E8%A6%81%E4%BD%BF%E7%94%A8%E4%B8%80%E6%AC%A1%EF%BC%9B**%0A%E8%BF%99%E4%B8%AA%E7%B1%BB%E5%8F%AF%E4%BB%A5%E8%A2%AB%E5%AE%9E%E4%BE%8B%E5%8C%96%E3%80%81%E4%BD%BF%E7%94%A8%E5%92%8C%E4%B8%A2%E5%BC%83%EF%BC%8C%E4%B8%80%E6%97%A6%E5%88%9B%E5%BB%BA%E4%BA%86%20SqlSessionFactory%EF%BC%8C%E5%B0%B1%E4%B8%8D%E5%86%8D%E9%9C%80%E8%A6%81%E5%AE%83%E4%BA%86%E3%80%82%20%E5%9B%A0%E6%AD%A4%20SqlSessionFactoryBuilder%20%E5%AE%9E%E4%BE%8B%E7%9A%84%E6%9C%80%E4%BD%B3%E4%BD%9C%E7%94%A8%E5%9F%9F%E6%98%AF%E6%96%B9%E6%B3%95%E4%BD%9C%E7%94%A8%E5%9F%9F%EF%BC%88%E4%B9%9F%E5%B0%B1%E6%98%AF%E5%B1%80%E9%83%A8%E6%96%B9%E6%B3%95%E5%8F%98%E9%87%8F%EF%BC%89%E3%80%82%20%E4%BD%A0%E5%8F%AF%E4%BB%A5%E9%87%8D%E7%94%A8%20SqlSessionFactoryBuilder%20%E6%9D%A5%E5%88%9B%E5%BB%BA%E5%A4%9A%E4%B8%AA%20SqlSessionFactory%20%E5%AE%9E%E4%BE%8B%EF%BC%8C%E4%BD%86%E6%98%AF%E6%9C%80%E5%A5%BD%E8%BF%98%E6%98%AF%E4%B8%8D%E8%A6%81%E8%AE%A9%E5%85%B6%E4%B8%80%E7%9B%B4%E5%AD%98%E5%9C%A8%EF%BC%8C%E4%BB%A5%E4%BF%9D%E8%AF%81%E6%89%80%E6%9C%89%E7%9A%84%20XML%20%E8%A7%A3%E6%9E%90%E8%B5%84%E6%BA%90%E5%8F%AF%E4%BB%A5%E8%A2%AB%E9%87%8A%E6%94%BE%E7%BB%99%E6%9B%B4%E9%87%8D%E8%A6%81%E7%9A%84%E4%BA%8B%E6%83%85%E3%80%82%0A%0A%0A**%EF%BC%882%EF%BC%89SqlSessionFactory%E7%B1%BB%3A%20SqlSessionFactory%E5%BA%94%E8%AF%A5%E5%8F%AA%E8%A2%AB%E5%88%9B%E5%BB%BA%E4%B8%80%E6%AC%A1%EF%BC%8C%E5%BA%94%E7%94%A8%E8%BF%90%E8%A1%8C%E6%9C%9F%E9%97%B4%E4%B8%80%E7%9B%B4%E4%BD%BF%E7%94%A8%EF%BC%8C%E4%BD%BF%E7%94%A8%E5%8D%95%E4%BE%8B%EF%BC%9B**%0ASqlSessionFactory%20%E4%B8%80%E6%97%A6%E8%A2%AB%E5%88%9B%E5%BB%BA%E5%B0%B1%E5%BA%94%E8%AF%A5%E5%9C%A8%E5%BA%94%E7%94%A8%E7%9A%84%E8%BF%90%E8%A1%8C%E6%9C%9F%E9%97%B4%E4%B8%80%E7%9B%B4%E5%AD%98%E5%9C%A8%EF%BC%8C%E6%B2%A1%E6%9C%89%E4%BB%BB%E4%BD%95%E7%90%86%E7%94%B1%E4%B8%A2%E5%BC%83%E5%AE%83%E6%88%96%E9%87%8D%E6%96%B0%E5%88%9B%E5%BB%BA%E5%8F%A6%E4%B8%80%E4%B8%AA%E5%AE%9E%E4%BE%8B%E3%80%82%20%E4%BD%BF%E7%94%A8%20SqlSessionFactory%20%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E6%98%AF%E5%9C%A8%E5%BA%94%E7%94%A8%E8%BF%90%E8%A1%8C%E6%9C%9F%E9%97%B4%E4%B8%8D%E8%A6%81%E9%87%8D%E5%A4%8D%E5%88%9B%E5%BB%BA%E5%A4%9A%E6%AC%A1%EF%BC%8C%E5%A4%9A%E6%AC%A1%E9%87%8D%E5%BB%BA%20SqlSessionFactory%20%E8%A2%AB%E8%A7%86%E4%B8%BA%E4%B8%80%E7%A7%8D%E4%BB%A3%E7%A0%81%E2%80%9C%E5%9D%8F%E5%91%B3%E9%81%93%EF%BC%88bad%20smell%EF%BC%89%E2%80%9D%E3%80%82%E5%9B%A0%E6%AD%A4%20SqlSessionFactory%20%E7%9A%84%E6%9C%80%E4%BD%B3%E4%BD%9C%E7%94%A8%E5%9F%9F%E6%98%AF%E5%BA%94%E7%94%A8%E4%BD%9C%E7%94%A8%E5%9F%9F%E3%80%82%20%E6%9C%89%E5%BE%88%E5%A4%9A%E6%96%B9%E6%B3%95%E5%8F%AF%E4%BB%A5%E5%81%9A%E5%88%B0%EF%BC%8C%E6%9C%80%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%B1%E6%98%AF%E4%BD%BF%E7%94%A8%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F%E6%88%96%E8%80%85%E9%9D%99%E6%80%81%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F%E3%80%82%0A%0A**%EF%BC%883%EF%BC%89SqlSession%E7%B1%BB%3A%E6%AF%8F%E4%B8%AA%E7%BA%BF%E7%A8%8B%E9%83%BD%E5%BA%94%E8%AF%A5%E6%9C%89%E5%AE%83%E8%87%AA%E5%B7%B1%E7%9A%84%20SqlSession%20%E5%AE%9E%E4%BE%8B%EF%BC%8C%E4%BD%BF%E7%94%A8%E5%AE%8C%E6%AF%95%E8%AE%B0%E5%BE%97%E5%85%B3%E9%97%AD**%0A%26emsp%3B%26emsp%3B%E6%AF%8F%E4%B8%AA%E7%BA%BF%E7%A8%8B%E9%83%BD%E5%BA%94%E8%AF%A5%E6%9C%89%E5%AE%83%E8%87%AA%E5%B7%B1%E7%9A%84%20SqlSession%20%E5%AE%9E%E4%BE%8B%E3%80%82SqlSession%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E4%B8%8D%E6%98%AF%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%EF%BC%8C%E5%9B%A0%E6%AD%A4%E6%98%AF%E4%B8%8D%E8%83%BD%E8%A2%AB%E5%85%B1%E4%BA%AB%E7%9A%84%EF%BC%8C%E6%89%80%E4%BB%A5%E5%AE%83%E7%9A%84%E6%9C%80%E4%BD%B3%E7%9A%84%E4%BD%9C%E7%94%A8%E5%9F%9F%E6%98%AF%E8%AF%B7%E6%B1%82%E6%88%96%E6%96%B9%E6%B3%95%E4%BD%9C%E7%94%A8%E5%9F%9F%E3%80%82%0A%0A%26emsp%3B%26emsp%3B%E7%BB%9D%E5%AF%B9%E4%B8%8D%E8%83%BD%E5%B0%86%20SqlSession%20%E5%AE%9E%E4%BE%8B%E7%9A%84%E5%BC%95%E7%94%A8%E6%94%BE%E5%9C%A8%E4%B8%80%E4%B8%AA%E7%B1%BB%E7%9A%84%E9%9D%99%E6%80%81%E5%9F%9F%EF%BC%8C%E7%94%9A%E8%87%B3%E4%B8%80%E4%B8%AA%E7%B1%BB%E7%9A%84%E5%AE%9E%E4%BE%8B%E5%8F%98%E9%87%8F%E4%B9%9F%E4%B8%8D%E8%A1%8C%E3%80%82%20%E4%B9%9F%E7%BB%9D%E4%B8%8D%E8%83%BD%E5%B0%86%20SqlSession%20%E5%AE%9E%E4%BE%8B%E7%9A%84%E5%BC%95%E7%94%A8%E6%94%BE%E5%9C%A8%E4%BB%BB%E4%BD%95%E7%B1%BB%E5%9E%8B%E7%9A%84%E6%89%98%E7%AE%A1%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%B8%AD%EF%BC%8C%E6%AF%94%E5%A6%82%20Servlet%20%E6%A1%86%E6%9E%B6%E4%B8%AD%E7%9A%84%20HttpSession%E3%80%82%20%E5%A6%82%E6%9E%9C%E4%BD%A0%E7%8E%B0%E5%9C%A8%E6%AD%A3%E5%9C%A8%E4%BD%BF%E7%94%A8%E4%B8%80%E7%A7%8D%20Web%20%E6%A1%86%E6%9E%B6%EF%BC%8C%E8%A6%81%E8%80%83%E8%99%91%20SqlSession%20%E6%94%BE%E5%9C%A8%E4%B8%80%E4%B8%AA%E5%92%8C%20HTTP%20%E8%AF%B7%E6%B1%82%E5%AF%B9%E8%B1%A1%E7%9B%B8%E4%BC%BC%E7%9A%84%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%B8%AD%E3%80%82%20%E6%8D%A2%E5%8F%A5%E8%AF%9D%E8%AF%B4%EF%BC%8C%E6%AF%8F%E6%AC%A1%E6%94%B6%E5%88%B0%E7%9A%84%20HTTP%20%E8%AF%B7%E6%B1%82%EF%BC%8C%E5%B0%B1%E5%8F%AF%E4%BB%A5%E6%89%93%E5%BC%80%E4%B8%80%E4%B8%AA%20SqlSession%EF%BC%8C%E8%BF%94%E5%9B%9E%E4%B8%80%E4%B8%AA%E5%93%8D%E5%BA%94%EF%BC%8C%E5%B0%B1%E5%85%B3%E9%97%AD%E5%AE%83%E3%80%82%20%E8%BF%99%E4%B8%AA%E5%85%B3%E9%97%AD%E6%93%8D%E4%BD%9C%E6%98%AF%E5%BE%88%E9%87%8D%E8%A6%81%E7%9A%84%EF%BC%8C%E4%BD%A0%E5%BA%94%E8%AF%A5%E6%8A%8A%E8%BF%99%E4%B8%AA%E5%85%B3%E9%97%AD%E6%93%8D%E4%BD%9C%E6%94%BE%E5%88%B0%20finally%20%E5%9D%97%E4%B8%AD%E4%BB%A5%E7%A1%AE%E4%BF%9D%E6%AF%8F%E6%AC%A1%E9%83%BD%E8%83%BD%E6%89%A7%E8%A1%8C%E5%85%B3%E9%97%AD%E3%80%82%20%E4%B8%8B%E9%9D%A2%E7%9A%84%E7%A4%BA%E4%BE%8B%E5%B0%B1%E6%98%AF%E4%B8%80%E4%B8%AA%E7%A1%AE%E4%BF%9D%20SqlSession%20%E5%85%B3%E9%97%AD%E7%9A%84%E6%A0%87%E5%87%86%E6%A8%A1%E5%BC%8F%EF%BC%9A%0A%60%60%60java%0ASqlSession%20session%20%3D%20sqlSessionFactory.openSession()%3B%0Atry%20%7B%0A%20%20%2F%2F%20%E4%BD%A0%E7%9A%84%E5%BA%94%E7%94%A8%E9%80%BB%E8%BE%91%E4%BB%A3%E7%A0%81%0A%7D%20finally%20%7B%0A%20%20session.close()%3B%0A%7D%0A%60%60%60%0A%0A!%5B8c69a6174985784169461556798fc284.png%5D(en-resource%3A%2F%2Fdatabase%2F1207%3A1)%0A%0A%0A%23%23%23%23%23%202%E5%AF%B9%E8%B1%A1%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%92%8C%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5%E6%A1%86%E6%9E%B6%0A%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5%E6%A1%86%E6%9E%B6%E5%8F%AF%E4%BB%A5%E5%88%9B%E5%BB%BA%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%E3%80%81%E5%9F%BA%E4%BA%8E%E4%BA%8B%E5%8A%A1%E7%9A%84%20SqlSession%20%E5%92%8C%E6%98%A0%E5%B0%84%E5%99%A8%EF%BC%8C%E5%B9%B6%E5%B0%86%E5%AE%83%E4%BB%AC%E7%9B%B4%E6%8E%A5%E6%B3%A8%E5%85%A5%E5%88%B0%E4%BD%A0%E7%9A%84%20bean%20%E4%B8%AD%EF%BC%8C%E5%9B%A0%E6%AD%A4%E5%8F%AF%E4%BB%A5%E7%9B%B4%E6%8E%A5%E5%BF%BD%E7%95%A5%E5%AE%83%E4%BB%AC%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%EF%BC%9B%E4%BE%8B%E5%A6%82%E5%9C%A8spring%E4%B8%AD%E4%BD%BF%E7%94%A8mybatis%EF%BC%8C%E4%BD%A0%E4%B8%8D%E9%9C%80%E8%A6%81%E5%85%B3%E5%BF%83sqlsession%E7%9A%84%E5%88%9B%E5%BB%BA%EF%BC%8C%E7%94%B1%E6%A1%86%E6%9E%B6%E8%87%AA%E5%8A%A8%E5%AE%8C%E6%88%90%E3%80%82%0A%0A%0A%23%23%23%23%23%203%20%E6%99%AE%E9%80%9A%20java%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Mybatis%0A%EF%BC%880%EF%BC%89%E5%BC%95%E5%85%A5mybatis%0A%60%60%60xml%0A%3C!--%20https%3A%2F%2Fmvnrepository.com%2Fartifact%2Forg.mybatis%2Fmybatis%20--%3E%0A%3Cdependency%3E%0A%20%20%20%20%3CgroupId%3Eorg.mybatis%3C%2FgroupId%3E%0A%20%20%20%20%3CartifactId%3Emybatis%3C%2FartifactId%3E%0A%20%20%20%20%3Cversion%3E3.5.7%3C%2Fversion%3E%0A%3C%2Fdependency%3E%0A%0A%60%60%60%0A(1)%20%E5%AE%9A%E4%B9%89mybatis-config.xml%E6%96%87%E4%BB%B6%E9%85%8D%E7%BD%AESqlSessionFactoryBuilder%20%3A%0A%60%60%60xml%0A%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22UTF-8%22%20%3F%3E%0A%3C!DOCTYPE%20configuration%0A%20%20%20%20%20%20%20%20PUBLIC%20%22-%2F%2Fmybatis.org%2F%2FDTD%20Config%203.0%2F%2FEN%22%0A%20%20%20%20%20%20%20%20%22http%3A%2F%2Fmybatis.org%2Fdtd%2Fmybatis-3-config.dtd%22%3E%0A%0A%3C!--%0AXML%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%E5%8C%85%E5%90%AB%E4%BA%86%E5%AF%B9%20MyBatis%20%E7%B3%BB%E7%BB%9F%E7%9A%84%E6%A0%B8%E5%BF%83%E8%AE%BE%E7%BD%AE%EF%BC%8C%E5%8C%85%E5%90%AB%E8%8E%B7%E5%8F%96%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E5%AE%9E%E4%BE%8B%E7%9A%84%E6%95%B0%E6%8D%AE%E6%BA%90%0A%EF%BC%88DataSource%EF%BC%89%E5%92%8C%E5%86%B3%E5%AE%9A%E4%BA%8B%E5%8A%A1%E4%BD%9C%E7%94%A8%E5%9F%9F%E5%92%8C%E6%8E%A7%E5%88%B6%E6%96%B9%E5%BC%8F%E7%9A%84%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86%E5%99%A8%EF%BC%88TransactionManager%EF%BC%89%0A--%3E%0A%3Cconfiguration%3E%0A%20%20%20%20%3Cenvironments%20default%3D%22development%22%3E%0A%20%20%20%20%20%20%20%20%3Cenvironment%20id%3D%22development%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3CtransactionManager%20type%3D%22JDBC%22%2F%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3CdataSource%20type%3D%22POOLED%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Cproperty%20name%3D%22driver%22%20value%3D%22com.mysql.jdbc.Driver%22%2F%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Cproperty%20name%3D%22url%22%20value%3D%22jdbc%3Amysql%3A%2F%2Flocalhost%3A3306%2Fshuai%22%2F%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Cproperty%20name%3D%22username%22%20value%3D%22root%22%2F%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Cproperty%20name%3D%22password%22%20value%3D%221qaz%40WSX%22%2F%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2FdataSource%3E%0A%20%20%20%20%20%20%20%20%3C%2Fenvironment%3E%0A%20%20%20%20%3C%2Fenvironments%3E%0A%20%20%20%20%3Cmappers%3E%0A%20%20%20%20%20%20%20%20%3Cmapper%20resource%3D%22mapper%2Fuser-mapper.xml%22%2F%3E%0A%20%20%20%20%3C%2Fmappers%3E%0A%3C%2Fconfiguration%3E%0A%0A%3C!--%0A%E8%BF%98%E6%9C%89%E5%BE%88%E5%A4%9A%E5%8F%AF%E4%BB%A5%E5%9C%A8%20XML%20%E6%96%87%E4%BB%B6%E4%B8%AD%E8%BF%9B%E8%A1%8C%E9%85%8D%E7%BD%AE%EF%BC%8C%E4%B8%8A%E9%9D%A2%E7%9A%84%E7%A4%BA%E4%BE%8B%E6%8C%87%E5%87%BA%E7%9A%84%E5%88%99%E6%98%AF%E6%9C%80%E5%85%B3%E9%94%AE%E7%9A%84%E9%83%A8%E5%88%86%E3%80%82%20%E8%A6%81%E6%B3%A8%E6%84%8F%20XML%20%E5%A4%B4%E9%83%A8%E7%9A%84%E5%A3%B0%E6%98%8E%EF%BC%8C%E5%AE%83%E7%94%A8%E6%9D%A5%E9%AA%8C%E8%AF%81%20XML%20%E6%96%87%E6%A1%A3%E6%AD%A3%E7%A1%AE%E6%80%A7%E3%80%82%0Aenvironment%20%E5%85%83%E7%B4%A0%E4%BD%93%E4%B8%AD%E5%8C%85%E5%90%AB%E4%BA%86%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86%E5%92%8C%E8%BF%9E%E6%8E%A5%E6%B1%A0%E7%9A%84%E9%85%8D%E7%BD%AE%E3%80%82%0Amappers%20%E5%85%83%E7%B4%A0%E5%88%99%E6%98%AF%E5%8C%85%E5%90%AB%E4%B8%80%E7%BB%84%E6%98%A0%E5%B0%84%E5%99%A8%EF%BC%88mapper%EF%BC%89%EF%BC%8C%E8%BF%99%E4%BA%9B%E6%98%A0%E5%B0%84%E5%99%A8%E7%9A%84%20XML%20%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E4%BA%86%20SQL%20%E4%BB%A3%E7%A0%81%E5%92%8C%E6%98%A0%E5%B0%84%E5%AE%9A%E4%B9%89%E4%BF%A1%E6%81%AF--%3E%0A%60%60%60%0A%EF%BC%882%EF%BC%89%E5%AE%9A%E4%B9%89sql%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6%0A%60%60%60xml%0A%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22UTF-8%22%3F%3E%0A%3C!DOCTYPE%20mapper%0A%20%20%20%20%20%20%20%20PUBLIC%20%22-%2F%2Fmybatis.org%2F%2FDTD%20Mapper%203.0%2F%2FEN%22%0A%20%20%20%20%20%20%20%20%22http%3A%2F%2Fmybatis.org%2Fdtd%2Fmybatis-3-mapper.dtd%22%3E%0A%0A%3C!--%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4%E5%BA%94%E8%AF%A5%E6%98%AF%E5%AF%B9%E5%BA%94%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%8C%85%E5%90%8D%2B%E6%8E%A5%E5%8F%A3%E5%90%8D%20--%3E%0A%3Cmapper%20namespace%3D%22com.shuailee.dao.IUserDao%22%3E%0A%20%20%20%20%3C!--%E6%9F%A5%E8%AF%A2%E7%94%A8%E6%88%B7%E4%BF%A1%E6%81%AF%E5%B9%B6%E5%88%86%E9%A1%B5%20--%3E%0A%20%20%20%20%3Cselect%20id%3D%22queryUser%22%20resultType%3D%22com.shuailee.model.User%22%3E%0A%20%20%20%20%20%20%20%20select%20*%20from%20tb_user%0A%20%20%20%20%3C%2Fselect%3E%0A%3C%2Fmapper%3E%0A%60%60%60%0A%EF%BC%883%EF%BC%89%E6%98%A0%E5%B0%84%E6%8E%A5%E5%8F%A3%EF%BC%9A%0A%E6%98%A0%E5%B0%84%E5%99%A8%E6%8E%A5%E5%8F%A3%E4%B8%8D%E9%9C%80%E8%A6%81%E5%8E%BB%E5%AE%9E%E7%8E%B0%E4%BB%BB%E4%BD%95%E6%8E%A5%E5%8F%A3%E6%88%96%E7%BB%A7%E6%89%BF%E8%87%AA%E4%BB%BB%E4%BD%95%E7%B1%BB%2C%E5%8F%AA%E8%A6%81%E6%96%B9%E6%B3%95%E5%8F%AF%E4%BB%A5%E8%A2%AB%E5%94%AF%E4%B8%80%E6%A0%87%E8%AF%86%E5%AF%B9%E5%BA%94%E7%9A%84%E6%98%A0%E5%B0%84%E8%AF%AD%E5%8F%A5%E5%B0%B1%E5%8F%AF%E4%BB%A5%E4%BA%86%20%3A%60com.shuailee.dao.IUserDao%2BqueryUser%60%0A%60%60%60java%0A%2F%2F%20%E5%A4%96%E9%83%A8%E7%9B%B4%E6%8E%A5%E4%BD%BF%E7%94%A8%E8%AF%A5%E6%8E%A5%E5%8F%A3%E5%8D%B3%E5%8F%AF%20%0Apublic%20interface%20IUserDao%20%7B%20%0AList%3CUser%3E%20queryUser()%3B%20%0A%7D%0A%60%60%60%0A%0A%EF%BC%884%EF%BC%89%E6%B5%8B%E8%AF%95%E4%BB%A3%E7%A0%81%EF%BC%9A%0A%60%60%60%20java%0Apublic%20class%20App%20%0A%7B%0A%20%20%20%20%2F**%0A%20%20%20%20%20*%E4%BB%8E%20XML%20%E4%B8%AD%E6%9E%84%E5%BB%BA%20SqlSessionFactory%2C%20%E6%89%A7%E8%A1%8C%E6%9F%A5%E8%AF%A2%E5%92%8Cmap%0A%20%20%20%20%20*%20*%2F%0A%20%20%20%20public%20void%20queryUserTest()%20%7B%0A%20%20%20%20%20%20%20%20%2F**%0A%20%20%20%20%20%20%20%20%20*%20%E6%AF%8F%E4%B8%AA%E5%9F%BA%E4%BA%8E%20MyBatis%20%E7%9A%84%E5%BA%94%E7%94%A8%E9%83%BD%E6%98%AF%E4%BB%A5%E4%B8%80%E4%B8%AA%20SqlSessionFactory%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E4%B8%BA%E6%A0%B8%E5%BF%83%E7%9A%84%E3%80%82SqlSessionFactory%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%20SqlSessionFactoryBuilder%0A%20%20%20%20%20%20%20%20%20*%20%E8%8E%B7%E5%BE%97%E3%80%82%E8%80%8C%20SqlSessionFactoryBuilder%20%E5%88%99%E5%8F%AF%E4%BB%A5%E4%BB%8E%20XML%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E6%88%96%E4%B8%80%E4%B8%AA%E9%A2%84%E5%85%88%E5%AE%9A%E5%88%B6%E7%9A%84%20Configuration%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E6%9E%84%E5%BB%BA%E5%87%BA%20SqlSessionFactory%20%E7%9A%84%E5%AE%9E%E4%BE%8B%0A%20%20%20%20%20%20%20%20%20*%0A%20%20%20%20%20%20%20%20%20*%20%20%E4%BB%8E%20XML%20%E6%96%87%E4%BB%B6%E4%B8%AD%E6%9E%84%E5%BB%BA%20SqlSessionFactory%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E9%9D%9E%E5%B8%B8%E7%AE%80%E5%8D%95%EF%BC%8C%E5%BB%BA%E8%AE%AE%E4%BD%BF%E7%94%A8%E7%B1%BB%E8%B7%AF%E5%BE%84%E4%B8%8B%E7%9A%84%E8%B5%84%E6%BA%90%E6%96%87%E4%BB%B6%E8%BF%9B%E8%A1%8C%E9%85%8D%E7%BD%AE%0A%20%20%20%20%20%20%20%20%20*%20%E4%BD%86%E6%98%AF%E4%B9%9F%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E4%BB%BB%E6%84%8F%E7%9A%84%E8%BE%93%E5%85%A5%E6%B5%81%EF%BC%88InputStream%EF%BC%89%E5%AE%9E%E4%BE%8B%EF%BC%8C%E5%8C%85%E6%8B%AC%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%BD%A2%E5%BC%8F%E7%9A%84%E6%96%87%E4%BB%B6%E8%B7%AF%E5%BE%84%E6%88%96%E8%80%85%20file%3A%2F%2F%20%E7%9A%84%20URL%20%E5%BD%A2%E5%BC%8F%E7%9A%84%E6%96%87%E4%BB%B6%E8%B7%AF%E5%BE%84%E6%9D%A5%E9%85%8D%E7%BD%AE%0A%20%20%20%20%20%20%20%20%20*%20MyBatis%20%E5%8C%85%E5%90%AB%E4%B8%80%E4%B8%AA%E5%90%8D%E5%8F%AB%20Resources%20%E7%9A%84%E5%B7%A5%E5%85%B7%E7%B1%BB%EF%BC%8C%E5%AE%83%E5%8C%85%E5%90%AB%E4%B8%80%E4%BA%9B%E5%AE%9E%E7%94%A8%E6%96%B9%E6%B3%95%EF%BC%8C%E5%8F%AF%E4%BD%BF%E4%BB%8E%20classpath%20%E6%88%96%E5%85%B6%E4%BB%96%E4%BD%8D%E7%BD%AE%E5%8A%A0%E8%BD%BD%E8%B5%84%E6%BA%90%E6%96%87%E4%BB%B6%E6%9B%B4%E5%8A%A0%E5%AE%B9%E6%98%93%0A%20%20%20%20%20%20%20%20%20*%2F%0A%20%20%20%20%20%20%20%20String%20resource%20%3D%20%22mybatis-config.xml%22%3B%0A%20%20%20%20%20%20%20%20InputStream%20inputStream%20%3D%20null%3B%0A%20%20%20%20%20%20%20%20try%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20inputStream%20%3D%20Resources.getResourceAsStream(resource)%3B%0A%20%20%20%20%20%20%20%20%7D%20catch%20(IOException%20e)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20e.printStackTrace()%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%2F%2F%E4%BB%8E%20SqlSessionFactory%20%E4%B8%AD%E8%8E%B7%E5%8F%96%20SqlSession%0A%20%20%20%20%20%20%20%20%2F%2F%E6%97%A2%E7%84%B6%E6%9C%89%E4%BA%86%20SqlSessionFactory%EF%BC%8C%E9%A1%BE%E5%90%8D%E6%80%9D%E4%B9%89%EF%BC%8C%E6%88%91%E4%BB%AC%E5%B0%B1%E5%8F%AF%E4%BB%A5%E4%BB%8E%E4%B8%AD%E8%8E%B7%E5%BE%97%20SqlSession%20%E7%9A%84%E5%AE%9E%E4%BE%8B%E4%BA%86%E3%80%82%0A%20%20%20%20%20%20%20%20%2F%2FSqlSession%20%E5%AE%8C%E5%85%A8%E5%8C%85%E5%90%AB%E4%BA%86%E9%9D%A2%E5%90%91%E6%95%B0%E6%8D%AE%E5%BA%93%E6%89%A7%E8%A1%8C%20SQL%20%E5%91%BD%E4%BB%A4%E6%89%80%E9%9C%80%E7%9A%84%E6%89%80%E6%9C%89%E6%96%B9%E6%B3%95%E3%80%82%E4%BD%A0%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%20SqlSession%20%E5%AE%9E%E4%BE%8B%E6%9D%A5%E7%9B%B4%E6%8E%A5%E6%89%A7%E8%A1%8C%E5%B7%B2%E6%98%A0%E5%B0%84%E7%9A%84%20SQL%20%E8%AF%AD%E5%8F%A5%0A%20%20%20%20%20%20%20%20SqlSessionFactory%20sqlSessionFactory%20%3D%20%20%20new%20SqlSessionFactoryBuilder().build(inputStream)%3B%0A%20%20%20%20%20%20%20%20SqlSession%20sqlSession%3D%20sqlSessionFactory.openSession()%3B%0A%20%20%20%20%20%20%20try%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%E6%89%A7%E8%A1%8C%E6%9F%A5%E8%AF%A2%E5%92%8C%E6%98%A0%E5%B0%84%0A%20%20%20%20%20%20%20%20%20%20%20%20IUserDao%20userMapper%20%3D%20sqlSession.getMapper(IUserDao.class)%3B%20%2F%2F%E4%BD%BF%E7%94%A8%E6%AD%A3%E7%A1%AE%E6%8F%8F%E8%BF%B0%E6%AF%8F%E4%B8%AA%E8%AF%AD%E5%8F%A5%E7%9A%84%E5%8F%82%E6%95%B0%E5%92%8C%E8%BF%94%E5%9B%9E%E5%80%BC%E7%9A%84%E6%8E%A5%E5%8F%A3%0A%20%20%20%20%20%20%20%20%20%20%20%20List%3CUser%3E%20user%20%3D%20userMapper.queryUser()%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20System.out.println(user.get(0).toString())%3B%0A%20%20%20%20%20%20%20%20%7Dfinally%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20sqlSession.close()%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%0A%20%20%20%20public%20static%20void%20main(%20String%5B%5D%20args%20)%0A%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20App%20app%3Dnew%20App()%3B%0A%20%20%20%20%20%20%20%20app.queryUserTest()%3B%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%0A%0A%23%23%23%23%23%203%20Springboot%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Mybatis%0A**%EF%BC%880%EF%BC%89%E5%BC%95%E5%85%A5mybatis**%0A%60%60%60xml%0A%3C!--mybatis--%3E%0A%3Cdependency%3E%0A%20%20%20%20%3CgroupId%3Eorg.mybatis.spring.boot%3C%2FgroupId%3E%0A%20%20%20%20%3CartifactId%3Emybatis-spring-boot-starter%3C%2FartifactId%3E%0A%20%20%20%20%3Cversion%3E2.1.3%3C%2Fversion%3E%0A%3C%2Fdependency%3E%0A%0A%3C!--druid--%3E%0A%3Cdependency%3E%0A%20%20%20%20%3CgroupId%3Ecom.alibaba%3C%2FgroupId%3E%0A%20%20%20%20%3CartifactId%3Edruid-spring-boot-starter%3C%2FartifactId%3E%0A%20%20%20%20%3Cversion%3E1.1.9%3C%2Fversion%3E%0A%3C%2Fdependency%3E%0A%3C!--mysql%20driven--%3E%0A%3Cdependency%3E%0A%20%20%20%20%3CgroupId%3Emysql%3C%2FgroupId%3E%0A%20%20%20%20%3CartifactId%3Emysql-connector-java%3C%2FartifactId%3E%0A%20%20%20%20%3Cversion%3E8.0.19%3C%2Fversion%3E%0A%3C%2Fdependency%3E%0A%0A%3C!--mybatis%E5%88%86%E9%A1%B5%E6%8F%92%E4%BB%B6--%3E%0A%3Cdependency%3E%0A%20%20%20%20%3CgroupId%3Ecom.github.pagehelper%3C%2FgroupId%3E%0A%20%20%20%20%3CartifactId%3Epagehelper-spring-boot-starter%3C%2FartifactId%3E%0A%20%20%20%20%3Cversion%3E1.2.3%3C%2Fversion%3E%0A%3C%2Fdependency%3E%0A%60%60%60%0A**%EF%BC%881%EF%BC%89%E6%95%B0%E6%8D%AE%E6%BA%90%E9%85%8D%E7%BD%AE%EF%BC%9A**%0Ahttps%3A%2F%2Fgithub.com%2Falibaba%2Fdruid%2Ftree%2Fmaster%2Fdruid-spring-boot-starter%0A%60%60%60sh%0A%23%20jdbc%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE%EF%BC%8C%E5%9C%A8%E8%BF%9E%E6%8E%A5%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%B8%AD%E5%8A%A0%E5%85%A5%20%26allowMultiQueries%3Dtrue%20%20%E5%BC%80%E5%90%AF%E6%89%B9%E9%87%8F%E8%AF%AD%E5%8F%A5%E6%89%A7%E8%A1%8C%0Aspring.datasource.url%20%3D%20jdbc%3Amysql%3A%2F%2Faaa.com%3A3306%2Fshuai%3FuseUnicode%3Dtrue%26characterEncoding%3Dutf8%26serverTimezone%3DGMT%252B8%26zeroDateTimeBehavior%3DconvertToNull%26allowMultiQueries%3Dtrue%0Aspring.datasource.username%20%3D%20test%0Aspring.datasource.password%20%3D%20123456%0Aspring.datasource.driver-class-name%20%3D%20com.mysql.cj.jdbc.Driver%0Aspring.datasource.type%20%3D%20com.alibaba.druid.pool.DruidDataSource%0A%0A%23%20druid%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E6%B1%A0%E9%85%8D%E7%BD%AE%0A%23%20%E5%88%9D%E5%A7%8B%E8%BF%9E%E6%8E%A5%E6%95%B0%0Aspring.datasource.druid.initial-size%3D10%0A%23%20%E6%9C%80%E5%A4%A7%E8%BF%9E%E6%8E%A5%E6%95%B0%0Aspring.datasource.druid.max-active%3D30%0A%23%20%E6%9C%80%E5%B0%8F%E8%BF%9E%E6%8E%A5%E6%95%B0%0Aspring.datasource.druid.min-idle%3D5%0Aspring.datasource.druid.max-wait%3D%0A%23%20%E8%8E%B7%E5%8F%96%E8%BF%9E%E6%8E%A5%E7%AD%89%E5%BE%85%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%0Aspring.datasource.druid.max-wait%20%3D60000%0A%23%20%E9%85%8D%E7%BD%AE%E9%97%B4%E9%9A%94%E5%A4%9A%E4%B9%85%E6%89%8D%E8%BF%9B%E8%A1%8C%E4%B8%80%E6%AC%A1%E6%A3%80%E6%B5%8B%EF%BC%8C%E6%A3%80%E6%B5%8B%E9%9C%80%E8%A6%81%E5%85%B3%E9%97%AD%E7%9A%84%E7%A9%BA%E9%97%B2%E8%BF%9E%E6%8E%A5%EF%BC%8C%E5%8D%95%E4%BD%8D%E6%98%AF%E6%AF%AB%E7%A7%92%0Aspring.datasource.druid.time-between-eviction-runs-millis%20%3D%2060000%0A%23%20%E9%85%8D%E7%BD%AE%E4%B8%80%E4%B8%AA%E8%BF%9E%E6%8E%A5%E5%9C%A8%E6%B1%A0%E4%B8%AD%E6%9C%80%E5%B0%8F%E7%94%9F%E5%AD%98%E7%9A%84%E6%97%B6%E9%97%B4%EF%BC%8C%E5%8D%95%E4%BD%8D%E6%98%AF%E6%AF%AB%E7%A7%92%0Aspring.datasource.druid.min-evictable-idle-time-millis%20%3D%20300000%0A%0A%23%20%E9%85%8D%E7%BD%AE%E7%9B%91%E6%8E%A7%E7%BB%9F%E8%AE%A1%E6%8B%A6%E6%88%AA%E7%9A%84filters%EF%BC%8C%E5%8E%BB%E6%8E%89%E5%90%8E%E7%9B%91%E6%8E%A7%E7%95%8C%E9%9D%A2SQL%E6%97%A0%E6%B3%95%E8%BF%9B%E8%A1%8C%E7%BB%9F%E8%AE%A1%EF%BC%8C%E2%80%99wall%E2%80%99%E7%94%A8%E4%BA%8E%E9%98%B2%E7%81%AB%E5%A2%99%0A%23%20spring.datasource.druid.filters%3Dconfig%2Cstat%2Cwall%2Clog4j%20%0A%0A%0A%0A%23%20mybatis%20%E9%85%8D%E7%BD%AE%EF%BC%88%E5%8F%AF%E4%BB%A5%E4%B8%8D%E9%9C%80%E8%A6%81%EF%BC%89%0A%23%20mybatis.mapper-locations%3Dclasspath%3Amapper%2F*.xml%20%0A%23%20mybatis.type-aliases-package%3Dcom.wdbyte.domain%0A%60%60%60%0A**%EF%BC%882%EF%BC%89%E5%AE%9A%E4%B9%89sql%E6%98%A0%E5%B0%84xml%E6%96%87%E4%BB%B6%E5%92%8C%E6%98%A0%E5%B0%84%E6%8E%A5%E5%8F%A3**%0A%60%60%60xml%0A%3C!--%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4%E5%BA%94%E8%AF%A5%E6%98%AF%E5%AF%B9%E5%BA%94%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%8C%85%E5%90%8D%2B%E6%8E%A5%E5%8F%A3%E5%90%8D--%3E%0A%3Cmapper%20namespace%3D%22com.shuailee.dao.IUserDao%22%3E%0A%20%20%20%20%3Cselect%20id%3D%22queryUserByID%22%20parameterType%3D%22int%22%20resultType%3D%22hashmap%22%20%3E%0A%20%20%20%20%20%20%20%20SELECT%20*%20FROM%20tb_user%20WHERE%20user_id%20%3D%20%23%7Bid%7D%0A%20%20%20%20%3C%2Fselect%3E%0A%3C%2Fmapper%3E%0A%60%60%60%0A%60%60%60java%20%E6%98%A0%E5%B0%84%E6%8E%A5%E5%8F%A3%0A%2F%2F%20%E5%A4%96%E9%83%A8%E7%9B%B4%E6%8E%A5%E4%BD%BF%E7%94%A8%E8%AF%A5%E6%8E%A5%E5%8F%A3%E5%8D%B3%E5%8F%AF%0Apublic%20interface%20IUserDao%20%7B%0A%20%20%20%20%20List%3CUser%3E%20queryUser()%3B%0A%7D%0A%60%60%60%0A**%EF%BC%883%EF%BC%89%E5%A6%82%E6%9E%9C%E6%8E%A5%E5%8F%A3%E4%BD%8D%E4%BA%8E%E4%B8%8D%E5%90%8C%E7%9A%84%E6%A8%A1%E5%9D%97%EF%BC%8C%E9%9C%80%E8%A6%81%E5%9C%A8%E9%A1%B9%E7%9B%AE%E5%90%AF%E5%8A%A8%E7%B1%BB%E9%85%8D%E7%BD%AEbean%E6%89%AB%E6%8F%8F%E8%B7%AF%E5%BE%84**%0A%60%40MapperScan(basePackages%20%3D%20%7B%22com.shuailee.repository.dao%22%7D)%0A%60%0A%0A%23%23%23%23%20%E4%B8%89%20Mybatis%E8%AF%A6%E8%A7%A3%0A%23%23%23%23%23%201%20SQL%E6%98%A0%E5%B0%84%0A%EF%BC%881%EF%BC%89.%20xml%E6%96%B9%E5%BC%8F%0A%60%60%60xml%0A%3C!--%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4%E5%BA%94%E8%AF%A5%E6%98%AF%E5%AF%B9%E5%BA%94%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%8C%85%E5%90%8D%2B%E6%8E%A5%E5%8F%A3%E5%90%8D--%3E%0A%3Cmapper%20namespace%3D%22com.shuailee.dao.IUserDao%22%3E%0A%20%20%20%20%3Cselect%20id%3D%22queryUserByID%22%20parameterType%3D%22int%22%20resultType%3D%22hashmap%22%20%3E%0A%20%20%20%20%20%20%20%20SELECT%20*%20FROM%20tb_user%20WHERE%20user_id%20%3D%20%23%7Bid%7D%0A%20%20%20%20%3C%2Fselect%3E%0A%3C%2Fmapper%3E%0A%60%60%60%0A%0A%E8%BF%99%E4%B8%AA%E8%AF%AD%E5%8F%A5%E8%A2%AB%E7%A7%B0%E4%BD%9C%20queryUserByID%EF%BC%8C%E6%8E%A5%E5%8F%97%E4%B8%80%E4%B8%AA%20int%EF%BC%88%E6%88%96%20Integer%EF%BC%89%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%8F%82%E6%95%B0%EF%BC%8C%E5%B9%B6%E8%BF%94%E5%9B%9E%E4%B8%80%E4%B8%AA%20HashMap%20%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%AF%B9%E8%B1%A1%EF%BC%8C%E5%85%B6%E4%B8%AD%E7%9A%84%E9%94%AE%E6%98%AF%E5%88%97%E5%90%8D%EF%BC%8C%E5%80%BC%E4%BE%BF%E6%98%AF%E7%BB%93%E6%9E%9C%E8%A1%8C%E4%B8%AD%E7%9A%84%E5%AF%B9%E5%BA%94%E5%80%BC%0A%0A%3E1.%20parameterType%20%E5%85%A5%E5%8F%82%E7%B1%BB%E5%9E%8B%0A%3E2.%20resultType%20%20%E8%BF%94%E5%9B%9E%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%EF%BC%8C%E5%A6%82%E6%9E%9C%E8%BF%94%E5%9B%9E%E7%9A%84%E6%98%AF%E9%9B%86%E5%90%88%EF%BC%8C%E9%82%A3%E5%BA%94%E8%AF%A5%E8%AE%BE%E7%BD%AE%E4%B8%BA%E9%9B%86%E5%90%88%E5%8C%85%E5%90%AB%E7%9A%84%E7%B1%BB%E5%9E%8B%EF%BC%8C%E8%80%8C%E4%B8%8D%E6%98%AF%E9%9B%86%E5%90%88%E6%9C%AC%E8%BA%AB%EF%BC%8CresultType%20%2C%20resultMap%EF%BC%8C%E4%B8%8D%E8%83%BD%E5%90%8C%E6%97%B6%E4%BD%BF%E7%94%A8%E3%80%82%0A%3E3.%20resultMap%20%E5%A4%96%E9%83%A8%20resultMap%20%E7%9A%84%E5%91%BD%E5%90%8D%E5%BC%95%E7%94%A8%0A%3E4.%20flushCache%20%E5%B0%86%E5%85%B6%E8%AE%BE%E7%BD%AE%E4%B8%BA%20true%20%E5%90%8E%EF%BC%8C%E5%8F%AA%E8%A6%81%E8%AF%AD%E5%8F%A5%E8%A2%AB%E8%B0%83%E7%94%A8%EF%BC%8C%E9%83%BD%E4%BC%9A%E5%AF%BC%E8%87%B4%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E5%92%8C%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E8%A2%AB%E6%B8%85%E7%A9%BA%EF%BC%8C%E9%BB%98%E8%AE%A4%E5%80%BC%EF%BC%9Afalse%E3%80%82%0A%3E5.%20useCache%20%20%20%E5%B0%86%E5%85%B6%E8%AE%BE%E7%BD%AE%E4%B8%BA%20true%20%E5%90%8E%EF%BC%8C%E5%B0%86%E4%BC%9A%E5%AF%BC%E8%87%B4%E6%9C%AC%E6%9D%A1%E8%AF%AD%E5%8F%A5%E7%9A%84%E7%BB%93%E6%9E%9C%E8%A2%AB%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E7%BC%93%E5%AD%98%E8%B5%B7%E6%9D%A5%EF%BC%8C%E9%BB%98%E8%AE%A4%E5%80%BC%EF%BC%9A%E5%AF%B9%20select%20%E5%85%83%E7%B4%A0%E4%B8%BA%20true%E3%80%82%0A%0A%EF%BC%882%EF%BC%89.%20%E6%B3%A8%E8%A7%A3%E6%96%B9%E5%BC%8F%0A%60%60%60%20java%0A%0A%40Mapper%0Apublic%20interface%20IUserDao%20%7B%0A%20%20%20%20%20%40ResultType(User.class)%0A%20%20%20%20%20%40Select(%22select%20*%20from%20tb_user%22)%0A%20%20%20%20%20List%3CUser%3E%20queryUser()%3B%0A%7D%0A%60%60%60%0A%0A%0A%23%23%23%23%23%202.%20%E5%91%BD%E5%90%8D%E8%A7%A3%E6%9E%90%0AMyBatis%20%E5%AF%B9%E6%89%80%E6%9C%89%E7%9A%84%E5%91%BD%E5%90%8D%E9%85%8D%E7%BD%AE%E5%85%83%E7%B4%A0%EF%BC%88%E5%8C%85%E6%8B%AC%E8%AF%AD%E5%8F%A5%EF%BC%8C%E7%BB%93%E6%9E%9C%E6%98%A0%E5%B0%84%EF%BC%8C%E7%BC%93%E5%AD%98%E7%AD%89%EF%BC%89%E4%BD%BF%E7%94%A8%E4%BA%86%E5%A6%82%E4%B8%8B%E7%9A%84%E5%91%BD%E5%90%8D%E8%A7%A3%E6%9E%90%E8%A7%84%E5%88%99%EF%BC%9A%0A%0A*%20**%E5%AE%8C%E5%85%A8%E9%99%90%E5%AE%9A%E5%90%8D%EF%BC%88%E6%AF%94%E5%A6%82%20%E2%80%9Ccom.shuailee.dao.IUserDao.queryUser%EF%BC%89**%20%E5%B0%86%E8%A2%AB%E7%9B%B4%E6%8E%A5%E7%94%A8%E4%BA%8E%E6%9F%A5%E6%89%BE%E5%8F%8A%E4%BD%BF%E7%94%A8%E3%80%82%E5%88%A9%E7%94%A8%E6%9B%B4%E9%95%BF%E7%9A%84%E5%AE%8C%E5%85%A8%E9%99%90%E5%AE%9A%E5%90%8D%E6%9D%A5%E5%B0%86%E4%B8%8D%E5%90%8C%E7%9A%84%E8%AF%AD%E5%8F%A5%E9%9A%94%E7%A6%BB%E5%BC%80%E6%9D%A5%EF%BC%8C%E5%90%8C%E6%97%B6%E4%B9%9F%E5%AE%9E%E7%8E%B0%E4%BA%86%E6%8E%A5%E5%8F%A3%E7%BB%91%E5%AE%9A%0A*%20**%E7%9F%AD%E5%90%8D%E7%A7%B0%EF%BC%88%E6%AF%94%E5%A6%82%20%E2%80%9CqueryUser%E2%80%9D%EF%BC%89**%20%E5%A6%82%E6%9E%9C%E5%85%A8%E5%B1%80%E5%94%AF%E4%B8%80%E4%B9%9F%E5%8F%AF%E4%BB%A5%E4%BD%9C%E4%B8%BA%E4%B8%80%E4%B8%AA%E5%8D%95%E7%8B%AC%E7%9A%84%E5%BC%95%E7%94%A8%E3%80%82%20%E5%A6%82%E6%9E%9C%E4%B8%8D%E5%94%AF%E4%B8%80%EF%BC%8C%E6%9C%89%E4%B8%A4%E4%B8%AA%E6%88%96%E4%B8%A4%E4%B8%AA%E4%BB%A5%E4%B8%8A%E7%9A%84%E7%9B%B8%E5%90%8C%E5%90%8D%E7%A7%B0%EF%BC%88%E6%AF%94%E5%A6%82%20%E2%80%9Ccom.foo.selectAllThings%E2%80%9D%20%E5%92%8C%20%E2%80%9Ccom.bar.selectAllThings%E2%80%9D%EF%BC%89%EF%BC%8C%E9%82%A3%E4%B9%88%E4%BD%BF%E7%94%A8%E6%97%B6%E5%B0%B1%E4%BC%9A%E4%BA%A7%E7%94%9F%E2%80%9C%E7%9F%AD%E5%90%8D%E7%A7%B0%E4%B8%8D%E5%94%AF%E4%B8%80%E2%80%9D%E7%9A%84%E9%94%99%E8%AF%AF%EF%BC%8C%E8%BF%99%E7%A7%8D%E6%83%85%E5%86%B5%E4%B8%8B%E5%B0%B1%E5%BF%85%E9%A1%BB%E4%BD%BF%E7%94%A8%E5%AE%8C%E5%85%A8%E9%99%90%E5%AE%9A%E5%90%8D%E3%80%82%0A%23%23%23%23%23%203.%20%E6%98%A0%E5%B0%84%E5%99%A8%EF%BC%88IUserDao%E6%8E%A5%E5%8F%A3%EF%BC%89%E3%80%90%E9%87%8D%E8%A6%81%E3%80%91%0A%E6%98%AF%E4%B8%80%E4%BA%9B%E7%94%B1%E4%BD%A0%E5%88%9B%E5%BB%BA%E7%9A%84%E3%80%81%E7%BB%91%E5%AE%9A%E4%BD%A0%E6%98%A0%E5%B0%84%E7%9A%84%E8%AF%AD%E5%8F%A5%E7%9A%84%E6%8E%A5%E5%8F%A3%E3%80%82%E6%98%A0%E5%B0%84%E5%99%A8%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%AE%9E%E4%BE%8B%E6%98%AF%E4%BB%8E%20SqlSession%20%E4%B8%AD%E8%8E%B7%E5%BE%97%E7%9A%84%EF%BC%8C%E5%AE%83%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%8F%AF%E4%BB%A5%E5%90%8CSqlSession%20%E7%9B%B8%E5%90%8C%E5%B1%9E%E4%BA%8E%E7%BA%BF%E7%A8%8B%EF%BC%8C%E8%99%BD%E7%84%B6%E5%9C%A8%E6%95%B4%E4%B8%AA%E8%AF%B7%E6%B1%82%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%BF%9D%E6%8C%81%E6%98%A0%E5%B0%84%E5%99%A8%E5%AE%9E%E4%BE%8B%E4%B8%8D%E4%BC%9A%E6%9C%89%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98%EF%BC%8C%E4%BD%86%E6%98%AF%E8%BF%99%E4%BC%9A%E8%AE%A9%E4%BD%9C%E7%94%A8%E5%9F%9F%E7%AE%A1%E7%90%86%E5%A4%AA%E5%A4%9A%E8%B5%84%E6%BA%90%EF%BC%8C%E6%8F%90%E5%8D%87%E4%BA%86%E5%A4%8D%E6%9D%82%E6%80%A7%E3%80%82%E4%B8%BA%E4%BA%86%E9%81%BF%E5%85%8D%E8%BF%99%E7%A7%8D%E5%A4%8D%E6%9D%82%E6%80%A7%EF%BC%8C%E6%9C%80%E5%A5%BD%E6%8A%8A%E6%98%A0%E5%B0%84%E5%99%A8%E6%94%BE%E5%9C%A8%E6%96%B9%E6%B3%95%E4%BD%9C%E7%94%A8%E5%9F%9F%E5%86%85%EF%BC%8C%E5%9C%A8%E6%96%B9%E6%B3%95%E6%89%A7%E8%A1%8C%E6%97%B6%E8%A2%AB%E5%88%9B%E5%BB%BA%EF%BC%8C%E7%94%A8%E8%BF%87%E4%B9%8B%E5%90%8E%E5%8D%B3%E5%8F%AF%E4%B8%A2%E5%BC%83%0A%0A**%E3%80%90%E9%87%8D%E8%A6%81%E3%80%91%E6%8E%A5%E5%8F%A3%E6%98%A0%E5%B0%84%E7%9A%84%E5%8E%9F%E7%90%86%EF%BC%9A**%0A%0A1.%20Dao%E6%8E%A5%E5%8F%A3%E5%8D%B3%20Mapper%E6%98%A0%E5%B0%84%E6%8E%A5%E5%8F%A3%E3%80%82%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%AE%8C%E5%85%A8%E9%99%90%E5%AE%9A%E5%90%8D%E5%B0%B1%E6%98%AF%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84namespace%E7%9A%84%E5%80%BC%E3%80%82%E6%8E%A5%E5%8F%A3%E7%9A%84%E6%96%B9%E6%B3%95%E5%90%8D%E5%B0%B1%E6%98%AF%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6%E4%B8%AD%20Mapper%20%E7%9A%84%20Statement%20%E7%9A%84%20id%20%E5%80%BC%EF%BC%9B%E6%98%A0%E5%B0%84%E6%8E%A5%E5%8F%A3%E5%86%85%E7%9A%84Mapper%E6%8E%A5%E5%8F%A3%E6%98%AF%E6%B2%A1%E6%9C%89%E5%AE%9E%E7%8E%B0%E7%B1%BB%E7%9A%84%EF%BC%8C%E5%BD%93%E8%B0%83%E7%94%A8%E6%8E%A5%E5%8F%A3%E6%96%B9%E6%B3%95%E6%97%B6%E6%8E%A5%E5%8F%A3%E5%AE%8C%E5%85%A8%E9%99%90%E5%AE%9A%E5%90%8D%2B%E6%96%B9%E6%B3%95%E5%90%8D%E4%BC%9A%E6%8B%BC%E6%8E%A5%E6%88%90%E4%B8%80%E4%B8%AA%20key%20%E5%80%BC%EF%BC%8C%E5%8F%AF%E5%94%AF%E4%B8%80%E5%AE%9A%E4%BD%8D%E4%B8%80%E4%B8%AA%20MapperStatement%E3%80%82%0A2.%20%E5%9C%A8%20Mybatis%20%E4%B8%AD%EF%BC%8C%E6%AF%8F%E4%B8%80%E4%B8%AA%3Cselect%3E%E3%80%81%3Cinsert%3E%E3%80%81%3Cupdate%3E%E3%80%81%3Cdelete%3E%20%E6%A0%87%E7%AD%BE%EF%BC%8C%E9%83%BD%E4%BC%9A%E8%A2%AB%E8%A7%A3%E6%9E%90%E4%B8%BA%E4%B8%80%E4%B8%AAMapperStatement%20%E5%AF%B9%E8%B1%A1%E3%80%82%E8%80%8C%E5%8F%82%E6%95%B0%E5%B0%B1%E6%98%AF%E4%BC%A0%E9%80%92%E7%BB%99%20sql%20%E7%9A%84%E5%8F%82%E6%95%B0%E3%80%82%0A3.%20%E4%BE%8B%E5%A6%82%E9%80%9A%E8%BF%87%60com.shuailee.dao.IUserDao.queryUserByID%60%E5%8F%AF%E4%BB%A5%E6%89%BE%E5%88%B0namespace%E4%B8%BA%60com.shuailee.dao.IUserDao%60%E7%9A%84%E4%B8%8B%E9%9D%A2id%E4%B8%BAqueryUserByID%E7%9A%84MapperStatement%20%0A4.%20Mapper%20%E6%8E%A5%E5%8F%A3%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E6%98%AF%20JDK%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%EF%BC%8CMybatis%20%E8%BF%90%E8%A1%8C%E6%97%B6%E4%BC%9A%E4%BD%BF%E7%94%A8%20JDK%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E4%B8%BA%20Mapper%20%E6%8E%A5%E5%8F%A3%E7%94%9F%E6%88%90%E4%BB%A3%E7%90%86%E5%AF%B9%E8%B1%A1%20proxy%EF%BC%8C%E4%BB%A3%E7%90%86%E5%AF%B9%E8%B1%A1%E4%BC%9A%E6%8B%A6%E6%88%AA%E6%8E%A5%E5%8F%A3%E6%96%B9%E6%B3%95%EF%BC%8C%E8%BD%AC%E8%80%8C%E6%89%A7%E8%A1%8C%20MapperStatement%20%E6%89%80%E4%BB%A3%E8%A1%A8%E7%9A%84%20sql%EF%BC%8C%E7%84%B6%E5%90%8E%E5%B0%86%20sql%20%E6%89%A7%E8%A1%8C%E7%BB%93%E6%9E%9C%E8%BF%94%E5%9B%9E%E3%80%82%0A%0A%23%23%23%23%23%204%20%E5%8A%A8%E6%80%81sql%0A%E5%8A%A8%E6%80%81sql%E5%B0%B1%E6%98%AF%E6%94%AF%E6%8C%81%E9%80%BB%E8%BE%91%E8%AF%AD%E6%B3%95%E7%9A%84sql%EF%BC%8C%E4%B8%BB%E8%A6%81%E7%94%A8%E6%9D%A5%E8%A7%A3%E5%86%B3%E4%B8%8D%E5%90%8C%E6%9D%A1%E4%BB%B6%E4%B8%8B%E6%8B%BC%E6%8E%A5sql%E8%AF%AD%E5%8F%A5%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%9A%E4%BE%8B%E5%A6%82%E6%8B%BC%E6%8E%A5%E6%97%B6%E8%A6%81%E7%A1%AE%E4%BF%9D%E4%B8%8D%E8%83%BD%E5%BF%98%E8%AE%B0%E6%B7%BB%E5%8A%A0%E5%BF%85%E8%A6%81%E7%9A%84%E7%A9%BA%E6%A0%BC%EF%BC%8C%E8%BF%98%E8%A6%81%E6%B3%A8%E6%84%8F%E5%8E%BB%E6%8E%89%E5%88%97%E8%A1%A8%E6%9C%80%E5%90%8E%E4%B8%80%E4%B8%AA%E5%88%97%E5%90%8D%E7%9A%84%E9%80%97%E5%8F%B7%E7%AD%89%E3%80%82MyBatis%20%E9%87%87%E7%94%A8%E5%8A%9F%E8%83%BD%E5%BC%BA%E5%A4%A7%E7%9A%84%E5%9F%BA%E4%BA%8E%20OGNL%20%E7%9A%84%E8%A1%A8%E8%BE%BE%E5%BC%8F%E6%9D%A5%E6%B7%98%E6%B1%B0%E5%85%B6%E5%AE%83%E5%A4%A7%E9%83%A8%E5%88%86%E5%85%83%E7%B4%A0%E3%80%82%0A%60%E4%B8%BB%E8%A6%81%E8%AF%AD%E6%B3%95%EF%BC%9Aif%20%20%20choose%20(when%2C%20otherwise)%20%20%20trim%20(where%2C%20set)%20%20%20foreach%60%0A%23%23%23%23%23%23%20**(1)%20%E5%A4%9A%E6%9D%A1%E4%BB%B6%E6%9F%A5%E8%AF%A2%E5%88%A4%E6%96%AD%20if**%0A%60%60%60%20xml%0A%3Cselect%20id%3D%22findActiveBlogWithTitleLike%22%20%20%20resultType%3D%22Blog%22%3E%0A%20%20SELECT%20*%20FROM%20BLOG%20%20%20WHERE%20state%20%3D%20%E2%80%98ACTIVE%E2%80%99%0A%20%20%3C!--%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%9C%89%E4%BC%A0%E5%85%A5%E2%80%9Ctitle%E2%80%9D%E5%88%99%E4%B8%8D%E6%8B%BC%E6%8E%A5title%E6%9F%A5%E8%AF%A2%E6%9D%A1%E4%BB%B6%EF%BC%9A--%3E%0A%20%20%3Cif%20test%3D%22title%20!%3D%20null%22%3E%0A%20%20%20%20AND%20title%20like%20%23%7Btitle%7D%0A%20%20%3C%2Fif%3E%0A%3C%2Fselect%3E%0A%60%60%60%0A%23%23%23%23%23%23%20**(2)%20%E5%A4%9A%E6%9D%A1%E4%BB%B6%E6%9F%A5%E8%AF%A2%E5%88%A4%E6%96%AD%E8%BF%9E%E7%BB%AD%E6%8B%BC%E6%8E%A5%20if%2Bwhere**%0A%E2%80%9Cwhere%E2%80%9D%E6%A0%87%E7%AD%BE%E5%A6%82%E6%9E%9C%E5%AE%83%E5%8C%85%E5%90%AB%E7%9A%84%E6%A0%87%E7%AD%BE%E4%B8%AD%E6%9C%89%E8%BF%94%E5%9B%9E%E5%80%BC%E7%9A%84%E8%AF%9D%EF%BC%8C%E5%AE%83%E5%B0%B1%E6%8F%92%E5%85%A5%E4%B8%80%E4%B8%AA%E2%80%98where%E2%80%99%2C%E5%A6%82%E6%9E%9C%E6%A0%87%E7%AD%BE%E8%BF%94%E5%9B%9E%E7%9A%84%E5%86%85%E5%AE%B9%E6%98%AF%E4%BB%A5AND%E6%88%96OR%E5%BC%80%E5%A4%B4%E7%9A%84%EF%BC%8C%E5%88%99%E4%BC%9A%E5%89%94%E9%99%A4%E6%8E%89AND%20%E6%88%96OR%0A%60%60%60xml%0A%3Cselect%20id%3D%22queryUserinfo2%22%20parameterType%3D%22java.util.Map%22%20%20resultType%3D%22com.shuailee.model.UserModel%22%3E%0A%20%20%20%20SELECT%20*%20FROM%20tb_user%0A%20%20%20%20%3Cwhere%3E%0A%20%20%20%20%20%20%20%20%3Cif%20test%3D%22sex%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20AND%20user_sex%20%3D%20%23%7Bsex%7D%0A%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%3Cif%20test%3D%22name%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20AND%20user_name%20like%20%23%7Bname%7D%0A%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%3C%2Fwhere%3E%0A%3C%2Fselect%3E%0A%60%60%60%0A%23%23%23%23%23%23%20**(3)%20choose%20when%E6%A0%87%E7%AD%BE%2C%E5%BD%93%E6%88%91%E4%BB%AC%E5%80%BC%E9%9C%80%E8%A6%81%E5%A4%9A%E4%B8%AA%E6%9D%A1%E4%BB%B6%E4%B8%AD%E7%9A%84%E4%B8%80%E4%B8%AA%E6%97%B6%E4%BD%BF%E7%94%A8**%0A%60%60%60xml%0A%3C!--%0A%E5%A6%82%E6%9E%9C%20sex%20%E4%B8%8D%E4%B8%BA%E7%A9%BA%EF%BC%8C%E9%82%A3%E4%B9%88%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E4%B8%BA%EF%BC%9Aselect%20*%20from%20user%20where%20user_sex%3D%3F%0A%E5%A6%82%E6%9E%9C%20sex%20%E4%B8%BA%E7%A9%BA%EF%BC%8Cname%20%E6%98%AF%E5%90%A6%E4%B8%BA%E7%A9%BA%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%8D%E4%B8%BA%E7%A9%BA%EF%BC%8C%E9%82%A3%E4%B9%88%E8%AF%AD%E5%8F%A5%E4%B8%BA%20select%20*%20from%20user%20where%20user_name%3D%3F%3B%0A%E5%A6%82%E6%9E%9C%20sex%20%E4%B8%BA%E7%A9%BA%EF%BC%8Cname%20%E4%B8%BA%E7%A9%BA%EF%BC%8C%E9%82%A3%E4%B9%88%E8%AF%AD%E5%8F%A5%E4%B8%BA%20select%20*%20from%20user%20where%20user_email%3D%3F%3B%0A%0A--%3E%0A%3Cselect%20id%3D%22queryUserinfo3%22%20parameterType%3D%22java.util.Map%22%20resultType%3D%22com.shuailee.model.UserModel%22%3E%0A%20%20%20%20SELECT%20*%20FROM%20tb_user%0A%20%20%20%20%3Cwhere%3E%0A%20%20%20%20%20%20%20%20%3Cchoose%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cwhen%20test%3D%22sex%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20AND%20user_sex%20%3D%20%23%7Bsex%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fwhen%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cwhen%20test%3D%22name%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20AND%20user_name%20like%20%23%7Bname%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fwhen%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cotherwise%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20and%20user_email%3D%23%7Bemail%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fotherwise%3E%0A%20%20%20%20%20%20%20%20%3C%2Fchoose%3E%0A%20%20%20%20%3C%2Fwhere%3E%0A%3C%2Fselect%3E%0A%60%60%60%0A%0A%23%23%23%23%23%23%20**(4)%20%E5%A4%9A%E5%AD%97%E6%AE%B5%E6%9B%B4%E6%96%B0%E5%88%A4%E6%96%AD%20update%20if**%0A%60%60%60xml%0A%3Cupdate%20id%3D%22updateUserById%22%20parameterType%3D%22com.shuailee.model.UserModel%22%3E%0A%20update%20tb_user%0A%20%20%20%20%20%3Cset%3E%0A%20%20%20%20%20%20%20%20%20%3Cif%20test%3D%22userName!%3Dnull%20and%20userName!%3D''%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20user_name%20%3D%20%23%7BuserName%7D%2C%0A%20%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%20%3Cif%20test%3D%22userSex!%3Dnull%20and%20userSex!%3D''%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20user_sex%20%3D%20%23%7BuserSex%7D%0A%20%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%3C%2Fset%3E%0A%20%20%20%20%20%20%20%20where%20user_id%20%3D%20%23%7BuserId%7D%0A%3C%2Fupdate%3E%0A%60%60%60%0A%23%23%23%23%23%23%20**(5)%20trim%20%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%A0%87%E8%AE%B0**%0A%0A**%E5%AE%8C%E6%88%90where%E6%A0%87%E8%AE%B0%E7%9A%84%E5%8A%9F%E8%83%BD%EF%BC%9A**%0A%0A*%20prefix%EF%BC%9A%E5%89%8D%E7%BC%80%E8%BF%BD%E5%8A%A0%0A*%20prefixoverride%EF%BC%9A%E5%8E%BB%E6%8E%89%E7%AC%AC%E4%B8%80%E4%B8%AAand%E6%88%96%E8%80%85%E6%98%AFor%0A%60%60%60xml%0A%3Cselect%20id%3D%22queryUserinfo4%22%20parameterType%3D%22java.util.Map%22%20resultType%3D%22com.shuailee.model.UserModel%22%3E%0A%20%20%20%20SELECT%20*%20FROM%20tb_user%0A%20%20%20%20%3Ctrim%20prefix%3D%22where%22%20prefixOverrides%3D%22and%20%7C%20or%22%3E%0A%20%20%20%20%20%20%20%20%3Cif%20test%3D%22sex%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20AND%20user_sex%20%3D%20%23%7Bsex%7D%0A%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%3Cif%20test%3D%22name%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20AND%20user_name%20like%20%23%7Bname%7D%0A%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%3C%2Ftrim%3E%0A%3C%2Fselect%3E%0A%60%60%60%0A**%E5%AE%8C%E6%88%90set%E6%A0%87%E8%AE%B0%E7%9A%84%E5%8A%9F%E8%83%BD**%0A%0A*%20suffix%EF%BC%9A%E5%90%8E%E7%BC%80%E8%BF%BD%E5%8A%A0%E3%80%80%E3%80%80%0A*%20suffixoverride%EF%BC%9A%E5%8E%BB%E6%8E%89%E6%9C%80%E5%90%8E%E4%B8%80%E4%B8%AA%E9%80%97%E5%8F%B7%EF%BC%88%E4%B9%9F%E5%8F%AF%E4%BB%A5%E6%98%AF%E5%85%B6%E4%BB%96%E7%9A%84%E6%A0%87%E8%AE%B0%EF%BC%8C%E5%B0%B1%E5%83%8F%E6%98%AF%E4%B8%8A%E9%9D%A2%E5%89%8D%E7%BC%80%E4%B8%AD%E7%9A%84and%E4%B8%80%E6%A0%B7%EF%BC%89%0A%0A%60%60%60xml%0A%3Cupdate%20id%3D%22updateUserById2%22%20parameterType%3D%22com.shuailee.model.UserModel%22%3E%0A%20%20%20%20update%20tb_user%0A%20%20%20%20%3Ctrim%20prefix%3D%22set%22%20suffixOverrides%3D%22%2C%22%3E%0A%20%20%20%20%20%20%20%20%3Cif%20test%3D%22userName!%3Dnull%20and%20userName!%3D''%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20user_name%20%3D%20%23%7BuserName%7D%2C%0A%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%3Cif%20test%3D%22userSex!%3Dnull%20and%20userSex!%3D''%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20user_sex%20%3D%20%23%7BuserSex%7D%0A%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%3C%2Ftrim%3E%0A%20%20%20%20where%20user_id%20%3D%20%23%7BuserId%7D%0A%3C%2Fupdate%3E%0A%60%60%60%0A%23%23%23%23%23%23%20**(6)%20%20include%E5%91%BD%E4%BB%A4%20%E5%AE%9A%E4%B9%89sql%E7%89%87%E6%AE%B5%E5%9C%A8%E5%88%AB%E7%9A%84%E5%9C%B0%E6%96%B9%E5%BC%95%E7%94%A8%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%A2%9E%E5%8A%A0%E4%BB%A3%E7%A0%81%E7%9A%84%E9%87%8D%E7%94%A8%E6%80%A7**%0A%60%60%60xml%0A%3C!--%E5%AE%9A%E4%B9%89%E4%B8%80%E4%B8%AA%E6%9D%A1%E4%BB%B6%E7%AD%9B%E9%80%89%E7%9A%84%E5%88%A4%E6%96%AD--%3E%0A%3Csql%20id%3D%22selectUserByPara%22%3E%0A%20%20%20%20%3Cif%20test%3D%22userName!%3Dnull%20and%20userName!%3D''%22%3E%0A%20%20%20%20%20%20%20%20user_name%20%3D%20%23%7BuserName%7D%2C%0A%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%3Cif%20test%3D%22userSex!%3Dnull%20and%20userSex!%3D''%22%3E%0A%20%20%20%20%20%20%20%20user_sex%20%3D%20%23%7BuserSex%7D%0A%20%20%20%20%3C%2Fif%3E%0A%3C%2Fsql%3E%0A%0A%3C!--%E4%BF%AE%E6%94%B9%E5%BC%95%E7%94%A8--%3E%0A%3Cupdate%20id%3D%22updateUserById3%22%20parameterType%3D%22com.shuailee.model.UserModel%22%3E%0A%20%20%20%20update%20tb_user%0A%20%20%20%20%3Ctrim%20prefix%3D%22set%22%20suffixOverrides%3D%22%2C%22%3E%0A%20%20%20%20%20%20%20%20%3C!--%20%E5%BC%95%E7%94%A8%20sql%20%E7%89%87%E6%AE%B5%EF%BC%8C%E5%A6%82%E6%9E%9Crefid%20%E6%8C%87%E5%AE%9A%E7%9A%84%E4%B8%8D%E5%9C%A8%E6%9C%AC%E6%96%87%E4%BB%B6%E4%B8%AD%EF%BC%8C%0A%20%20%20%20%20%20%20%20%E9%82%A3%E4%B9%88%E9%9C%80%E8%A6%81%E5%9C%A8%E5%89%8D%E9%9D%A2%E5%8A%A0%E4%B8%8A%20namespace%20--%3E%0A%20%20%20%20%20%20%20%20%3Cinclude%20refid%3D%22selectUserByPara%22%3E%3C%2Finclude%3E%0A%20%20%20%20%3C%2Ftrim%3E%0A%20%20%20%20where%20user_id%20%3D%20%23%7BuserId%7D%0A%3C%2Fupdate%3E%0A%60%60%60%0A%0A%23%23%23%23%23%23%20**(7)%20foreach%20%E6%A0%87%E8%AE%B0**%0A%0A%60%60%60xml%0A%3C!--%0Acollection%3A%E6%8C%87%E5%AE%9A%E8%BE%93%E5%85%A5%E5%AF%B9%E8%B1%A1%E4%B8%AD%E7%9A%84%E9%9B%86%E5%90%88%E5%B1%9E%E6%80%A7%0Aitem%3A%E6%AF%8F%E6%AC%A1%E9%81%8D%E5%8E%86%E7%94%9F%E6%88%90%E7%9A%84%E5%AF%B9%E8%B1%A1%0Aopen%3A%E5%BC%80%E5%A7%8B%E9%81%8D%E5%8E%86%E6%97%B6%E7%9A%84%E6%8B%BC%E6%8E%A5%E5%AD%97%E7%AC%A6%E4%B8%B2%0Aclose%3A%E7%BB%93%E6%9D%9F%E6%97%B6%E6%8B%BC%E6%8E%A5%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2%0Aseparator%3A%E9%81%8D%E5%8E%86%E5%AF%B9%E8%B1%A1%E4%B9%8B%E9%97%B4%E9%9C%80%E8%A6%81%E6%8B%BC%E6%8E%A5%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2%0A--%3E%0A%3C!--1%20%E5%A4%9A%E8%A1%8C%E6%8F%92%E5%85%A5--%3E%0A%3Cinsert%20id%3D%22batchInsertUser2%22%20parameterType%3D%22com.shuailee.model.User%22%0A%20%20%20%20%20%20%20%20useGeneratedKeys%3D%22true%22%20keyProperty%3D%22user_id%22%3E%0A%20%20%20%20insert%20into%20tb_user%20(user_name%2Cuser_sex%2Cuser_birthday%2Cuser_address%2Cuser_email)%20%0A%20%20%20%20values%0A%20%20%20%20%3Cforeach%20item%3D%22item%22%20collection%3D%22list%22%20separator%3D%22%2C%22%3E%0A%20%20%20%20%20%20%20%20(%23%7Bitem.user_name%7D%2C%20%23%7Bitem.user_sex%7D%2C%20%23%7Bitem.user_birthday%7D%2C%20%23%7Bitem.user_address%7D%2C%20%23%7Bitem.user_email%7D)%0A%20%20%20%20%3C%2Fforeach%3E%0A%3C%2Finsert%3E%0A%0A%E6%9C%80%E7%BB%88%E5%BE%97%E5%88%B0sql%E8%AF%AD%E5%8F%A5%EF%BC%9A%0A%0Ainsert%20into%20redeem_code%20(batch_id%2C%20code%2C%20type%2C%20facevalue%2Ccreate_user%2Ccreate_time)%20values%20(%3F%2C%3F%2C%3F%2C%3F%2C%3F%2C%3F%20)%2C(%3F%2C%3F%2C%3F%2C%3F%2C%3F%2C%3F%20)%2C(%3F%2C%3F%2C%3F%2C%3F%2C%3F%2C%3F%20)%2C(%3F%2C%3F%2C%3F%2C%3F%2C%3F%2C%3F%20)%0A%0A%60%60%60%0A%0A%23%23%23%23%23%23%20**(8)%20%20%E6%8B%BC%E6%8E%A5OR%E6%9F%A5%E8%AF%A2%E6%9D%A1%E4%BB%B6**%0A%60%60%60%20xml%0A%3C!--2%20%E6%88%91%E4%BB%AC%E7%94%A8%20foreach%20%E6%9D%A5%E6%94%B9%E5%86%99%20select%20*%20from%20user%20where%20id%3D1%20or%20id%3D2%20or%20id%3D3%0A%E6%B3%A8%E6%84%8F%EF%BC%9A%0AparameterType%E5%8F%82%E6%95%B0%E7%B1%BB%E5%9E%8B%E4%B8%BAList%20%20collection%3D%22list%22%0AparameterType%E5%8F%82%E6%95%B0%E7%B1%BB%E5%9E%8B%E4%B8%BAArrayList%20%20collection%3D%22values%5B%5D%22%0A--%3E%0A%3Cselect%20id%3D%22queryUserbyids%22%20resultType%3D%22com.shuailee.model.User%22%0A%20parameterType%3D%22java.util.List%22%3E%0A%20%20%20%20SELECT%20*%20FROM%20tb_user%0A%20%20%20%20%3Cwhere%3E%0A%20%20%20%20%20%20%20%20%3Cforeach%20collection%3D%22list%22%20item%3D%22id%22%20open%3D%22%20(%22%20close%3D%22)%22%0A%20%20%20%20%20%20%20%20%20separator%3D%22or%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20user_id%20%3D%20%23%7Bid%7D%0A%20%20%20%20%20%20%20%20%3C%2Fforeach%3E%0A%20%20%20%20%3C%2Fwhere%3E%0A%3C%2Fselect%3E%0A%60%60%60%0A%23%23%23%23%23%23%20**(9)%20in%E6%9F%A5%E8%AF%A2**%0A**%E6%8E%A5%E5%8F%A3%EF%BC%9A**%60int%20selectInformationAppid(%40Param(%22appid%22)%20String%20appid%2C%40Param(%22infoSourceList%22)List%20infoSourceList)%3B%60%0A%60%60%60xml%0A%3Cselect%20id%3D%22selectInformationAppid%22%20resultType%3D%22java.lang.Integer%22%3E%0A%20%20%20%20select%20count(1)%20from%20info_source%20where%20app_id%20%3D%20%23%7Bappid%7D%20and%20information_source%0A%20%20%20%20in%0A%20%20%20%20%3Cforeach%20item%3D%22item%22%20%20collection%3D%22infoSourceList%22%20separator%3D%22%2C%22%20open%3D%22(%22%20%20close%3D%22)%22%3E%0A%20%20%20%20%20%20%20%20%23%7Bitem.informationSource%2CjdbcType%3DVARCHAR%7D%0A%20%20%20%20%3C%2Fforeach%3E%0A%3C%2Fselect%3E%0A%60%60%60%0A%0A%23%23%23%23%23%23%20**(10)%20%20%E6%89%B9%E9%87%8F%E6%9B%B4%E6%96%B0%EF%BC%9A%E9%9C%80%E8%A6%81%E5%9C%A8%E8%BF%9E%E6%8E%A5%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%B8%AD%E5%8A%A0%E5%85%A5%20%26allowMultiQueries%3Dtrue%20%E5%BC%80%E5%90%AF%E6%89%B9%E9%87%8F%E8%AF%AD%E5%8F%A5%E6%89%A7%E8%A1%8C**%0A%60%60%60sh%0A%23%20Druid%20%E6%95%B0%E6%8D%AE%E6%BA%90%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE%20spring.datasource.url%20%3D%20jdbc%5C%3Amysql%5C%3A%2F%2F192.168.5.15%5C%3A3306%2Fmiddleware%3FuseUnicode%5C%3Dtrue%26characterEncoding%5C%3Dutf8%26serverTimezone%5C%3DGMT%252B8%26zeroDateTimeBehavior%5C%3DconvertToNull%26allowMultiQueries%5C%3Dtrue%0A%0A%2F%2F%20%E6%8E%A5%E5%8F%A3%E5%AE%9A%E4%B9%89%0Aint%20batchUpdate(List%20infoSourceList)%3B%0A%60%60%60%0A**sql%E6%98%A0%E5%B0%84%EF%BC%9A**%0A%60%60%60xml%0A%3Cupdate%20id%3D%22batchUpdate%22%20parameterType%3D%22com.web.entity.InfoSource%22%3E%0A%0A%20%20%20%20%3Cforeach%20item%3D%22item%22%20collection%3D%22list%22%20separator%3D%22%3B%22%3E%0A%20%20%20%20%20%20%20%20update%20info_source%0A%20%20%20%20%20%20%20%20%3Cset%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cif%20test%3D%22item.informationSource%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20information_source%20%3D%20%23%7Bitem.informationSource%2CjdbcType%3DVARCHAR%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cif%20test%3D%22item.informationSourceName%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20information_source_name%20%3D%20%23%7Bitem.informationSourceName%2CjdbcType%3DVARCHAR%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cif%20test%3D%22item.informationAppid%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20information_appid%20%3D%20%23%7Bitem.informationAppid%2CjdbcType%3DVARCHAR%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cif%20test%3D%22item.informationAppkey%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20information_appkey%20%3D%20%23%7Bitem.informationAppkey%2CjdbcType%3DVARCHAR%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cif%20test%3D%22item.updateUser%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20update_user%20%3D%20%23%7Bitem.updateUser%2CjdbcType%3DVARCHAR%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%3Cif%20test%3D%22item.delete%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%60delete%60%20%3D%20%23%7Bitem.delete%2CjdbcType%3DBIT%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%20%20%20%20%3C%2Fset%3E%0A%20%20%20%20%20%20%20%20where%20id%20%3D%20%23%7Bitem.id%2CjdbcType%3DINTEGER%7D%0A%20%20%20%20%3C%2Fforeach%3E%0A%3C%2Fupdate%3E%0A%60%60%60%0A%0A%23%23%23%23%23%23%20**(11)%20%20MyBatis%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2**%0A%0A**1.%20%E7%9B%B4%E6%8E%A5%E4%BC%A0%E5%8F%82**%0A%60%60%60%20java%0Apublic%20void%20selectBykeyWord(String%20keyword)%20%7B%0A%20%20%20%20%20String%20roleName%20%3D%20%22%25%22%20%2B%20keyword%20%2B%20%22%25%22%3B%0A%20%20%20%20%20userDao.selectBykeyWord(roleName)%3B%0A%20%7D%0A%2F%2F%20%E6%8E%A5%E5%8F%A3%E6%98%A0%E5%B0%84%E5%AE%9A%E4%B9%89%0AList%3CRoleEntity%3E%20selectBykeyWord(%40Param(%22roleName%22)%20String%20roleName)%3B%0A%0A%60%60%60%0A%60%60%60xml%0A%3Cselect%20id%3D%22selectBykeyWord%22%20parameterType%3D%22string%22%20resultType%3D%22com.why.mybatis.entity.RoleEntity%22%3E%0A%09SELECT%20*%20FROM%20t_role%0A%09WHERE%0A%09%20%20%20%20role_name%20LIKE%20%23%7BroleName%7D%0A%3C%2Fselect%3E%0A%0A%60%60%60%0A%0A**2.%20CONCAT%EF%BC%88%EF%BC%89%E5%87%BD%E6%95%B0**%0AMySQL%E7%9A%84%20CONCAT%EF%BC%88%EF%BC%89%E5%87%BD%E6%95%B0%E7%94%A8%E4%BA%8E%E5%B0%86%E5%A4%9A%E4%B8%AA%E5%AD%97%E7%AC%A6%E4%B8%B2%E8%BF%9E%E6%8E%A5%E6%88%90%E4%B8%80%E4%B8%AA%E5%AD%97%E7%AC%A6%E4%B8%B2%EF%BC%8C%E6%98%AF%E6%9C%80%E9%87%8D%E8%A6%81%E7%9A%84mysql%E5%87%BD%E6%95%B0%E4%B9%8B%E4%B8%80%0A%0A%60%20List%3CRoleEntity%3E%20selectBykeyWord(%40Param(%22keyword%22)%20String%20keyword)%3B%60%0A%0A%60%60%60xml%0A%3Cselect%20id%3D%22selectBykeyWord%22%20parameterType%3D%22string%22%20resultType%3D%22com.why.mybatis.entity.RoleEntity%22%3E%0A%20%20%20%20%20%20%20%20SELECT%20%20*%20FROM%20t_role%0A%20%20%20%20%20%20%20%20WHERE%0A%20%20%20%20%20%20%20%20%20%20%20%20role_name%20LIKE%20CONCAT('%25'%2C%23%7Bkeyword%7D%2C'%25')%0A%20%20%20%20%3C%2Fselect%3E%0A%0A%60%60%60%0A%0A%0A%23%23%23%23%23%23%20**(12)%20%20%E6%89%B9%E9%87%8F%E6%8F%92%E5%85%A5%E5%8E%BB%E9%87%8D**%0A%E5%8F%82%E8%80%83%EF%BC%9Ahttps%3A%2F%2Fwww.cnblogs.com%2Fbetter-farther-world2099%2Farticles%2F11737376.html%0A%E5%AF%B9%E5%A4%A7%E9%87%8F%E7%9A%84%E6%95%B0%E6%8D%AE%E8%BF%9B%E8%A1%8C%E5%86%99%E5%85%A5%E6%95%B0%E6%8D%AE%E5%BA%93%E6%93%8D%E4%BD%9C%E6%97%B6%EF%BC%8C%E4%BC%9A%E5%87%BA%E7%8E%B0%E8%BF%99%E6%A0%B7%E7%9A%84%E9%97%AE%E9%A2%98%EF%BC%9A%E5%AE%8C%E5%85%A8%E4%B8%8D%E4%B8%80%E6%A0%B7%EF%BC%8C%E4%B8%80%E9%83%A8%E5%88%86%E4%B8%8D%E4%B8%80%E6%A0%B7%E3%80%82%E6%88%91%E4%BB%AC%E4%B8%80%E8%88%AC%E5%B8%8C%E6%9C%9B%E5%AD%98%E5%9C%A8%E5%B0%B1%E6%9B%B4%E6%96%B0%EF%BC%8C%E4%B8%8D%E5%AD%98%E5%9C%A8%E5%B0%B1%E6%96%B0%E5%A2%9E%E5%BE%97%E6%95%88%E6%9E%9C%E3%80%82mybatis%E4%B8%AD%E5%BE%97%E5%AE%9E%E7%8E%B0%E6%80%9D%E8%B7%AF%E6%98%AF%E9%80%9A%E8%BF%87%E7%BB%99mysql%E6%95%B0%E6%8D%AE%E8%A1%A8%E5%BB%BA%E7%AB%8B%E4%B8%80%E4%B8%AA%E5%94%AF%E4%B8%80%E7%B4%A2%E5%BC%95%E6%9D%A5%E5%88%A4%E6%96%AD%E6%98%AF%E5%90%A6%E9%87%8D%E5%A4%8D%EF%BC%8C%E5%A6%82%E6%9E%9C%E9%87%8D%E5%A4%8D%E5%88%99%E6%9B%B4%E6%96%B0%EF%BC%88%E6%88%96%E8%80%85%E8%BF%94%E5%9B%9E%E9%94%99%E8%AF%AF%EF%BC%89%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%8D%E9%87%8D%E5%A4%8D%E5%88%99%E6%96%B0%E5%A2%9E%E3%80%82%E5%90%8C%E6%97%B6%E5%86%8DMySQL%20%E4%B8%AD%E4%BD%BF%E7%94%A8%20REPLACE%20%E5%92%8CINSERT%20%E2%80%A6%20ON%20DUPLICATE%20KEY%20UPDATE%E8%AF%AD%E6%B3%95%0A%0A1%EF%BC%89%E9%9C%80%E8%A6%81%E8%A1%A8%E5%AD%98%E5%9C%A8%E4%B8%80%E4%B8%AA%E5%94%AF%E4%B8%80%E7%B4%A2%E5%BC%95%E4%BD%9C%E4%B8%BA%E6%9D%A1%E4%BB%B6%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%B8%8D%E5%B8%8C%E6%9C%9B%E6%9C%89%E4%B8%A4%E6%9D%A1%E4%B8%80%E6%91%B8%E4%B8%80%E6%A0%B7%E7%9A%84%E5%B0%B1%E5%BB%BA%E7%AB%8B%E5%A4%8D%E5%90%88%E5%94%AF%E4%B8%80%E7%B4%A2%E5%BC%95%EF%BC%9A%0A%60alter%20table%20trading_sync_product%20add%20unique%20index%20sku_code_sync(sku_code%2Csync_time)%3B%60%0A%0A2%EF%BC%89INSERT%20%E2%80%A6%20ON%20DUPLICATE%20KEY%20UPDATE%0A%E5%8F%AA%E9%9C%80%E8%A6%81%E5%9C%A8%20INSERT%20%E8%AF%AD%E5%8F%A5%E5%90%8E%E9%9D%A2%E5%A3%B0%E6%98%8E%20ON%20DUPLICATE%20KEY%20UPDATE%20%E5%AD%90%E5%8F%A5%EF%BC%8C%E6%8F%92%E5%85%A5%E6%95%B0%E6%8D%AE%E6%97%B6%20MySQL%20%E5%B0%B1%E4%BC%9A%E6%A0%B9%E6%8D%AE%E5%94%AF%E4%B8%80%E7%B4%A2%E5%BC%95%E5%92%8C%E4%B8%BB%E9%94%AE%E8%BF%9B%E8%A1%8C%E5%88%A4%E6%96%AD%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%9C%89%E5%94%AF%E4%B8%80%E7%B4%A2%E5%BC%95%E5%92%8C%E4%B8%BB%E9%94%AE%E6%9C%89%E9%87%8D%E5%A4%8D%EF%BC%8C%E5%88%99%E4%BC%9A%E8%B7%9F%E6%96%B0%E6%95%B0%E6%8D%AE%EF%BC%8C%E5%90%A6%E5%88%99%E4%BC%9A%E6%8F%92%E5%85%A5%E6%95%B0%E6%8D%AE%0A%0A**%E6%89%B9%E9%87%8F%E6%8F%92%E5%85%A5%EF%BC%9A%E6%9C%89%E5%B0%B1%E6%9B%B4%E6%96%B0%EF%BC%8C%E6%B2%A1%E6%9C%89%E5%B0%B1%E6%96%B0%E5%A2%9E**%0A%0A%60%60%60xml%0A%3Cinsert%20id%3D%22batchInsert%22%20useGeneratedKeys%3D%22true%22%20keyProperty%3D%22id%22%20parameterType%3D%22com.ghub.middleware.repository.entity.TradingSyncProduct%22%3E%0A%20%20insert%20into%20trading_sync_product%20(sync_time%2C%20sku_code%2C%0A%20%20%20%20trade_type%2C%20stock_num_init%2C%20stock_num_new%2C%0A%20%20%20%20remark%2C%20create_by%2C%20create_time%2C%20update_time%2C%20update_by%2Csync_status)%0A%20%20values%0A%20%20%3Cforeach%20item%3D%22item%22%20collection%3D%22list%22%20separator%3D%22%2C%22%3E%0A%20%20%20%20(%23%7Bitem.syncTime%2CjdbcType%3DDATE%7D%2C%20%23%7Bitem.skuCode%2CjdbcType%3DVARCHAR%7D%2C%0A%20%20%20%20%23%7Bitem.tradeType%2CjdbcType%3DINTEGER%7D%2C%20%23%7Bitem.stockNumInit%2CjdbcType%3DINTEGER%7D%2C%20%23%7Bitem.stockNumNew%2CjdbcType%3DINTEGER%7D%2C%0A%20%20%20%20%23%7Bitem.remark%2CjdbcType%3DVARCHAR%7D%2C%20%23%7Bitem.createBy%2CjdbcType%3DVARCHAR%7D%2C%20%23%7Bitem.createTime%2CjdbcType%3DTIMESTAMP%7D%2C%0A%20%20%20%20%23%7Bitem.updateTime%2CjdbcType%3DTIMESTAMP%7D%2C%20%23%7Bitem.updateBy%2CjdbcType%3DVARCHAR%7D%2C%20%23%7Bitem.syncStatus%2CjdbcType%3DINTEGER%7D)%0A%20%20%3C%2Fforeach%3E%0A%20%20ON%20DUPLICATE%20KEY%20UPDATE%0A%20%20sync_time%3DVALUES(sync_time)%2C%0A%20%20sku_code%3DVALUES(sku_code)%2C%0A%20%20trade_type%3DVALUES(trade_type)%2C%0A%20%20stock_num_init%3DVALUES(stock_num_init)%2C%0A%20%20stock_num_new%3DVALUES(stock_num_new)%2C%0A%20%20remark%3DVALUES(remark)%2C%0A%20%20update_time%3DVALUES(update_time)%2C%0A%20%20update_by%3DVALUES(update_by)%2C%0A%20%20sync_status%3DVALUES(sync_status)%0A%3C%2Finsert%3E%0A%60%60%60%0A%0A**%E4%BD%BF%E7%94%A8%E9%99%90%E5%88%B6%EF%BC%9A**%0A%26emsp%3B%26emsp%3B%E6%9B%B4%E6%96%B0%E7%9A%84%E5%86%85%E5%AE%B9%E4%B8%ADunique%20key%E6%88%96%E8%80%85primary%20key%E6%9C%80%E5%A5%BD%E4%BF%9D%E8%AF%81%E4%B8%80%E4%B8%AA%EF%BC%8C%E4%B8%8D%E7%84%B6%E4%B8%8D%E8%83%BD%E4%BF%9D%E8%AF%81%E8%AF%AD%E5%8F%A5%E6%89%A7%E8%A1%8C%E6%AD%A3%E7%A1%AE(%E6%9C%89%E4%BB%BB%E6%84%8F%E4%B8%80%E4%B8%AAunique%20key%E9%87%8D%E5%A4%8D%E5%B0%B1%E4%BC%9A%E8%B5%B0%E6%9B%B4%E6%96%B0%2C%E5%BD%93%E7%84%B6%E5%A6%82%E6%9E%9C%E6%9B%B4%E6%96%B0%E7%9A%84%E8%AF%AD%E5%8F%A5%E4%B8%AD%E5%9C%A8%E8%A1%A8%E4%B8%AD%E4%B9%9F%E6%9C%89%E9%87%8D%E5%A4%8D%E6%A0%A1%E9%AA%8C%E7%9A%84%E5%AD%97%E6%AE%B5%EF%BC%8C%E9%82%A3%E4%B9%88%E4%B9%9F%E4%B8%8D%E4%BC%9A%E6%9B%B4%E6%96%B0%E6%88%90%E5%8A%9F%E8%80%8C%E5%AF%BC%E8%87%B4%E6%8A%A5%E9%94%99%2C%E5%8F%AA%E6%9C%89%E5%BD%93%E8%AF%A5%E6%9D%A1%E8%AF%AD%E5%8F%A5%E6%B2%A1%E6%9C%89%E4%BB%BB%E4%BD%95%E4%B8%80%E4%B8%AAunique%20key%E9%87%8D%E5%A4%8D%E6%89%8D%E4%BC%9A%E6%8F%92%E5%85%A5%E6%96%B0%E8%AE%B0%E5%BD%95)%EF%BC%9B%0A%0A*%20%E5%B0%BD%E9%87%8F%E4%B8%8D%E5%AF%B9%E5%AD%98%E5%9C%A8%E5%A4%9A%E4%B8%AA%E5%94%AF%E4%B8%80%E9%94%AE%E7%9A%84table%E4%BD%BF%E7%94%A8%E8%AF%A5%E8%AF%AD%E5%8F%A5%EF%BC%8C%E9%81%BF%E5%85%8D%E5%8F%AF%E8%83%BD%E5%AF%BC%E8%87%B4%E6%95%B0%E6%8D%AE%E9%94%99%E4%B9%B1%E3%80%82%0A*%20%E5%9C%A8%E6%9C%89%E5%8F%AF%E8%83%BD%E6%9C%89%E5%B9%B6%E5%8F%91%E4%BA%8B%E5%8A%A1%E6%89%A7%E8%A1%8C%E7%9A%84insert%20%E8%AF%AD%E5%8F%A5%E6%83%85%E5%86%B5%E4%B8%8B%E4%B8%8D%E4%BD%BF%E7%94%A8%E8%AF%A5%E8%AF%AD%E5%8F%A5%EF%BC%8C%E5%8F%AF%E8%83%BD%E5%AF%BC%E8%87%B4%E4%BA%A7%E7%94%9Fdeath%20lock%E3%80%82%0A*%20%E5%A6%82%E6%9E%9C%E6%95%B0%E6%8D%AE%E8%A1%A8id%E6%98%AF%E8%87%AA%E5%8A%A8%E9%80%92%E5%A2%9E%E7%9A%84%E4%B8%8D%E5%BB%BA%E8%AE%AE%E4%BD%BF%E7%94%A8%E8%AF%A5%E8%AF%AD%E5%8F%A5%EF%BC%9Bid%E4%B8%8D%E8%BF%9E%E7%BB%AD%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%89%8D%E9%9D%A2%E6%9B%B4%E6%96%B0%E7%9A%84%E6%AF%94%E8%BE%83%E5%A4%9A%EF%BC%8C%E6%96%B0%E5%A2%9E%E7%9A%84%E4%B8%8B%E4%B8%80%E6%9D%A1%E4%BC%9A%E7%9B%B8%E5%BA%94%E8%B7%B3%E8%B7%83%E7%9A%84%E6%9B%B4%E5%A4%A7%E3%80%82%0A*%20%E8%AF%A5%E8%AF%AD%E5%8F%A5%E6%98%AFmysql%E7%8B%AC%E6%9C%89%E7%9A%84%E8%AF%AD%E6%B3%95%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%8F%AF%E8%83%BD%E4%BC%9A%E8%AE%BE%E8%AE%A1%E5%88%B0%E5%85%B6%E4%BB%96%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AF%AD%E8%A8%80%E8%B7%A8%E5%BA%93%E8%A6%81%E8%B0%A8%E6%85%8E%E4%BD%BF%E7%94%A8%E3%80%82%0A*%20%E5%A6%82%E6%9E%9C%E4%B8%8D%E4%BD%BF%E7%94%A8VALUES%EF%BC%9A%EF%BC%88trade_type%3Dtrade_type%EF%BC%89%2C%E5%88%99%E8%BF%98%E6%98%AF%E5%8E%9F%E6%9D%A5%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%88%E6%9C%89%E5%B0%B1%E4%B8%8D%E6%9B%B4%E6%96%B0%EF%BC%8C%E6%B2%A1%E6%9C%89%E5%B0%B1%E6%96%B0%E5%A2%9E%EF%BC%89%EF%BC%8C%E4%BD%BF%E7%94%A8VALUES%EF%BC%88trade_type%3DVALUES(trade_type)%EF%BC%89%E5%88%99%E6%98%AF%EF%BC%88%E6%9C%89%E5%B0%B1%E6%9B%B4%E6%96%B0%EF%BC%8C%E6%B2%A1%E6%9C%89%E5%B0%B1%E6%96%B0%E5%A2%9E%EF%BC%89%0A%0A**%E4%BA%A7%E7%94%9Fdeath%20lock%E5%8E%9F%E7%90%86%EF%BC%9A**%0Ainsert%20...%20on%20duplicate%20key%20%E5%9C%A8%E6%89%A7%E8%A1%8C%E6%97%B6%EF%BC%8Cinnodb%E5%BC%95%E6%93%8E%E4%BC%9A%E5%85%88%E5%88%A4%E6%96%AD%E6%8F%92%E5%85%A5%E7%9A%84%E8%A1%8C%E6%98%AF%E5%90%A6%E4%BA%A7%E7%94%9F%E9%87%8D%E5%A4%8Dkey%E9%94%99%E8%AF%AF%EF%BC%8C%E5%A6%82%E6%9E%9C%E5%AD%98%E5%9C%A8%EF%BC%8C%E5%9C%A8%E5%AF%B9%E8%AF%A5%E7%8E%B0%E6%9C%89%E7%9A%84%E8%A1%8C%E5%8A%A0%E4%B8%8AS%EF%BC%88%E5%85%B1%E4%BA%AB%E9%94%81%EF%BC%89%E9%94%81%EF%BC%8C%E5%A6%82%E6%9E%9C%E8%BF%94%E5%9B%9E%E8%AF%A5%E8%A1%8C%E6%95%B0%E6%8D%AE%E7%BB%99mysql%2C%E7%84%B6%E5%90%8Emysql%E6%89%A7%E8%A1%8C%E5%AE%8Cduplicate%E5%90%8E%E7%9A%84update%E6%93%8D%E4%BD%9C%EF%BC%8C%E7%84%B6%E5%90%8E%E5%AF%B9%E8%AF%A5%E8%AE%B0%E5%BD%95%E5%8A%A0%E4%B8%8AX%EF%BC%88%E6%8E%92%E4%BB%96%E9%94%81%EF%BC%89%EF%BC%8C%E6%9C%80%E5%90%8E%E8%BF%9B%E8%A1%8Cupdate%E5%86%99%E5%85%A5%E3%80%82%0A%0A%23%23%23%23%20%20%E5%9B%9B%20%E7%BC%93%E5%AD%98%0AMybatis%20%E4%BD%BF%E7%94%A8%E5%88%B0%E4%BA%86%E4%B8%A4%E7%A7%8D%E7%BC%93%E5%AD%98%EF%BC%9A%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%EF%BC%88local%20cache%EF%BC%89%E5%92%8C%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%EF%BC%88second%20level%20cache%EF%BC%89%E3%80%82%0A%23%23%23%23%23%20(1)%20%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%0A*%20%E6%AF%8F%E5%BD%93%E4%B8%80%E4%B8%AA%E6%96%B0%20session%20%E8%A2%AB%E5%88%9B%E5%BB%BA%EF%BC%8CMyBatis%20%E5%B0%B1%E4%BC%9A%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E4%B8%8E%E4%B9%8B%E7%9B%B8%E5%85%B3%E8%81%94%E7%9A%84%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E3%80%82%E4%BB%BB%E4%BD%95%E5%9C%A8%20session%20%E6%89%A7%E8%A1%8C%E8%BF%87%E7%9A%84%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E6%9C%AC%E8%BA%AB%E9%83%BD%E4%BC%9A%E8%A2%AB%E4%BF%9D%E5%AD%98%E5%9C%A8%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E4%B8%AD%EF%BC%8C%E9%82%A3%E4%B9%88%EF%BC%8C%E7%9B%B8%E5%90%8C%E7%9A%84%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E5%92%8C%E7%9B%B8%E5%90%8C%E7%9A%84%E5%8F%82%E6%95%B0%E6%89%80%E4%BA%A7%E7%94%9F%E7%9A%84%E6%9B%B4%E6%94%B9%E5%B0%B1%E4%B8%8D%E4%BC%9A%E4%BA%8C%E5%BA%A6%E5%BD%B1%E5%93%8D%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BA%86%E3%80%82%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E4%BC%9A%E8%A2%AB%E5%A2%9E%E5%88%A0%E6%94%B9%E3%80%81%E6%8F%90%E4%BA%A4%E4%BA%8B%E5%8A%A1%E3%80%81%E5%85%B3%E9%97%AD%E4%BA%8B%E5%8A%A1%E4%BB%A5%E5%8F%8A%E5%85%B3%E9%97%AD%20session%20%E6%89%80%E6%B8%85%E7%A9%BA%E3%80%82%0A*%20%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E9%9A%8F%E7%9D%80session%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%BB%93%E6%9D%9F%E8%80%8C%E7%BB%93%E6%9D%9F%EF%BC%8C%E4%B9%9F%E5%8F%AF%E4%BB%A5%E8%B0%83%E7%94%A8void%20clearCache()%E6%88%96session.close()%E9%83%BD%E4%BC%9A%E6%B8%85%E7%A9%BA%E7%BC%93%E5%AD%98%EF%BC%9B%0A*%20%E5%8F%AF%E4%BB%A5%E8%AE%BE%E7%BD%AE%20localCacheScope%E6%9D%A5%E8%AE%BE%E7%BD%AE%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E7%9A%84%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F%EF%BC%8CSTATEMENT%20%E8%A1%A8%E7%A4%BA%E7%BC%93%E5%AD%98%E4%BB%85%E5%9C%A8%E8%AF%AD%E5%8F%A5%E6%89%A7%E8%A1%8C%E6%97%B6%E6%9C%89%E6%95%88%EF%BC%8C%E5%A6%82%E6%9E%9C%20localCacheScope%20%E8%A2%AB%E8%AE%BE%E7%BD%AE%E4%B8%BA%20SESSION%EF%BC%8C%E9%82%A3%E4%B9%88%20MyBatis%20%E6%89%80%E8%BF%94%E5%9B%9E%E7%9A%84%E5%BC%95%E7%94%A8%E5%B0%86%E4%BC%A0%E9%80%92%E7%BB%99%E4%BF%9D%E5%AD%98%E5%9C%A8%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E9%87%8C%E7%9A%84%E7%9B%B8%E5%90%8C%E5%AF%B9%E8%B1%A1%E3%80%82%E5%AF%B9%E8%BF%94%E5%9B%9E%E7%9A%84%E5%AF%B9%E8%B1%A1%EF%BC%88%E4%BE%8B%E5%A6%82%20list%EF%BC%89%E5%81%9A%E5%87%BA%E4%BB%BB%E4%BD%95%E6%9B%B4%E6%96%B0%E5%B0%86%E4%BC%9A%E5%BD%B1%E5%93%8D%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E7%9A%84%E5%86%85%E5%AE%B9%EF%BC%8C%E8%BF%9B%E8%80%8C%E5%BD%B1%E5%93%8D%E5%AD%98%E6%B4%BB%E5%9C%A8%20session%20%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E4%B8%AD%E7%9A%84%E7%BC%93%E5%AD%98%E6%89%80%E8%BF%94%E5%9B%9E%E7%9A%84%E5%80%BC%E3%80%82%E5%9B%A0%E6%AD%A4%EF%BC%8C%E4%B8%8D%E8%A6%81%E5%AF%B9%20MyBatis%20%E6%89%80%E8%BF%94%E5%9B%9E%E7%9A%84%E5%AF%B9%E8%B1%A1%E4%BD%9C%E5%87%BA%E6%9B%B4%E6%94%B9%EF%BC%8C%E4%BB%A5%E9%98%B2%E5%90%8E%E6%82%A3%E3%80%82%EF%BC%88%E5%AF%B9%E8%BF%94%E5%9B%9E%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%9B%B4%E6%94%B9%E4%BC%9A%E5%BD%B1%E5%93%8D%E7%BC%93%E5%AD%98%E9%87%8C%E7%9A%84%E5%80%BC%EF%BC%89%0A%0A**mybatis-config.xml%20%E5%85%B3%E4%BA%8E%E7%BC%93%E5%AD%98%E7%9A%84%E9%85%8D%E7%BD%AE%EF%BC%9A**%0A%60%60%60xml%0A%3Csettings%3E%0A%20%20%20%20%3C!--%E5%85%A8%E5%B1%80%E5%9C%B0%E5%BC%80%E5%90%AF%E6%88%96%E5%85%B3%E9%97%AD%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84%E6%89%80%E6%9C%89%E6%98%A0%E5%B0%84%E5%99%A8%E5%B7%B2%E7%BB%8F%E9%85%8D%E7%BD%AE%E7%9A%84%E4%BB%BB%E4%BD%95%E7%BC%93%E5%AD%98%E3%80%82%0A%20%20%20%20%E5%85%A8%E5%B1%80%E5%85%B3%E9%97%AD%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%20--%3E%0A%20%20%20%20%3Csetting%20name%3D%22cacheEnabled%22%20value%3D%22false%22%2F%3E%0A%20%20%20%20%3C!--%E8%AE%BE%E7%BD%AE%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E7%9A%84%E4%BD%9C%E7%94%A8%E5%9F%9F%2C%E9%BB%98%E8%AE%A4SESSION%EF%BC%8CSESSION%E4%BC%9A%E7%BC%93%E5%AD%98%E4%B8%80%E4%B8%AA%E4%BC%9A%E8%AF%9D%E4%B8%AD%E7%9A%84%E6%89%80%E6%9C%89%E6%9F%A5%E8%AF%A2%EF%BC%8CSTATEMENT%EF%BC%8C%0A%20%20%20%20%E6%9C%AC%E5%9C%B0%E4%BC%9A%E8%AF%9D%E4%BB%85%E7%94%A8%E5%9C%A8%E8%AF%AD%E5%8F%A5%E6%89%A7%E8%A1%8C%E4%B8%8A%EF%BC%8C%E5%AF%B9%E7%9B%B8%E5%90%8C%20SqlSession%20%E7%9A%84%E4%B8%8D%E5%90%8C%E8%B0%83%E7%94%A8%E5%B0%86%E4%B8%8D%E4%BC%9A%E5%85%B1%E4%BA%AB%E6%95%B0%E6%8D%AE--%3E%0A%20%20%20%20%3Csetting%20name%3D%22localCacheScope%22%20value%3D%22SESSION%22%2F%3E%0A%3C%2Fsettings%3E%0A%60%60%60%0A%0A%0A%23%23%23%23%23%23%20**%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%EF%BC%9A**%0A%0A%E6%BA%90%E7%A0%81%E4%B8%AD%EF%BC%9APerpetualCache%E6%98%AF%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E7%9A%84%E5%AE%9E%E7%8E%B0%E7%B1%BB%EF%BC%8C%E5%AE%9E%E7%8E%B0%E4%BA%86Cache%E6%8E%A5%E5%8F%A3%EF%BC%9B%E9%BB%98%E8%AE%A4%E7%9A%84%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E4%BD%BF%E7%94%A8%E9%9D%9E%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E9%9B%86%E5%90%88%E7%B1%BBHashMap%E6%9D%A5%E5%AE%9E%E7%8E%B0%EF%BC%8C%E7%94%B1%E4%BA%8E%E5%AE%83%E5%90%8Csqlsession%E7%9B%B8%E5%90%8C%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%89%80%E4%BB%A5%E4%B8%8D%E4%BC%9A%E6%9C%89%E5%B9%B6%E5%8F%91%E9%97%AE%E9%A2%98%EF%BC%9B%0A%0Amybatis%E7%9A%84Executor%E6%A8%A1%E5%9D%97%E6%98%AF%E6%89%A7%E8%A1%8CSQL%E7%9A%84%E6%89%A7%E8%A1%8C%E5%99%A8%EF%BC%8C%E5%AE%83%E6%9C%893%E4%B8%AA%E5%AE%9E%E7%8E%B0%E7%B1%BB%E5%A6%82%E5%9B%BE%EF%BC%9A%0A!%5Bfa0f74a604a89b6ac55ea85aae60f5b0.png%5D(en-resource%3A%2F%2Fdatabase%2F1213%3A1)%0A%0A1.%20SimpleExecutor%E6%98%AF%E9%BB%98%E8%AE%A4%E4%BD%BF%E7%94%A8%E7%9A%84%E6%89%A7%E8%A1%8C%E5%99%A8%EF%BC%9B%0A2.%20ReuseExecutor%E5%88%99%E6%98%AF%E5%B0%86Statement%E4%B8%8ESQL%E5%BB%BA%E7%AB%8B%E5%85%B3%E8%81%94%E5%85%B3%E7%B3%BB%E7%BC%93%E5%AD%98%E8%B5%B7%E6%9D%A5%EF%BC%8C%E8%BF%99%E6%A0%B7%E5%B0%B1%E4%B8%8D%E7%94%A8%E6%AF%8F%E6%AC%A1%E9%83%BD%E8%A6%81%E9%87%8D%E5%A4%8D%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84Statement%EF%BC%9B%0A3.%20BatchExecutor%E5%88%99%E6%98%AF%E6%89%B9%E9%87%8F%E6%89%A7%E8%A1%8C%E5%99%A8%EF%BC%8C%E9%80%9A%E8%BF%87%E5%B0%81%E8%A3%85%E5%BA%95%E5%B1%82JDBC%E7%9A%84BATCH%E7%9B%B8%E5%85%B3%E7%9A%84API%EF%BC%8C%E6%9D%A5%E5%8A%A0%E9%80%9F%E6%89%B9%E9%87%8F%E7%9B%B8%E5%85%B3%E7%9A%84%E6%93%8D%E4%BD%9C%E3%80%82%0A%0A**%E8%80%8C%E6%88%91%E4%BB%AC%E5%85%B3%E5%BF%83%E7%9A%84%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98PerpetualCache%E5%88%99%E4%BB%A5%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%E5%AD%98%E5%9C%A8%E4%BA%8EBaseExecutor%E4%B8%AD**%0A!%5B5c6daafd5f19485fea53ebff9c2c37e6.png%5D(en-resource%3A%2F%2Fdatabase%2F1215%3A1)%0A%0A**%EF%BC%881%EF%BC%89Executor%E6%89%A7%E8%A1%8C%E5%99%A8%E7%9A%84%E5%88%9B%E5%BB%BA%E6%B5%81%E7%A8%8B%EF%BC%9A**%0A!%5Bccfc2a7ac4659558a47b1394d2a9f0e3.png%5D(en-resource%3A%2F%2Fdatabase%2F1217%3A1)%0A%0A%E5%BD%93%E9%80%9A%E8%BF%87sqlsessionfactory%E8%B0%83%E7%94%A8opensession%E6%96%B9%E6%B3%95%E5%88%9B%E5%BB%BAsqlsession%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E6%80%BB%E6%98%AF%E4%BC%9A%E5%88%9B%E5%BB%BAExecutor%E6%89%A7%E8%A1%8C%E5%99%A8%EF%BC%8Csqlsession%E9%9A%8F%E7%9D%80%E8%AF%B7%E6%B1%82%E7%9A%84%E7%BA%BF%E7%A8%8B%E8%A2%AB%E5%88%9B%E5%BB%BA%EF%BC%8C%E9%9A%8F%E7%9D%80%E8%AF%B7%E6%B1%82%E7%BB%93%E6%9D%9F%E8%80%8C%E6%B6%88%E4%BA%A1%EF%BC%8C%E4%BD%9C%E7%94%A8%E5%9F%9F%E5%9C%A8%E6%96%B9%E6%B3%95%E6%88%96%E7%BA%BF%E7%A8%8B%E7%9A%84%E4%B8%8A%E4%B8%8B%E6%96%87%E3%80%82%E5%9C%A8%E5%B9%B6%E5%8F%91%E5%9C%BA%E6%99%AF%E4%B8%8B%E4%BC%9A%E5%88%9B%E5%BB%BA%E5%A4%9A%E4%B8%AAsqlsession%2CExecutor%E6%89%A7%E8%A1%8C%E5%99%A8%E4%B9%9F%E6%9C%80%E6%98%AF%E4%BC%9A%E8%A2%AB%E5%88%9B%E5%BB%BA%EF%BC%9B%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98%E5%9C%A8BaseExecutor%E4%B8%AD%EF%BC%8C%E6%AF%8F%E6%AC%A1%E4%B9%9F%E9%83%BD%E4%BC%9A%E8%A2%AB%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84%E5%AE%9E%E4%BE%8B%EF%BC%9B%E6%89%80%E4%BB%A5%E4%B8%8D%E4%BC%9A%E5%AD%98%E5%9C%A8%E5%B9%B6%E5%8F%91%E9%97%AE%E9%A2%98%EF%BC%9B%0A%0A**%EF%BC%882%EF%BC%89%E6%9F%A5%E8%AF%A2%E6%B5%81%E7%A8%8B%EF%BC%9A**%0A!%5B8dd5cfa602fff54b446413b0de7291c7.png%5D(en-resource%3A%2F%2Fdatabase%2F1219%3A1)%0A%E6%9C%80%E7%BB%88%E4%BC%9A%E8%B0%83%E7%94%A8BaseExecutor%E4%B8%AD%E7%9A%84query%E6%96%B9%E6%B3%95%EF%BC%8C%E6%96%B9%E6%B3%95%E4%B8%AD%E4%BC%9A%E8%AF%BB%E5%8F%96%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84%E7%BC%93%E5%AD%98%E9%85%8D%E7%BD%AE%E6%9D%A5%E5%86%B3%E5%AE%9A%E6%98%AF%E5%90%A6%E8%AF%BB%E5%8F%96%E6%88%96%E6%B8%85%E7%A9%BA%E7%BC%93%E5%AD%98%EF%BC%9A%0A%60%60%60java%0Apublic%20%3CE%3E%20List%3CE%3E%20query(MappedStatement%20ms%2C%20Object%20parameter%2C%20RowBounds%20rowBounds%2C%20ResultHandler%20resultHandler%2C%20CacheKey%20key%2C%20BoundSql%20boundSql)%20throws%20SQLException%20%7B%0A%20%20%20%20ErrorContext.instance().resource(ms.getResource()).activity(%22executing%20a%20query%22).object(ms.getId())%3B%0A%20%20%20%20if%20(closed)%20%7B%0A%20%20%20%20%20%20throw%20new%20ExecutorException(%22Executor%20was%20closed.%22)%3B%0A%20%20%20%20%7D%0A%20%20%20%20%2F%2F%20%E5%A6%82%E6%9E%9C%E4%B8%8D%E6%98%AF%E5%B5%8C%E5%A5%97%E6%9F%A5%E8%AF%A2%E6%88%96%E8%80%85%E5%B5%8C%E5%A5%97%E6%9F%A5%E8%AF%A2%E6%89%A7%E8%A1%8C%E5%AE%8C%E6%AF%95%EF%BC%8C%E5%B9%B6%E4%B8%94flushCache%E9%85%8D%E7%BD%AE%E4%B8%BAtrue%0A%20%20%20%20if%20(queryStack%20%3D%3D%200%20%26%26%20ms.isFlushCacheRequired())%20%7B%0A%20%20%20%20%20%20%2F%2F%20%E6%B8%85%E7%A9%BA%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%0A%20%20%20%20%20%20clearLocalCache()%3B%0A%20%20%20%20%7D%0A%20%20%20%20List%3CE%3E%20list%3B%0A%20%20%20%20try%20%7B%0A%20%20%20%20%20%20%2F%2F%20%E6%9C%AC%E6%AC%A1%E6%9F%A5%E8%AF%A2%E6%89%A7%E8%A1%8C%E6%9F%A5%E8%AF%A2%E6%AC%A1%E6%95%B0%2B1%0A%20%20%20%20%20%20queryStack%2B%2B%3B%0A%20%20%20%20%20%20%2F%2F%E8%8E%B7%E5%8F%96%E7%BC%93%E5%AD%98%0A%20%20%20%20%20%20list%20%3D%20resultHandler%20%3D%3D%20null%20%3F%20(List%3CE%3E)%20localCache.getObject(key)%20%3A%20null%3B%0A%20%20%20%20%20%20if%20(list%20!%3D%20null)%20%7B%0A%20%20%20%20%20%20%20%20%2F%2F%20%E5%A4%84%E7%90%86%E5%82%A8%E5%AD%98%E8%BF%87%E7%A8%8B%EF%BC%8C%E9%92%88%E5%AF%B9OUT%E7%9A%84%E5%8F%82%E6%95%B0%E8%AE%BE%E5%AE%9A%E7%BC%93%E5%AD%98%E5%80%BC%0A%20%20%20%20%20%20%20%20handleLocallyCachedOutputParameters(ms%2C%20key%2C%20parameter%2C%20boundSql)%3B%0A%20%20%20%20%20%20%7D%20else%20%7B%0A%20%20%20%20%20%20%20%20%2F%2F%20%E6%B2%A1%E6%9C%89%E5%91%BD%E4%B8%AD%E7%BC%93%E5%AD%98%E5%88%99%E5%88%B0%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%AD%E6%9F%A5%E8%AF%A2%0A%20%20%20%20%20%20%20%20list%20%3D%20queryFromDatabase(ms%2C%20parameter%2C%20rowBounds%2C%20resultHandler%2C%20key%2C%20boundSql)%3B%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%20finally%20%7B%0A%20%20%20%20%20%20%2F%2F%20%E6%9C%AC%E6%AC%A1%E6%9F%A5%E8%AF%A2%E6%89%A7%E8%A1%8C%E6%9F%A5%E8%AF%A2%E6%AC%A1%E6%95%B0-1%0A%20%20%20%20%20%20queryStack--%3B%0A%20%20%20%20%7D%0A%20%20%20%20%2F%2F%20%E6%9F%A5%E8%AF%A2%E6%89%A7%E8%A1%8C%E5%AE%8C%E6%AF%95%0A%20%20%20%20if%20(queryStack%20%3D%3D%200)%20%7B%0A%20%20%20%20%20%20for%20(DeferredLoad%20deferredLoad%20%3A%20deferredLoads)%20%7B%0A%20%20%20%20%20%20%20%20deferredLoad.load()%3B%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20deferredLoads.clear()%3B%0A%20%20%20%20%20%20%2F%2F%20%E5%A6%82%E6%9E%9C%E6%9C%AC%E5%9C%B0%E7%BC%93%E5%AD%98Scope%E8%A2%AB%E9%85%8D%E7%BD%AE%E4%B8%BASTATEMENT%EF%BC%8C%E5%88%99%E6%AF%8F%E6%AC%A1%E6%89%A7%E8%A1%8C%E7%9A%84SQL%E7%9A%84%E6%97%B6%E5%80%99%E9%83%BD%E8%A6%81%E6%B8%85%E7%A9%BA%E7%BC%93%E5%AD%98%E3%80%82%0A%20%20%20%20%20%20%2F%2F%20%E5%9B%A0%E4%B8%BA%E6%9C%89queryStack%20%3D%3D%200%E7%9A%84%E5%88%A4%E6%96%AD%EF%BC%8C%E6%89%80%E4%BB%A5%E6%9F%A5%E8%AF%A2%E5%A6%82%E6%9E%9C%E6%98%AF%E5%B5%8C%E5%A5%97%E6%9F%A5%E8%AF%A2%EF%BC%8C%E7%BC%93%E5%AD%98%E8%BF%98%E6%98%AF%E8%B5%B7%E4%BD%9C%E7%94%A8%E7%9A%84%E3%80%82%0A%20%20%20%20%20%20if%20(configuration.getLocalCacheScope()%20%3D%3D%20LocalCacheScope.STATEMENT)%20%7B%0A%20%20%20%20%20%20%20%20clearLocalCache()%3B%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20return%20list%3B%0A%20%20%7D%0A%60%60%60%0A%0A%E6%89%A7%E8%A1%8Csql%E6%9F%A5%E8%AF%A2%EF%BC%8C%E5%B9%B6%E5%86%99%E5%85%A5%E7%BC%93%E5%AD%98%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%9A%0A%60%60%60java%0Aprivate%20%3CE%3E%20List%3CE%3E%20queryFromDatabase(MappedStatement%20ms%2C%20Object%20parameter%2C%20RowBounds%20rowBounds%2C%20ResultHandler%20resultHandler%2C%20CacheKey%20key%2C%20BoundSql%20boundSql)%20throws%20SQLException%20%7B%0A%20%20List%3CE%3E%20list%3B%0A%20%20%2F%2F%20%E7%BC%93%E5%AD%98%E9%A2%84%E5%8D%A0%E4%BD%8D%0A%20%20localCache.putObject(key%2C%20EXECUTION_PLACEHOLDER)%3B%0A%20%20try%20%7B%0A%20%20%20%20%2F%2F%20%E6%89%A7%E8%A1%8CDB%E6%9F%A5%E8%AF%A2%E6%93%8D%E4%BD%9C%0A%20%20%20%20list%20%3D%20doQuery(ms%2C%20parameter%2C%20rowBounds%2C%20resultHandler%2C%20boundSql)%3B%0A%20%20%7D%20finally%20%7B%0A%20%20%20%20%2F%2F%20%E9%87%8A%E6%94%BE%E9%A2%84%E5%8D%A0%E4%BD%8D%0A%20%20%20%20localCache.removeObject(key)%3B%0A%20%20%7D%0A%20%20%2F%2F%20%E6%94%BE%E5%85%A5%E7%9C%9F%E6%AD%A3%E7%9A%84%E7%BC%93%E5%AD%98%0A%20%20localCache.putObject(key%2C%20list)%3B%0A%20%20%2F%2F%20%E5%A6%82%E6%9E%9C%E6%98%AF%E5%AD%98%E5%82%A8%E8%BF%87%E7%A8%8B%E7%9A%84%E8%B0%83%E7%94%A8%E5%88%99%E5%B0%86%E5%82%A8%E5%AD%98%E8%BF%87%E7%A8%8B%E7%9A%84%E6%89%A7%E8%A1%8C%E7%9A%84%E7%BB%93%E6%9E%9C%E4%BF%9D%E5%AD%98%E8%87%B3localOutputParameterCache%E4%B8%AD%0A%20%20if%20(ms.getStatementType()%20%3D%3D%20StatementType.CALLABLE)%20%7B%0A%20%20%20%20localOutputParameterCache.putObject(key%2C%20parameter)%3B%0A%20%20%7D%0A%20%20return%20list%3B%0A%7D%0A%60%60%60%0A%0A**%EF%BC%883%EF%BC%89%E6%9B%B4%E6%96%B0%E6%B5%81%E7%A8%8B%EF%BC%9A**%0A!%5Bf8c2b4798049b28f58dbc5c42f2455e6.png%5D(en-resource%3A%2F%2Fdatabase%2F1221%3A1)%0A%E5%A2%9E%E5%88%A0%E6%94%B9%E4%B8%8D%E5%90%8C%E7%9A%84%E6%93%8D%E4%BD%9C%E6%9C%80%E7%BB%88%E5%88%B0%E6%89%A7%E8%A1%8C%E5%99%A8%E6%97%B6%E4%BB%85%E5%AF%B9%E5%BA%94%E5%88%B0%E4%B8%80%E4%B8%AAupdate%E7%9A%84%E6%96%B9%E6%B3%95%EF%BC%8C%E5%8F%AA%E8%A6%81%E5%9C%A8%E4%B8%80%E4%B8%AASqlSession%E5%A3%B0%E6%98%8E%E5%91%A8%E6%9C%9F%E5%86%85%E6%9C%89%E6%89%A7%E8%A1%8C%E5%A2%9E%E5%88%A0%E6%94%B9%E7%9A%84%E4%BB%BB%E4%BD%95%E4%B8%80%E7%A7%8D%E6%93%8D%E4%BD%9C%EF%BC%8C%E9%82%A3%E4%B9%88%E4%B8%8ESqlSession%E5%85%B3%E8%81%94%E7%9A%84%E6%89%80%E6%9C%89%E7%9A%84%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%E5%86%85%E5%AE%B9%E5%B0%86%E8%A2%AB%E6%B8%85%E7%A9%BA%0A%60%60%60%20java%0A%40Override%0Apublic%20int%20update(MappedStatement%20ms%2C%20Object%20parameter)%20throws%20SQLException%20%7B%0A%20%20ErrorContext.instance().resource(ms.getResource()).activity(%22executing%20an%20update%22).object(ms.getId())%3B%0A%20%20if%20(closed)%20%7B%0A%20%20%20%20throw%20new%20ExecutorException(%22Executor%20was%20closed.%22)%3B%0A%20%20%7D%0A%20%20%2F%2F%20%E6%B8%85%E7%A9%BA%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%E3%80%81%E6%B8%85%E7%A9%BA%E5%AD%98%E5%82%A8%E8%BF%87%E7%A8%8B%E7%BC%93%E5%AD%98%0A%20%20clearLocalCache()%3B%0A%20%20%2F%2F%20%E6%89%A7%E8%A1%8CDB%E6%93%8D%E4%BD%9C%0A%20%20return%20doUpdate(ms%2C%20parameter)%3B%0A%7D%0A%60%60%60%0A!%5B4c3c083c05fade9161c94c3d3d12fe99.png%5D(en-resource%3A%2F%2Fdatabase%2F1223%3A1)%0A%0A%0A%23%23%23%23%23%20(2)%20%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%0A**mapper%20%E6%98%A0%E5%B0%84xml%E4%B8%AD%E7%9A%84%E9%85%8D%E7%BD%AE%EF%BC%9A**%0A%0A%60%60%60%20xml%0A%3Cmapper%20namespace%3D%22com.shuailee.dao.IUserDao%22%3E%0A%20%20%20%20%3C!--%E5%BC%80%E5%90%AF%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98--%3E%0A%20%20%20%20%3Ccache%20%2F%3E%0A%20%20%20%20%3C!--mapper%E8%AF%AD%E5%8F%A5%0A%20%20%20%201%20%20SQL%E8%AF%AD%E5%8F%A5%E4%B8%AD%E7%9A%84flushCache%3D%22true%22%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%90%86%E8%A7%A3%E4%B8%BA%E6%AF%8F%E6%AC%A1%E6%89%A7%E8%A1%8CSQL%E4%B9%8B%E5%89%8D%E9%83%BD%E4%BC%9A%E4%B8%BB%E5%8A%A8%E6%B8%85%E7%A9%BA%E7%BC%93%E5%AD%98%EF%BC%8C%0A%20%20%20%20%E7%84%B6%E5%90%8E%E5%86%8D%E4%BB%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%AD%E6%9F%A5%E8%AF%A2%E7%BB%93%E6%9E%9C%EF%BC%8C%E5%9C%A8%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E4%B8%AD%E9%BB%98%E8%AE%A4%E5%80%BC%E4%B8%BAfalse%EF%BC%8C%E5%85%B6%E4%BB%96%E6%83%85%E5%86%B5%E4%B8%8B%E4%B8%BAtrue%0A%20%20%20%202%20SQL%E8%AF%AD%E5%8F%A5%E4%B8%AD%E7%9A%84useCache%3D%22true%22%20%E9%BB%98%E8%AE%A4%E4%B8%BAtrue%20%E6%89%A7%E8%A1%8C%E8%AF%AD%E5%8F%A5%E6%97%B6%E4%BD%BF%E7%94%A8%E7%BC%93%E5%AD%98%0A%20%20%20%20--%3E%0A%20%20%20%20%3Cselect%20id%3D%22queryUser%22%20resultType%3D%22com.shuailee.model.User%22%20flushCache%3D%22true%22%3E%0A%20%20%20%20SELECT%20*%20FROM%20tb_user%0A%20%20%20%20%3C%2Fselect%3E%0A%3C%2Fmapper%3E%0A%60%60%60%0A!%5Bf8a09823a58e7062981b6ac51f98304d.png%5D(en-resource%3A%2F%2Fdatabase%2F1209%3A1)%0A%0A%26emsp%3B%26emsp%3B%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E5%9C%A8Mybatis%E6%98%AF%E9%BB%98%E8%AE%A4%E5%85%B3%E9%97%AD%EF%BC%8C%E6%88%91%E4%BB%AC%E9%80%9A%E8%BF%87%E9%85%8D%E7%BD%AE%E9%A6%96%E5%85%88%E5%90%AF%E7%94%A8%E5%AE%83%E3%80%82%E5%90%AF%E7%94%A8%E5%AE%83%E7%9A%84%E9%85%8D%E7%BD%AE%E5%BE%88%E7%AE%80%E5%8D%95%E5%8F%AA%E8%A6%81%E5%9C%A8Mapper%E6%96%87%E4%BB%B6%E4%B8%AD%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%8A%A0%E5%85%A5%3Ccache%2F%3E%E5%8D%B3%E5%8F%AF%E3%80%82%0A%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E5%B1%9E%E4%BA%8E%E6%9F%90%E4%B8%AAmapper%E7%9A%84%EF%BC%8C%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%E5%B1%9E%E4%BA%8E%E6%9F%90%E4%B8%AAsqlsession%2C%20%E5%BC%80%E5%90%AF%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E4%BC%9A%EF%BC%9A%0A1.%20Mapper%E4%B8%AD%E6%89%80%E6%9C%89%E7%9A%84%E6%9F%A5%E8%AF%A2%E7%BB%93%E6%9E%9C%E5%B0%86%E4%BC%9A%E8%A2%AB%E7%BC%93%E5%AD%98%E3%80%82%0A2.%20insert%E3%80%81update%E3%80%81delete%E8%AF%AD%E5%8F%A5%E4%BC%9A%E5%88%B7%E6%96%B0%E7%BC%93%E5%AD%98%E3%80%82%0A3.%20%E9%BB%98%E8%AE%A4%E4%BD%BF%E7%94%A8LRU%E7%AE%97%E6%B3%95%E6%9D%A5%E5%88%B7%E6%96%B0%E7%BC%93%E5%AD%98%E3%80%82%0A4.%20%E7%BC%93%E5%AD%98%E4%B8%8D%E4%BC%9A%E5%AE%9A%E6%97%B6%E8%BF%9B%E8%A1%8C%E5%88%B7%E6%96%B0%E3%80%82%0A5.%20%E7%BC%93%E5%AD%98%E5%AF%B9%E8%B1%A1%E7%9A%84%E4%B8%8A%E9%99%90%E9%BB%98%E8%AE%A4%E8%AE%BE%E7%BD%AE%E4%B8%BA1024%EF%BC%88%E4%B8%8D%E7%AE%A1%E8%BF%94%E5%9B%9E%E7%9A%84%E7%BB%93%E6%9E%9C%E6%98%AF%E5%AF%B9%E8%B1%A1%E5%92%8C%E8%BF%98%E6%98%AF%E5%88%97%E8%A1%A8%EF%BC%89%0A6.%20%E7%BC%93%E5%AD%98%E7%BB%93%E6%9E%9C%E6%98%AF%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%E3%80%82%0A%0A**%E5%85%A8%E5%B1%80%E9%85%8D%E7%BD%AE**%0A%0A*%20cacheEnabled%EF%BC%8C%E5%8F%AF%E9%80%89%E9%A1%B9%E4%B8%BAtrue%E6%88%96%E8%80%85false%EF%BC%8C%E9%BB%98%E8%AE%A4%E4%B8%BAtrue%E3%80%82%E5%BD%93%E8%AE%BE%E7%BD%AE%E4%B8%BAfalse%E6%97%B6%EF%BC%8CSQL%E6%89%A7%E8%A1%8C%E5%99%A8%E7%94%B1%E9%BB%98%E8%AE%A4%E7%9A%84CachingExecutor%E8%BD%AC%E5%8F%98%E4%B8%BASimpleExecutor%EF%BC%8C%E8%80%8C%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E5%8F%88%E4%BE%9D%E8%B5%96%E4%BA%8ECachingExecutor%E6%89%A7%E8%A1%8C%E5%99%A8%E3%80%82%E5%BA%9F%E8%AF%9D%E4%BA%86%E8%BF%99%E4%B9%88%E5%85%B6%E5%AE%9E%E5%B0%B1%E6%98%AF%E8%AE%BE%E7%BD%AE%E4%B8%BAfalse%E6%97%B6%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E5%B0%B1%E6%B2%A1%E4%BA%86%E3%80%82%0A*%20localCacheScope%EF%BC%8C%E5%8F%AF%E9%80%89%E9%A1%B9%E4%B8%BASESSION%20%E6%88%96%E8%80%85STATEMENT%EF%BC%8C%E9%BB%98%E8%AE%A4%E4%B8%BASESSION%E3%80%82%E5%A6%82%E6%9E%9C%E9%85%8D%E7%BD%AE%E4%B8%BASTATEMENT%E6%9C%AC%E5%9C%B0%E4%BC%9A%E8%AF%9D%E4%BB%85%E7%94%A8%E5%9C%A8%E8%AF%AD%E5%8F%A5%E6%89%A7%E8%A1%8C%E4%B8%8A%EF%BC%88%E5%B5%8C%E5%A5%97%E6%9F%A5%E8%AF%A2%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E5%88%B0%E7%BC%93%E5%AD%98%EF%BC%89%EF%BC%8C%E5%AF%B9%E7%9B%B8%E5%90%8C%20SqlSession%20%E7%9A%84%E4%B8%8D%E5%90%8C%E8%B0%83%E7%94%A8%E5%B0%86%E4%B8%8D%E4%BC%9A%E5%85%B1%E4%BA%AB%E6%95%B0%E6%8D%AE%E3%80%82%0A%0A**Mapper%E4%B8%AD%E7%9A%84%E9%85%8D%E7%BD%AE**%0A%0A*%20%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87cache%E5%AE%9A%E5%88%B6%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%E7%9A%84%E8%A1%8C%E4%B8%BA%EF%BC%8C%E5%85%B7%E4%BD%93%E7%9A%84%E9%80%89%E9%A1%B9%E5%8F%AF%E4%BB%A5%E5%8F%82%E7%85%A7%E4%B8%8A%E9%9D%A2%E8%AF%A6%E7%BB%86%E6%8F%8F%E8%BF%B0%E3%80%82%0A*%20%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%3Ccache-ref%20namespace%3D%22%22%2F%3E%E6%9D%A5%E5%BC%95%E7%94%A8%E5%85%B6%E4%BB%96Mapper%E7%9A%84%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98%0A*%20SQL%E8%AF%AD%E5%8F%A5%E4%B8%AD%E7%9A%84flushCache%3D%22true%22%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%90%86%E8%A7%A3%E4%B8%BA%E6%AF%8F%E6%AC%A1%E6%89%A7%E8%A1%8CSQL%E4%B9%8B%E5%89%8D%E9%83%BD%E4%BC%9A%E4%B8%BB%E5%8A%A8%E6%B8%85%E7%A9%BA%E7%BC%93%E5%AD%98%EF%BC%8C%E7%84%B6%E5%90%8E%E5%86%8D%E4%BB%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%AD%E6%9F%A5%E8%AF%A2%E7%BB%93%E6%9E%9C%EF%BC%8C%E5%9C%A8%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E4%B8%AD%E9%BB%98%E8%AE%A4%E5%80%BC%E4%B8%BAfalse%EF%BC%8C%E5%85%B6%E4%BB%96%E6%83%85%E5%86%B5%E4%B8%8B%E4%B8%BAtrue%E3%80%82%0A*%20SQL%E8%AF%AD%E5%8F%A5%E4%B8%AD%E7%9A%84useCache%3D%22true%22%EF%BC%8C%E6%AF%8F%E6%AC%A1%E6%89%A7%E8%A1%8C%E8%BF%99%E6%9D%A1SQL%E7%9A%84%E6%97%B6%E5%80%99%E9%83%BD%E4%BC%9A%E4%B8%BB%E5%8A%A8%E7%9A%84%E4%BD%BF%E7%94%A8%E7%BC%93%E5%AD%98%EF%BC%8C%E5%9C%A8%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E4%B8%AD%E9%BB%98%E8%AE%A4%E5%80%BC%E4%B8%BAtrue%2C%E5%85%B6%E4%BB%96%E6%83%85%E5%86%B5%E4%B8%8B%E4%B8%BAfalse%E3%80%82%0A%0A**%E7%BC%93%E5%AD%98%E4%BD%BF%E7%94%A8%E6%B5%81%E7%A8%8B**%0A%0A!%5B67b08df3f65c246ed8cf90c36eb3ba44.png%5D(en-resource%3A%2F%2Fdatabase%2F1211%3A1)%0A%0A%0A%23%23%23%23%20%E4%BA%94%20Mybatis%E6%8F%92%E4%BB%B6%0A%26emsp%3B%26emsp%3BMyBatis%20%E5%85%81%E8%AE%B8%E4%BD%A0%E5%9C%A8%E5%B7%B2%E6%98%A0%E5%B0%84%E8%AF%AD%E5%8F%A5%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%E4%B8%AD%E7%9A%84%E6%9F%90%E4%B8%80%E7%82%B9%E8%BF%9B%E8%A1%8C%E6%8B%A6%E6%88%AA%E8%B0%83%E7%94%A8%E3%80%82%E9%BB%98%E8%AE%A4%E6%83%85%E5%86%B5%E4%B8%8B%EF%BC%8CMyBatis%20%E5%85%81%E8%AE%B8%E4%BD%BF%E7%94%A8%E6%8F%92%E4%BB%B6%E6%9D%A5%E6%8B%A6%E6%88%AA%E7%9A%84%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8%E5%8C%85%E6%8B%AC%EF%BC%9A%0A%0A1.%20%60Executor%20(update%2C%20query%2C%20flushStatements%2C%20commit%2C%20rollback%2C%20getTransaction%2C%20close%2C%20isClosed)%60%EF%BC%8C**Executor%20%E6%89%AE%E6%BC%94%E8%80%85%E6%89%A7%E8%A1%8C%E5%99%A8%E7%9A%84%E8%A7%92%E8%89%B2%EF%BC%8C%E6%98%AF%E6%89%80%E6%9C%89%E7%9A%84SQL%E6%89%A7%E8%A1%8C%E7%9A%84%E6%80%BB%E5%85%A5%E5%8F%A3%EF%BC%9B**%0A2.%20%60ParameterHandler%20(getParameterObject%2C%20setParameters)%60%EF%BC%8C**ParameterHandler%E8%B4%9F%E8%B4%A3%E5%AF%B9SQL%E8%AF%AD%E5%8F%A5%E7%9A%84%E8%BE%93%E5%85%A5%E5%8F%82%E6%95%B0%E8%BF%9B%E8%A1%8C%E5%A4%84%E7%90%86%EF%BC%9B**%0A3.%20%60StatementHandler%20(prepare%2C%20parameterize%2C%20batch%2C%20update%2C%20query)%60%EF%BC%8C**StatementHandler%20%E8%B4%9F%E8%B4%A3SQL%E8%AF%AD%E5%8F%A5%E6%89%A7%E8%A1%8C%E8%B0%83%E7%94%A8%E3%80%81Statement%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%E3%80%81%E8%B0%83%E7%94%A8ParameterHandler%E5%B0%86%E8%BE%93%E5%85%A5%E5%8F%82%E6%95%B0%E7%BB%91%E5%AE%9A%E5%88%B0Statement%E4%B8%8A%EF%BC%9B**%0A4.%20%60ResultSetHandler%20(handleResultSets%2C%20handleOutputParameters)%60%EF%BC%8C**ResultSetHandler%20%E5%AF%B9%E6%89%A7%E8%A1%8C%E7%9A%84%E7%BB%93%E6%9E%9C%E9%9B%86%E5%92%8C%E5%AE%9E%E4%BD%93%E8%BF%9B%E8%A1%8C%E6%98%A0%E5%B0%84%E5%A4%84%E7%90%86**%E3%80%82%0A%0A%26emsp%3B%26emsp%3B%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%87%BAMybatis%E5%9C%A8SQL%E6%89%A7%E8%A1%8C%E5%89%8D%E3%80%81%E8%BE%93%E5%85%A5%E5%8F%82%E6%95%B0%E5%A4%84%E7%90%86%E3%80%81SQL%E6%89%A7%E8%A1%8C%E3%80%81%E7%BB%93%E6%9E%9C%E9%9B%86%E5%A4%84%E7%90%86%E7%AD%89%E8%BF%99%E4%BA%9B%E9%87%8D%E8%A6%81%E7%9A%84%E6%AD%A5%E9%AA%A4%E4%B8%AD%E9%83%BD%E9%A2%84%E7%95%99%E4%BA%86%E4%B8%B0%E5%AF%8C%E7%9A%84%E6%8B%A6%E6%88%AA%E7%82%B9%E3%80%82%E5%8D%B3%E4%BE%BF%E6%88%91%E4%BB%AC%E5%B9%B3%E6%97%B6%E7%94%A8%E5%88%B0%E7%9A%84%E5%88%86%E9%A1%B5%E6%8F%92%E4%BB%B6PageHelper%E4%B9%9F%E6%98%AF%E5%9F%BA%E4%BA%8E%E8%BF%99%E4%BA%9B%E5%9F%BA%E6%9C%AC%E7%9A%84%E6%8B%A6%E6%88%AA%E7%82%B9%E5%BC%80%E5%8F%91%E7%9A%84%E3%80%82%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E6%98%AF%E5%A6%82%E6%9E%9C%E5%9C%A8%E8%AF%95%E5%9B%BE%E4%BF%AE%E6%94%B9%E6%88%96%E9%87%8D%E5%86%99%E5%B7%B2%E6%9C%89%E6%96%B9%E6%B3%95%E7%9A%84%E8%A1%8C%E4%B8%BA%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E4%BD%A0%E5%BE%88%E5%8F%AF%E8%83%BD%E5%9C%A8%E7%A0%B4%E5%9D%8F%20MyBatis%20%E7%9A%84%E6%A0%B8%E5%BF%83%E6%A8%A1%E5%9D%97%E3%80%82%20%E8%BF%99%E4%BA%9B%E9%83%BD%E6%98%AF%E6%9B%B4%E4%BD%8E%E5%B1%82%E7%9A%84%E7%B1%BB%E5%92%8C%E6%96%B9%E6%B3%95%EF%BC%8C%E6%89%80%E4%BB%A5%E4%BD%BF%E7%94%A8%E6%8F%92%E4%BB%B6%E7%9A%84%E6%97%B6%E5%80%99%E8%A6%81%E7%89%B9%E5%88%AB%E5%BD%93%E5%BF%83%E3%80%82%0A%0A**%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8F%92%E4%BB%B6%EF%BC%9A**%0A%E9%80%9A%E8%BF%87%20MyBatis%20%E6%8F%90%E4%BE%9B%E7%9A%84%E5%BC%BA%E5%A4%A7%E6%9C%BA%E5%88%B6%EF%BC%8C%E4%BD%BF%E7%94%A8%E6%8F%92%E4%BB%B6%E6%98%AF%E9%9D%9E%E5%B8%B8%E7%AE%80%E5%8D%95%E7%9A%84%EF%BC%8C%E5%8F%AA%E9%9C%80%E5%AE%9E%E7%8E%B0%20Interceptor%20%E6%8E%A5%E5%8F%A3%EF%BC%8C%E5%B9%B6%E6%8C%87%E5%AE%9A%E6%83%B3%E8%A6%81%E6%8B%A6%E6%88%AA%E7%9A%84%E6%96%B9%E6%B3%95%E7%AD%BE%E5%90%8D%E5%8D%B3%E5%8F%AF%E3%80%82%0A%0A%60%60%60%20java%0A%2F**%0A%20*%20%40program%3A%20study-gupao%0A%20*%20%40description%3A%20%E5%BC%80%E5%8F%91%E4%B8%80%E4%B8%AA%E8%87%AA%E5%B7%B1%E7%9A%84%E7%9B%91%E6%8E%A7%E7%9A%84%E6%8F%92%E4%BB%B6%EF%BC%8C%E7%94%A8%E4%BA%8E%E7%9B%91%E6%8E%A7%E6%AF%8F%E4%B8%AASQL%E6%89%A7%E8%A1%8C%E7%9A%84%E7%9C%9F%E6%AD%A3%E6%97%B6%E9%95%BF%0A%20*%20%E4%BD%BF%E7%94%A8%E6%8F%92%E4%BB%B6%E6%98%AF%E9%9D%9E%E5%B8%B8%E7%AE%80%E5%8D%95%E7%9A%84%EF%BC%8C%E5%8F%AA%E9%9C%80%E5%AE%9E%E7%8E%B0%20Interceptor%20%E6%8E%A5%E5%8F%A3%EF%BC%8C%E5%B9%B6%E6%8C%87%E5%AE%9A%E6%83%B3%E8%A6%81%E6%8B%A6%E6%88%AA%E7%9A%84%E6%96%B9%E6%B3%95%E7%AD%BE%E5%90%8D%E5%8D%B3%E5%8F%AF%0A%20*%20%40author%3A%20shuai.li%0A%20*%20%40create%3A%202019-06-18%2021%3A09%0A%20**%2F%0A%0A%2F%2F%E6%8F%92%E4%BB%B6%E5%B0%86%E4%BC%9A%E6%8B%A6%E6%88%AA%E5%9C%A8%20Executor%20%E5%AE%9E%E4%BE%8B%E4%B8%AD%E6%89%80%E6%9C%89%E7%9A%84%20%E2%80%9Cupdate%E2%80%9D%2C%22query%22%20%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8%EF%BC%8C%20%E8%BF%99%E9%87%8C%E7%9A%84%20Executor%20%E6%98%AF%E8%B4%9F%E8%B4%A3%E6%89%A7%E8%A1%8C%E4%BD%8E%E5%B1%82%E6%98%A0%E5%B0%84%E8%AF%AD%E5%8F%A5%E7%9A%84%E5%86%85%E9%83%A8%E5%AF%B9%E8%B1%A1%0A%40Intercepts(%7B%0A%40Signature(type%20%3D%20Executor.class%2C%0A%20%20%20%20%20%20%20%20method%20%3D%20%22query%22%2C%0A%20%20%20%20%20%20%20%20args%20%3D%20%7BMappedStatement.class%2C%20Object.class%2C%20RowBounds.class%2C%20ResultHandler.class%7D)%2C%0A%40Signature(type%20%3D%20Executor.class%2C%0A%20%20%20%20%20%20%20%20method%20%3D%20%22query%22%2C%0A%20%20%20%20%20%20%20%20args%20%3D%20%7BMappedStatement.class%2C%20Object.class%2C%20RowBounds.class%2C%20ResultHandler.class%2C%20CacheKey.class%2C%20BoundSql.class%7D)%2C%0A%40Signature(type%20%3D%20Executor.class%2C%0A%20%20%20%20%20%20%20%20method%20%3D%20%22update%22%2C%0A%20%20%20%20%20%20%20%20args%20%3D%20%7BMappedStatement.class%2C%20Object.class%7D)%0A%7D)%0A%40Component%0Apublic%20class%20MonitorSQLExecutionTimePlugin%20implements%20Interceptor%20%7B%0A%0A%0A%20%20%20%20%40Override%0A%20%20%20%20public%20Object%20intercept(Invocation%20invocation)%20throws%20Throwable%20%7B%0A%20%20%20%20%20%20%20%20%2F%2F%E8%8E%B7%E5%8F%96invocation%E4%B8%AD%E6%89%A7%E8%A1%8C%E7%9A%84sql%0A%20%20%20%20%20%20%20%20Object%5B%5D%20args%20%3D%20invocation.getArgs()%3B%0A%20%20%20%20%20%20%20%20MappedStatement%20mappedStatement%20%3D%20(MappedStatement)%20args%5B0%5D%3B%0A%20%20%20%20%20%20%20%20Object%20param%20%3D%20args%5B1%5D%3B%0A%20%20%20%20%20%20%20%20BoundSql%20boundSql%20%3D%20mappedStatement.getBoundSql(param)%3B%0A%20%20%20%20%20%20%20%20String%20sql%20%3D%20boundSql.getSql()%3B%0A%0A%20%20%20%20%20%20%20%20%2F%2F%E8%AE%A1%E7%AE%97%E6%97%B6%E9%97%B4%E5%B9%B6%E6%89%93%E5%8D%B0%0A%20%20%20%20%20%20%20%20long%20beginTime%20%3D%20System.currentTimeMillis()%3B%0A%20%20%20%20%20%20%20%20Object%20result%20%3D%20invocation.proceed()%3B%0A%20%20%20%20%20%20%20%20long%20endTime%20%3D%20System.currentTimeMillis()%3B%0A%20%20%20%20%20%20%20%20System.out.println(MessageFormat.format(%22sql%20%3A%20%7B0%7D%2C%20%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8F%92%E4%BB%B6elapsed%20time%20%7B1%7D%20ms%22%2C%20sql%2C%20endTime%20-%20beginTime))%3B%0A%20%20%20%20%20%20%20%20return%20result%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20%40Override%0A%20%20%20%20public%20Object%20plugin(Object%20o)%20%7B%0A%20%20%20%20%20%20%20%20return%20Plugin.wrap(o%2Cthis)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20%40Override%0A%20%20%20%20public%20void%20setProperties(Properties%20properties)%20%7B%0A%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%0A%0A%E4%B8%8A%E9%9D%A2%E7%9A%84%E6%8F%92%E4%BB%B6%E5%B0%86%E4%BC%9A%E6%8B%A6%E6%88%AA%E5%9C%A8%20Executor%20%E5%AE%9E%E4%BE%8B%E4%B8%AD%E6%89%80%E6%9C%89%E7%9A%84%20%E2%80%9Cupdate%E2%80%9D%EF%BC%8C%E2%80%9Cselect%E2%80%9D%20%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8%EF%BC%8C%20%E8%BF%99%E9%87%8C%E7%9A%84%20Executor%20%E6%98%AF%E8%B4%9F%E8%B4%A3%E6%89%A7%E8%A1%8C%E4%BD%8E%E5%B1%82%E6%98%A0%E5%B0%84%E8%AF%AD%E5%8F%A5%E7%9A%84%E5%86%85%E9%83%A8%E5%AF%B9%E8%B1%A1%E3%80%82%E5%9C%A8springboot%E9%A1%B9%E7%9B%AE%E4%B8%AD%20%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8F%92%E4%BB%B6%E4%B8%8Emybatis%E9%9B%86%E6%88%90%E5%8F%AA%E9%9C%80%E8%A6%81%E5%9C%A8%E6%8F%92%E4%BB%B6%E7%B1%BB%E4%B8%8A%E6%89%93%E4%B8%8A%40Component%20%E5%8D%B3%E5%8F%AF%0Ahttps%3A%2F%2Fgithub.com%2FEvanDylan%2Farticle%2Fblob%2Fmaster%2Fmybatis%2Fplugin.md%20%0A%0A%23%23%23%23%20%E5%85%AD%20%E5%AD%97%E6%AE%B5%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86%E5%99%A8%EF%BC%88typeHandlers%EF%BC%89%0A%E6%97%A0%E8%AE%BA%E6%98%AF%20MyBatis%20%E5%9C%A8%E9%A2%84%E5%A4%84%E7%90%86%E8%AF%AD%E5%8F%A5%EF%BC%88PreparedStatement%EF%BC%89%E4%B8%AD%E8%AE%BE%E7%BD%AE%E4%B8%80%E4%B8%AA%E5%8F%82%E6%95%B0%E6%97%B6%EF%BC%8C%E8%BF%98%E6%98%AF%E4%BB%8E%E7%BB%93%E6%9E%9C%E9%9B%86%E4%B8%AD%E5%8F%96%E5%87%BA%E4%B8%80%E4%B8%AA%E5%80%BC%E6%97%B6%EF%BC%8C%20%E9%83%BD%E4%BC%9A%E7%94%A8%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86%E5%99%A8%E5%B0%86%E8%8E%B7%E5%8F%96%E7%9A%84%E5%80%BC%E4%BB%A5%E5%90%88%E9%80%82%E7%9A%84%E6%96%B9%E5%BC%8F%E8%BD%AC%E6%8D%A2%E6%88%90%20Java%20%E7%B1%BB%E5%9E%8B%0A!%5B3d3fd2e78918608c8dadf38d0fb3f4f5.png%5D(en-resource%3A%2F%2Fdatabase%2F1225%3A1)%0A%E4%BD%A0%E5%8F%AF%E4%BB%A5%E9%87%8D%E5%86%99%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86%E5%99%A8%E6%88%96%E5%88%9B%E5%BB%BA%E4%BD%A0%E8%87%AA%E5%B7%B1%E7%9A%84%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86%E5%99%A8%E6%9D%A5%E5%A4%84%E7%90%86%E4%B8%8D%E6%94%AF%E6%8C%81%E7%9A%84%E6%88%96%E9%9D%9E%E6%A0%87%E5%87%86%E7%9A%84%E7%B1%BB%E5%9E%8B%2C%C2%A0%E5%85%B7%E4%BD%93%E5%81%9A%E6%B3%95%E4%B8%BA%EF%BC%9A%E5%AE%9E%E7%8E%B0%C2%A0org.apache.ibatis.type.TypeHandler%C2%A0%E6%8E%A5%E5%8F%A3%EF%BC%8C%20%E6%88%96%E7%BB%A7%E6%89%BF%E4%B8%80%E4%B8%AA%E5%BE%88%E4%BE%BF%E5%88%A9%E7%9A%84%E7%B1%BB%C2%A0org.apache.ibatis.type.BaseTypeHandler%EF%BC%8C%20%E7%84%B6%E5%90%8E%E5%8F%AF%E4%BB%A5%E9%80%89%E6%8B%A9%E6%80%A7%E5%9C%B0%E5%B0%86%E5%AE%83%E6%98%A0%E5%B0%84%E5%88%B0%E4%B8%80%E4%B8%AA%20JDBC%20%E7%B1%BB%E5%9E%8B%E3%80%82%0A%60%60%60%20java%0A%2F**%0A%20*%20%40program%3A%20study-gupao%0A%20*%20%40description%3A%20%E4%BD%A0%E5%8F%AF%E4%BB%A5%E9%87%8D%E5%86%99%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86%E5%99%A8%E6%88%96%E5%88%9B%E5%BB%BA%E4%BD%A0%E8%87%AA%E5%B7%B1%E7%9A%84%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86%E5%99%A8%E6%9D%A5%E5%A4%84%E7%90%86%E4%B8%8D%E6%94%AF%E6%8C%81%E7%9A%84%E6%88%96%E9%9D%9E%E6%A0%87%E5%87%86%E7%9A%84%E7%B1%BB%E5%9E%8B%0A%20*%20%E5%85%B7%E4%BD%93%E5%81%9A%E6%B3%95%E4%B8%BA%EF%BC%9A%E5%AE%9E%E7%8E%B0%20org.apache.ibatis.type.TypeHandler%20%E6%8E%A5%E5%8F%A3%EF%BC%8C%20%E6%88%96%E7%BB%A7%E6%89%BF%E4%B8%80%E4%B8%AA%E5%BE%88%E4%BE%BF%E5%88%A9%E7%9A%84%E7%B1%BB%20org.apache.ibatis.type.BaseTypeHandler%EF%BC%8C%0A%20*%20%E7%84%B6%E5%90%8E%E5%8F%AF%E4%BB%A5%E9%80%89%E6%8B%A9%E6%80%A7%E5%9C%B0%E5%B0%86%E5%AE%83%E6%98%A0%E5%B0%84%E5%88%B0%E4%B8%80%E4%B8%AA%20JDBC%20%E7%B1%BB%E5%9E%8B%E3%80%82%0A%20*%20%40author%3A%20shuai.li%0A%20*%20%40create%3A%202019-06-17%2021%3A44%0A%20**%2F%0A%40MappedJdbcTypes(JdbcType.VARCHAR)%0Apublic%20class%20ExampleTypeHandler%20extends%20BaseTypeHandler%3CString%3E%20%7B%0A%20%20%20%20%40Override%0A%20%20%20%20public%20void%20setNonNullParameter(PreparedStatement%20preparedStatement%2C%20int%20i%2C%20String%20s%2C%20JdbcType%20jdbcType)%20throws%20SQLException%20%7B%0A%20%20%20%20%20%20%20%20preparedStatement.setString(i%2C%20s)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20%40Override%0A%20%20%20%20public%20String%20getNullableResult(ResultSet%20resultSet%2C%20String%20s)%20throws%20SQLException%20%7B%0A%20%20%20%20%20%20%20%20return%20resultSet.getString(s)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20%40Override%0A%20%20%20%20public%20String%20getNullableResult(ResultSet%20resultSet%2C%20int%20i)%20throws%20SQLException%20%7B%0A%20%20%20%20%20%20%20%20return%20resultSet.getString(i)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20%40Override%0A%20%20%20%20public%20String%20getNullableResult(CallableStatement%20callableStatement%2C%20int%20i)%20throws%20SQLException%20%7B%0A%20%20%20%20%20%20%20%20return%20callableStatement.getString(i)%3B%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%0A%0A%E4%BD%BF%E7%94%A8%E4%B8%8A%E8%BF%B0%E7%9A%84%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86%E5%99%A8%E5%B0%86%E4%BC%9A%E8%A6%86%E7%9B%96%E5%B7%B2%E7%BB%8F%E5%AD%98%E5%9C%A8%E7%9A%84%E5%A4%84%E7%90%86%20Java%20%E7%9A%84%20String%20%E7%B1%BB%E5%9E%8B%E5%B1%9E%E6%80%A7%E5%92%8C%20VARCHAR%20%E5%8F%82%E6%95%B0%E5%8F%8A%E7%BB%93%E6%9E%9C%E7%9A%84%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86%E5%99%A8%E3%80%82%20%E8%A6%81%E6%B3%A8%E6%84%8F%20MyBatis%20%E4%B8%8D%E4%BC%9A%E9%80%9A%E8%BF%87%E7%AA%A5%E6%8E%A2%E6%95%B0%E6%8D%AE%E5%BA%93%E5%85%83%E4%BF%A1%E6%81%AF%E6%9D%A5%E5%86%B3%E5%AE%9A%E4%BD%BF%E7%94%A8%E5%93%AA%E7%A7%8D%E7%B1%BB%E5%9E%8B%EF%BC%8C%E6%89%80%E4%BB%A5%E4%BD%A0%E5%BF%85%E9%A1%BB%E5%9C%A8%E5%8F%82%E6%95%B0%E5%92%8C%E7%BB%93%E6%9E%9C%E6%98%A0%E5%B0%84%E4%B8%AD%E6%8C%87%E6%98%8E%E9%82%A3%E6%98%AF%20VARCHAR%20%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%AD%97%E6%AE%B5%EF%BC%8C%20%E4%BB%A5%E4%BD%BF%E5%85%B6%E8%83%BD%E5%A4%9F%E7%BB%91%E5%AE%9A%E5%88%B0%E6%AD%A3%E7%A1%AE%E7%9A%84%E7%B1%BB%E5%9E%8B%E5%A4%84%E7%90%86%E5%99%A8%E4%B8%8A%E3%80%82%E8%BF%99%E6%98%AF%E5%9B%A0%E4%B8%BA%20MyBatis%20%E7%9B%B4%E5%88%B0%E8%AF%AD%E5%8F%A5%E8%A2%AB%E6%89%A7%E8%A1%8C%E6%97%B6%E6%89%8D%E6%B8%85%E6%A5%9A%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E3%80%82%0A%0A%0A%23%23%23%23%20%E4%B8%83%20%E5%85%B6%E4%BB%96%E7%9F%A5%E8%AF%86%E7%82%B9%0A%23%23%23%23%23%201.%20%3C!%5BCDATA%5B%22%20...%20%22%5D%5D%3E%20%E8%AF%AD%E6%B3%95%0AXML%20%E5%85%83%E7%B4%A0%E4%B8%AD%EF%BC%8C%22%3C%22%20%E5%92%8C%20%22%26%22%20%E6%98%AF%E9%9D%9E%E6%B3%95%E7%9A%84%EF%BC%8C%22%3C%22%20%E4%BC%9A%E4%BA%A7%E7%94%9F%E9%94%99%E8%AF%AF%EF%BC%8C%E5%9B%A0%E4%B8%BAxml%E8%A7%A3%E6%9E%90%E5%99%A8%E4%BC%9A%E6%8A%8A%E8%AF%A5%E5%AD%97%E7%AC%A6%E8%A7%A3%E9%87%8A%E4%B8%BA%E6%96%B0%E5%85%83%E7%B4%A0%E7%9A%84%E5%BC%80%E5%A7%8B%E3%80%82%22%26%22%20%E4%B9%9F%E4%BC%9A%E4%BA%A7%E7%94%9F%E9%94%99%E8%AF%AF%EF%BC%8C%E5%9B%A0%E4%B8%BA%E8%A7%A3%E6%9E%90%E5%99%A8%E4%BC%9A%E6%8A%8A%E8%AF%A5%E5%AD%97%E7%AC%A6%E8%A7%A3%E9%87%8A%E4%B8%BA%E5%AD%97%E7%AC%A6%E5%AE%9E%E4%BD%93%E7%9A%84%E5%BC%80%E5%A7%8B%E3%80%82%E8%80%8C%E8%84%9A%E6%9C%AC%E4%BB%A3%E7%A0%81%E4%B8%AD%E7%BB%8F%E5%B8%B8%E5%90%AB%E6%9C%89%E8%BF%99%E4%BA%9B%E7%89%B9%E6%AE%8A%E5%AD%97%E7%AC%A6%EF%BC%8C%E4%B8%BA%E4%BA%86%E9%81%BF%E5%85%8D%E9%94%99%E8%AF%AF%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%B0%86%E8%84%9A%E6%9C%AC%E4%BB%A3%E7%A0%81%E5%AE%9A%E4%B9%89%E5%9C%A8%20CDATA%E6%A0%87%E7%AD%BE%E4%B8%AD%EF%BC%8CCDATA%20%E4%B8%AD%E7%9A%84%E6%89%80%E6%9C%89%E5%86%85%E5%AE%B9%E9%83%BD%E4%BC%9A%E8%A2%AB%E8%A7%A3%E6%9E%90%E5%99%A8%E5%BF%BD%E7%95%A5%E3%80%82CDATA%20%E7%94%B1%20%22%3C!%5BCDATA%5B%22%20%E5%BC%80%E5%A7%8B%EF%BC%8C%E7%94%B1%20%22%5D%5D%3E%22%20%E7%BB%93%E6%9D%9F%0A%0A%E4%BE%8B%E5%A6%82%E4%B8%80%E4%B8%AA%E6%89%B9%E9%87%8F%E6%9B%B4%E6%96%B0%EF%BC%8C%E5%B0%86%E6%97%B6%E9%97%B4%E4%BD%9C%E4%B8%BA%E6%AF%94%E8%BE%83%E6%9D%A1%E4%BB%B6%E7%9A%84%E4%BE%8B%E5%AD%90%EF%BC%9A%0A%60%60%60xml%0A%3Cupdate%20id%3D%22batchUpdate%22%20parameterType%3D%22com.ghub.middleware.repository.entity.TradingSyncProduct%22%3E%0A%20%20%3Cforeach%20item%3D%22item%22%20collection%3D%22list%22%20separator%3D%22%3B%22%3E%0A%20%20%20%20update%20trading_sync_product%0A%20%20%20%20%3Cset%3E%0A%20%20%20%20%20%20%3Cif%20test%3D%22item.syncStatus%20!%3D%20null%22%3E%0A%20%20%20%20%20%20%20%20sync_status%20%3D%20%23%7Bitem.syncStatus%2CjdbcType%3DINTEGER%7D%2C%0A%20%20%20%20%20%20%3C%2Fif%3E%0A%20%20%20%20%3C%2Fset%3E%0A%20%20%20%20WHERE%20sku_code%20%3D%20%23%7Bitem.skuCode%2CjdbcType%3DVARCHAR%7D%20AND%0A%20%20%20%20%3C!%5BCDATA%5B%20sync_time%20%3C%3D%20%23%7Bitem.syncTime%2CjdbcType%3DDATE%7D%20%5D%5D%3E%0A%20%20%3C%2Fforeach%3E%0A%3C%2Fupdate%3E%0A%60%60%60%0A%23%23%23%23%23%202.%20%23%7B%7D%20%E5%92%8C%20%24%7B%7D%E7%9A%84%E5%8C%BA%E5%88%AB%0A*%20Mybatis%20%E5%9C%A8%E5%A4%84%E7%90%86%23%7B%7D%E6%97%B6%EF%BC%8C%E4%BC%9A%E5%B0%86%20sql%20%E4%B8%AD%E7%9A%84%23%7B%7D%E6%9B%BF%E6%8D%A2%E4%B8%BA%3F%E5%8F%B7%EF%BC%8C%E7%84%B6%E5%90%8E%E8%B0%83%E7%94%A8%20PreparedStatement%20%E7%9A%84set%20%E6%96%B9%E6%B3%95%E6%9D%A5%E8%B5%8B%E5%80%BC%EF%BC%88%E4%BC%9A%E6%8A%8A%EF%BC%9F%E5%8F%B7%E6%9B%BF%E6%8D%A2%E4%B8%BA%E5%8F%82%E6%95%B0%E5%80%BC%E7%9A%84%E5%90%8C%E6%97%B6%E4%BC%9A%E5%8A%A0%E4%B8%8A%E8%BD%AC%E4%B9%89%E7%AC%A6%EF%BC%8C%E4%BB%8E%E8%80%8C%E9%81%BF%E5%85%8D%E4%BA%86sql%E6%B3%A8%E5%85%A5%EF%BC%89%0A*%20%24%7B%7D%E4%BB%85%E4%BB%85%E6%98%AF%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%9B%BF%E6%8D%A2%EF%BC%8C%E6%97%A0%E6%B3%95%E9%81%BF%E5%85%8Dsql%E6%B3%A8%E5%85%A5%0A%23%23%23%23%23%203.%20%E7%BB%93%E6%9E%9C%E9%9B%86%E6%98%A0%E5%B0%84%EF%BC%88%E6%95%B0%E6%8D%AE%E5%BA%93%E5%AD%97%E6%AE%B5%E5%92%8C%E5%AE%9E%E4%BD%93%E5%AD%97%E6%AE%B5%E6%98%A0%E5%B0%84%EF%BC%89%0A**1.%20sql%E8%AF%AD%E5%8F%A5%E4%B8%AD%20as%20%E5%AE%9A%E4%B9%89%E5%88%AB%E5%90%8D%0A2.%20%E9%80%9A%E8%BF%87%C2%A0%3CresultMap%3E%C2%A0%E6%9D%A5%E6%98%A0%E5%B0%84%E5%AD%97%E6%AE%B5%E5%90%8D%E5%92%8C%E5%AE%9E%E4%BD%93%E7%B1%BB%E5%B1%9E%E6%80%A7%E5%90%8D%E7%9A%84%E4%B8%80%E4%B8%80%E5%AF%B9%E5%BA%94%E7%9A%84%E5%85%B3%E7%B3%BB**%0A%60%60%60xml%0A%3Cmapper%20namespace%3D%22com.shuailee.dao.IUserDao%22%3E%0A%20%20%20%20%3Cselect%20id%3D%22getOrder%22%20parameterType%3D%22int%22%20resultMap%3D%22orderresultmap%22%3E%0A%20%20%20%20%20%20%20%20select%20*%20from%20orders%20where%20order_id%3D%23%7Bid%7D%0A%20%20%20%20%3C%2Fselect%3E%0A%20%20%20%20%3CresultMap%20type%3D%E2%80%9Dme.gacl.domain.order%E2%80%9D%20id%3D%E2%80%9Dorderresultmap%E2%80%9D%3E%0A%20%20%20%20%20%20%20%20%3C!%E2%80%93%E7%94%A8%20id%20%E5%B1%9E%E6%80%A7%E6%9D%A5%E6%98%A0%E5%B0%84%E4%B8%BB%E9%94%AE%E5%AD%97%E6%AE%B5%E2%80%93%3E%0A%20%20%20%20%20%20%20%20%3Cid%20property%3D%E2%80%9Did%E2%80%9D%20column%3D%E2%80%9Dorder_id%E2%80%9D%3E%0A%20%20%20%20%20%20%20%20%3C!%E2%80%93%E7%94%A8%20result%20%E5%B1%9E%E6%80%A7%E6%9D%A5%E6%98%A0%E5%B0%84%E9%9D%9E%E4%B8%BB%E9%94%AE%E5%AD%97%E6%AE%B5%EF%BC%8Cproperty%20%E4%B8%BA%E5%AE%9E%E4%BD%93%E7%B1%BB%E5%B1%9E%E6%80%A7%E5%90%8D%EF%BC%8Ccolumn%0A%20%20%20%20%20%20%20%20%E4%B8%BA%E6%95%B0%E6%8D%AE%E8%A1%A8%E4%B8%AD%E7%9A%84%E5%B1%9E%E6%80%A7%E2%80%93%3E%0A%20%20%20%20%20%20%20%20%3Cresult%20property%20%3D%20%E2%80%9Corderno%E2%80%9D%20column%20%3D%E2%80%9Dorder_no%E2%80%9D%2F%3E%0A%20%20%20%20%20%20%20%20%3Cresult%20property%3D%E2%80%9Dprice%E2%80%9D%20column%3D%E2%80%9Dorder_price%E2%80%9D%20%2F%3E%0A%20%20%20%20%3C%2FreslutMap%3E%0A%3C%2Fmapper%3E%0A%60%60%60%0A**3.%20%E8%87%AA%E5%8A%A8%E6%98%A0%E5%B0%84**%0A*%20%E5%BD%93%E8%87%AA%E5%8A%A8%E6%98%A0%E5%B0%84%E6%9F%A5%E8%AF%A2%E7%BB%93%E6%9E%9C%E6%97%B6%EF%BC%8CMyBatis%20%E4%BC%9A%E8%8E%B7%E5%8F%96%E7%BB%93%E6%9E%9C%E4%B8%AD%E8%BF%94%E5%9B%9E%E7%9A%84%E5%88%97%E5%90%8D%E5%B9%B6%E5%9C%A8%20Java%20%E7%B1%BB%E4%B8%AD%E6%9F%A5%E6%89%BE%E7%9B%B8%E5%90%8C%E5%90%8D%E5%AD%97%E7%9A%84%E5%B1%9E%E6%80%A7%EF%BC%88%E5%BF%BD%E7%95%A5%E5%A4%A7%E5%B0%8F%E5%86%99%EF%BC%89%E3%80%82%20%E8%BF%99%E6%84%8F%E5%91%B3%E7%9D%80%E5%A6%82%E6%9E%9C%E5%8F%91%E7%8E%B0%E4%BA%86%20ID%20%E5%88%97%E5%92%8C%20id%20%E5%B1%9E%E6%80%A7%EF%BC%8CMyBatis%20%E4%BC%9A%E5%B0%86%E5%88%97%20ID%20%E7%9A%84%E5%80%BC%E8%B5%8B%E7%BB%99%20id%20%E5%B1%9E%E6%80%A7%E3%80%82%0A*%20%E9%80%9A%E5%B8%B8%E6%95%B0%E6%8D%AE%E5%BA%93%E5%88%97%E4%BD%BF%E7%94%A8%E5%A4%A7%E5%86%99%E5%AD%97%E6%AF%8D%E7%BB%84%E6%88%90%E7%9A%84%E5%8D%95%E8%AF%8D%E5%91%BD%E5%90%8D%EF%BC%8C%E5%8D%95%E8%AF%8D%E9%97%B4%E7%94%A8%E4%B8%8B%E5%88%92%E7%BA%BF%E5%88%86%E9%9A%94%EF%BC%9B%E8%80%8C%20Java%20%E5%B1%9E%E6%80%A7%E4%B8%80%E8%88%AC%E9%81%B5%E5%BE%AA%E9%A9%BC%E5%B3%B0%E5%91%BD%E5%90%8D%E6%B3%95%E7%BA%A6%E5%AE%9A%E3%80%82%E4%B8%BA%E4%BA%86%E5%9C%A8%E8%BF%99%E4%B8%A4%E7%A7%8D%E5%91%BD%E5%90%8D%E6%96%B9%E5%BC%8F%E4%B9%8B%E9%97%B4%E5%90%AF%E7%94%A8%E8%87%AA%E5%8A%A8%E6%98%A0%E5%B0%84%EF%BC%8C%E9%9C%80%E8%A6%81%E5%B0%86%20mapUnderscoreToCamelCase%20%E8%AE%BE%E7%BD%AE%E4%B8%BA%20true%0A*%20%E7%94%9A%E8%87%B3%E5%9C%A8%E6%8F%90%E4%BE%9B%E4%BA%86%E7%BB%93%E6%9E%9C%E6%98%A0%E5%B0%84%E5%90%8E%EF%BC%8C%E8%87%AA%E5%8A%A8%E6%98%A0%E5%B0%84%E4%B9%9F%E8%83%BD%E5%B7%A5%E4%BD%9C%E3%80%82%E5%9C%A8%E8%BF%99%E7%A7%8D%E6%83%85%E5%86%B5%E4%B8%8B%EF%BC%8C%E5%AF%B9%E4%BA%8E%E6%AF%8F%E4%B8%80%E4%B8%AA%E7%BB%93%E6%9E%9C%E6%98%A0%E5%B0%84%EF%BC%8C%E5%9C%A8%20ResultSet%20%E5%87%BA%E7%8E%B0%E7%9A%84%E5%88%97%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%9C%89%E8%AE%BE%E7%BD%AE%E6%89%8B%E5%8A%A8%E6%98%A0%E5%B0%84%EF%BC%8C%E5%B0%86%E8%A2%AB%E8%87%AA%E5%8A%A8%E6%98%A0%E5%B0%84%E3%80%82%E5%9C%A8%E8%87%AA%E5%8A%A8%E6%98%A0%E5%B0%84%E5%A4%84%E7%90%86%E5%AE%8C%E6%AF%95%E5%90%8E%EF%BC%8C%E5%86%8D%E5%A4%84%E7%90%86%E6%89%8B%E5%8A%A8%E6%98%A0%E5%B0%84%E3%80%82%0A%0A%23%23%23%23%23%23%204%20Mybatis%E6%97%A5%E5%BF%97%0AMybatis%20%E7%9A%84%E5%86%85%E7%BD%AE%E6%97%A5%E5%BF%97%E5%B7%A5%E5%8E%82%E6%8F%90%E4%BE%9B%E6%97%A5%E5%BF%97%E5%8A%9F%E8%83%BD%EF%BC%8C%E5%86%85%E7%BD%AE%E6%97%A5%E5%BF%97%E5%B7%A5%E5%8E%82%E5%B0%86%E6%97%A5%E5%BF%97%E4%BA%A4%E7%BB%99%E4%BB%A5%E4%B8%8B%E5%85%B6%E4%B8%AD%E4%B8%80%E7%A7%8D%E5%B7%A5%E5%85%B7%E4%BD%9C%E4%BB%A3%E7%90%86%EF%BC%9A%60SLF4J%20%2CApache%20Commons%20Logging%20%EF%BC%8C%20Log4j2%20%EF%BC%8CLog4j%EF%BC%8C%20JDK%20logging%60%E5%AE%83%E4%BC%9A%E4%BD%BF%E7%94%A8%E7%AC%AC%E4%B8%80%E4%B8%AA%E6%9F%A5%E6%89%BE%E5%BE%97%E5%88%B0%E7%9A%84%E5%B7%A5%E5%85%B7%EF%BC%88%E6%8C%89%E4%B8%8A%E6%96%87%E5%88%97%E4%B8%BE%E7%9A%84%E9%A1%BA%E5%BA%8F%E6%9F%A5%E6%89%BE%EF%BC%89%E3%80%82%E5%A6%82%E6%9E%9C%E4%B8%80%E4%B8%AA%E9%83%BD%E6%9C%AA%E6%89%BE%E5%88%B0%EF%BC%8C%E6%97%A5%E5%BF%97%E5%8A%9F%E8%83%BD%E5%B0%B1%E4%BC%9A%E8%A2%AB%E7%A6%81%E7%94%A8%EF%BC%9B%E5%A6%82%E6%9E%9C%E4%BD%A0%E6%8C%87%E5%AE%9A%E4%BA%86%E6%97%A5%E5%BF%97%E4%BD%86%E6%98%AF%E6%B2%A1%E6%9C%89%E6%89%BE%E5%88%B0%EF%BC%8C%E5%88%99%E4%BC%9A%E5%BF%BD%E7%95%A5%E7%BB%A7%E7%BB%AD%E6%8C%89%E7%85%A7%E9%A1%BA%E5%BA%8F%E6%9F%A5%E6%89%BE%EF%BC%9B%0A%0A%23%23%23%23%20%E5%85%AB%20%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90%E9%85%8D%E7%BD%AE%0A%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90%E5%B0%B1%E6%98%AF%E9%85%8D%E7%BD%AE%E5%A4%9A%E4%B8%AA%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%EF%BC%8C%E4%B8%8D%E5%90%8C%E8%B4%A6%E5%8F%B7%E5%92%8C%E5%AF%86%E7%A0%81%EF%BC%8C%E6%AF%8F%E4%B8%AA%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%90%8C%E7%9A%84%E6%95%B0%E6%8D%AE%E6%BA%90%E3%80%82%0A%E8%BF%98%E8%A6%81%E6%89%8B%E5%8A%A8%E9%85%8D%E7%BD%AEDatasouce%E5%92%8CSqlsessionfactory%E3%80%82%E5%85%B6%E4%BB%96%E5%B0%B1%E5%92%8C%E5%8D%95%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BD%BF%E7%94%A8%E4%B8%80%E6%A0%B7%E4%BA%86%0A**Mybatis%E7%BB%93%E5%90%88Druid%E6%95%B0%E6%8D%AE%E8%BF%9E%E6%8E%A5%E6%B1%A0%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90%E9%85%8D%E7%BD%AE**%0A**1%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E9%85%8D%E7%BD%AE%E5%A4%9A%E6%95%B0%E6%8D%AE%E6%BA%90**%0A%60%60%60sh%0A%23%20%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE%0A%23%23%23%23%23%23%23%23%23%23%23%23%23%23%20%E6%95%B0%E6%8D%AE%E5%BA%93%201%23%23%23%23%23%23%23%23%23%23%23%23%23%0Aspring.datasource.db1.url%20%3D%20jdbc%3Amysql%3A%2F%2F127.0.0.1%3A4000%2Fdb1%3FuseUnicode%3Dtrue%26characterEncoding%3DUTF-8%26autoReconnect%3Dtrue%26autoReconnectForPools%3Dtrue%26noAccessToProcedureBodies%3Dtrue%0Aspring.datasource.db1.username%20%3D%20test%0Aspring.datasource.db1.password%20%3D%20123%0Aspring.datasource.db1.driver-class-name%20%3D%20com.mysql.jdbc.Driver%0Aspring.datasource.db1.initialSize%20%3D%205%0Aspring.datasource.db1.minIdle%20%3D%205%0Aspring.datasource.db1.maxActive%20%3D%20100%0A%23%20%E9%85%8D%E7%BD%AE%E8%8E%B7%E5%8F%96%E8%BF%9E%E6%8E%A5%E7%AD%89%E5%BE%85%E8%B6%85%E6%97%B6%E7%9A%84%E6%97%B6%E9%97%B4%0Aspring.datasource.db1.maxWait%20%3D%2060000%0A%23%20%E9%85%8D%E7%BD%AE%E9%97%B4%E9%9A%94%E5%A4%9A%E4%B9%85%E6%89%8D%E8%BF%9B%E8%A1%8C%E4%B8%80%E6%AC%A1%E6%A3%80%E6%B5%8B%EF%BC%8C%E6%A3%80%E6%B5%8B%E9%9C%80%E8%A6%81%E5%85%B3%E9%97%AD%E7%9A%84%E7%A9%BA%E9%97%B2%E8%BF%9E%E6%8E%A5%EF%BC%8C%E5%8D%95%E4%BD%8D%E6%98%AF%E6%AF%AB%E7%A7%92%0Aspring.datasource.db1.timeBetweenEvictionRunsMillis%20%3D%2060000%0A%23%20%E9%85%8D%E7%BD%AE%E4%B8%80%E4%B8%AA%E8%BF%9E%E6%8E%A5%E5%9C%A8%E6%B1%A0%E4%B8%AD%E6%9C%80%E5%B0%8F%E7%94%9F%E5%AD%98%E7%9A%84%E6%97%B6%E9%97%B4%EF%BC%8C%E5%8D%95%E4%BD%8D%E6%98%AF%E6%AF%AB%E7%A7%92%0Aspring.datasource.db1.minEvictableIdleTimeMillis%20%3D%20300000%0Aspring.datasource.db1.maxEvictableIdleTimeMillis%20%3D%201200000%0Aspring.datasource.db1.validationQuery%20%3D%20SELECT%201%20FROM%20DUAL%0Aspring.datasource.db1.testWhileIdle%20%3D%20true%0Aspring.datasource.db1.testOnBorrow%20%3D%20true%0Aspring.datasource.db1.testOnReturn%20%3D%20false%0Aspring.datasource.db1.keepAlive%20%3D%20true%0A%23%20%E6%89%93%E5%BC%80PSCache%EF%BC%8C%E5%B9%B6%E4%B8%94%E6%8C%87%E5%AE%9A%E6%AF%8F%E4%B8%AA%E8%BF%9E%E6%8E%A5%E4%B8%8APSCache%E7%9A%84%E5%A4%A7%E5%B0%8F%0Aspring.datasource.db1.poolPreparedStatements%20%3D%20true%0Aspring.datasource.db1.maxPoolPreparedStatementPerConnectionSize%20%3D%2020%0A%23%20%E9%85%8D%E7%BD%AE%E7%9B%91%E6%8E%A7%E7%BB%9F%E8%AE%A1%E6%8B%A6%E6%88%AA%E7%9A%84filters%EF%BC%8C%E5%8E%BB%E6%8E%89%E5%90%8E%E7%9B%91%E6%8E%A7%E7%95%8C%E9%9D%A2sql%E6%97%A0%E6%B3%95%E7%BB%9F%E8%AE%A1%EF%BC%8C'wall'%E7%94%A8%E4%BA%8E%E9%98%B2%E7%81%AB%E5%A2%99%0Aspring.datasource.db1.filters%20%3D%20stat%2Cwall%0A%23%20%E9%80%9A%E8%BF%87connectProperties%E5%B1%9E%E6%80%A7%E6%9D%A5%E6%89%93%E5%BC%80mergeSql%E5%8A%9F%E8%83%BD%EF%BC%9B%E6%85%A2SQL%E8%AE%B0%E5%BD%95%0Aspring.datasource.db1.connectionProperties%20%3D%20druid.stat.mergeSql%3Dtrue%3Bdruid.stat.slowSqlMillis%3D5000%0A%0Aspring.datasource.db1.mapperLocation%20%3D%20classpath*%3Adb1%2Fmapper%2F*.xml%0Aspring.datasource.db1.typeAliasesPackage%20%3D%20com.shuailee.repository.model.db1%0A%0A%0A%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%23%E6%95%B0%E6%8D%AE%E5%BA%93%202%23%23%23%23%23%23%23%23%23%0A%23%E6%95%B0%E6%8D%AE%E5%BA%93%202%20%E9%85%8D%E7%BD%AE%E5%8F%AA%E9%9C%80%E8%A6%81%E4%BF%AE%E6%94%B9%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E5%92%8C%E8%B4%A6%E5%8F%B7%E5%AF%86%E7%A0%81%E5%8D%B3%E5%8F%AF%0Aspring.datasource.db2.url%20%3D%20jdbc%3Amysql%3A%2F%2F127.0.0.1%3A4000%2Fdb2%3FuseUnicode%3Dtrue%26characterEncoding%3DUTF-8%26autoReconnect%3Dtrue%26autoReconnectForPools%3Dtrue%26noAccessToProcedureBodies%3Dtrue%0Aspring.datasource.db2.username%20%3D%20test2%0Aspring.datasource.db2.password%20%3D%201234%0A...%0Aspring.datasource.db2.mapperLocation%20%3D%20classpath*%3Adb2%2Fmapper%2F*.xml%0Aspring.datasource.db2.typeAliasesPackage%20%3D%20com.shuailee.repository.model.db2%0A%60%60%60%0A**2%20%E4%B8%BA%E6%AF%8F%E4%B8%AA%E6%95%B0%E6%8D%AE%E5%BA%93%E9%85%8D%E7%BD%AE%E7%8B%AC%E7%AB%8B%E7%9A%84%E6%95%B0%E6%8D%AE%E6%BA%90bean**%0A%0A**db1%E6%95%B0%E6%8D%AE%E5%BA%93**%0A%60%60%60java%0A%40Data%0A%40Configuration%0A%40ConfigurationProperties(prefix%20%3D%20%22spring.datasource.db1%22)%0A%40MapperScan(basePackages%20%3D%20%22com.shuailee.repository.mapper%22%2C%20sqlSessionFactoryRef%20%3D%20%22db1SqlSessionFactory%22)%0A%40Slf4j%0Apublic%20class%20DB1DataSource%20%7B%0A%0A%20%20%20%20private%20String%20url%3B%0A%20%20%20%20private%20String%20username%3B%0A%20%20%20%20private%20String%20password%3B%0A%20%20%20%20private%20String%20driverClassName%3B%0A%20%20%20%20private%20int%20initialSize%3B%0A%20%20%20%20private%20int%20minIdle%3B%0A%20%20%20%20private%20int%20maxActive%3B%0A%20%20%20%20private%20long%20maxWait%3B%0A%20%20%20%20private%20long%20timeBetweenEvictionRunsMillis%3B%0A%20%20%20%20private%20long%20minEvictableIdleTimeMillis%3B%0A%20%20%20%20private%20long%20maxEvictableIdleTimeMillis%3B%0A%20%20%20%20private%20String%20validationQuery%3B%0A%20%20%20%20private%20boolean%20testWhileIdle%3B%0A%20%20%20%20private%20boolean%20testOnBorrow%3B%0A%20%20%20%20private%20boolean%20testOnReturn%3B%0A%20%20%20%20private%20boolean%20poolPreparedStatements%3B%0A%20%20%20%20private%20int%20maxPoolPreparedStatementPerConnectionSize%3B%0A%20%20%20%20private%20String%20connectionProperties%3B%0A%20%20%20%20private%20String%20filters%3B%0A%20%20%20%20private%20boolean%20keepAlive%3B%0A%0A%20%20%20%20private%20String%20mapperLocation%3B%0A%20%20%20%20private%20String%20typeAliasesPackage%3B%0A%0A%0A%20%20%20%20%2F**%0A%20%20%20%20%20*%20%20DB%0A%20%20%20%20%20*%0A%20%20%20%20%20*%20%40return%0A%20%20%20%20%20*%2F%0A%20%20%20%20%40Bean(name%20%3D%20%22db1Ds%22)%0A%20%20%20%20public%20DataSource%20gsuperDataSourceConfig()%20throws%20Exception%20%7B%0A%20%20%20%20%20%20%20%20log.info(%22db1Ds%20db%20build%22)%3B%0A%20%20%20%20%20%20%20%20DruidDataSource%20druid%20%3D%20new%20DruidDataSource()%3B%0A%20%20%20%20%20%20%20%20%2F%2F%20%E7%9B%91%E6%8E%A7%E7%BB%9F%E8%AE%A1%E6%8B%A6%E6%88%AA%E7%9A%84filters%0A%20%20%20%20%20%20%20%20druid.setFilters(filters)%3B%0A%0A%20%20%20%20%20%20%20%20%2F%2F%20%E9%85%8D%E7%BD%AE%E5%9F%BA%E6%9C%AC%E5%B1%9E%E6%80%A7%0A%20%20%20%20%20%20%20%20druid.setDriverClassName(driverClassName)%3B%0A%20%20%20%20%20%20%20%20druid.setUsername(username)%3B%0A%20%20%20%20%20%20%20%20druid.setPassword(password)%3B%0A%20%20%20%20%20%20%20%20druid.setUrl(url)%3B%0A%0A%20%20%20%20%20%20%20%20%2F%2F%E5%88%9D%E5%A7%8B%E5%8C%96%E6%97%B6%E5%BB%BA%E7%AB%8B%E7%89%A9%E7%90%86%E8%BF%9E%E6%8E%A5%E7%9A%84%E4%B8%AA%E6%95%B0%0A%20%20%20%20%20%20%20%20druid.setInitialSize(initialSize)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%A4%A7%E8%BF%9E%E6%8E%A5%E6%B1%A0%E6%95%B0%E9%87%8F%0A%20%20%20%20%20%20%20%20druid.setMaxActive(maxActive)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E6%9C%80%E5%B0%8F%E8%BF%9E%E6%8E%A5%E6%B1%A0%E6%95%B0%E9%87%8F%0A%20%20%20%20%20%20%20%20druid.setMinIdle(minIdle)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E8%8E%B7%E5%8F%96%E8%BF%9E%E6%8E%A5%E6%97%B6%E6%9C%80%E5%A4%A7%E7%AD%89%E5%BE%85%E6%97%B6%E9%97%B4%EF%BC%8C%E5%8D%95%E4%BD%8D%E6%AF%AB%E7%A7%92%E3%80%82%0A%20%20%20%20%20%20%20%20druid.setMaxWait(maxWait)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E9%97%B4%E9%9A%94%E5%A4%9A%E4%B9%85%E8%BF%9B%E8%A1%8C%E4%B8%80%E6%AC%A1%E6%A3%80%E6%B5%8B%EF%BC%8C%E6%A3%80%E6%B5%8B%E9%9C%80%E8%A6%81%E5%85%B3%E9%97%AD%E7%9A%84%E7%A9%BA%E9%97%B2%E8%BF%9E%E6%8E%A5%0A%20%20%20%20%20%20%20%20druid.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E4%B8%80%E4%B8%AA%E8%BF%9E%E6%8E%A5%E5%9C%A8%E6%B1%A0%E4%B8%AD%E6%9C%80%E5%B0%8F%E7%94%9F%E5%AD%98%E7%9A%84%E6%97%B6%E9%97%B4%0A%20%20%20%20%20%20%20%20druid.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis)%3B%0A%20%20%20%20%20%20%20%20druid.setMaxEvictableIdleTimeMillis(maxEvictableIdleTimeMillis)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E7%94%A8%E6%9D%A5%E6%A3%80%E6%B5%8B%E8%BF%9E%E6%8E%A5%E6%98%AF%E5%90%A6%E6%9C%89%E6%95%88%E7%9A%84sql%0A%20%20%20%20%20%20%20%20druid.setValidationQuery(validationQuery)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E5%BB%BA%E8%AE%AE%E9%85%8D%E7%BD%AE%E4%B8%BAtrue%EF%BC%8C%E4%B8%8D%E5%BD%B1%E5%93%8D%E6%80%A7%E8%83%BD%EF%BC%8C%E5%B9%B6%E4%B8%94%E4%BF%9D%E8%AF%81%E5%AE%89%E5%85%A8%E6%80%A7%E3%80%82%0A%20%20%20%20%20%20%20%20druid.setTestWhileIdle(testWhileIdle)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E7%94%B3%E8%AF%B7%E8%BF%9E%E6%8E%A5%E6%97%B6%E6%89%A7%E8%A1%8CvalidationQuery%E6%A3%80%E6%B5%8B%E8%BF%9E%E6%8E%A5%E6%98%AF%E5%90%A6%E6%9C%89%E6%95%88%0A%20%20%20%20%20%20%20%20druid.setTestOnBorrow(testOnBorrow)%3B%0A%20%20%20%20%20%20%20%20druid.setTestOnReturn(testOnReturn)%3B%0A%20%20%20%20%20%20%20%20druid.setKeepAlive(keepAlive)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%E6%98%AF%E5%90%A6%E7%BC%93%E5%AD%98preparedStatement%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AFPSCache%EF%BC%8Coracle%E8%AE%BE%E4%B8%BAtrue%EF%BC%8Cmysql%E8%AE%BE%E4%B8%BAfalse%E3%80%82%E5%88%86%E5%BA%93%E5%88%86%E8%A1%A8%E8%BE%83%E5%A4%9A%E6%8E%A8%E8%8D%90%E8%AE%BE%E7%BD%AE%E4%B8%BAfalse%0A%20%20%20%20%20%20%20%20druid.setPoolPreparedStatements(poolPreparedStatements)%3B%0A%20%20%20%20%20%20%20%20%2F%2F%20%E6%89%93%E5%BC%80PSCache%E6%97%B6%EF%BC%8C%E6%8C%87%E5%AE%9A%E6%AF%8F%E4%B8%AA%E8%BF%9E%E6%8E%A5%E4%B8%8APSCache%E7%9A%84%E5%A4%A7%E5%B0%8F%0A%20%20%20%20%20%20%20%20druid.setMaxPoolPreparedStatementPerConnectionSize(maxPoolPreparedStatementPerConnectionSize)%3B%0A%20%20%20%20%20%20%20%20druid.setConnectionProperties(connectionProperties)%3B%0A%20%20%20%20%20%20%20%20return%20druid%3B%0A%20%20%20%20%7D%0A%0A%0A%0A%20%20%20%20%2F**%0A%20%20%20%20%20*%20%40return%0A%20%20%20%20%20*%20%40throws%20Exception%0A%20%20%20%20%20*%2F%0A%20%20%20%20%40Bean(name%20%3D%20%22db1SqlSessionFactory%22)%0A%20%20%20%20public%20SqlSessionFactory%20sqlSessionFactory()%20throws%20Exception%20%7B%0A%20%20%20%20%20%20%20%20SqlSessionFactoryBean%20factoryBean%20%3D%20new%20SqlSessionFactoryBean()%3B%0A%20%20%20%20%20%20%20%20factoryBean.setDataSource(gsuperDataSourceConfig())%3B%0A%20%20%20%20%20%20%20%20val%20resolver%20%3D%20new%20PathMatchingResourcePatternResolver()%3B%0A%20%20%20%20%20%20%20%20factoryBean.setMapperLocations(resolver.getResources(mapperLocation))%3B%0A%20%20%20%20%20%20%20%20factoryBean.setTypeAliasesPackage(typeAliasesPackage)%3B%0A%20%20%20%20%20%20%20%20PageInterceptor%20pageInterceptor%20%3D%20new%20PageInterceptor()%3B%0A%20%20%20%20%20%20%20%20Properties%20properties%20%3D%20new%20Properties()%3B%0A%20%20%20%20%20%20%20%20properties.setProperty(%22helperDialect%22%2C%20%22mysql%22)%3B%0A%20%20%20%20%20%20%20%20properties.setProperty(%22reasonable%22%2C%20%22true%22)%3B%0A%20%20%20%20%20%20%20%20properties.setProperty(%22supportMethodsArguments%22%2C%20%22true%22)%3B%0A%20%20%20%20%20%20%20%20properties.setProperty(%22autoRuntimeDialect%22%2C%20%22true%22)%3B%0A%20%20%20%20%20%20%20%20pageInterceptor.setProperties(properties)%3B%0A%20%20%20%20%20%20%20%20PageInterceptor%5B%5D%20pageInterceptors%20%3D%20%7BpageInterceptor%7D%3B%0A%20%20%20%20%20%20%20%20factoryBean.setPlugins(pageInterceptors)%3B%0A%20%20%20%20%20%20%20%20return%20factoryBean.getObject()%3B%0A%20%20%20%20%7D%0A%0A%0A%20%20%20%20%2F**%0A%20%20%20%20%20*%20%40return%0A%20%20%20%20%20*%20%40throws%20Exception%0A%20%20%20%20%20*%2F%0A%20%20%20%20%40Bean(name%20%3D%20%22db1SessionTemplate%22)%0A%20%20%20%20public%20SqlSessionTemplate%20sqlSessionTemplate()%20throws%20Exception%20%7B%0A%20%20%20%20%20%20%20%20SqlSessionTemplate%20template%20%3D%20new%20SqlSessionTemplate(sqlSessionFactory())%3B%0A%20%20%20%20%20%20%20%20return%20template%3B%0A%0A%20%20%20%20%7D%0A%0A%0A%20%20%20%20%2F**%0A%20%20%20%20%20*%20%40return%0A%20%20%20%20%20*%20%40throws%20Exception%0A%20%20%20%20%20*%2F%0A%20%20%20%20%40Bean(name%20%3D%20%22db1Transaction%22)%0A%20%20%20%20public%20DataSourceTransactionManager%20transactionManager()%20throws%20Exception%20%7B%0A%20%20%20%20%20%20%20%20return%20new%20DataSourceTransactionManager(gsuperDataSourceConfig())%3B%0A%20%20%20%20%7D%0A%0A%0A%7D%0A%60%60%60%0A**db2%E6%95%B0%E6%8D%AE%E5%BA%93**DB2%E6%95%B0%E6%8D%AE%E5%BA%93%E9%85%8D%E7%BD%AE%E5%8F%AA%E9%9C%80%E8%A6%81%E4%BF%AE%E6%94%B9%E5%AF%B9%E5%BA%94bean%E5%90%8D%E7%A7%B0%E5%92%8C%E8%B7%AF%E5%BE%84%E5%8D%B3%E5%8F%AF%E3%80%82%0A%0A%0A%23%23%23%23%20%E4%B9%9D%20Mybatis-Plus%0A%0A**%E6%96%87%E6%A1%A3%3A**%20https%3A%2F%2Fmp.baomidou.com%2Fguide%2F%0A%0A**mybatis-plus%E5%92%8Cmybatis%E7%9A%84%E4%B8%8D%E5%90%8C%EF%BC%9A**%0A*%20mybatis-plus%E6%98%AF%E5%AF%B9mybatis%E7%9A%84%E5%A2%9E%E5%BC%BA%EF%BC%8C%E6%9C%AA%E5%AF%B9mybatis%E5%81%9A%E4%BF%AE%E6%94%B9%EF%BC%8C%E6%89%80%E4%BB%A5mybatis%E7%9A%84%E6%89%80%E6%9C%89%E9%85%8D%E7%BD%AE%E5%92%8C%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F%E9%83%BD%E4%BF%9D%E6%8C%81%E4%B8%8D%E5%8F%98%E3%80%82%E4%BD%BF%E7%94%A8%E6%97%B6%E5%8F%AA%E9%9C%80%E8%A6%81%E5%BC%95%E5%85%A5mybatis-plus%E7%9A%84maven%E4%BE%9D%E8%B5%96%E5%8D%B3%E5%8F%AF%E3%80%82%0A*%20%E5%9C%A8%E4%B8%8D%E9%9C%80%E8%A6%81%E8%87%AA%E5%AE%9A%E4%B9%89sql%E7%9A%84%E6%83%85%E5%86%B5%E4%B8%8Bmybatis-plus%E4%B8%8D%E9%9C%80%E8%A6%81%E9%85%8D%E7%BD%AExml%EF%BC%8C%E5%8F%AA%E9%9C%80%E8%A6%81%E5%AE%9A%E4%B9%89mapper%E6%8E%A5%E5%8F%A3%E7%BB%A7%E6%89%BFBaseMapper%20%E5%8D%B3%E5%8F%AF%E5%AE%9E%E7%8E%B0%E5%A4%A7%E9%83%A8%E5%88%86CRUD%E6%93%8D%E4%BD%9C%0A*%20%E5%85%B3%E4%BA%8E%E5%88%86%E9%A1%B5%EF%BC%8Cmybatis%E5%88%86%E9%A1%B5%E9%9C%80%E8%A6%81%E4%BD%BF%E7%94%A8PageHelper%20%E6%8F%92%E4%BB%B6%E5%AE%8C%E6%88%90%E5%88%86%E9%A1%B5%EF%BC%8C%E8%80%8Cmybatis-plus%E5%86%85%E7%BD%AE%E4%BA%86%E5%88%86%E9%A1%B5%E6%8B%A6%E6%88%AA%E5%99%A8PaginationInnerInterceptor%E5%AE%9E%E7%8E%B0%EF%BC%8C%E9%9C%80%E8%A6%81%E9%85%8D%E7%BD%AE%E5%9C%A8MybatisPlusInterceptor%20%E7%B1%BB%0A*%20%E6%9D%A1%E4%BB%B6%E6%9E%84%E9%80%A0%E5%99%A8%EF%BC%9A%E9%80%9A%E8%BF%87EntityWrapper%3CT%3E%EF%BC%88%E5%AE%9E%E4%BD%93%E5%8C%85%E8%A3%85%E7%B1%BB%EF%BC%89%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%94%A8%E4%BA%8E%E6%8B%BC%E6%8E%A5SQL%20%E8%AF%AD%E5%8F%A5%EF%BC%8C%E5%B9%B6%E4%B8%94%E6%94%AF%E6%8C%81%E6%8E%92%E5%BA%8F%E3%80%81%E5%88%86%E7%BB%84%E6%9F%A5%E8%AF%A2%E7%AD%89%E5%A4%8D%E6%9D%82%E7%9A%84SQL%E3%80%82%0Apom%3A%0A%60%60%60xml%0A%3C!--druid%E8%BF%9E%E6%8E%A5%E6%B1%A0--%3E%0A%3Cdependency%3E%0A%20%20%20%20%3CgroupId%3Ecom.alibaba%3C%2FgroupId%3E%0A%20%20%20%20%3CartifactId%3Edruid-spring-boot-starter%3C%2FartifactId%3E%0A%20%20%20%20%3Cversion%3E1.1.9%3C%2Fversion%3E%0A%3C%2Fdependency%3E%0A%3C!--mybatis-plus--%3E%0A%3Cdependency%3E%0A%20%20%20%20%3CgroupId%3Ecom.baomidou%3C%2FgroupId%3E%0A%20%20%20%20%3CartifactId%3Emybatis-plus-boot-starter%3C%2FartifactId%3E%0A%20%20%20%20%3Cversion%3E3.4.2%3C%2Fversion%3E%0A%3C%2Fdependency%3E%0A%3C!--mysql--%3E%0A%3Cdependency%3E%0A%20%20%20%20%3CgroupId%3Emysql%3C%2FgroupId%3E%0A%20%20%20%20%3CartifactId%3Emysql-connector-java%3C%2FartifactId%3E%0A%20%20%20%20%3Cversion%3E%24%7Bmysql-connecto.version%7D%3C%2Fversion%3E%0A%3C%2Fdependency%3E%0A%60%60%60%0A**%E5%88%86%E9%A1%B5%E6%8B%A6%E6%88%AA%E5%99%A8**%0A%60%60%60java%0A%2F**%0A%20*%20%40package%3A%20com.klein.config%0A%20*%20%40description%3A%20%E5%88%86%E9%A1%B5%E6%8B%A6%E6%88%AA%E5%99%A8%E9%85%8D%E7%BD%AE%20%203.4%E4%B9%8B%E5%89%8D%E4%BD%BF%E7%94%A8PaginationInterceptor%20%203.4%E4%B9%8B%E5%90%8E%E4%BD%BF%E7%94%A8MybatisPlusInterceptor%0A%20*%20%40author%3A%20klein%0A%20*%20%40date%3A%202021-06-10%2010%3A36%0A%20**%2F%0A%40Configuration%0Apublic%20class%20MyBatisPlusConfig%20%7B%0A%0A%20%20%20%20%40Bean%0A%20%20%20%20public%20MybatisPlusInterceptor%20paginationInterceptor()%20%7B%0A%20%20%20%20%20%20%20%20MybatisPlusInterceptor%20interceptor%20%3D%20new%20MybatisPlusInterceptor()%3B%0A%20%20%20%20%20%20%20%20PaginationInnerInterceptor%20paginationInnerInterceptor%20%3D%20new%20PaginationInnerInterceptor()%3B%0A%20%20%20%20%20%20%20%20paginationInnerInterceptor.setOverflow(false)%3B%0A%20%20%20%20%20%20%20%20paginationInnerInterceptor.setMaxLimit(500L)%3B%0A%20%20%20%20%20%20%20%20interceptor.addInnerInterceptor(paginationInnerInterceptor)%3B%0A%20%20%20%20%20%20%20%20return%20interceptor%3B%0A%20%20%20%20%7D%0A%7D%0A%60%60%60%0A**%E6%8E%A5%E5%8F%A3%E6%98%A0%E5%B0%84%E5%99%A8**%0A%60%60%60%0A%2F**%0A%20*%20%40package%3A%20com.klein.dao.mapper%0A%20*%20%40description%3A%20mybatisplus%20mapper%E6%8E%A5%E5%8F%A3%E5%AE%9A%E4%B9%89%0A%20*%20%40author%3A%20klein%0A%20*%20%40date%3A%202021-06-09%2018%3A13%0A%20**%2F%0Apublic%20interface%20UserMapper%20extends%20BaseMapper%3CUser%3E%20%7B%0A%0A%7D%0A%60%60%60%0A**%E5%AE%9E%E4%BD%93%E5%AF%B9%E8%B1%A1**%0A%60%60%60java%0A%40Data%0A%40TableName(value%20%3D%20%22User%22)%2F%2F%E6%8C%87%E5%AE%9A%E8%A1%A8%E5%90%8D%0Apublic%20class%20User%20%7B%0A%20%20%20%20%2F**%0A%20%20%20%20%20*%20value%E4%B8%8E%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%BB%E9%94%AE%E5%88%97%E5%90%8D%E4%B8%80%E8%87%B4%EF%BC%8C%E8%8B%A5%E5%AE%9E%E4%BD%93%E7%B1%BB%E5%B1%9E%E6%80%A7%E5%90%8D%E4%B8%8E%E8%A1%A8%E4%B8%BB%E9%94%AE%E5%88%97%E5%90%8D%E4%B8%80%E8%87%B4%E5%8F%AF%E7%9C%81%E7%95%A5value%0A%20%20%20%20%20*%20%E6%8C%87%E5%AE%9A%E8%87%AA%E5%A2%9E%E7%AD%96%E7%95%A5%0A%20%20%20%20%20*%20*%2F%0A%20%20%20%20%40TableId(value%20%3D%20%22id%22%2Ctype%20%3D%20IdType.AUTO)%0A%20%20%20%20private%20Long%20id%3B%0A%20%20%20%20%2F**%0A%20%20%20%20%20*%20%E8%8B%A5%E6%B2%A1%E6%9C%89%E5%BC%80%E5%90%AF%E9%A9%BC%E5%B3%B0%E5%91%BD%E5%90%8D%EF%BC%8C%E6%88%96%E8%80%85%E8%A1%A8%E4%B8%AD%E5%88%97%E5%90%8D%E4%B8%8D%E7%AC%A6%E5%90%88%E9%A9%BC%E5%B3%B0%E8%A7%84%E5%88%99%EF%BC%8C%E5%8F%AF%E9%80%9A%E8%BF%87%E8%AF%A5%E6%B3%A8%E8%A7%A3%E6%8C%87%E5%AE%9A%E6%95%B0%E6%8D%AE%E5%BA%93%E8%A1%A8%E4%B8%AD%E7%9A%84%E5%88%97%E5%90%8D%EF%BC%8Cexist%E6%A0%87%E6%98%8E%E6%95%B0%E6%8D%AE%E8%A1%A8%E4%B8%AD%E6%9C%89%E6%B2%A1%E6%9C%89%E5%AF%B9%E5%BA%94%E5%88%97%0A%20%20%20%20%20*%20*%2F%0A%20%20%20%20%40TableField(value%20%3D%20%22name%22%2Cexist%20%3D%20true)%0A%20%20%20%20private%20String%20name%3B%0A%20%20%20%20private%20Integer%20age%3B%0A%20%20%20%20private%20String%20email%3B%0A%7D%0A%60%60%60%0A**%E5%88%86%E9%A1%B5%E6%9F%A5%E8%AF%A2**%0A%60%60%60%20java%0A%20%40Test%0A%20%20%20%20public%20void%20testSelectPage()%20%7B%0A%0A%20%20%20%20%20%20%20%20System.out.println((%22-----%20selectList%20method%20process%20------%22))%3B%0A%20%20%20%20%20%20%20%20List%3CUser%3E%20userList%20%3D%20userMapper.selectList(null)%3B%0A%20%20%20%20%20%20%20%20userList.forEach(System.out%3A%3Aprintln)%3B%0A%0A%20%20%20%20%20%20%20%20System.out.println((%22-----%20selectPage%20method%20test%20------%22))%3B%0A%0A%0A%20%20%20%20%20%20%20%20%2F%2F%20%E5%88%86%E9%A1%B5%E9%9C%80%E8%A6%81%E9%85%8D%E7%BD%AE%20MybatisPlusInterceptor%20%E5%88%86%E9%A1%B5%E6%8B%A6%E6%88%AA%E5%99%A8%0A%20%20%20%20%20%20%20%20%2F%2F%20%E4%BD%BF%E7%94%A8QueryWrapper%E6%9D%A1%E4%BB%B6%E6%9E%84%E9%80%A0%E5%99%A8%E8%BF%9B%E8%A1%8C%E5%88%86%E9%A1%B5%EF%BC%8C%E7%89%A9%E7%90%86%E5%88%86%E9%A1%B5%0A%20%20%20%20%20%20%20%20Page%3CUser%3E%20userPage%20%3D%20userMapper.selectPage(new%20Page%3CUser%3E(1%2C%202)%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20new%20QueryWrapper%3CUser%3E()%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.between(%22age%22%2C%2018%2C%2050)%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20.in(%22id%22%2C%20Arrays.asList(1%2C%202%2C%203%2C%204%2C%205))%0A%20%20%20%20%20%20%20%20)%3B%0A%0A%20%20%20%20%20%20%20%20List%3CUser%3E%20userPageList%20%3D%20userPage.getRecords()%3B%0A%20%20%20%20%20%20%20%20userPageList.forEach(System.out%3A%3Aprintln)%3B%0A%20%20%20%20%7D%0A%60%60%60%0A%0A**git%E6%BA%90%E7%A0%81%EF%BC%9A%5Blab-01-springboot-hub%5D(git%40gitee.com%3Ashuai1990%2Fspringboot-labs.git)**

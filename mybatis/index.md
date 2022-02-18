# Mybatis


# Mybatis配置

### 1. 搭建实验数据库

- 创建一个MyBatis实验数据库,并创建一个user表

### 2. Idea新建项目,连接数据库

- 新建一个普通的maven项目
- pom.xml中导入相关的maven依赖
  - Mysql驱动
  - Mybatis驱动
  - junit驱动

```xml
<dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.21</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.5</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

- 连接我们的Mysql
  - 解决时区问题

### 3. 编写MyBatis核心配置文件

- 创建一个子模块mybatis-01
- 在该模块下的resource目录下,新建mybatis-config.xml文件,这是MyBatis的核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSH=true&amp;serverTimezone=UTC&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

### 4. 编写MyBatis工具类

在子模块mybatis-01/src/main/java目录下,新建一个utils工具类包,新建MybatisUntils类 * 该类用来从XML中构建SqlSessionFactory

```java
package utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

//从 SqlSessionFactory 中获取 SqlSession
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            //获取SqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //从 SqlSessionFactory 中获取 SqlSession
    public static SqlSession getSqlSession() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        return sqlSession;
    }
}
```

### 5. 创建表对应实体类

```java
package pojo;

public class User {
    private int id;
    private String name;
    private String pwd;

    public User() {
    }

    public User(int id, String name, String pwd) {
        this.id = id;
        this.name = name;
        this.pwd = pwd;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPwd() {
        return pwd;
    }

    public void setPwd(String pwd) {
        this.pwd = pwd;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }
}
```

### 6.编写Mapper接口

在mybatis-01/src/main/java目录下,新建mapper包,其中新建UerMapper接口 * 其中抽象方法getUerList()用来返回Uer对象集合

```java
package mapper;

import pojo.User;

import java.util.List;

public interface UserMapper {
    List<User> getUserList();
}
```

### 7.编写Mapper.xml配置文件

在子模块/src/main/java/mapper下,新建UerMappering.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mapper.UserMapper">
    <select id="getUserList" resultType="pojo.User">
    select * from mybatis.user
    </select>
</mapper>
```

- namespase=自己创建Mapper接口
- id=对应Mapper接口的方法名
- resuletType=返回结果类型
- select中间是Sql语句

### 8. 编写junit测试类

在mybatis-01/test/java下新建mapper包，该包下新建测试类UserMapperTest *junit测试

```java
package mapper;

import mapper.UserMapper;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import pojo.User;
import utils.MybatisUtils;

import java.util.List;

public class UserMapperTest {
    @Test
    public void test() {
        //获取sqlSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        //执行SQL
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = mapper.getUserList();

        for (User user : userList) {
            System.out.println(user);
        }
        //关闭sqlSession
        sqlSession.close();
    }
}
```

### 9. 给Maper.xml添加注册

每一个Mapper.xml都需要在MyBatis核心配置文件中注册

我们要在mybatis-config.xml中进行注册，最后加上以下代码

```xml
<mappers>
        <mapper resource="mapper/UserMapping.xml"/>
    </mappers>
```

这里的路径是我们定义的xml配置文件的路径 **注意:改路径的中间要用/进行分割**

### 10. 测试运行

### 11. 可能遇到的问题

- maven配置文件无法被到处或生效
  - maven默认资源文件配置在resource目录下，但是我们放在了java目录下，该目录下无法导出，所以需要手动配置资源过滤，让src/main/java下的 .properties 或 .xml 可以导出

解决方案是:将以下设置写在pom.xml中

```xml
<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```

# CRUD

### 1. namespace:

- namespace中的包名要和Mapper接口的包名一致;

### 2. select:

选择,查询语句;

- id:就是对应的namespace中的方法名;
- resultType:Sql语句执行的返回值;
- parameterType:参数类型;

1. 编写接口

```java
User getUserById(int id);
```

1. 编写对应Mapper中的Sql语句

```xml
<select id="getUserById" parameterType="int" resultType="pojo.User">
    select * from mybatis.user where id=#{id}
    </select>
```

1. 测试

```java
@Test
    public void getUserById(){

        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);

        User user = mapper.getUserById(1);
        System.out.println(user);

        sqlSession.close();
    }
```

### 3. insert

1. 编写接口

```java
int addUser(User user);
```

1. 编写对应Mapper中的Sql语句

```xml
<insert id="addUser" parameterType="pojo.User">
        insert into mybatis.user (id,name,pwd) values (#{id},#{name},#{pwd});
    </insert>
```

### 4. update

1. 编写接口

```java
User updateUser(int id);
```

1. 编写对应Mapper中的Sql语句

```xml
<update id="updateUser" parameterType="pojo.User">
        update mybatis.user set name=#{name},pwd=#{pwd} where id=#{id};
    </update>
```

### 5. delete

1. 编写接口

```java
int deleteUser(int id);
```

1. 编写对应的Mapper中的Sql语句

```xml
<delete id="deleteUser" parameterType="int">
        delete from mybatis.user where id=#{id};
    </delete>
```

### 6. 错误分析

- 标签不要匹配错;
- resource编订mapper,需要使用路径;
- 陈旭配置文件必须符合规范;
- 空指针异常,没有注册到资源;
- 输出的 xml文件存在中文乱码问题;
- maven资源没有导出问题;

### 7. 万能Map

假设,我们的实体类,或者数据库中的表,字段或者参数过多,我们应该烤炉使用Map;

```java
int addUser2(Map<String,Object> map);
<insert id="addUser2" parameterType="map">
        insert into mybatis.user (id,name,pwd) values (#{userid},#{username},#{userpwd});
    </insert>
public void addUser2(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);

        HashMap<String, Object> map = new HashMap<String, Object>();
        map.put("userid",6);
        map.put("username","gongxiwu");
        map.put("userpwd","12344454");

        mapper.addUser2(map);

        sqlSession.commit();
        sqlSession.close();
    }
```

- Map传递参数,直接在sql中取出key即可;
- 对象传递参数,直接在sql中取出属性即可;
- 只有一个基本类型参数的情况下,可以直接在sql中取到;

### 8. 模糊查询

1. java代码执行的时候,传递通配符% %

```java
List<User> userList = mapper.getUserLike("%王%");
```

1. 在sql拼接中使用通配符;

```xml
<select id="getUserLike"  resultType="pojo.User">
        select * from mybatis.user where name like "%"#{value}"%"
    </select>
```

# 配置解析

### 1. 核心配置文件

- configuration（配置）
- properties（属性）
- settings（设置）
- typeAliases（类型别名）
- typeHandlers（类型处理器）
- objectFactory（对象工厂）
- plugins（插件）
- environments（环境配置）
- environment（环境变量）
- transactionManager（事务管理器）
- dataSource（数据源）
- databaseIdProvider（数据库厂商标识）
- mappers（映射器）

### 2 环境配置(environment)

- MyBatis可以配置成适应多种环境;
- 尽管可以配置多个环境,但每个SqlSessionFactory实例只能使用一种环境;
- MyBatis默认的事务管理器就是JDBC, 连接池: POOLED

### 3. 属性

- xml文件中所有标签都可以规定顺序;
- 编写一个配置文件db.properties

```yaml
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSH=true&serverTimezone=GMT&characterEncoding=UTF-8
username=root
password=root
```

- 在核心配置文件映入

```xml
    <properties resource="db.properties">
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </properties>
```

- 可以直接引入外部文件
- 可以在其中增加一些属性配置
- 如果两个文件有同一字段,优先使用外部配置文件的

### 4. 类型别名(typeAliases)

- 类型别名是为Java类型设置自一个短的名字;
- 存在的意义仅在于用来减少类完全限定名的冗余'

```xml
    <typeAliases>
        <typeAlias type="pojo.User" alias="User"/>
    </typeAliases>
```

也可以指定一个包名,MyBaties会在包名下面搜索需要的Java Bean,比如:扫描实体类的包名,他的默认别名就是这个类的类名;

```xml
<!--给实体类起别名-->
    <typeAliases>
        <package name="pojo"/>
    </typeAliases>
```

实体类较少的时候建议使用第一种;

实体类较多建议使用第二种;

第一种可以DIY别名,第二种则不行;如果非要修改,可以在实体类上加注解;

```java
@Alias("hello")
public class User {}
```

### 5. 设置(settings)

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项设置的含义、默认值等;

### 6. 其他设置

- typeHandlers(类型处理器)
- objectFactory(对象工厂)
- plugins插件
  - mybatis-generator-core
  - mybatis-plus
  - 通用mapper

### 7. 映射器(mappers)

**MapperRegistry:注册绑定我们的Mapper文件**

##### 方式一:

```xml
<mappers>
    <mapper resource="mapper/UserMapping.xml"/>
</mappers>
```

##### 方式二:使用class文件绑定注册

```xml
<mappers>
    <mapper class="mapper.UserMapper"/>
</mappers>
```

注意点:

```
* 接口和他的Mappe配置文件必须同名;
* 接口和他的Mapper配置文件必须在同一个包下;
```

##### 方式三:使用扫描包进行注册绑定

```xml
<mappers>
    <package name="mapper"/>
</mappers>
```

注意点:

```
* 接口和他的Mapper配置文件必须同名;
* 接口和他的Mapper文件必须在同一个包下;
```

### 8.生命周期和作用域

生命周期和作用域是至关重要的,因为错误的使用会导致非常严重的**并发问题**;

##### SqlSessionFactoryBuilder:

- 一旦创建了SqlSessionFactory,就不在需要它了
- 局部变量

##### SqlSessionFactory:

- 说白了就是可以想象为:数据库连接池
- SqlSessionFactory一旦被创建就应该在应用的运行期间一直存在,没有任何理由丢弃它或重新创建另一个实例.
- 因此SqlSessionFactory的最佳作用域是应用作用域;
- 最简单的就是使用单例模式或者静态单看例模式;

##### SqlSession

- 连接到连接池的一个请求;
- SqlSession的实例不是线程安全的,因此是不能被共享的,所以他的最佳的作用域是请求或方法作用域;
- 用完之后需要赶紧关闭,否则资源被占用;

# 解决属性名和字段名不一致问题

1. 数据库中的字段是: id,name,pwd;
2. 实体类中的字段是:id,name,password;

```java
public class User {
    private int id;
    private String name;
    private String password;
    }
```

查出password字段为null;

```xml
select *  password from mybatis.user
select id,name,pwd as password from mybatis.user
```

##### 解决方法:

- 起别名

```xml
<select id="getUserList" resultType="pojo.User">
    select id,name,pwd as password from mybatis.user
</select>
```

### resultMap

结果集映射

```xml
id name pwd
id name password
<resultMap id="UserMap" type="User">
        <!--column对应数据库中的字段,property对应实体类的属性-->
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="pwd" property="password"/>
    </resultMap>
    <!--select查询语句-->
    <select id="getUserList" resultMap="UserMap">
    select *  from mybatis.user
    </select>
```

- ResultMap 元素是MyBatis中最重要的最强大的元素;
- ResultMap的设计思想是,对于简单的于都根本不需要配置显示的结果映射,而对于复杂一点的语句只需要描述他们的关系就行了;
- ResultMap 最优秀的地方在于,虽然你已经对他相当了解了,但是根本就不需要显示地用到他们;

# 日志

如果一个数据库操作,出现了异常,我们需要拍错,日志就是最好的助手;

曾经:sout;debug

现在:日志工厂

**logImpl:指定 MyBatis 所用日志的具体实现，未指定时将自动查找;**

- SLF4J
- LOG4J (掌握)
- LOG4J2
- JDK_LOGGING
- COMMONS_LOGGING
- STDOUT_LOGGING(掌握)
- NO_LOGGING

在MyBatis中具体使用哪一个日志实现,在设置中设定;

**STDOUT_LOGGING是标准日志输出**

### log4j:

- Log4j是Apache的一个开源项目，通过使用Log4j,我们可以控制日志信息输送的目的地是控制台、文件、GUI组件;
- 我们也可以控制每一条日志的输出格式;
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程;
- 这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码;

1. 先导入log4j的包

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

1. log4j.properties

```
#将登记为DEBUG的日志信息输出到console和file这两个目的地,console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Target=System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

#文件输出的相关设置
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/gong.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PrepareStatement=DEBUG
```

1. 配置log4j为日志的实现

```xml
<settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>
```

##### 简单使用:

1. 导包:import org.apache.log4j.Logger;
2. 日志对象,参数为当前类的class

```java
static Logger logger = Logger.getLogger(UserMapperTest.class);
```

1. 日志级别

```java
logger.info("info进入了log4jTest");
logger.debug("debug进入了log4jTest");
logger.error("error进入了log4jTest");
```

# 多对一处理

### 创建表

```java
CREATE TABLE teacher (
	id INT (10) NOT NULL,
	name VARCHAR(30) DEFAULT NULL,
	PRIMARY KEY (id)
) ENGINE=INNODB 

INSERT INTO TEACHER (id,name) VALUES (1,'李四');

CREATE TABLE student(
	id INT (10) NOT NULL,
	name VARCHAR(30) DEFAULT NULL,
	tid INT(10) DEFAULT NULL,
	PRIMARY KEY (id),
	FOREIGN KEY (tid) REFERENCES teacher (id)
)ENGINE=INNODB 

INSERT INTO student(id,name,tid) VALUES (1,'小明',1);
INSERT INTO student(id,name,tid) VALUES (2,'小红',1);
INSERT INTO student(id,name,tid) VALUES (3,'小蓝',1);
INSERT INTO student(id,name,tid) VALUES (4,'小绿',1);
INSERT INTO student(id,name,tid) VALUES (5,'小黄',1);
```

### 测试环境搭建

1. 导入lombok
2. 建立实体类Student,Teacher
3. 创建Mapper接口
4. 创建Mapper.xml文件
5. 在核心配置文件中注册
6. 测试

### 按照嵌套查询处理

```xml
<mapper namespace="mapper.StudentMapper">
    <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <!--复杂的属性需要单独处理
            对象:association
            集合:collection
        -->
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
    </resultMap>

    <select id="getStudent" resultMap="StudentTeacher">
        select * from student
    </select>

    <select id="getTeacher" resultType="Teacher">
        select * from teacher where id=#{id}
    </select>
```

### 按照结果嵌套查询

```xml
<resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <association property="teacher" javaType="Teacher">
            <result property="name" column="tname"/>
        </association>
    </resultMap>
    <select id="getStudent2" resultMap="StudentTeacher2">
        select s.id sid,s.name sname ,t.name tname 
        from student s,teacher t 
        where s.tid=t.id
    </select>
```

mysql多对一查询方式:

1. 子查询
2. 联表查询



# 一对多处理

### 测试环境搭建

1. 导入lombok
2. 建立实体类Student,Teacher
3. 创建Mapper接口
4. 创建Mapper.xml文件
5. 在核心配置文件中注册
6. 测试

### 按照嵌套查询处理

```xml
<select id="getTeacher2" resultMap="TeacherStudent2">
        select * from teacher where id=#{id}
    </select>
    <resultMap id="TeacherStudent2" type="Teacher">
        <collection property="students" column="id" javaType="ArrayList" ofType="Student" select="getTeacherByTeacherId"/>
    </resultMap>
    <select id="getTeacherByTeacherId" resultType="student">
        select * from student where tid=#{tid}
    </select>
```

### 按照结果嵌套查询

```xml
<resultMap id="TeacherStudent" type="Teacher">
        <result property="id" column="tid"/>
        <result property="name" column="tname"/>
        <!--复杂的属性,需要单独处理,对象:association 集合:collection-->
        <!--集合中的泛型信息,使用ofType获取-->
        <collection property="students" ofType="Student">
            <result property="id" column="sid"/>
            <result property="name" column="sname"/>
            <result property="tid" column="tid"/>
        </collection>
    </resultMap>

    <select id="getTeacher" resultMap="TeacherStudent">
        select s.id sid,s.name sname,t.name tname,t.id tid
        from student s,teacher t
        where s.tid=t.id and t.id=#{tid}
    </select>
```

### 小结

1. 关联-association [多对一]
2. 集合-collection [一对多]
3. javaType & ofType
   1. javaType 用来指定实体类中属性的类型
   2. ofType 用来指定映射到List或者集合中的pojo类型,泛型中的约束类型

### 注意点:

- 保证sql的可读性,尽量保证通俗易懂
- 注意一对多和多对一种,属性名和字段的问题
- 如果问题不好排查错误,可以使用日志,建议使用log4j

# 动态SQl

**什么是动态SQL:就是指不同条件生成不同语句;**

使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

```
if
choose (when, otherwise)
trim (where, set)
foreach
```

### 搭建环境

```sql
CREATE TABLE `blog`  (
  `id` varchar(50)  NOT NULL COMMENT '博客id',
  `title` varchar(100)  NOT NULL COMMENT '博客标题',
  `author` varchar(30)  NOT NULL COMMENT '博客作者',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `views` int NOT NULL COMMENT '浏览量'
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;
```

### 创建一个基础工程

1. 导包
2. 编写配置文件
3. 编写实体类

```java
@Data
public class Blog {
    private String id;
    private String title;
    private String author;
    private Date create_time;
    private int views;
}
```

1. 编写实体类对应的Mapper接口及Mapper.xml文件
2. 测试

```xml
<insert id="addBlog" parameterType="blog">
        insert into mybatis.blog (id,title,author,create_time,views)
        values (#{id},#{title},#{author},#{create_time},#{views})
    </insert>
public void addBlog(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);

        Blog blog = new Blog();
        blog.setId(IdUtils.getId());
        blog.setAuthor("宫习武");
        blog.setCreate_time(new Date());
        blog.setTitle("java");
        blog.setViews(99999);

        mapper.addBlog(blog);

        blog.setId(IdUtils.getId());
        blog.setTitle("c#");
        mapper.addBlog(blog);

        blog.setId(IdUtils.getId());
        blog.setTitle("数据库");
        mapper.addBlog(blog);

        blog.setId(IdUtils.getId());
        blog.setTitle("操作系统");
        mapper.addBlog(blog);

        sqlSession.commit();
        sqlSession.close();

    }
```

### IF语句

```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
        select * from mybatis.blog where 1=1
        <if test="title!=null">
            and title=#{title}
        </if>
        <if test="author!=null">
            and author=#{author}
        </if>
    </select>
```

### Choose(when,otherwise)

```xml
<select id="queryBlogChoose" parameterType="map" resultType="blog">
        select * from mybatis.blog
        <where>
            <choose>
                <when test="title!=null">
                    title=#{title}
                </when>
                <when test="author!=null">
                    and author!=null
                </when>
                <otherwise>
                    and views=#{views}
                </otherwise>
            </choose>
        </where>
</select>
```

### trim(where,set)

```xml
<update id="updateBlog" parameterType="map" >
        update mybatis.blog
        <set>
            <if test="title!=null">
                title=#{title},
            </if>
            <if test="author!=null">
                author=#{author}
            </if>
        </set>
        where id=#{id}
    </update>
```

**所谓动态SQL,本质还是SQL语句,只是我们可以在SQL层面,去执行一个逻辑代码**

### SQL片段

有的时候,我们可能会将一些功能的部分抽取出来,方便复用!

1. 使用SQL标签抽取公共的部分

```xml
<sql id="If-Sql">
        <if test="title!=null">
            title=#{title}
        </if>
        <if test="author!=null">
            and author=#{author}
        </if>
    </sql>
```

1. 在需要使用的地方使用include标签引用即可

```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
        select * from mybatis.blog
        <where>
            <include refid="If-Sql"></include>
        </where>
    </select>
```

**注意事项:**

- 最好基于单表来定义SQL片段
- 不要存在where标签

### foreach

```xml
<select id="queryBlogForeach" parameterType="map" resultType="blog">
        select * from mybatis.blog
        <where>
            <foreach collection="ids" item="id" open="(" close=")" separator="or">
                id=#{id}
            </foreach>
        </where>
    </select>
```

**动态SQL就是在拼接SQL语句,我们只要保证SQL的准确性,按照SQL的格式,去排列组合就可以了**

建议:

- 先在Mysql中写出完整的SQL,再去对应的修改成为我们的动态SQL实现通用即可;

# 缓存

### 一级缓存

- 一级缓存也叫本地缓存:SqlSession
  - 与数据库同义词回话期间查询到的数据会放在本地缓存中;
  - 以后如果需要获取相同的数据,直接从缓存中拿,没必要再去查询数据库;

**测试步骤**

1. 开启日志;
2. 测试在一个Session中查询两次记录
3. 查看日志输出

缓存失效的情况:

1. 查询不同的东西
2. 增删改操作,可能会改变原来的数据,所以必定会刷新缓存;
3. 查询不同的Mapper.xml
4. 手动清理缓存;

```java
SqlSession.clearCache();//手动清理缓存
```

#### 小结:一级缓存默认是开启的,只在一次SqlSession中有效,也就是拿到连接到关闭连接这个区间段;一级缓存就是一个Map;

### 二级缓存

- 二级缓存也叫全局缓存,一级缓存作用域太低了,所以诞生了二级缓存;
- 基于namespace级别的缓存,一个名称空间,对应一个二级缓存;
- 工作机制
  - 一个会话查询一条数据,这个数据就会被放在当前会话的一级缓存中;
  - 如果当前会话关闭了,这个会话对应的一级缓存就没了,但是我们想要的实,会话关闭了,一级缓存中的数据被保存到二级缓存中;
  - 新的会话查询信息,就可以从二级缓存中获取内容;
  - 不同的mapper查出的数据会放在自己对应的缓存中;

#### 步骤:

1. 开启全局的二级缓存

```xml
<setting name="cacheEnabled" value="true"/>
```

1. 在Mapper.xml中使用二级缓存

```xml
<cache/>
```

也可以自定义参数

```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

1. 测试
   1. 问题:我们需要将实体类序列化,否则会报错!(实体类实现Serializable接口)

### 小结

- 只要开启了二级缓存,在同一个Mapper下就有效;
- 所有的数据都会先放在一级缓存中;
- 只有当会话提交,或者关闭的时候,才会提交到二级缓存中;


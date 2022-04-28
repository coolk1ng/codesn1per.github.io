# 面试题目整理


### 1. JavaSE

##### StringBuffer,StringBuilder区别?

1. 两者都是可变的字符串对象,有共同的父类AbstractStringBuilder,并且两个类的构造方法和成员方法也基本相同,不同的是,StringBuffer是线程安全的,而StringBUilder是非线程安全的,所以StringBuilder性能略高





### 2. Spring

##### Spring是什么?

1. 轻量级开源JavaEE框架,它是一个容器框架,用来装javabean,中间层框架可以起一个连接作用,可以让我们企业开发更快速,更简洁.

1. Spring中比较核心的是IOC(控制反转)和AOP(面向切面编程)

* IOC可以帮助我们维护对象与对象之间的依赖关系,降低了对象之间的耦合度,DI是依赖注入的意思,他是IOC实现的方式,指应用运行过程中,对象之间的关系不再需要对象的使用者来维护,而是被IOC容器进行管理注入
* AOP是面向切面编程,是对OOP的补充,可以同一解决一批组件的共性需求(如权限检查,记录日志,事务管理等).我们可以将解决共性需求的代码独立出来,通过配置的方式,声明这些代码在什么地方,什么时机调用.满足调用条件,AOP会将业务代码织入到我们指定的位置,从而同一解决问题,不需要修改这一批组件的代码

##### BeanFactory和ApplicationContext有什么区别?

1. ApplicationContext是BeanFacotry的子接口
2. ApplicationContext提供更完整的功能:
   * 继承MessageSource,支持国际化
   * 同一的资源文件访问方式
   * 提供在监听器中注册bean的事件
   * 同时加载多个配置文件
3. BeanFactory采用延迟加载形式来注入Bean的,即只有使用某个Bean时,才对该Bean进行加载实例化.如果Bean的某一个属性没有注入,BeanFactory加载后,知道第一次使用这个Bean才会抛出异常
4. ApplicationContext是在容器启动时,一次性创建所有的bean,在启动时就可以发现spring中存在的配置问题,这样有利于检查所依赖属性是否注入.
   1. 两者都都支持BeanPostProcessor,BeanFactoryPostProcessor的使用,但两者的区别是:BeanFactory需要手动注册,而ApplicationContext则是自动注册

##### SpringBean的生命周期?

1. Spring中的bean的生命主要包含四个阶段:实例化Bean->Bean属性填充->初始化Bean->销毁Bean

2. 首先是实例化Bean,当客户端想容器请求一个尚未初始化的bean时,或初始化bean的时候需要注入另一个尚未初始化的bean的依赖时,容器就会调用doCreatebean()方法进行实例化,实际上就是通过反射的方式创建出一个bean对象

3. Bean实例创建出来后,接着就是给这个bean对象进行属性填充,也就是注入这个bean依赖的其他bean对象

4. 属性填充完成后,进行初始化Bean操作,初始化阶段可以分为几个步骤

   1. 执行Aware接口的方法

      Spring会检测该对象是否实现了xxxAware接口,通过Aware类型的接口,可以让我们拿到Spring容器的一些资源.如实现BeanNameAware接口可以获取到BeanName,实现BeanFactoryAware接口可以获取工厂对象BeanFactory等等

   2. 执行BeanPostProcessor的前置处理方法PostProcessbeforeInitiaization(),对bean进行一些自定义的前置处理

   3. 判断bean是否实现了initializingBean接口,如果实现了,将会执行initializingBean的afterPropertiesSet()初始化方法

   4. 执行用户自定义的初始化方法,如init-method等

   5. 执行BeanPostProcessor的后置处理方法postProcessAfterInitialization()

5. 初始化完成后,Bean就成功创建了,之后就可以使用这个bean,当bean不在需要时,会进行销毁操作

   1. 首先判断bean是否实现了DestructionAwarebeanPostProcessor接口,如果实现了,则会执行DestructionAwareBeanPostProcessor后置处理器的销毁回调方法
   2. 其次会判断Bean是否实现了DisposableBean接口,如果实现了将会调用其实现destroy方法
   3. 最后判断这个bean是否配置了destroy-method等自定义的销毁方法,如果有的话,则会自动调用其配置的销毁方法

##### Spring的作用域?

1. singleton: 默认,每个容器中只有一个bean的实例,单例的模式由BeanFactory自身来维护.该对象的生命周期与Spring IOC容器一致的(但在第一次被注入时才会创建)
2. prototype: 为每个bean请求提供一个实例,在每次注入时都会创建一个新的对象
3. request: bean被定义每个HTTP请求中创建一个单例对象,也就是说在单个请求中都会复用这一个单例对象
4. session: 与request范围类似,确保每个session中都有一个bean的实例,在session过期后,bean会随之失效
5. application: bean被定义在ServletContext的生命周期中复用一个单例对象
6. websocket: bean被定义在websocket的生命周期中复用一个单例对象

##### SpringMVC的执行流程?

![image-20220407230755791](/image-20220407230755791.png)

1. DispatcherServlet接受请求,调用HandleMapping(处理器映射器)
2. HandleMapping找到具体的处理器(可查找xml配置或注解配置),生成处理器对象及处理器拦截器(如果有),在一起返回给DispatcherServlet
3. DispatcherServlet调用HandleAdapter(处理器适配器)
4. HandleAdapter经过适配器调用具体的处理器(Handle/Controller),返回ModelAndView给DispatcherServlet
5. DispatcherServlet调用ViewResolver(视图解析器),根据返回的ModelAndView中的逻辑视图名为DispatcherServlet返回一个可用的view实例
6. DispatcherServlet根据View渲染视图,响应给用户

##### Spring中的Bean是线程安全的么?

1. Spring本省没有针对Bean做线程安全的处理
2. 如果Bean是无状态的,那么Bean则是线程安全的,反之
3. 另外,Bean是不是线程安全,跟Bean的作用于没有关系,Bean的作用于至少表示Bean的生命周期范围,对于任何声明周期的Bean都是一个对象,是不是线程安全还得看Bean本身

##### Spring中的事务是如何实现的?

1. Spring事务底层是基于数据库事务和AOP机制
2. 对使用了@Transactional注解的Bean,Spring会创建一个代理对象作为Bean
3. 如果代理对象的方法加了注解,则利用事务管理器创建数据连接
4. 修改数据库连接的autocommit为false,禁止此连接的自动提交
5. 执行方法,方法会执行sql,没有异常就提交事务
6. Spring事务的隔离级别就是数据库的隔离级别



### 3. MySQL

##### 对MySQL索引的理解?

1. 索引时单独的,存储在磁盘上的数据结构,包含对数据表中所有记录的引用指针.索引可以快速找到在某个或多个列有一特定值的行,所有MySQL列的类型都可以被索引.
2. 索引时在存储引擎中实现的,因此,每种存储引擎的所有都不一定完全相同,并且每种存储引擎也不一定支持所有索引类型.MySQL中所有的存储类型有两种,即B+树和Hash,具体和表的存储引擎相关.

##### 索引的优点?

1. 通过创建唯一索引,可以保证数据库中的每一行数据的唯一性
2. 加快数据的查询速度,降低数据库的IO成本,这也是创建索引的主要原因
3. 使用分组和排序子句进行数据检索时,同样可以显著减少查询中分组和排序的时间
4. 进行表连接操作时,索引加速表连接操作

##### 索引的缺点?

1. 占用一定的磁盘空间
2. 在频繁插入,删除操作时,MySQL需要动态维护索引

##### 索引有哪几种?

1. 普通索引和唯一索引
   * 普通索引时MySQL中的基本索引类型,允许在定义索引的列中插入重复值和空值
   * 唯一索引要求索引列的值必须唯一,但允许有空值,如果是组合索引,则列值的组合必须唯一
   * 主键索引时一种特殊的唯一索引,不允许有空值
2. 单列索引和组合索引
   * 单列索引即一个索引只包含单个列,一个表可以有多个单列索引
   * 组合索引是指在表的多个字段组合上组件的索引,只有在查询条件中使用了这些字段的左边字段时,索引次啊会被使用,使用组合索引时遵循最左前缀法则
3. 全文索引
   * 全文索引类型为FULLTEXT,在定义一索引的列上支持值的全文查找,允许在这些索引列中插入重复值和空值.全文索引可以在CHAR,VARCHAR或者TEXT类型的列上创建
4. 空间索引
   * 空间索引是对空间数据类型的字段建立的索引，MySQL中的空间数据类型有4种，分别是GEOMETRY、POINT、LINESTRING和POLYGON。MySQL使用SPATIAL关键字进行扩展，使得能够用创建正规索引类似的语法创建空间索引。创建空间索引的列，必须将其声明为NOT NULL，空间索引只能在存储引擎为MyISAM的表中创建。

##### 如何避免索引失效?

1. 使用组合索引,遵循"最左前缀"原则
2. 不在索引列上做任何操作,例如计算,函数,类型转换,会导致索引失效而转向全表扫描
3. 尽量使用覆盖索引(只访问索引列的查询),减少select * 覆盖索引能减少回表次数
4. 在使用不等于(!=或者<>)的时候无法使用索引会导致全表扫描
5. LIKE以通配符开头(%abc),MySQL会失效变成全表扫描的操作
6. 字符串不加单引号会导致索引失效(可能发生索引列的隐式转换)
7. 少用or,用它来连接时会索引失效

##### 不适合建索引的场合

1. 频繁更新的字段
2. where条件中用不到的字段不适合建立索引
3. 数据比较少的表不需要建索引
4. 数据重复且分布比较均匀的字段不适合建索引,例如性别,真假值
5. 参与列计算的列不是建索引





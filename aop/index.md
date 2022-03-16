# Aop


### AOP是什么

* AOP(Aspect Oriented Programming),意味面向切面编程,可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态同一添加功能的一种技术.

* AOP的编程思想就是把很多类对象的横切问题点,从业务逻辑中分离出来,从而达到解耦的目的,增加代码的重用性,提高开发效率

### AOP应用场景

* 日志记录
* 异常处理
* 权限验证
* 缓存处理
* 事务处理
* 数据持久化
* 效率检查
* 内容分发

### Aspect概念

* aspect:切面,切面有切点和通知组成,即包括横切逻辑的定义也包括连接点的定义
* pointcut: 切点,每个类都拥有多个连接点,可以理解是连接点的集合
* joinpoint: 连接点,程序执行的某个特定位置,如某个方法调用前后等
* weaving: 织入,将增强添加到目标类的具体连接点的过程
* advice: 通知,是织入到目标类连接点上的一段代码,就是增强到什么地方?增强什么内容?
* target: 目标对象,通知织入的目标类
* aop Proxy: 代理对象,即增强后产生的对象

**Spring AOP底层实现,是通过JDK动态代理或CGlib代理在运行时期在对象初始化阶段织入代码的**



> * Before advice
>
>   前置通知,即在目标方法调用之前执行.注意: 无论方法是否遇到异常都执行
>
> * After returning advice
>
>   后置通知,在目标方法执行后执行,前提是目标方法没有遇到异常,如果有异常则不执行通知
>
> * After throwing advice
>
>   异常通知,在目标方法抛出异常时执行,可以获取异常信息
>
> * After finally advice
>
>   最终通知,在目标方法执行后执行,无论是否是异常执行
>
> * Around advice
>
>   环绕通知,最强大的通知类型,可以控制目标方法的执行(通过调用ProceedingJoinPoint.proceed()),可以在目标执行全过程进行执行

> **@Aspect驱动**
>
> * 定义一个切面类Aspect
>
>   即在声明的类,增加@Component @Aspect两个注解,Springboot中引入Spring-boot-starter-aop依赖
>
> * 定义切点Pointcut
>
>   定义切点,并定义切点在哪些地方执行,采用@pointcut注解完成,如@Pointcut(public * com.xxx.xxx.*.*(..))
>
>   **规则:修饰符(可以不写,但不能用*)+返回类型+哪些包下的类+那些方法+方法参数"*"代表不限,".."两个点代表参数不限 **
>
> * 定义Advice通知
>
>   利用通知的5中类型注解@Before,@After,@AfterReturning,@AfterThrowing,@Around来完成在某些切点的增强动作,如@Before("myPointcut(),mypointcut为第二步骤定义的切点")



# SpringBean的生命周期


* 阶段一: 处理名称,检查缓存
* 阶段二: 检查父工厂
* 阶段三: 检查DependsOn
* 阶段四: 按Scope创建bean
  * 创建singleton
  * 创建prototype
  * 创建其他scope
* 阶段五: 创建bean
  * 创建bean实例
  * 依赖注入
  * 初始化
  * 登记可销毁bean
* 阶段六: 类型转换
* 阶段七: 销毁bean

### 阶段一: 处理名称,检查缓存

* 先把别名解析为实际名称,再进行后续处理
* 若要FactoryBean本身,需要使用&名称获取
* singletonObjects是一级缓存,放单例成品对象
* singletonFactories是三级缓存,放单例工厂
* earlySingletonObjects是耳机缓存,放单例工厂的产品,可成为提前单例对象

### 阶段二: 处理父子容器

### 阶段三: 

### 阶段四: 按Scope创建bean

* scope理解为从xxx范围内找到bean更加贴切
* singleton scope表示从单例池范围获取bean,如果没有,则创建并放入单例池
* prototype scope表示从不缓存bean,每次都创建新的
* request scope表示从request对象范围内获取bean,如果没有,则创建并放入request

### 阶段五: 创建bean

##### 1. 创建bean实例

![image-20220327134624984](/image-20220327134624984.png)

* FactoryMethod方式创建bean实例
  * 分成静态工厂与实例工厂
  * 工厂方法若有参数,需要对工厂方法参数进行解析,利用resolveDependency
  * 如果有多个工厂方法候选者,还要进一步按权重筛选

* AutowiredAnnotationBeanPostProcessor 
  * 优先选择带@Autowired注解的构造
  * 若有唯一的带参构造,也会入选
* 才有默认构造
  * 如果上面的后处理器和BeanDefiniation都没找到构造,采用默认构造,即使是私有的

##### 2. 依赖注入

* AutowireAnnotationBeanPostProcessor(注解匹配)
  * 识别@Autowired及@Value标注的成员,封装为InjectionMetadata进行依赖注入
* CommonAnnotationBeanPostProcessor(注解匹配)
  * 识别@Resource标注的成员,封装为InjectionMetadata进行依赖注入
* AUTOWIRE_BY_NAME(根据名字匹配)
  * 根据成员名字找bean对象,修改mbd的propertyValues,不会考虑简单类型的成员
* AUTOWIRE_BY_TYPE(根据类型匹配)
  * 根据成员类型执行resolveDependency找到依赖注入的值,修改mbd的propertyValues
* applyPropertyValues(xml中<property name ref|value/>)
  * 根据mbd的propertyValues进行依赖注入


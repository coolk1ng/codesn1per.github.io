# SpringMVC


## 1. SpringMVC组件

![image-20210727011720811](../../static/image-20210727011720811.png)

## 2. 请求方式

![image-20210727012056120](../../static/image-20210727012056120.png)

## 3. 处理器方法的参数

处理器方法可以包含一下四类参数,这些参数会在系统调用时由系统自动赋值,即程序员可在方法内直接使用;

* HttpServletRequest
* HttpServletResponse
* HttpServletSession
* 请求中所携带的请求参数

### 3.1 逐个参数接收

逐个接收:请求中的参数名和控制器方法的形参名一样;

接受参数的问题:

1. 参数最好使用包装类型,例如Integer,能接收空值情况,接收的是null
2. post请求中有乱码问题,使用字符集过滤器

### 3.2 请求中参数名和形参名不一样,使用@RequestParam

@RequestParam: 解决名称不一样的问题

1. 属性: value 请求中的参数名称

   ​			required: boolean类型的,默认是true

   ​							  true: 请求中必须有此参数

   ​							  false:请求中可以没有此参数

2. 位置: 在形参定义的前面

### 3.3 对象接收

对象接收: 在控制器方法的形参是java对象,使用java对象的属性接收请求中的参数值

要求: java对象的属性名和请求中的参数名一样



## 4. SpringMVC执行流程

![image-20210727023639950](../../static/image-20210727023639950.png)

SpringMVC内部请求的处理过程:

1. 用户发起请求给DispatcherServlet

2. DispatcherServlet把请求(request)交给处理器映射器

   处理器映射器:springmvc框架中的对象,需要实现HandlerMapping接口

   映射器作用: 从springmvc容器中,获取控制器对象,把找到的控制器和拦截器对象都放到处理器执行链对象中,保存,并返回给中央调度器;

3. DispatcherServlet把获取到的处理器执行链中的控制器对象,交给处理器适配器;

   处理器适配器: 是springmvc框架中的实现.实现HandlerAdapter接口

   适配器作用: 执行控制器的方法,得到结果ModelAndView;

4. DispatcherServlet把控制器执行结果交给了视图解析器

   视图解析器: springmvc中的对象,需要实现ViewResolver接口

   视图解析器作用:处理视图的,组成视图的完整路径,能创建View类型的对象;

5. DispatcherServlet调用View类的方法,把Model中的数据放到request作用域;


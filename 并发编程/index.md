# 并发编程


## 1. 进程与线程

### 1.1 进程与线程

##### 1.1.1 进程

> * 程序由指令和数据组成,但这些指令要运行,数据要读写,就必须将指令加载至CPU,数据加载至内存.在指令运行过程中还需要用到磁盘,网络等设备.进程用来加载指令,管理内存,管理IO的.
> * 当一个程序被运行,从磁盘加载这个程序至内存,这时就开启了一个进程.
> * 进程就可以视为程序的一个实例.大部分程序可以同时运行多个实例进程,也有的程序只能启动一个实例进程.

##### 1.1.2 线程

> * 一个进程之内可以分一到多个线程.
> * 一个线程就是一个指令流,将指令流中的一条条指令以一定的顺序交给CPU执行
> * Java中,线程作为最小调度单位,进程作为资源分配的最小单位,在Windows中进程是不活动的,知识作为线程的容器.

##### 1.1.3 二者对比

* 进程基本上相互独立,线程在进程内,是进程的一个子集.
* 进程拥有共享的资源,如内存空间等,供内部的线程共享.
* 进程间通信较为复杂
  * 同一台计算机的进程通信成为IPC
  * 不同计算机之间的进程通信,需要通过网络,并遵守共同的协议.
* 线程通信相对简单,因为他们共享进程的内存,一个例子是多个线程可以访问同一个共享变量
* 线程更轻量,线程上下文切换成本一般要比进程上下文切换低.

### 1.2 并行与并发

> * 单核cpu下,线程实际还是串行执行的.操作系统中有一个组件叫做任务调度器,将cpu的时间片分给不同的线程使用,只是由于cpu在线程间的切换非常快,感觉是同时运行的.
> * 一般会将这种线程轮流使用cpu的做法成为并发.
> * 多核cpu下,每个核都可以调度运行线程,这时候线程是并行的.

> * 并发: 同一时间应对多件事情的能力.
> * 并行: 同一时间动手做多件事情的能力.

### 1.3 应用

##### 1.3.1 异步调用案例

> 从方法调用角度来说: 
>
> * 需要等地啊结果返回,才能继续运行就是同步
> * 不需要等待结果返回,就能继续运行就是异步
>
> 注意: 同步在多线程中海油另外一层的意思,是让多个线程步调一致
>
> ##### 设计: 
>
> 多线程可以让方法执行变为异步的,比如说读取磁盘文件时,假设读取操作话费了5s,如果没有线程调度机制,这5s调用者什么都做不了,其代码都得暂停.
>
> ##### 结论: 
>
> * 比如在项目中,视频文件需要转换格式等操作比较费时,这时开一个新线程处理视频转换,避免阻塞主线程.
> * tomcat的异步servlet也是类似的目的,让用户线程处理耗时比较长的操作,避免阻塞tomcat的工作线程

##### 1.3.2 提高效率案例

> 充分利用多核cpu的有事,提高运行效率.想象下面的场景,执行3个计算,最后将计算结果汇总.
>
> ```
> 计算 1 花费 10 ms
> 计算 2 花费 11 ms
> 计算 3 花费 5 ms
> 汇总需要 1 ms
> ```
>
> * 如果是串行执行,name总共花费 10 + 11 + 5 + 1 = 27ms
> * 但如果是四核cpu,各个核心分别使用线程1执行计算1 ,线程2执行计算2.....花费时间只取决于最长的那个线程运行的时间,及11ms 最后加上汇总的时间只会花费12ms
>
> ```
> 注意: 需要在多核cpu才能提高效率,单核仍然是轮流执行
> ```
>
> ##### 结论
>
> * 单核cpu下,多线程不能实际提高程序运行效率,知识为了能够在不同的任务之间切换,不同线程轮流使用cpu,不至于一个线程总占用cpu,别的线程没法干活
> * 多核cpu可以并行跑多个线程,但能否提高程序运行效率还是要分情况的
>   * 有些任务,经过精心设计,将任务拆分,并行执行,当然可以提高程序的运行效率,但不是所有计算任务都能拆分
>   * 也不是所有任务都需要拆分,任务的目的如果不同,谈拆分和效率没啥意义
> * IO操作不占用cpu,知识我们一般拷贝文件使用的是阻塞IO,这相当于线程虽然不用cpu,但需要一直等待IO结束,没能充分利用线程,所以才有后面的非阻塞IO和异步IO优化

## 2. Java线程

### 2.1 创建和运行线程

##### 2.1.1 方法一,直接使用Thread

```java
public class CreateThread {
    public static void main(String[] args) {
        Thread t = new Thread(()->{
            System.out.println(Thread.currentThread().getName()+ "->" + "running...");
        });
        t.setName("t1");
        t.start();
        System.out.println(Thread.currentThread().getName()+ "->" + "running...");
    }
}		
```

##### 2.1.2 方法二,使用Runnable配合Thread

```java
public class CreateThreadByRunnable {
    public static void main(String[] args) {
        //创建任务对象
        Runnable runnable = () -> System.out.println(Thread.currentThread().getName() + "->" + "running");
        Thread t1 = new Thread(runnable, "t1");
        t1.start();

        System.out.println(Thread.currentThread().getName() + "->" + "running");
    }
}
```

##### 2.1.3 方法三,FutureTask配置Thread

```java
public class CreateThreadByFutureTask {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask<>(()->{
            System.out.println(Thread.currentThread().getName() + "running...");
            Thread.sleep(2000);
            return "continue running...";
        });
        Thread thread = new Thread(futureTask,"t1");
        thread.start();
        System.out.println(futureTask.get());
    }
}
```



### 2.2 观察多个线程同时运行

主要是理解

* 交替执行
* 谁先谁后,不由我们控制

```java
public class ThreadRunSameTime {
    public static void main(String[] args) {
        //线程t1
        new Thread(()->{
            while (true) {
                System.out.println(Thread.currentThread().getName()+ "running...");
            }
        },"t1").start();

        //线程t2
        new Thread(()->{
            while (true) {
                System.out.println(Thread.currentThread().getName()+ "running...");
            }
        },"t2").start();
    }
}							
```

### 2.3 查看进程线程的方法

##### linux

>* ps -fe 查看所有进程
>
>* ps -fT -p <PID> 查看某个进程(PID) 的所有信息
>
>* kill 杀死进程
>
>* top 按大写H切换是否显示进程
>
>* top -H -p <PID> 查看某个进程(PID)的所有信息

##### Java

>* jps 查看所有Java进程
>* jstack <PID> 查看某个Java进程(PID)的所有线程状态
>* jconsole 查看某个Java进程中线程的运行状态(图形界面)

### 2.4 线程运行原理

##### 2.4.1 栈与栈帧

> 我们知道JVM中由堆,栈,方法区所组成,其中栈内存是给谁用的呢?其实就是线程,每个线程启动后,虚拟机就会为其分配一块栈内存.
>
> * 每个栈由多个栈帧(Frame)组成,对应着每个方法调用时所占用的内存
> * 每个线程只能有一个活动栈帧,对应着当前正在执行的那个方法

##### 2.4.2 线程上下文切换(Thread Context Switch)

> 因为以下一些原因导致cpu不在执行当前线程,转而执行另一个线程的代码
>
> * 线程的cpu时间片用完
> * 垃圾回收
> * 有更高优先级的线程需要运行
> * 线程自己调用sleep,yield,wait,join,park,synchronized,lock等方法

> 当Context Switch 发生时,需要由操作系统保存当前线程的状态,并恢复另一个线程的状态,Java中对应的概念就是程序计数器,它的作用就是记住下一条jvm指令的执行地址,是线程私有的
>
> * 状态包括程序计数器,虚拟机栈中的每个栈帧的信息,如局部变量,操作数栈,返回地址等
> * COntext Switch 频繁发生会影响性能

### 2.5 常见方法

|          方法名          | 功能说明                                                  | 注意                                                         |
| :----------------------: | :-------------------------------------------------------- | ------------------------------------------------------------ |
|         start()          | 启动一个新线程,在新的线程运行run方法中的代码              | start方法只是让线程进入就绪,里面代码不一定立刻执行(cpu的时间片还没分给它).每个线程对象的start方法只能调用一次,如果调用了多次会出现IllegalThreadStateException |
|          run()           | 新线程启动后会调用的方法                                  | 如果在构造Thread对象时传递了Runnable参数,则线程启动后会调用Runnable中的run方法,否则默认不执行任何操作,但可以创建Thread的子类对象,来覆盖默认行为 |
|          join()          | 等待线程运行结束                                          |                                                              |
|       join(long n)       | 等待线程运行结束最多等待n毫秒                             |                                                              |
|         getId()          | 获取线程长整形id                                          | id唯一                                                       |
|        getName()         | 获取线程名                                                |                                                              |
|     setName(String)      | 修改线程名                                                |                                                              |
|      getPriority()       | 获取线程优先级                                            |                                                              |
|     setPriority(int)     | 修改线程优先级                                            | Java中规定线程优先级是1~10的整数,较大的优先级能提高该线程被cpu调用的几率 |
|        getState()        | 获取线程黄台                                              | Java中线程状态是用6个enum表示,分别是: NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING,TERMINATED |
|     isInterrupted()      | 判断是否被打断                                            | 不会清除                                                     |
|         isAlive          | 线程是否存活                                              |                                                              |
|       interrupt()        | 打断线程                                                  | 如果被打断线程正在sleep,wait,join会导致被打断的线程抛出InterruptedExcepiton,并清除打断标记;如果打断的正在运行的线程,则会设置打断标记;park的线程被打断,也会设置打断标记 |
|  interrupted() (static)  | 判断当前线程是否打断                                      | 会清除打断标记                                               |
| currentThread() (static) | 获取当前正在执行的线程                                    |                                                              |
|  sleep(long n) (static)  | 让当前执行的线程休眠n毫秒,休眠时让出cpu的时间片给其他线程 |                                                              |
|     yield() (static)     | 提示线程调度器让出当前线程对cpu的使用                     | 主要用于测试和调试                                           |

### 2.6 start与run

##### 调用run

> ```java
> public class StartAndRun {
>     public static void main(String[] args) {
>         Thread thread = new Thread("t1"){
>             @Override
>             public void run() {
>                 System.out.println(Thread.currentThread().getName() + "->" +"running...");
>             }
>         };
> 
>         thread.run();
>         System.out.println(Thread.currentThread().getName() + "->" +"do other things...");
>     }
> }
> ```

![image-20220223001157228](/image-20220223001157228.png)

>程序仍然在main线程运行,方法调用还是同步

##### 调用start

> ```java
> public class StartAndRun {
>     public static void main(String[] args) {
>         Thread thread = new Thread("t1"){
>             @Override
>             public void run() {
>                 System.out.println(Thread.currentThread().getName() + "->" +"running...");
>             }
>         };
> 
>         //thread.run();
>         thread.start();
>         System.out.println(thread.getState());
>         System.out.println(Thread.currentThread().getName() + "->" +"do other things...");
>     }
> }
> ```
>
> 



![image-20220223001800453](/image-20220223001800453.png)

>方法调用是异步的!!!!

### 2.7 sleep与yield

##### sleep

>1. 调用sleep会让当前线程从Running进入Timed Waiting状态
>2. 其他线程可以使用interrupt方法打断正在睡眠的线程,这是sleep方法会抛出InterrupttedException
>3. 睡眠结束后的线程尾部会立刻得到执行
>4. 建议用TimeUnit的sleep代替Thread的sleep来获得更好的可读性

##### yield

>1. 调用yield会让当前线程从Running进入Runnable状态,然后调度执行其他同优先级的线程.如果这是没有同优先级的线程,name不能保证让当前线程暂停的效果
>2. 具体的实现依赖操作系统的任务调度器

##### 线程优先级

> * 线程优先级会提示(hint)调度器优先级调度该线程,但他仅仅是一个提示,调度器可以忽略他
>
> * 如果cpu比较忙,name优先级高的线程会获得更多的时间片,但cpu闲时,优先级几乎没有用

```java
public class Priority {
    public static void main(String[] args) {
        //任务一
        Runnable task1 = ()->{
            int count = 0;
          for(;;){
              System.out.println(Thread.currentThread().getName() + "->" +count++);
          }
        };

        //任务二
        Runnable task2 = ()->{
            int count = 0;
            for(;;){
                //让出时间片
                Thread.yield();
                System.out.println(Thread.currentThread().getName() + "----------------->" +count++);
            }
        };
        Thread t1 = new Thread(task1, "t1");
        Thread t2 = new Thread(task2, "t2");
        t1.setPriority(Thread.MAX_PRIORITY);
        t2.setPriority(Thread.MIN_PRIORITY);
        t1.start();
        t2.start();
    }
}

```

##### 案例-防止CPU占用100%

##### sleep实现:

> 在没有利用cpu计算时,不要让while(true)空转浪费cpu,这时可以使用yield或sleep来让出cpu的使用权给其他程序
>
> ```java
> while (true){
>             try {
>                 Thread.sleep(500);
>             }catch (InterruptedException e){
>                 e.printStackTrace();
>             }
>             }
>         }
> ```
>
> * 可以用wait或条件变量达到类似的效果
> * 不同的是,或两种都需要加锁,并且需要响应的唤醒操作,一般适用于进行同步的场景
> * sleep使用与无需锁同步的场景

### 2.8 join方法详解

###### 为什么需要join?

```java
public class JoinThread {
    static int count = 0;
    public static void main(String[] args) {
        test();
    }

    public static void test(){
        Thread t1 = new Thread(()->{
            try {
                System.out.println("start");
                sleep(1);
                System.out.println("end");
                count = 10;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        thread.start();
        System.out.println("结果是--->" +count);
        System.out.println("end");
    }
}
```

##### 分析:

* 因为主线程和线程t1是并行执行的,t1线程需要1秒后才能得到count = 10
* 而主线程一开始就要打印count的结果,所以只能打印count = 0

##### 解决方法

主线程中对t1调用join()方法,等待t1线程执行完毕

```java
public class JoinThread {
    static int count = 0;
    public static void main(String[] args) throws InterruptedException {
        test();
    }

    public static void test() throws InterruptedException {
        Thread t1 = new Thread(()->{
            try {
                System.out.println("start");
                sleep(1);
                System.out.println("end");
                count = 10;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        t1.start();
        t1.join();
        System.out.println("结果是--->" +count);
        System.out.println("end");
    }
}
```

### 2.9 interrupt方法详解

##### 打断sleep,wait,join的线程

```java
public class InterruptThread {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(()->{
            try {
                System.out.println("sleep...");
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t1");

        t1.start();
        Thread.sleep(2000);
        t1.interrupt();
        System.out.println("打断标记->" + t1.isInterrupted());
    }
}
```

##### 打断正常运行的线程(不会清空打断标记)

```java
public class InterruptNormalThread {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(()->{
            while (true){
                boolean flag = Thread.currentThread().isInterrupted();
                if (flag){
                    System.out.println("打断状态->" + flag);
                    break;
                }
                System.out.println("打断状态->" + flag);
            }
        },"t1");

        t1.start();
        Thread.sleep(1000);
        System.out.println("interrupt");
        t1.interrupt();
    }
}
```

### 2.10 两阶段终止模式

在一个线程T1中如何"优雅"的终止线程T2? 这里的"优雅" 指的是给T2一个机会去料理后事;

##### 错误思路:

>* 使用线程对象的stop方法停止
>  * stop方法会真正杀死线程,如果这是线程锁住了共享资源,那么当他被杀死后就再没机会释放锁,其他区线程将永远无法获取锁
>* 使用System.exit(int) 方法停止线程
>  * 目的仅仅是停止一个线程,但这种做法会让整个程序都停

```mermaid
graph TD
w("while(true)") --> a
a("有没有被打断?") -- 是 --> b(料理后事)
b--> c((结束循环))
a -- 否 --> d(睡眠2s)
d -- 无异常 --> e(执行监控记录)
d -- 有异常 --> i(设置打断标记)
i --> w
e --> w
```



##### 实现:

```java
public class TwoPhaseTerminationTest {
    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination twoPhaseTermination = new TwoPhaseTermination();
        twoPhaseTermination.start();
        Thread.sleep(3000);
        twoPhaseTermination.stop();
    }
}
    class TwoPhaseTermination {
        private Thread monitor;

        /**
         * 启动线程
         * @return void
         */
        public void start(){
           monitor = new Thread(()->{
               Thread current = Thread.currentThread();
               while (true){
                   if (current.isInterrupted()){
                       System.out.println("线程被打断了");
                       break;
                   }
                   try {
                       // 情况一: 睡眠状态被打断
                       Thread.sleep(1000);
                       // 情况二: 执行监控记录被打断
                       System.out.println("执行监控记录");
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                       //重新设置打断标记
                       monitor.interrupt();
                   }
               }

           });
           // 启动线程
           monitor.start();
        }

        /**
         * 停止线程
         * @return void
         */
        public void stop(){
            monitor.interrupt();
        }
    }
```

### 2.11 打断park线程

```java
public class InterruptParkThread {
    public static void main(String[] args) throws InterruptedException {
        test();
    }

    public static void test() throws InterruptedException {
        Thread t1 = new Thread(()->{
            System.out.println("park...");
            LockSupport.park();
            System.out.println("unPark...");
            System.out.println("打断状态->" + Thread.currentThread().isInterrupted());
        },"t1");
        t1.start();

        Thread.sleep(1000);
        t1.interrupt();
    }
}
```

打断标记为true时,park失效

### 2.12 不推荐的方法

还有一些不推荐的方法,这些方法已经过世,容易破坏同步代码块,造成线程死锁

| 方法名    | 功能说明           |
| --------- | ------------------ |
| stop()    | 停止线程运行       |
| suspend() | 挂起(暂停)线程运行 |
| resume()  | 恢复线程运行       |

### 2.13 主线程与守护线程

> 默认情况下,Java进程需要等待所有线程都运行结束,才会结束,有一种特殊的线程叫做守护线程,只要其他守护线程运行结束,即使守护线程的代码没有执行完,也会强制结束.

例如:

```java
public class DaemonThread {
    public static void main(String[] args) throws InterruptedException {
        Thread daemon = new Thread(()->{
            while (true){
                System.out.println("running...");
            }
        });
        // 设置为守护线程
        daemon.setDaemon(true);
        daemon.start();

        Thread.sleep(1000);
        System.out.println("end...");
    }
}
```

##### 注意:

> * 垃圾回收线程就是一种守护线程
> * Tomcat中的Acceptor和Poller线程都是守护线程,所以Tomcat接受到shutdown命令后,不会等待他们处理完当前请求

### 2.14 五种状态

##### 这是从操作系统层面来描述的:

![image-20220224043238257](/image-20220224043238257.png)

* 初始状态: 仅是在语言层面创建了线程对象,还未与操作系统线程关联
* 可运行状态: (就绪状态) 指该线程已经被创建(与操作系统线程关联).可以由cpu调度执行
* 运行状态: 指获取了cpu时间片运行中的状态
  * 当cpu时间片用完,会从运行状态转换至可运行状态,会导致线程的上下文切换
* 阻塞状态
  * 如果调用了阻塞API,如BIO读写文件,这时该线程实际不会用到cpu,会导致线程上下文切换,进入阻塞状态
  * 等BIO操作完毕,会由操作系统唤醒阻塞的线程,转换至可运行状态
  * 与可运行状态的区别是,对阻塞状态的线程来说只要他们一直不唤醒,调度一直不会考虑调度他们
* 终止状态: 表示线程已经执行完毕,声明周期已经结束,不会再转换为其他状态

### 2.15 六种状态

##### 这是从Java API层面来描述的,根据Thread.State枚举,分为6种状态

![image-20220224044431446](/image-20220224044431446.png)

* NEW: 线程刚被创建,但是还没有调用start()方法
* RUNNABLE当调用了start()方法之后,注意,Java API层面的RUNNABLE状态涵盖了操作系统层面的 可运行状态,运行状态和阻塞状态(由于BIO导致的线程阻塞,在Java中无法区分,仍然认为是可运行的)
* BLOCKED,WAITING,TIMED_WAITING都Java API层面对阻塞状态的细分
* TERMINATED: 当线程代码运行结束


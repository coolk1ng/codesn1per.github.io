# IO流


## 1. 文件

### 1. 什么是文件?

**保存数据的地方**

### 2. 文件流

文件在程序中是以流的形式来操作的

![image-20210710181321189](/image-20210710181321189.png)

流: 数据在数据源(文件)和程序(内存)之间经历的路径

输入流: 数据从数据源(文件)到程序(内存)的路径

输出流: 数据从程序(内存)到数据源(文件)的路径

### 3. 常用文件操作

* new File(String pathname) //根据路径创建一个File对象
* new File(File parent,String child) //根据父目录文件+子路径创建
* new File(String parent,String child) //根据父目录+子路径构建
* createNewFile 创建文件
* getName,getAbsoutePath,getParent,length,exists,isFile,isDirectory
* mkdir:创建一级目录,mkdirs: 创建多级目录,delete:删除空目录或文件

## 2. IO流原理及流的分类

### 1. io流原理

1. I/O是input/output的缩写,I/O技术是非常实用的技术,用于处理数据传输,如读写文件,网络通信等;
2. java程序中,对于数据的输入/输出操作以流(stream)的方式进行;
3. java.io包下提供各种"流"类和接口,用以获取不同种类的数据,并通过方法输入或输出数据;
4. 输入input:读取外部数据(磁盘,光盘等存储设备的数据)到程序(内存)中;
5. 输出output:将程序数据输出到磁盘,光盘等存储设备中;

### 2. 流的分类

1. 按操作数据单位不同分为: 字节流(8bit) 二进制文件,字符流(按字符) 文本文件;

2. 按数据流的流向不同分为: 输入流,输出流

3. 按流的角色的不同分为: 节点流,处理流/包装流

   ![image-20210710225844363](/image-20210710225844363.png)

   1. java的IO流共设计40多个类,实际上非常规则,都是从如上4个抽象基类派生的.
   2. 由这四个类派生出来的子类名称都是以其父类作为子类名后缀;

## 3. IO流常用的类

![image-20210710230634863](/image-20210710230634863.png)

### 1. FileInputStream/FileOutStream

### 2. FileReader/FileWriter

是字符流,按照字符来操作io

**FileReader相关方法:**

* new FileReader(File/String)
* read: 每次读取单个字符,返回该字符,如果到文件末尾返回-1;
* read(char[]): 批量读取多个字符到数组,返回读取到的字符数,如果到文件末尾返回-1;
* new String(char[]): 将char[]转换成String
* new String(char[],off,len): 将char[]的指定部分转换成String

**FileWriter相关方法:**

* new FileWriter(File/String): 覆盖模式,相当于流的指针在首端
* new FileWriter(File/String,true): 追加模式,相当于流的指针在末尾
* write(int) : 写入单个字符
* write(char[]):写入指定数组
* write(char[],off,len): 写入指定数组的指定部分
* write(string):写入整个字符串
* write(string,off,len): 写入字符串的指定部分
* toCharArray:将String转换成char[]

**注意:FileWriter使用后,必须要关闭或刷新,否则写入不到指定的文件**

### 3. 节点流和处理流

![image-20210711205024120](/image-20210711205024120.png)



#### 1. 基本介绍

1. 节点流可以从一个特定的数据源读写数据,如FileReader,FileWriter

   ![image-20210711163537911](/image-20210711163537911.png)

   

2. 处理流(也叫包装流)是"连接"在已存在的流(节点流或处理流)之上,为程序提供更为强大读写功能,如BufferedReader,BufferedWreter

   ![image-20210711163726284](/image-20210711163726284.png)

#### 2. 节点流和处理流的区别和联系

1. 节点流是底层流/低级流,直接跟数据源相接;
2. 处理流(包装流)包装节点流,既可以消除不同节点流的实现差异,也可以提供更方便的方法来完成输入输出;
3. 处理流对节点流进行包装,使用了修饰器设计模式,不会直接与数据源相连;

#### 3. 处理流的功能主要提现在以下两个方面:

1. 性能的提高: 主要以增加缓冲的方式来提高输入输出的效率;
2. 操作的编写: 处理流可以提供一系列便捷的方法来一次输入输出大批量的数据,使用更加灵活方便;

#### 4. 处理流-BufferedReader和BufferedWriter

1. BufferedReader和BufferedWriter属于字符流,是按照字符来读取数据的;
2. 关闭时,只需要关闭外层流即可;

#### 5. 处理流-BufferedInputStream和BufferedOutputStream

1. BufferedInputStream是字节流,在创建BufferedInputStream时,会创建一个内部缓冲区数组;

   ![image-20210711230300043](/image-20210711230300043.png)

   

![image-20210711230331714](/image-20210711230331714.png)

#### 6. 对象流-ObectInputStream和ObjectOutputStream

![image-20210712161641079](/image-20210712161641079.png)

**序列化和反序列化**

1. 序列化就是在保存数据时,保存数据的值和数据类型
2. 反序列化就是在恢复数据时,恢复数据的值和数据类型
3. 需要让某个对象支持序列化机制,则必须让其类是可序列化的,为了让某个类是可序列化的,该类必须实现如下两个接口之一;
   1. Serializable //这是标记接口,没有方法
   2. Externalizable //该接口有方法实现,所以一般使用Serializable

##### 1. 基本介绍

![image-20210712162251276](/image-20210712162251276.png)

1. 功能: 提供了对基本类型或对象类型的序列化和反序列化的方法
2. ObjectOutputStream提供序列化功能
3. ObjectInputStream提供反序列化功能

**调用对象的方法,需要向下转型**

##### 2. 注意事项和细节说明

1. 读写顺序要一致
2. 要求序列化或反序列化对象,需要实现Serializable
3. 序列化的类中建议添加SerialVersionUID,为了提高版本的兼容性,private static final long serialVersionUID=1L;
4. 序列化对象时,默认将里面所有属性都进行序列化,但除了static或transient修饰的成员
5. 序列化对象时,要求里面属性的类型也需要实现序列化接口
6. 序列化具备可继承性,也就是如果某类已经实现了序列化,则他的所有子类也已经 默认实现了序列化

#### 7. 标准输入输出流

##### 1. 介绍

![image-20210712170338174](/image-20210712170338174.png)

##### 2. System.in

* 编译类型 InputStream
* 运行类型 BufferedInputStream
* 标准输入 :键盘

##### 3. System.out

* 编译类型 ,运行类型 PrintStream 

#### 8. 转换流-InputStreamReader和OutputStreamWriter

##### 1. 介绍

1. InputStreamReader: Reader的子类,可以将InputStream(字节流)包装成(转换成)Reader(字符流)
2. OutputStreamWriter: Writer的子类,实现将OutputStream(字节流)包装成Writer(字符流)
3. 当处理纯文本数据时,如果使用字符流效率更高,并且可以有效解决中文问题,所以讲义将字节转换成字符流
4. 可以在使用时指定编码格式(比如utf-8,gbk,gb2312)

#### 9. 打印流-PrintStream和PrintWriter

## 3. Properties类

![image-20210712220156466](/image-20210712220156466.png)

### 1. 基本介绍

1. 专门用于读写配置文件的集合类

2. 配置文件的格式: 键=值 

   注意: 键值对不需要有空格,值不需要用引号,默认类型是String

3. Properties的常见方法

   * load: 加载配置文件的键值对到Properties对象
   * list: 将数据显示到指定设备
   * getProperty(key): 根据键获取值
   * getProperty(key,value): 设置键值对到Properties对象
   * store: 将Properties中的键值对存储到配置文件,在idea中,保存信息到配置文件,如果含有中文,会存储为unicode码


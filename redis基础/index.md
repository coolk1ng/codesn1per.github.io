# Redis基础


### 1. NoSQL数据库概述

NoSQL泛指非关系型数据库.NoSQL不依赖业务逻辑方式存储,而已简单的key_value模式存储.因此大大增加了数据库的拓展能力;

* 不遵循SQL标准
* 不支持ACID
* 远超于SQL的性能

### 2. NoSQL使用场景

* 对数据高并发的读写
* 海量数据的读写
* 对数据高可拓展性的

### 3. NoSQL不适用场景

* 需要事务支持
* 基于sql的结构查询存储,处理复杂的关系,需要即席查询
* 用不着sql和用了sql也不行的情况,考虑用NoSQL

### 4. 常见NoSQL数据库

* Memcache
* Redis
* MongoDB

## 2. Redis概述安装

* Redis是一个开源的key-value存储系统

* 和Memcached类似,他支持存储的value类型相对更对,包括string,list,set,zset(sorted set--有序集合)和hash

* 这些数据类型都支持push/pop add/remove及取交集并集和差集及更丰富的操作,而且这些操作都是原子性的;

* 在此基础上,Redis支持各种不同方式的排序

* 与memcached一样,为了保证效率,数据都是缓存在内存中.

* 区别的是Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的几率文件;

* 并且在此基础上实现了master-slave(主从)同步;

  #### 1. redis安装

  ##### 1. 安装目录: /usr/local/bin

  查看默认安装目录:

  * redis-benchmark: 性能检测工具,可以在自己本子运行,看看自己本子性能如何
  * redis-check-aof: 修复有问题的AOF文件,rdb和aof后面讲 
  * redis-check-rdb: 修复有问题的dump.rdb文件
  * redis-sentinel: Redis集群使用
  * redis-server: Redis服务器启动命令
  * redis-cli: 客户端,操作入口

  ##### 2. 后台启动(前台启动不推荐)

  * 备份redis.conf 文件 sudo cp redis.conf  /etc/redis.conf ,将redis.conf里面的daemonize no改成yes 
  * redis-server /etc/redis.conf
  * 客户端访问:redis-cli

  #### 2. Redis 相关知识介绍

  * 默认16个数据库,类似数组下标从0开始,初始默认使用0号库
  * 使用命令 select index 来切换数据库 ,例如select 7
  * 同一密码管理,所有库相同密码
  * dbsize: 查看当前数据库的key数量
  * flushdb: 清空当前库
  * flushall: 通杀全部库

  **Redis是单线程+多路IO复用技术.与Mencache三点不同: 支持多数据类型,支持持久化,单线程+多路IO复用**

## 3. 常用5大数据类型

### 1. Redis键(key) 

* keys * : 查看当前库所有key
* exists key: 判断某个key是否存在
* type key: 查看你的key是什么类型
* del key: 删除指定key数据
* unlink key: 根据value选择非阻塞删除 (仅将keys从keyspace元数据中删除,真正的删除会在后续异步操作)
* expire key 10 : 10秒钟: 为给定的key设置过期时间
* ttl key: 查看还有多少, -1表示永不过期,-2表示过期

### 2. Redis字符串

#### 1. 简介

* String是Redis最基本的类型,可以理解成与Memcacahed一样的类型,一个key对应一个value
* String类型是二进制安全的,以为这Redis的String可以包含任何数据,比如jp图片或者序列化的对象
* String类型是Redis最基本的数据类型,一个Redis中字符串value最多可以是512M

#### 2. 常用命令

* set key value : 添加键值对
* get key: 查询对应键值
* append key value: 将给定的value追加到原值的末尾
* strlen key: 戳去值的长度
* setnx key value: 只有在key不存在是,设置key的值
* incr key: 将key中存储的数字增1,只能对数字值操作,如果为空,新值值为1
* decr key: 将key中存储的数字减1,只能对数字值操作,如果为空,新值值为-1
* incrby/decrby key 步长 : 将key中存储的数字值增减.自定义步长
* mset key1 value1 key2 value2: 设置多个键值对
* mget key1 key2 key3: 查询多个对应键值
* getrange key 起始位置 结束位置: 获得值的范围
* setrange key 起始位置 value: 用value覆写key所存储的字符串值,从起始位置开始
* setex key 过期时间 value: 设置键值的同时,设置过期时间,单位秒
* getset key value: 以新换旧,设置了新值同时获得旧值

**原子性**:

* 所谓原子操作是指不会被线程调度机制打断的操作;
* 这种操作一旦开始,就一直运行到结束,中间不会有任何context switch(切换到另一个线程)
* 在单线程中,能够在单挑指令中完成的操作都可以认为是原子操作,因为终端只发生在指令之间
* 在多线程中,不能被其他进程打断的操作就叫原子操作,Redis单命令的原子性主要得益于Redis的单线程

#### 3. 数据结构

String的数据结构为简单动态字符串,是可以修改的字符串,颞部结构实现上类似java的ArrayList,.采用预分配冗余空间的 方式来减少内存的频繁分配;

![image-20210503015610691](../../static/image-20210503015610691.png)

如图所示,内部为当前字符串实际分配的空间capacity一般要高于时间字符串长度

len.当字符串长度小于1M时,扩容都是加倍现有空间,如果超过1m,扩容时一次智慧多扩1M的空间,需要注意的是字符串崔大长度为512M

### 2. Redis列表(List)

#### 1. 简介(单列多值)

* Redis列表是简单的字符串列表,按照插入顺序排序,你可以添加一个元素到列表的头部或者尾部

* 他的底层实际是一个双向链表,对两端的操作性能很高,通过索引下标的操作中间的就节点性能较差

  ![image-20210503020332882](/image-20210503020332882.png)

  

**问题: 列表插入出错**

![image-20210504230916970](/image-20210504230916970.png)

强制停止redis快照导致，redis运行用户没有权限写rdb文件或者磁盘空间满了

#### 2. 常用命令

* lpush/rpush key value1 value2.. : 从左边/右边插入一个或多个值
* lpop/rpop key : 从左边/右边吐出一个值.值在键在,值光键亡
* rpoplpush key1 key2 : 从key1列表右边吐出一个值,插入到key2列表的左边
* lrange key start stop: 参照索引下标获得元素(从左到右,0 -1表示所有)
* lindex key index:  按照索引下标获得元素(从左到右)
* llen key: 获得列表长度
* linsert key before/after value (new value): 在value的后面插入newvalue
* lrem key n value: 从左边删除n个value(从左到右)
* lset key index value: 将列表key下标为index的值替换成value

#### 3. 数据结构

Lisst的数据结构为快速链表qucikList

1. 首先在列表元素较少的情况下会使用一块连续存储的内存存储,这个结构是ziplist,也就是压缩列表;
2. 他将所有的元素紧挨着一起存储,分配的是一块连续的内存,当数据量比较多的时候才会改成quicklist;
3. 因为普通的链表需要的附加指针空间太大,会比较浪费空间;比如这个列表里村的只是int类型的数据,结构上还需要两个额外的指针prev和next;

![image-20210505020146359](/image-20210505020146359.png)

### 3. Redis集合(Set)

#### 1. 简介

1. Redis Set对外提供的功能与list类似是一个列表的功能,特殊之处在于set是可以自动排重的,当你需要存储一个列表数据,又不希望出现重复数据时,set是一个很好的选择;
2. set提供了判断某个成员是否在一个set集合的重要接口,这个也是list没有的;
3. Redis的Set是string类型的无序集合,底层是一个value为null的hash表,所以添加,删除,查找的复杂度都是O(1);

#### 2. 常用命令

* sadd key value1 value2: 将一个或多个member元素加入到集合key中,已经存在的member元素将被忽略
* smembers key: 取出该集合的所有值
* sismember key value : 判断集合key是否为含有该value值,有1,没有0
* scard key : 返回该 集合的元素个数
* srem key value1 value2: 删除集合中的某个元素
* spop key: 随机从该集合中吐出一个值
* srandmember key n: 随机从该集合中取出n个值,不会从集合中删除
* smove <source> <destination> value : 把集合中一个值从一个集合移动到另一个集合
* sinter key1 key2: 返回两个集合的交集元素
* sunion key1 key2: 返回两个集合的并集元素
* sdiff key1 key2 : 返回两个集合的差集元素(key1中的,不包含key2中的)

#### 3. 数据结构

1. Set数据结构是dict字典,字典是用哈希表实现的;
2. Java中的HashSet的每部实现使用的是HashMap,只不过所有的value都指向同一个对象
3. Redis的set结构也是一样,他的内部也使用hash结构,所有的value都直线沟通一个内部值;

![image-20210505022613120](/image-20210505022613120.png)

#### 2. 常用命令

* hset key field value给key:集合中的field键复制value
* hget key field : 从key集合field取出value
* hmset key1 field1 value1 field2 value2: 批量设置hash的值
* hexists key1 fidld: 查看哈希表key中,给定域field是否存在
* hkeys key: 列出该hash集合的所有field
* hvals key: 列出该hash集合的所有value
* hincrby key fidld increment: 为哈希表key中的域field的值加上增量
* hsetnx key field value : 将哈希表key中的域 field的值设置为value(当且仅当field不存在)

### 4. Redis有序集合Zset(sorted set)

#### 1. 简介

1. Redis有序集合zset与普通集合set非常相似,是一个没有重复元素的字符串集合.
2. 不同之处是有序集合的每个成员都关联一个评分score,这个评分被用来按照从最低分到最高分的方式排序集合中的成员.集合的成员是唯一的,但是评分可以是重复的
3. 因为元素是有序的,所有你也可以很快的根据评分或者次序来获取一个范围的元素
4. 访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的只能列表

#### 2. 常用命令

* zadd key score1 value1 score2 value2 : 将一个或多个member元素及其score值加入到有序集key当中
* zrange key start stop (withscores): 返回有序集key中,下标在start stop之间的元素,逮withscores,可以让分数一起和值返回到结果集
* zrangebyscore key minmax (withscores) (limit offset count): 返回有序集key中,所有score值结余min和max之间(包括等于min或max的成员).有序集成员按score值递增次序排列
* zrevrangebyscore key maxmin (withscores ) (limit offset count):同上,改为从大到小排列

#### 3. 数据结构

SortedSet(zset)是Redis提供的一个非常忒别的数据结构,一方面他等价于java的数据结构map,可以给每个元素value 赋予一个权重score,另一方面他又类似于treeset,内部的元素会按照权重score进行排序,可以得到每个元素的名次,还可以通过score的范围来获取元素的列表

zset底层使用两个数据结构

1. hash,hash的作用就是关联元素value和权重score,保障元素value的唯一性,可以通过元素value找到相应的score值
2. 跳跃表,跳跃表的目的在于给元素value排序,根据score的范围获取元素列表

## 4. Redis 配置文件介绍

### 1. ###Units单位###

配置大小单位,开头定义了一些基本的度量单位,只支持bytes,不支持bit

## 5.Redis的发布和订阅

### 5.1 什么是发布和订阅

Redis发布订阅是一种消息通信模式: 发送者发送消息,订阅者接受消息

Redis客户端可以订阅任意数量的频道

### 5.2 Redis的发布和订阅

![image-20210705185221134](/image-20210705185221134.png)

![image-20210705185230592](/image-20210705185230592.png)

**注: 发布的消息没有持久化,如果在订阅的客户端收不到,只能收到订阅后发布消息**

## 6. Redis新数据类型

### 6.1 简介

Redis提供了Bitmaps这个"数据类型" 可以实现对位的操作:

1. Bitmaps本身不是一种数据类型,实际上它就是字符串,但是他可以对字符串的位进行操作;
2. Bitmaps单独提供一套命令,所以在Redis中使用Bitmaps和使用字符串的方法不太相同.可以把Bitmaps想象成一个以位为单位的数组,数组的每个的那位只能存储0和1,数组的下标在Bitmaps中叫做偏移量;

### 6.2 命令

1. setbit 

   setbit <key> <offset><value>设置Bitmaps中某个偏移量的值(0或1)

   **offset:偏移量从0开始**

2. getbit

   getbit <key><offset> 获取某个偏移量的值

   **获取键的第offset位的值(从0开始计算)**

3. bittop 


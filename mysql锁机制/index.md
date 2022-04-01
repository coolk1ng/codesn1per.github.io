# MySQL锁机制


### 1. 锁的分类

* 共享锁 Share Lock (简称S锁,属于行锁)
* 排它锁 Exclusive Locks (简称X锁,属于行锁)
* 意向共享锁 Intention Shared Locks (简称IS锁,属于表锁)
* 意向排他锁 Intention Exclusive Locks (简称IX锁,属于表锁)
* 自增锁 AUTO-INC Locks

### 2. 共享锁

> 共享锁就是多个事务对同一数据可以共享一把锁,都能访问到数据库,但是只能读不能修改

> **事务A**: select * from student where id = 1 lock in share mode;
>
> **事务B**: select * from student where id = 1 (读数据没有问题)
>
> **事务B**: update student set name = "haha" where id = 1
>
> **注意: 无法修改会卡死,当事务A提交事务之后,会立刻修改成功**

### 3. 排它锁

> 排它锁不能与其他锁并存,如一个事务获取了一个数据行的排它锁,其他事务就不能再获取该行的锁,只有当前获取了排它锁的事务可以对数据进行读取和修改
>
> **delete,update,insert默认是排它锁**

> **事务A:** select * from student where id = 1 for update;
>
> **事务B:** select * from student where id = 1 for update;
>
> ​	  select * from student where id = 1 lock in share mode; 
>
> **注意: 事务B操作的时候会卡死,提交事务立马成功**

### 4. 意向共享锁

> **意向共享锁:** 表示事务准备给数据行加入共享锁,也就是说给一个数据行在加共享锁之前必须先获得该表的IS锁

> **意向排它锁:** 表示事务准备给数据行加入排它锁,也就是说一个数据行加排它锁之前必须取得该表的IX锁

**意向锁是InnoDB数据操作之前自动加的,不需要用户干预**

### 5. 自增锁

> 针对自增列自增张的一个特殊的表级别锁

```sql
SHOW VARIABLES LIKE 'innodb_autoinc_lock_mod;de
```



# 事务

> show VARIABLES like 'autocommit';  查看事务状态



### 事务:

数据库操作的最小工作单元，是作为单个逻辑工作单元执行的一系列操作，事务是一组不可再分割的操作集合（工作逻辑单元）;



#### 典型事务场景(转账)：

```mysql
update user_account set balance = balance - 1000 where userID = 3;

update user_account set balance = balance +1000 where userID = 1;
```

### 如何开启事务

> begin / start transaction 			-- 手工
>
> 
> commit / rollback 				-- 事务提交或回滚
>
> 
> set session autocommit = on/off;	 -- 设定事务是否自动开启  



> SQL : 	set session autocommit = off;   
>
> 
>
> JDBC:  	connection.setAutoCommit（boolean）;
>
> 
>
> spring:	connection.commit()

#### autocommit 默认为ON  

update user set name ='zhangsan' where id =1;

> 自动提交



#### 如何开启事务



```mysql
BEGIN; 
update user set name ='zhangsan' where id =1;
COMMIT;

BEGIN; 
update user set name ='zhangsan' where id =1;
ROLLBACK;


update user set name ='zhangsan' where id =1;
COMMIT;

START TRANSACTION;
update user set name ='zhangsan' where id =1;
ROLLBACK;

SET SESSION autocommit = OFF;
update user set name ='zhangsan' where id =1;
COMMIT;

SET SESSION autocommit = OFF;
update user set name ='zhangsan' where id =1;
ROLLBACK;
```



###  事务的四大特性

> 原子性（Atomicity）
> 最小的工作单元，整个工作单元要么一起提交成功，要么全部失败回滚
>
>
> 一致性（Consistency）
> 事务中操作的数据及状态改变是一致的，即写入资料的结果必须完全符合预设的规则，
> 不会因为出现系统意外等原因导致状态的不一致
>
>
> 隔离性（Isolation）
> 一个事务所操作的数据在提交之前，对其他事务的可见性设定（一般设定为不可见）
>
>
> 持久性（Durability）
> 事务所做的修改就会永久保存，不会因为系统意外导致数据的丢失
>
> 



### 事务带来的问题



> 脏读   不可重读的  幻读



####  脏读

![](assert/cangdu.jpg)



#### 不可重读的

![](assert/bukechongfudu.jpg)



#### 幻读



![](assert/huandu.jpg)



### 隔离级别

>  SQL92 ANSI/ISO标准：http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt


> 隔离级别的实现: 锁  MVCC

#### Read Uncommitted（未提交读） --未解决并发问题

事务未提交对其他事务也是可见的，脏读（dirty read）



#### Read Committed（提交读） --解决脏读问题

一个事务开始之后，只能看到自己提交的事务所做的修改，不可重复读（nonrepeatable
read）



#### Repeatable Read (可重复读) --解决不可重复读问题

在同一个事务中多次读取同样的数据结果是一样的，这种隔离级别未定义解决幻读的问题



#### Serializable（串行化） --解决所有问题

最高的隔离级别，通过强制事务的串行执行



#### innodb引擎对隔离级别的支持程度



![](assert/innodb_zhichi.jpg)

> innodb的默认隔离级别是可重复读，但是不存在幻读的情况





# 锁

> 锁是用于管理不同事务对共享资源的并发访问


> 支持行锁与表锁

#### 表锁与行锁的区别

> 粒度  效率   概率 性能

锁定粒度：表锁 > 行锁


加锁效率：表锁 > 行锁


冲突概率：表锁 > 行锁


并发性能：表锁 < 行锁



#### innodb锁的类型

> 共享锁（行锁）：Shared Locks
>
> 排它锁（行锁）：Exclusive Locks
>
> 意向锁共享锁（表锁）：Intention Shared Locks
>
> 意向锁排它锁（表锁）：Intention Exclusive Locks
>
> 自增锁：AUTO-INC Locks
>
> 

#### innodb行锁的算法

> 记录锁 Record Locks


> 
> 间隙锁 Gap Locks


> 
> 临键锁 Next-key Locks



##  共享锁

> 又称为读锁，简称S锁，顾名思义，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改;



### 加锁的方式



```mysql
-- 共享锁加锁
BEGIN;
select * from users WHERE id=1 LOCK IN SHARE MODE;
rollback; 
commit;

-- 其他事务执行
select * from users where id =1;  -- 查询成功

update users set age=19 where id =1; -- 插入失败

```



## 排他锁



> 又称为写锁，简称X锁，排他锁不能与其他锁并存，如一个事务获取了一个数据行的排他
> 锁，其他事务就不能再获取该行的锁（共享锁、排他锁），只有该获取了排他锁的事务是可以对
> 数据行进行读取和修改，（其他事务要读取数据可来自于快照）



```mysql
delete / update / insert 默认加上X锁
SELECT * FROM table_name WHERE ... FOR UPDATE
commit
rollback


set session autocommit = OFF;
update users set age = 23 where id =1;
select * from users where id =1;
update users set age = 26 where id =1;
commit;
ROLLBACK;

-- 手动获取排它锁
set session autocommit = ON;
begin
select * from users where id =1 for update;
commit;

-- 其他事务执行
select * from users where id =1 lock in share mode;
select * from users where id =1 for update;
select * from users where id =1;
```



## 行锁到底锁了什么东西



> InnoDB的行锁是通过给索引上的索引项加锁来实现的



> 只有通过索引条件进行数据检索，InnoDB才使用行级锁，否则，InnoDB将使用表锁（锁住索引的所有记录）















#  MVCC




# mysql优化配置

#### 参数的作用域

##### 全局参数

> set global autocommit = ON/OFF;



##### 会话参数

> ( 会话参数不单独设置则会采用全局参数)
>
> set session autocommit = ON/OFF;

##### 注意

> - 全局参数的设定对于已经存在的会话无法生效
> - 会话参数的设定随着会话的销毁而失效
> - 全局类的统一配置建议配置在默认配置文件中，否则重启服务会导致配置失效

#### 寻找配置文件

mysql --help 

> 寻找配置文件的位置和加载顺序

```mysql
Default options are read from the following files in the given order:

/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
```

```mysql
mysql --help | grep -A 1 'Default options are read from the following
files in the given order'
```

#### 最大连接数配置

>  max_connections
>
> 约束mysql最大连接数值的因素：系统句柄数配置   mysql 句柄数配置

##### 系统句柄数配置

> 查看系统句柄设置：ulimit -a

> /etc/security/limits.conf

##### mysql 句柄数配置
> /usr/lib/systemd/system/mysqld.service
>
> limitONFILE

#### 常用全局配置

```mysql
port = 3306
socket = /tmp/mysql.sock
basedir = /usr/local/mysql
datadir = /data/mysql
pid-file = /data/mysql/mysql.pid
user = mysql
bind-address = 0.0.0.0
max_connections=2000
lower_case_table_names = 0 # 表名区分大小写
server-id = 1   # 主从配置
tmp_table_size=16M  # memory引擎的临时表空间大小
transaction_isolation = REPEATABLE-READ
ready_only=1 # 对数据的操作必须拥有超级用户的权限，主要用到主从复制里面
```

#### mysql内存参数配置

##### connection 内存参数配置

###### sort_buffer_size connection

> 每一个connection 内存参数配置：
>
> sort_buffer_size connection 
>
> 排序缓冲区大小建议256K( 默认值)-> 2M 之内
>
> 当查询语句中有需要文件排序功能时，马上为connection 分配配置的内存大小

###### join_buffer_size connection

> join_buffer_size connection (可能多个)
>
> 关联查询缓冲区大小建议256K( 默认值)-> 1M 之内
>
> 当查询语句中有关联查询时，马上分配配置大小的内存用这个关联查询，所以有可能在一个查询语句中会分配很多个关联查询缓冲区

###### 根据实际场景计算

> 上述配置4000 连接占用内存：4000*(0.256M+0.256M) = 2G



##### 缓冲区的设置

>  Innodb_buffer_pool_size
>
> innodb buffer/cache 的大小（默认128M） ）

- Innodb_buffer_pool

  > 数据缓存
  > 索引缓存
  > 缓冲数据
  > 内部结构

大的缓冲池可以减小多次磁盘I/O 访问相同的表数据以提高性能

###### 参考计算公式：
Innodb_buffer_pool_size =  （总物理内存 -  系统运行所用 - connection  所用）* 90%



#### 其他参数

wait_timeout

> 服务器关闭非交互连接之前等待活动的秒数

innodb_open_files

> 限制Innodb能打开的表的个数

innodb_write_io_threads
innodb_read_io_threads

> innodb使用后台线程处理innodb缓冲区数据页上的读写 I/O(输入输出)请求

innodb_lock_wait_timeout

> InnoDB事务在被回滚之前可以等待一个锁定的超时秒数



https://www.cnblogs.com/wyy123/p/6092976.html



# 数据库表设计

#### 第一范式（ 1NF）
> 字段具有原子性,不可再分。 所有关系型数据库系统都满足第一范式）数据库表中的字
> 段都是单一属性的， 不可再分；

#### 第二范式（ 2NF）

> 要求实体的属性完全依赖于主键。 所谓完全依赖是指不能存在仅依赖主键一部分的属性，
> 如果存在， 那么这个属性和主关键字的这一部分应该分离出来形成一个新的实体， 新实体与原
> 实体之间是一对多的关系。为实现区分通常需要为表加上一个列，以存储各个实例的惟一标识。
> 简而言之， 第二范式就是属性完全依赖主键。

#### 第三范式（ 3NF）

> 满足第三范式（ 3NF） 必须先满足第二范式（ 2NF）。 简而言之， 第三范式（ 3NF）
> 要求一个数据库表中不包含已在其它表中已包含的非主键信息。

#### 总结

1. 每一列只有一个 单一的 值 ，不可再拆分
2. 每一行都 有主键能进行 区分
3.  每一个表都不包含其他表已经包含的非 主键 信息。



#### 带来的问题

- 充分的满足第一范式设计将为表建立太量的列

> 数据从磁盘到缓冲区，缓冲区脏页到磁盘进行持久的过程中，列的数量过多会导致性能下降。过多的列影响转换和持久的性能



- 过分的满足第三范式化造成了太多的表关联

> 表的关联操作将带来额外的内存和性能开销



- 使用innodb 引擎的外键关系进行数据的完整性保证

> 外键表中数据的修改会导致Innodb引擎对外键约束进行检查，就带来了额外的开销
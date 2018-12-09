# mysql-summary

> https://www.cs.usfca.edu/~galles/visualization/Algorithms.html 



> 正确的创建合适的索引是提升数据库查询性能的



## 索引的定义

> 索引是为了加速对表中数据行的检索而创建的一种分散存储的数据结构

## 索引的作用

> 1. 索引能极大的减少 存储引擎需要扫描的数据 量
> 2. 索引可以把随机IO 变成顺序IO
> 3. 索引 可以帮助 我们在进行 分组、 排序等操作时，避免 使用临时 表

##  常见数据结构的对比

### 二叉树

 ![微信截图_20181208034600](image/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20181208034600.png)





### 平衡二叉树

![1544212070(1)](image/1544212070(1).png)

![微信图片编辑_20181208035254](image/微信图片编辑_20181208035254.jpg)



#### 缺点

- 太深了

>  数据处的（高）深度决定着他的IO 操作次数，IO 操作耗时大

- 太小了

1. 每一个磁盘块 （节点/ 页） 保存的数据量太小 了
2. ***没有很好的利用操作磁盘IO 的数据 交换 特性***
3. ***也没有利用好磁盘IO 的预 读能力（空间局部性原理 ），从而带来频繁的IO***

> mysql 每个页为16K ,为了在一页保存更多的数据，需要字段短一点



> (2)每次IO时(加载4K的时候)获得的数据很少    硬盘的4K对齐
>
> (3)



### 多路平衡查找树 （B-tree）

![微信图片编辑_20181208040909](image/微信图片编辑_20181208040909.jpg)



### 加强版多路平衡查找树 （B+tree）

![微信图片编辑_20181208041047](image/微信图片编辑_20181208041047.jpg)



###  B-tree 与 B+tee的对比

- B+ 节点关键字搜索采用闭合区间

- B+ 非叶节点不保存数据相关信息，只保存关键字和子节点的引用

  >  每个磁盘保存的节点数据更多


- B+ 关键字对应的数据保存在叶子节点中


- B+ 叶子节点是顺序排列的，并且相邻节点具有顺序引用的关系

  > 使B+tree具有了排序的功能



## 选择B+tree原因

- B+ 树的磁盘读写能力更 强

  > 每页存放的数据更好

- B+树 树 的排序能力更强

  > B+tree 叶子节点是顺序排序的的

- B+ 树的查询效率更加 稳定

  > 每次查询数据的时间，几乎相同



## MySQL中B+tree的体现实现

> 聚簇索引  非聚簇索引  主索引  副索引  IBD  MYI  MYD  默认以主键为索引



### myisam

###  ![微信图片编辑_20181208042710](image/微信图片编辑_20181208042710.jpg)

 ![微信图片编辑_20181208042745](image/微信图片编辑_20181208042745.jpg)



###  innodb

![微信图片编辑_20181208042937](image/微信图片编辑_20181208042937.jpg)

 ![微信图片编辑_20181208043004](image/微信图片编辑_20181208043004.jpg)



## 覆盖索引



> 如果查询列可通过索引节点中的关键字直接返回，则该索引称之为覆盖索引。



> ***覆盖索引可减少数据库IO，将随机IO变为顺序IO，可提高查询性能***







##  补充知识点



***列的离散性越高越好***





***最左匹配原则***

- ***1 ，经常用的列优先 【 【 最左匹配原则 】***
- ***2 ，选择性（离散度）高的列 优先 【 离散度高原则 】***
- ***3 ，宽度小的列 优先 【 最少空间原则***




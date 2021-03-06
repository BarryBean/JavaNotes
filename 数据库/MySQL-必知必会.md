# 1. 基础知识
## 1.1 三大范式
- 第一范式：每个列都不能再分；
- 第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能依赖主键的一部分；
- 第三范式：在第二范式的基础上，非主键列只依赖于主键，不依赖于其他非主键。

## 1.2 存储过程
答：存储过程是一个预编译的SQL语句，只要创建一次，就可以多次调用。

优点
- 预编译过的，执行效率高；
- 可重复使用；
- 存储过程代码放在数据库里，通过存储过程名可以直接调用。

缺点
- 引用关系对象改变，受影响的存储过程要重新编译；
- 维护麻烦。


## 1.3 触发器
答：触发器是用户定义在关系表上的一类由事件驱动的特殊存储过程。通俗理解就是，一段触发某事件后自动执行的代码。

在insert、update、delete前后都能加入触发器。




## 1.4 数据类型
1. 整数类型。tinyint,smallint,mediumint,int,bigint => 1,2,3,4,8字节。
2. 实数类型。float,double,decimal。
3. 字符串类型。varchar(变长),char(定长),text,blob。
4. 枚举类型。enum用来代替常用字符串类型。
5. 日期和时间类型。timestamp,datetime。

## 1.5 int(10) char(10) varchar(10)区别
- int(10)表示显示的数据长度，还是4字节存储；
- char(10)表示存储定长的10个字符，不够就用空格补齐；
- varchar(10)表示存储变长的10个字符，存储多少个就是多少个。


## 1.6 count(*),count(1)和count(col)
1. 执行效果上
    - count(*)和count(1)相当于统计行数，不忽略列值为null的；
    - count(1)统计col列，忽略null值。
2. 执行效率上
    - count(*)mysql做了优化处理，有主键的情况下，效率更优
    - count(1)当没有主键时使用。
    - count(col)最慢，因为不走索引。


# 2. 存储引擎
答：MySQL是一个关系型数据库，默认端口号是3306。5.5前用MyISAM，5.5后用InnoDB。
## 2.1 InnoDB
答：5.5版本后的默认引擎。默认可重复读的事务隔离级别，主索引是聚簇索引，内部做了诸多优化(插入缓冲区、哈希索引等)，支持热备份。
## 2.2 MyISAM
答：5.5版本前的默认引擎。全文搜索用 match against。
## 2.3 MyISAM和InnoDB区别
答：区别如下：
- 行级锁。MyISAM只支持表级锁，InnoDB支持表级锁和行级锁。
- 事务支持。MyISAM不支持事务，InnoDB提供事务支持和崩溃恢复。
- 外键。MyISAM不支持外键，InnoDB支持外键。
- InnoDB支持多版本并发控制MVVC，可以处理大量数据。
- MyISAM支持全文索引。

总结一下：**InnoDB支持行+表级锁、事务、外键、多版本并发控制**。


# 3. 索引
索引是一种数据结构，提供指向存储在表的指定列中数据值的指针，能加快检索速度，但同时索引也需要空间存储和定期维护。(通俗理解为书本的目录)
## 3.1 类型
- 主键索引：数据列不允许重复，不允许null，一个表主键唯一；
- 唯一索引：数据列不允许重复，允许null，一个表允许多个唯一索引。
- 普通索引：允许null，无唯一性要求。
- 全文索引：搜索时使用。

### 3.1.1 唯一索引影响插入速度，提高查找速度
前提：当使用辅助索引时，先判断插入索引页是否在内存，不在则先放在 Change Buffer 中，然后以一定频率进行合并操作。

1. 插入速度慢
    - 唯一索引的插入无法利用Change Buffer。
    - 因为唯一索引为了保证唯一性，要将数据页全部加载进内存，不用进入缓存。
2. 查找速度快
    - 普通索引在找到满足条件的第一条记录后，还需要继续查找，直到第一条不满足条件的记录出现；
    - 唯一索引在找到满足条件的第一条记录后，直接返回。




## 3.2 InnoDB索引类型
### 3.2.1 B+ Tree索引
答：**InnoDB默认的索引方式**。有序索引，将相邻数据都存在一起，把随机IO变成顺序IO。

#### B树和B+树的区别
- B树，叶子和非叶子节点都存放数据；B+树，只在叶子节点存放数据。
- B+树的叶子节点有一条链相连，能进行顺序索引，而B树的叶子节点各自独立。

####  数据库为什么用B+树
答：总结：
- B+树支持**随机和顺序索引**，B树只支持随机索引；
- B+树查询效率稳定。B+树数据存储在叶子结点中，B树存储在叶子 + 非叶子节点，所以B+树的**查询是从root到叶子节点，不会中断**。
- B+树**内部结点更小**，没有指向关键字的指针，读写消耗小。


#### 为什么MongoDB使用B树
答：MySQL是关系型数据库，更多使用范围查询，所以用B+树；MongoDB是非关系型数据库，数据遍历的操作少，常用单一查询，所以用B树。


### 3.2.2 哈希索引
答：类似哈希表，将数据库字段转换成hash值和行指针放进hash表。以O(1)时间搜索，但无法排序分组，**只能精准查找**。

自适应哈希索引：当某个索引被频繁引用时，在B+树索引上再创建一个哈希索引，提供快速查找。
 。

## 3.3 聚簇索引(主索引)和非聚簇索引(辅助索引)
答：B+ Tree索引分为主索引和辅助索引。
- 聚簇索引/主索引：叶子**结点存储数据**。直接找key就能获得数据。一个表只能包含一个主索引。
- 非聚簇索引/辅助索引：叶子**结点存储主键**的值。需要先取得主键，再用此主键走一遍主索引。(回表，若全部命中索引，也可以不用回表)


## 3.4 索引优化
### 3.4.1 索引列顺序
答：把**选择性最强/使用最频繁的索引放在前面**。索引选择性=不重复的索引值 / 记录总数。
### 3.4.2 覆盖索引
答：覆盖索引是select的数据列只用从索引中就能够取得，不必读取数据行，即查询列要被所建的索引覆盖。
### 3.4.3 前缀索引
答：根据选择性，只**索引**字符串的**最左M个字符**，前提是前缀的标识度高。
### 3.4.4 索引下推
答：**联合索引**时，先对其余其他字段判断，**过滤不满足条件的记录**，减少回表次数和数据数量。
### 3.4.5 最左前缀匹配原则
联合索引时，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)停止，所以要注意联合索引的顺序。

## 3.5 SQL语句

```java
//增加索引
alter table user add index/unique/fulltext user_id(id) 
create index/unique user_id on user(id)

//删除索引
alter table user drop key user_id
```


## 3.6 小知识
### 3.6.1 回表
答：回表就是指普通索引查询，先搜索普通索引树获得主键值，再到主键索引树搜索一次。

### 3.6.2 百万级数据删除
答：百万级以上的数据一定存在索引，CRUD时都会对索引有影响，所以先删除索引，再删无用数据，循环。



# 4. 事务
答：事务就是逻辑上的一组操作，要么都执行，要么都不执行，执行结果必须让数据库从一种一致性状态到另一种一致性状态。
## 4.1 四大特性ACID
答：ACID。
- **原子性**(Atomicity)：事务是最小的执行单位，要么全部执行，要么全部不执行。
- **一致性**(Consistency)：保证数据库状态要一致，多个事务对同一数据读取结果是相同的。
- **隔离性**(Isolation)：多个事务并发，各个事务是不受其他事务干扰的。
- **持久性**(Durability)：一个事务一旦提交，此修改在数据库中是持久保存的。


## 4.2 并发事务的问题
- **脏读**(Dirty read)：事务A读取了事务B修改未提交的数据，B回滚，A的数据不一致。
- **丢失修改**(Lost to modify)：事务A和B同时读取数据，A修改后先提交，B修改后再提交会将A的修改覆盖。
- **不可重复读**(Unrepeatable read)：事务A两次读取数据间，有事务B修改更新，导致两次读取数据不一致。
- **幻读**(Phantom read)：事务A两次读取数据间，有事务B增加了记录，导致两次读取数据记录数不一致。

注：**幻读指结构**上发生变化。**不可重复读指数值**上发生变化。

## 4.3 隔离级别
答：MySQL定义了四个隔离级别。
- **读未提交**(Read-Uncommitted)：最低级别，允许读取尚未提交的数据。会发生脏读、幻读、不可重复读。
- **读已提交**(Read-Committed)：允许读取已经提交的数据。阻止脏读。**oracle默认**
- **可重复读**(Repeatable-Read)：读事务时禁止写事务，写事务时阻止一切。阻止脏读和不可重复读。**InnoDB默认隔离级别**。
- **可串行化**(Serialization)：最高级别。所有事务依次执行，不会互相干扰。阻止三个事务问题。


# 5. 锁机制
锁机制是在并发时，确保数据访问次序的。
## 5.1 按粒度分
答：分为表级锁，行级锁和页级锁。
- **表级锁**：粒度最大的锁。**对整张表加锁**，资源消耗少，加锁快，不会死锁，但锁冲突概率高。
- **行级锁**：粒度最小的锁。**对**当前**操作行加锁**，加锁慢，开销大，会死锁，但大大减少冲突。
- **页级锁**：介于行级锁和表级锁中间的锁。**一次锁定相邻的一组记录**，会死锁，其他均属于中等。

### 5.1.1 常见行级锁
答：有三种。
- Record Lock：单行锁，锁定符合条件的行。
- Gap Lock：间隙锁，锁定一个范围，不含记录本身。防止幻读。
- **Next-key Lock：Record+Gap，锁定一个范围，含记录本身**。

### 5.1.2 表级锁使用场景
答：当事务比较复杂，使用行级锁容易死锁回滚。更新大表的大部分数据，表级锁效率更好。

### 5.1.3 行级锁何时会锁住整张表
答：更新的列没有建立索引，会直接锁住整张表。


## 5.2 按是否可写分
答：可以进一步划分为共享锁和排他锁。
- **共享锁**(Shared Lock)：读锁。锁定的资源能被其他用户读取，但不能修改，直到资源上的S锁全部被释放。
- **排他锁**(Exclusive Lock)：写锁。事务T对数据A加X锁，则只允许T读取修改，直到X锁释放。在更新操作，如insert、update 或 delete时，始终应用排它锁。



## 5.3 死锁
答：MySQL中的死锁是**多个事务使用行级锁对某行数据加锁造成**。
### 5.3.1 解决方案
答：分为两个方面。
业务层面：
- 指定锁的获取资源顺序。(操作系统中的哲学家就餐问题)
- 同一个事务一次锁定尽可能多的资源。
- 事务拆分成小事务。

数据库设置：
- 设置超时时间。InnoDB默认是50s。
- 开启死锁检测。发生死锁时，回归死锁链上一个事务，让其他事务继续执行。

## 5.4 悲观锁和乐观锁
答：悲观锁和乐观锁是数据库管理系统并发控制的手段。
- **悲观锁**：**利用数据库的锁机制**实现，在整个数据处理过程都加锁，**保证排他性**。
- **乐观锁**：**CAS/版本号实现**。假定不会发生冲突，在提交时检查是否违反数据完整性。

### 5.4.1 乐观锁的ABA问题
答：加入**数据版本记录机制**或者使用时间戳。

ABA问题：事务X读取数据A时，事务Y修改成B，又修改回A，事实上数据发生过改变的，存在并发问题，但事务X无法得到数据发生过变化。

### 5.4.2 使用场景
答：多读用乐观，多写用悲观。



# 6. MySQL架构和执行流程
## 6.1 基础架构
答：MySQL主要分为Server层和存储引擎层。
- **Server层**：跨存储引擎的功能都在此实现，如视图、触发器、函数等，有一个binlog日志。
- **存储引擎**：负责数据的存储和读取。常用的是InnoDB，自带redolog模块。

## 6.2 基本组件
答：Server层有五个主要组件。
- 连接器：身份验证和权限设置。
- 查询缓存：执行查询语句前，先查有没有缓存的结果集。(8.0后废弃)
- 分析器：没有命中缓存，则进入分析器进行词法和语法分析。
- 优化器：按照MySQL认为最优的方案执行。
- 执行器：用户有权限，则调用存储引擎，执行语句。

## 6.3 执行流程
### 6.3.1 查询语句
权限校验---->查询缓存---->分析器---->优化器---->权限校验---->执行器---->引擎
### 6.3.2 更新语句
分析器---->权限校验---->执行器---->引擎---->redo log prepare---->binlog---->redo log commit

以修改张三的年龄为例：
- 先查询到张三这条数据，如果有缓存，用缓存。
- 拿到查询的语句，把 age 改为 19。
- 调用引擎 API 接口，写入这一行数据，InnoDB 引擎把数据保存在内存中，同时记录 redo log，此时 redo log 进入 prepare 状态。
- 通知执行器，执行完成，可以提交。
- 执行器收到通知后记录 binlog，调用引擎接口，提交 redo log 为提交状态。
- 更新完成。



## 6.4 日志模块
答：MySQL主要有redolog和binlog。
- **redolog(重做日志)**：InnoDB特有，物理日志。**记录哪个数据页做了修改**。
- **binlog(归档日志)**：Server层自带，逻辑日志，**记录本次修改的SQL语句**。通常使用row二进制格式，保证数据记录的准确性。

### 6.4.1 两阶段提交
答：保证数据一致性。

**redolog prepare引擎数据保存 -> binlog执行器收到信号 -> redolog commit执行器调用**

异常时，MySQL先判断redolog是否完整，完整则直接提交。redolog只是prepare状态，查看binlog是否完整，完整就继续提交redolog，否则回滚事务。

### 6.4.2 MySQL为什么会突然慢一下
答：更新数据库时，先写日志，等合适时间再更新磁盘。当redolog写满，需要**flush脏页**，将数据写入磁盘，这是会使执行速度慢一下。




# 7. 常用SQL
## 7.1 分类
- DDL：create、drop、alter；
- DQL：select；
- DML：insert、update、delete；
- DCL：grant、revoke、commit、rollback。

## 7.2 关联查询
### 7.2.1 内连接
- 等值连接：on A.id=B.id
- 不等值连接：on A.id>B.id
- 自连接：select * from A t1 inner join A t2 on t1.id=t2.pid


### 7.2.2 外连接
- 左外连接：left join，以左表为主，先查出左表，按照on的关联条件匹配右边，没匹配到就填充null
- 右外连接：right join，以右表为主，先查出右表，按照on的关联条件匹配左边，没匹配到就填充null

### 7.2.3 填充查询
把多个结果集集中在一起，以union前的结果为主。union all不合并重复的记录。

select * from A union select * from B

### 7.2.4 全连接
MySQL不支持全连接，用left join + union + right join代替。


## 7.3 子查询
把一条SQL的查询结果作为另一条SQL的条件或者结果。
- 子查询是单行单列，结果集是一个值
- 子查询是多行单列，结果集类似数组
- 子查询是多行多列，结果集类似表，用在select中。
```java
select * from employee where salary=(select max(salary) from employee);

select * from employee where salary in (select max(salary) from employee);

select * from dept d,(select * from employee where joindate > '2019-12-1') e where d.id=e.did;
```

## 7.4 分页

select * from table limit 5,10

两个参数，偏移量和返回行的数目。



# 8. 优化
## 8.1 结构优化
1. 拆表。把使用频率低的列单独分离成表。
2. 增加中间表。把经常联合查询数据插入中间表。
3. 增加冗余字段。

## 8.2 SQL优化

### 8.2.1 超大分页
答：两个思路。
- 减少load的数据，如先定位id再关联；
- 增加缓存，如redis。

### 8.2.2 慢查询日志
答：当执行时间超过临界值时，将SQL计入日志。慢查询优化主要从三方面入手。
- load了额外数据；
- 没有命中索引；
- 数据量太大，需要切分。

### 8.2.3 语句优化
参考 https://www.cnblogs.com/huchong/p/10219318.html 作者：听风。


## 8.3 大表优化
答：单表记录数过大时，数据库性能下降明显，需要进行一系列的优化。
### 8.3.1 限定数据范围
最简单的手段，限制数据范围条件来查询数据。
### 8.3.2 读写分离
**主服务器处理写操作**或实时性高的读操作，**从服务器处理读操作**。
- 实现方式：增设代理服务器，决定将应用传来的请求转发到哪个服务器。
- 原因：极大减少了锁的争用；用冗余换可用性；从服务器可用MyISAM节约资源。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200227140539433.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0NzYxMDEy,size_16,color_FFFFFF,t_70)

### 8.3.3 缓存
对基本不更新的数据使用应用级别的缓存。

### 8.3.4 水平拆分
水平拆分将一张表A拆分成多个相同结构的表a1,a2,a3存储，每个子表只占一部分数据。
又细分为库内分表和分库分表。
- **库内分表**：子表仍在一个数据库实例中，仍要竞争同一个物理机资源。
- **分库分表**：子表分散在不同数据库中。

优点：业务改造简单；解决了单表数据量过大的问题。
缺点：跨库的一致性难保证；join关联性能差；扩容和维护难度过大。


### 8.3.5 垂直拆分
又细分为垂直分库和垂直分表。
- **垂直分库**：基于业务划分数据库，让每个业务都有独立数据库。
- **垂直分表**：基于数据表的列切分，把一张列1-7的表拆成列1-4和列1 5-7的表。

优点：业务解耦；高并发下提升了性能。
缺点：增加了业务复杂度；**主键冗余**；数据量过大仍未解决，需要配合水平拆分。


## 8.4 拆分后的问题
1. 事务一致性
两阶段提交性能较差。通常**追求最终一致性**，出现问题进行事务补偿。
2. 分页和排序
字段排序，需要先在每个节点内排序，再将结果汇总再次排序，最后返回给用户。
3. 全局唯一主键
拆分后属于分布式应用，需要使用**分布式ID用作全局唯一主键**。一般用雪花模型。

注：所以推荐使用成熟的中间件，如sharding-jdbc，Atlas，Cobar

## 8.5 分布式ID
1. UUID：生成简单，但不推荐！长度过长，不适合实际的业务需求。
```java
public static void main(String[] args) { 
       String uuid = UUID.randomUUID().toString().replaceAll("-","");
       System.out.println(uuid);
 }
```
2. 数据库自增ID：用一个单独数据库实例生成ID，需要ID时，向表中插入记录并返回ID。
    - MySQL本身性能是瓶颈，而且存在宕机风险。
3. 数据库集群：对每个MySQL实例的自增ID设置起始值和自增步长，保证不冲突。
    - 后续扩容问题大，无法满足高并发。
4. 数据库号段模式：每次从数据库获取一个号段范围，具体业务将号段生成对应自增ID加载到内存。一批号段ID用完，再向数据库申请新的。
    - 用版本号乐观锁更新；不频繁访问数据库。
5. 雪花算法：twitter公司开源的算法。
    - Snowflake生成Long型64位ID。
    - 组成结构：正数位（1bit，默认0）+ 时间戳（41bit，存储 当前-固定开始 的差值）+ 机器ID（5bit）+ 数据中心（5bit）+ 自增值（12bit，1ms支持同一节点生成4096个ID）




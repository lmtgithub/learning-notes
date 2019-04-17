> 上文已经介绍了数据库隔离级别，本文对隔离级别进行验证。[了解数据库隔离级别](MySQL数据库隔离级别.md)
# 准备工作
首先创建一个表，用以接下来的验证。
```sql
-- 建表语句
create table verify(
    `id` int(5) NOT NULL AUTO_INCREMENT,
    `num` int(5) NOT NULL,
    PRIMARY KEY(`id`)
);
-- 初始化
insert into verify(num) values(10);
```
由于mysql默认隔离级别是可重复的，所以在每次验证之前需要修改数据库的隔离级别
```sql
-- 查看当前数据库隔离级别
select @@tx_isolation;

-- 设置数据库隔离级别。参数可以是'read-uncommitted'、'read-committed'、’repeatable-read‘、’serializable‘
set tx_isolation='read-uncommitted'
```
# 脏读
> set tx_isolation='read-uncommitted'  

|操作顺序|事务A|事务B|备注|
|-|-|-|-|
|第一步|start trasaction;||事务A开始事务|
|第二步|update verify set num = 20 where id = 1;||此时数据库num值为20，但是事务A并未提交|
|第三步||start transaction;|事务B开始事务|
|第四步||select * from verify where id = 1;|事务B查询num，值为20|
|第五步|rollback;||事务A回滚|
|第六步||select * from verify where id = 1;|事务B查询num，值为10|
|第七步||rollback;|事务B回滚|

从上面的第四步可以看出事务B读取的num是事务A未提交额数据，所以属于脏读。
# 不可重复读
> set tx_isolation='read-committed'

|操作顺序|事务A|事务B|备注|
|-|-|-|-|
|第一步|start trasaction;||事务A开始事务|
|第二步|update verify set num = 20 where id = 1;||此时数据库num值为20，但是事务A并未提交|
|第三步||start transaction;|事务B开始事务|
|第四步||select * from verify where id = 1;|事务B查询num，值为10|
|第五步|commit;||事务A提交|
|第六步||select * from verify where id = 1;|事务B查询num，值为20|
|第七步||commit;|事务B提交|

从上面的第四步和第七步可以发现『读提交』解决了脏读的问题。
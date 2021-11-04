---
title: mysql锁
date: 2021-11-02 11:25:21
tags:
categories: 
- MySQL
- 锁技术
---

## 基本概念

MySQL锁的大分类为两类: 悲观锁与乐观锁。两种锁实现的`位置`不同，悲观锁由数据库系统实现，乐观锁由用户自己实现。

### 使用场景

普遍的使用场景是: select&update, select出数据然后再此基础上进行更新操作，此期间需要使用锁来保证同一个事务的ACID (https://zh.wikipedia.org/wiki/ACID)

### 悲观锁

>  据从事证券行业的朋友说，证券银行等行业都是使用的悲观锁

悲观锁又分为两种: 共享锁与排他锁，顾名思义，这两种锁的严格级别(颗粒度)也是从低到高。

#### 共享锁: lock in share mode

其他的session可以读取数据，但是不能修改，如果其他事务修改了该行数据，会重新读取最新的数据

#### 排他锁: lock for update

相对于共享锁更加严格，其他session不能读也不能修改。

### 乐观锁

乐观锁一般由使用方来实现，实现方式是为数据库表增加一个version字段(自动更新), select时把version字段也带上，然后在update的时候再次去检查数据库中的version与事务中select出来的version是否一致，如果不一致就获取最新的version重试。

> 阿里java开发规范里有一则规范是乐观锁的重试次数不低于3次，可以作为参考

#### 对比优势与劣势

TODO

### 示例

示例数据

```sql
create table `goods` (
    id int unsigned auto_increment comment '自增主键',
    name varchar(35) not null default '' comment '商品名字',
    stock int unsigned not null default 0 comment '商品库存',
    create_time timestamp not null default current_timestamp comment '创建时间',
    update_time timestamp not null default current_timestamp on update current_timestamp comment '更新时间',
    primary key (`id`),
    unique key `uk__name` (`name`)
) engine=innodb default charset=utf8mb4 comment='商品表';

insert into `goods` (`name`, `stock`) values ('phone', 10000);
insert into `goods` (`name`, `stock`) values ('book', 1000);
```

关掉事务自动提交

```sql
set sesssion autocommit=0;
```

#### 悲观锁

事务A: 

```sql
start transaction;
select name as name, @a := stock as stock from goods where name = 'phone' for update;
update goods set stock=@a-1 where name = 'phone';
commit;
```

事务B: 

```sql
start transaction;
select name as name, @a:=stock as stock from goods where name = 'phone' for update;  # 这里事务B开始阻塞，直到事务A提交
update goods set stock=@a-1 where name = 'phone';
commit;
```

##### 注意点

`where`条件必须加索引，加上索引是行锁，否则就是锁表。索引类型可以是主键索引，唯一索引，普通索引。


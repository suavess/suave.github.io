---
title: MySQL中的触发器
date: 2022-02-24 19:36:53
tags: 
    - MySQL
    - 触发器
category: 
    - MySQL
---

## 1. 介绍

> 触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端**确保数据的完整性** , **日志记录** , **数据校验**等操作 。

使用别名 OLD 和 NEW 来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发。（Oracle既有行级触发器，又有语句级触发器）

| 触发器类型      | NEW 和 OLD的使用                                        |
| --------------- | ------------------------------------------------------- |
| INSERT 型触发器 | NEW 表示将要或者已经新增的数据                          |
| UPDATE 型触发器 | OLD 表示修改之前的数据 , NEW 表示将要或已经修改后的数据 |
| DELETE 型触发器 | OLD 表示将要或者已经删除的数据                          |

## 2. 创建触发器

### 2.1 语法结构 :

```sql
CREATE TRIGGER trigger_name trigger_time trigger_event ON tb_name FOR EACH ROW trigger_stmt
```

tb_name ：需要建立触发器的表名(只能是永久表，不能对临时表创建触发器)

trigger_name ：触发器名称，自行指定

trigger_time：触发时机，取值BEFORE、AFTER

trigger_event ：触发事件，INSERT、UPDATE、DELETE

trigger_stmt ： 触发程序体，可以是一条SQL语句或是BEGIN和END包含的多条语句

### 2.2 示例

#### 2.2.1 需求

```
通过触发器记录 emp 表的数据变更日志 , 包含增加, 修改 , 删除 ;
```

#### 2.2.2 首先创建一张日志表 :

```sql
create table emp_logs(
  id int(11) not null auto_increment,
  operation varchar(20) not null comment '操作类型, insert/update/delete',
  operate_time datetime not null comment '操作时间',
  operate_id int(11) not null comment '操作表的ID',
  operate_params varchar(500) comment '操作参数',
  primary key(`id`)
)engine=innodb default charset=utf8;
```

#### 2.2.3 创建 insert 型触发器，完成插入数据时的日志记录 :

```sql
DELIMITER $

create trigger emp_logs_insert_trigger
after insert 
on emp 
for each row 
begin
  insert into emp_logs (id,operation,operate_time,operate_id,operate_params) values(null,'insert',now(),new.id,concat('插入后(id:',new.id,', name:',new.name,', age:',new.age,', salary:',new.salary,')'));	
end $

DELIMITER ;
```

#### 2.2.4 创建 update 型触发器，完成更新数据时的日志记录 :

```sql
DELIMITER $

create trigger emp_logs_update_trigger
after update 
on emp 
for each row 
begin
  insert into emp_logs (id,operation,operate_time,operate_id,operate_params) values(null,'update',now(),new.id,concat('修改前(id:',old.id,', name:',old.name,', age:',old.age,', salary:',old.salary,') , 修改后(id',new.id, 'name:',new.name,', age:',new.age,', salary:',new.salary,')'));                                                                      
end $

DELIMITER ;
```

#### 2.2.5 创建delete 行的触发器 , 完成删除数据时的日志记录 :

```sql
DELIMITER $

create trigger emp_logs_delete_trigger
after delete 
on emp 
for each row 
begin
  insert into emp_logs (id,operation,operate_time,operate_id,operate_params) values(null,'delete',now(),old.id,concat('删除前(id:',old.id,', name:',old.name,', age:',old.age,', salary:',old.salary,')'));                                                                      
end $

DELIMITER ;
```

#### 2.2.6 测试：

```sql
insert into emp(id,name,age,salary) values(null, '光明左使',30,3500);
insert into emp(id,name,age,salary) values(null, '光明右使',33,3200);

update emp set age = 39 where id = 3;

delete from emp where id = 5;
```

## 3. 删除触发器

语法结构 :

```sql
drop trigger [schema_name.]trigger_name
```

如果没有指定 schema_name，默认为当前数据库 。

### 4. 查看触发器

可以通过执行 SHOW TRIGGERS 命令查看触发器的状态、语法等信息。

语法结构 ：

```sql    
show triggers ;
```

## 5. 总结

- 优点是可以在数据库层面保证数据的完整性，并且可以少写业务逻辑代码，使用方便。例如记录日志的功能，可以节省极大量的逻辑代码

- 缺点也很明显，逻辑在数据库层面，应用层调试困难，出现问题难以定位

- 注意点

  - 如果BEFORE触发器执行失败，SQL无法正确执行。

  - SQL执行失败时，AFTER型触发器不会触发。
  - AFTER类型的触发器执行失败，SQL会回滚。


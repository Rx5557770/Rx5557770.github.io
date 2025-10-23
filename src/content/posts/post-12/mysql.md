
---
title: mysql
published: 2025-10-23
description: ''
image: ''
tags: ['code', 'mysql']
category: 代码日常
draft: false
lang: zh-CN
---
    
## 基本命令
### 库操作

```mysql
# 显示数据库
show databases;
# 创建数据库
create database new_database charset utf8mb4;
# 选择数据库
use new_database;
# 删库
drop database new_database;
```

### 表操作

```mysql
# 显示表
show tables;
# 创建表
create table new_tb (
	id int primary key,
	name varchar(10) not null, # 大小最多十个字符
	age int
);
# 直接删除整个表的数据和结构
drop table new_tb;
# 添加数据
insert into new_tb (id, name, age) values (1, 'jack', 10), (2, 'rose', 12);

# 查询数据
select * from new_tb;
# 查询特定表列
select name, age from new_tb;
# 条件查询
select * from new_tb where id = 2;
# 更新值
update new_tb set age = 26 where name = 'jack';
# 删除特定数据
delete from new_tb where name = 'jack';
# 清空表数据
delete from new_tb;
```

### Pymysql

安装：`pip install pymysql`

```python
from pymysql import Connection

conn = Connection(
	host = "localhost",
	port = 3306,
	user = "root",
	password = "123456",
	autocommit = True
)

# 获取游标
cursor = conn.cursor()
db_name = 'new_db'
table_name = 'new_table'
db_comm = f'create database {db_name} charset utf8mb4;'
table_comm = f"""
create table {table_name} (
	id int primary key,
	name varchar(10) not null,
	age int
);
"""
cursor.execute(db_comm)

# 选择数据库
conn.select_db(db_name)
cursor.execute(table_comm)
conn.close()
```
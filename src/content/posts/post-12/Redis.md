
---
title: Redis
published: 2025-10-23
description: ''
image: ''
tags: ['code', 'redis']
category: 代码日常
draft: false
lang: zh-CN
---
    ## 类型
### String类型

> 最常见普通的类型
```redis
# set/ get
127.0.0.1:6379> set age 18
OK
127.0.0.1:6379> get age
"18"

# set ex 设置值的同时定下有效期（单位秒）
127.0.0.1:6379> set name jack ex 5
OK
127.0.0.1:6379> get name
"jack"
127.0.0.1:6379> get name
(nil)

# 同上
# 127.0.0.1:6379> setex rose 5 18
OK
127.0.0.1:6379> get rose
"18"
127.0.0.1:6379> get rose
(nil)

# setnx 当键不存在则设置，如果存在则设置失败。
127.0.0.1:6379> keys *
(empty array)
127.0.0.1:6379> set age 18
OK
127.0.0.1:6379> setnx age 19
(integer) 0
127.0.0.1:6379> setnx name jack
(integer) 1
127.0.0.1:6379> mget age name
1) "18"
2) "jack"

# incr 执行+1操作
127.0.0.1:6379> incr age
(integer) 19

# decr 执行-1操作
127.0.0.1:6379> decr age
(integer) 18

# append 可以创建/追加内容
127.0.0.1:6379> append name 01
(integer) 2
127.0.0.1:6379> get name
"01"
127.0.0.1:6379> append name 01
(integer) 4
127.0.0.1:6379> get name
"0101"

# mset/mget 批量设置与获取值
127.0.0.1:6379> mset name rose age 16
OK
127.0.0.1:6379> mget name age
1) "rose"
2) "16"
127.0.0.1:6379>
```

### hash类型

> 诸如对象形式的数据可以用hash类型

```redis
# hset/hget 设置字段与获取值
127.0.0.1:6379> hset user1 name jack
(integer) 1
127.0.0.1:6379> hset user1 age 18
(integer) 1
127.0.0.1:6379> hget user1 name
"jack"
127.0.0.1:6379> hget user1 age
"18"

# hmset/hmget 批量设置/获取
127.0.0.1:6379> hmset user2 name rose age 16
OK
127.0.0.1:6379> hmget user2 name age
1) "rose"
2) "16"
```

### 集合类型

> 集合类型里面元素不会重复，集合是无序的。

```redis
# sadd 添加元素
127.0.0.1:6379> sadd class jack rose ikun
(integer) 3
127.0.0.1:6379> smembers class
1) "jack"
2) "rose"
3) "ikun"

# sismember 判断元素是否在集合中
127.0.0.1:6379> sismember class jack
(integer) 1
127.0.0.1:6379> sismember class apple
(integer) 0

# sinter 求两个集合的交集
127.0.0.1:6379> smembers class
1) "jack"
2) "rose"
3) "ikun"
127.0.0.1:6379> sadd class2 rose ikun yoyo
(integer) 3
127.0.0.1:6379> sinter class class2
4) "rose"
5) "ikun"

# sunion 求两个集合的并集
127.0.0.1:6379> sunion class class2
1) "ikun"
2) "rose"
3) "yoyo"
4) "jack"
```

### 有序集合类型

> 有序集合需要在每个元素前定义一个权重

> 这个和列表的命令很像

```redis
# zadd用于添加元素
127.0.0.1:6379> zadd class 20 rose 10 jack 30 ikun
(integer) 3

# zrange 默认按权重从小到大排序
127.0.0.1:6379> zrange class 0 -1
1) "jack"
2) "rose"
3) "ikun"

# zrevrange 按权重从大到小排序
127.0.0.1:6379> zrevrange class 0 -1
1) "ikun"
2) "rose"
3) "jack"

# zrank 获取排名，也就是指定元素的权重值下面还有多少元素。
127.0.0.1:6379> zrank class jack
(integer) 0
127.0.0.1:6379> zrank class rose
(integer) 1
127.0.0.1:6379> zrank class ikun
(integer) 2
127.0.0.1:6379>

# zscore 获取权重值
127.0.0.1:6379> zscore class ikun
"30"
```

### 列表类型

```redis
# lpush代表从左侧插入，即每次都放到最左侧
127.0.0.1:6379> lpush list apple
(integer) 1
127.0.0.1:6379> lpush list banana
(integer) 2
127.0.0.1:6379> lpush list jack
(integer) 3

# lrange 获取整个列表 需要传入下标
127.0.0.1:6379> lrange list 0 -1
1) "jack"
2) "banana"
3) "apple"

# rpush 从右侧插入值
127.0.0.1:6379> rpush list rose
(integer) 4
127.0.0.1:6379> lrange list 0 -1
4) "jack"
5) "banana"
6) "apple"
7) "rose"

# lpop 从最左侧弹出这个元素并从列表删除
127.0.0.1:6379> lpop list
"jack"
127.0.0.1:6379> lrange list 0 -1
8) "banana"
9) "apple"
10) "rose"

# rpop同理... 
```

## 事物

> 事物会在一个块内 
```redis
MULTI
SET key1 value1
INCR key2
EXEC
```


## python+redis

### 安装

`pip install redis`

### 常用配置

```python
import redis  
  
# 基本连接  
r = redis.Redis(host='localhost', port=6379, db=0)  
  
# 测试连接  
try:  
    r.ping()  
    print("成功连接到Redis")  
except redis.ConnectionError:  
    print("无法连接到Redis")  
  
  
"""  
String类型  
"""  
r.set('age', '18')  # 单个设置  
r.mset({'key1': 'value1', 'key2': 'value2'})  # 多个设置  
r.setnx('age', '96') # 如果key不存在则设置，存在则失败  
r.setex('age', 5, 65) # 设置一个有效期的值  
  
r.get('age').decode('utf-8')  # 默认返回bytes类型  
values = r.mget('key1', 'key2')  # 获取多个值  
  
# 自增操作  
r.set('counter', 0)  
r.incr('counter')  # 1  
r.incrby('counter', 5)  # 6  
  
"""  
hash类型  
"""  
# 设置哈希字段  
r.hset('user:1000', 'name', 'Bob')  
r.hset('user:1000', 'email', 'bob@example.com')  
  
# 获取单个字段  
name = r.hget('user:1000', 'name')  
  
# 获取所有字段  
user_data = r.hgetall('user:1000')  # 返回字典  
  
# 设置多个字段  
r.hmset('user:1001', {'name': 'Charlie', 'email': 'charlie@example.com'})  
  
"""  
列表类型  
"""  
# 从左侧添加元素  
r.lpush('tasks', 'task1', 'task2', 'task3')  
  
# 从右侧添加元素  
r.rpush('tasks', 'task4')  
  
# 获取列表范围  
tasks = r.lrange('tasks', 0, -1)  # 获取所有元素  
  
# 弹出元素  
first_task = r.lpop('tasks')  
last_task = r.rpop('tasks')  
  
# 列表长度  
length = r.llen('tasks')  
  
"""  
集合类型  
"""  
# 添加元素  
r.sadd('tags', 'python', 'redis', 'database')  
  
# 获取所有元素  
tags = r.smembers('tags')  
  
# 检查元素是否存在  
has_python = r.sismember('tags', 'python')  
  
# 集合运算  
r.sadd('tags1', 'python', 'redis', 'database')  
r.sadd('tags2', 'python', 'mysql', 'web')  
common_tags = r.sinter('tags1', 'tags2')  # 交集  
all_tags = r.sunion('tags1', 'tags2')    # 并集  
diff_tags = r.sdiff('tags1', 'tags2')    # 差集  
  
"""  
有序集合类型  
"""  
# 添加带分数的成员  
r.zadd('leaderboard', {'player1': 100, 'player2': 85, 'player3': 95})  
  
# 获取排名  
rank = r.zrevrank('leaderboard', 'player1')  # 从高到低排名  
score = r.zscore('leaderboard', 'player1')  
  
# 获取范围  
top_players = r.zrevrange('leaderboard', 0, 2, withscores=True)  
  
# 增加分数  
r.zincrby('leaderboard', 10, 'player2')  
  
"""  
事物  
"""  
# 使用事务 (multi/exec)pipe = r.pipeline()  
pipe.multi()  
pipe.set('tx_key1', 'value1')  
pipe.set('tx_key2', 'value2')  
pipe.execute()
```


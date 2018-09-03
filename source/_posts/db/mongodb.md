---
title: mongodb基本操作
date: 2018-01-27
tags: [数据库, 后端]
categories: 数据库
---

### [官方文档](https://docs.mongodb.com/manual/crud/)

### 一、 新增(insert)
```sh
db.users.insert({ "email": "499871809@qq.com" })
insert into users(`email`) values("499871809@qq.com")
```

### 二、 查询(find)

#### 查询所有
```sh
db.users.find()
select * from users
```

#### 条件查询（where）
```sh
db.users.find({ "email": "499871809@qq.com" })
select * from users where email = "499871809@qq.com"
```

#### 指定键值（where）
```sh
db.users.find({ "email": "499871809@qq.com" }, {email: 1, name: 1})
select email, name from users where email = "499871809@qq.com"
```

#### 并（and）
```sh
db.users.find({ "email": "499871809@qq.com", name: "hap" }, {email: 1, name: 1})
select email, name from users where email = "499871809@qq.com" and name = "hap"
```

#### 或（or）
```sh
db.users.find("$or": [{ "email": "499871809@qq.com" }, {name: "hap"}])
select email, name from users where email = "499871809@qq.com" or name = "hap"
```

#### 符号对比
| mongodb  | mysql | 例子 |
| ----- | -----| -----| 
|  $eq  |  =   | { age: { $eq: 18 } }, age = 18   |
|  $ne  |  !=  | { age: { $ne: 18 } }, age != 18  |
|  $gt  |  >   | { age: { $gt: 18 } }, age > 18   |
|  $gte |  >=  | { age: { $gte: 18 } }, age >= 18 |
|  $lt  |  <   | { age: { $lt: 18 } }, age < 18   |
|  $lte |  <=  | { age: { $lte: 18 } }, age <= 18 |
|  $in  |  in  | { age: { $in: [12, 15, 18] } }, age IN (12, 15, 18) |
|  $nin | not in | { age: { $nin: [12, 15, 18] } }, age NOT IN (12, 15, 18) |

```sh
db.users.find({"age": {$gt: 12, $lt: 18}})
select * from users where age>12 and age<18
```

#### 模糊查询(like)
```sh
db.users.find({"email": /4998/)
select * from users where email like "%4998%"
```

#### 数目(count)
```sh
db.users.count({"email": /4998/)
select count(*) from users where email like "%4998%"
```

#### 排序(sort, order by)
```sh
db.users.find({"email": /4998/).sort({createAt: -1}) #降序
select count(*) from users where email like "%4998%" order by createAt desc

db.users.find({"email": /4998/).sort({createAt: 1}) #升序
select count(*) from users where email like "%4998%" order by createAt asc
```

### 三、 删除(remove)
```sh
db.users.remove({"email": "499871809@qq.com"})
delete from users where email = "499871809@qq.com"
```

### 四、 更新(update)
#### update
mongodb更新命令
```sh
db.collecions.update(query, update[, options] )
```

```sh
query = { "email": "499871809@qq.com" }
update = { age: 24 }
options = { update: true }
db.users.update(query, update, options)

update users set age = 24 where email = "499871809@qq.com"
```

#### save
```sh
query = { "email": "499871809@qq.com" }
user = await db.users.findOne(query).exec()
user.age = 18
user.save()
```

#### 更新特定字段($set)
对于更新对象的某个属性时特别有用

```sh
update users set age = 24 where email = 499871809@qq.com
query = { "email": "499871809@qq.com" }
update = { $set: { age: 24 } }
options = { update: true }
db.users.update(query, update, options)

query = { "email": "499871809@qq.com" }
update = { $set: { ip.address: 'aaaaaa' } }
options = { update: true }
db.users.update(query, update, options)
```

#### 删除特定字段($unset)
```sh
update users set age = 24 where email = 499871809@qq.com
query = { "email": "499871809@qq.com" }
update = { $unset: { age: 1 } }
db.users.update(query, update)
```
#### 自增($inc)

```sh
{$inc: 1} 加一
{$inc: -1} 减一

update users set age = age+1 where email = 499871809@qq.com
query = { "email": "499871809@qq.com" }
update = { $inc: { age: 1 } }
db.users.update(query, update)
```

#### 数组操作
##### 一次追加多个元素($pushAll)

```sh
query = { "email": "499871809@qq.com" }
update = { $pushAll: { score: [0, 9, 0] } }
db.users.update(query, update)
```

##### 追加不重复元素($addToSet)
```sh
query = { "email": "499871809@qq.com" }
update = { $addToSet: { score: [1] } }
db.users.update(query, update)
```
##### 删除元素($pop)
```sh
query = { "email": "499871809@qq.com" }
update = { $addToSet: { score: 1 } } //删除最后一个元素
update = { $addToSet: { score: -1 } } //删除第一个元素
db.users.update(query, update)
```

#####删除多个特定元素($popAll)
```sh
query = { "email": "499871809@qq.com" }
update = { $pullAll: { score: [0, 9] } }
db.users.update(query, update)
```

### 五、 其他

####数组查询
    
只要匹配其中一个就可以

```sh
languages: ['javasctipt', 'c++']
db.users.find({ languages: 'javasctipt' }) 可以匹配到
```

多个条件都符合($all)

```sh
db.users.find({ languages: $all: ['javasctipt', 'c++'] }) 可以匹配到
```

限制长度($size)

```sh
db.users.find({ languages: $size: 2}) 可以匹配到
```

返回特定偏移量($slice)
```sh
db.users.find({ languages: $slice: 1 }) //第一个
db.users.find({ languages: $slice: -1 }) //倒数第一个
db.users.find({ languages: $slice: [1, 1] }) //index为1开始，返回一个
```
元素匹配($elemMatch) 常用于对象数组
```sh
friends = [{ name: 'xiaoming', age: 22 }, { name: 'xiaoyu', age: 23 }]
db.users.find({ friends: $elemMatch: { age: { $gt: 22 } } }) 
```

#### 取模($mod)
```sh
 read % 2 == 1 奇数
db.users.find({ age: { mod: [2, 1] } }) 
```

#### 取反
```sh
$not是元语句，即可以用在任何其他条件之上, 可以是正则或对象
db.users.find({ age: { $not: { $gt: 22} }) 
```
#### 存在($exists)
```sh
query = { age: { $exists: true } } 存在
query = { age: { $exists: false } } 不存在
db.users.find(query) 
```

#### 正则表达式($regex)
You cannot use $regex operator expressions inside an $in.
```sh
{ name: { $regex: /acme.*corp/i, $nin: [ 'acmeblahcorp' ] } }
{ name: { $regex: /acme.*corp/, $options: 'i', $nin: [ 'acmeblahcorp' ] } }
```

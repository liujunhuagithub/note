# 基本介绍

## 概念

1. MongoDB是跨平台的，基于分布式文件存储的数据库，由C++语言编写的NoSQL数据库。
2. 数据结构由键值(key:value)对组成，类似JSON。
3. table——collection(表)        raw——document(记录行)
4. 语法
   - 支持JavaScript语法和V8引擎，true和false和1/0可代替
   - 支持for循环和forEach
   - 支持地理排序及操作
5. 没有复杂的表连接，使用内存存储工作集，可以更快地访问数据，易于扩展
6. 不适于强事务业务
7. 

#数据类型

#基本命令

##数据库db级

| 命令                                                | 作用                                         |
| --------------------------------------------------- | -------------------------------------------- |
| show dbs                                            | 查询所有数据库                               |
| use 某db名称                                        | 选择某数据库，不存在自动创建                 |
| db.dropDatabase()                                   | 删除当前使用数据库                           |
| db.copyDatabase("源dbname", "目的dbname", "源地址") | 从指定的机器上复制指定数据库数据到某个数据库 |
| db.repairDatabase()                                 | 修复当前数据库                               |
| db.getName()                                        | 查看当前使用的数据库                         |
| db.stats()                                          | 显示当前db状态                               |
| db.getMongo()                                       | 当前db的链接机器地址                         |
|                                                     |                                              |

## 集合Collection级

| 命令                                   | 作用                                       |
| -------------------------------------- | ------------------------------------------ |
| show collections                       | 显示本db所有collection                     |
| db.createCollection(“集合名”, option); | option为{size: 20, capped: true, max: 100} |
| db.getCollectionNames();               | 获得本db下所有集合的数组                   |
| db.getCollection("name")               | 获得到单个的集合信息                       |
| db.user.drop()                         | 删除当前库下集合                           |
|                                        |                                            |

###option参数

1. capped(布尔)：是否创建固定集合(固定大小，==自动覆盖最早记录==)默认false；为true必须指定size。
2. autoindexid：是否在_id自动建立索引，默认false
3. size：最大字节数
4. max：最大document数

##文档document级

## 用户管理

| 命令                                   | 作用             |
| -------------------------------------- | ---------------- |
| show users;                            | 显示当前所有用户 |
| db.removeUser("userName");             | 删除用户         |
| db.addUser("用户名", "密码", 是否只读) | 添加用户         |
| db.auth("userName", "123123");         | 数据库认证       |
|                                        |                  |
|                                        |                  |



#CRUD(参数为js对象或json均可)

##修改

1. **insert**：db.集合名.insert(==js对象或数组(多条记录)==)       _id存在则报错

2. **save**：db.集合名.save(js对象)            _id存在则update原document

3. **update**：

   - db.集合名.(criteria条件,新值 ,无则新建默认false,支持多条修改默认false)    不使用修改器为整体替换

   - 修改器{符号：{field：value},{$...}}

     | 符号    | 含义       | 示例                       |
     | ------- | ---------- | -------------------------- |
     | $set    | 修改字段   | {$set:{age:100}}           |
     | $inc    | 自增       | {$inc:{age:100}}           |
     | $rename | 重命名字段 | {$rename:{age:"newfield"}} |
     | $unset  | 删除字段   | {$unset:{age:true}}        |

     

   - db.集合名.(criteria条件,{==$set==:{field:value}，$inc:[field:自增数字]}  ,{multi:true是否修改多行,默认否})  

4. **remove**：db.集合名.(criteria条件,是否仅删一条默认false)    无条件删除全部document

5. _id可指定，但不推荐

##查询 find(query,field,option)

1. 条件运算符   {field:{ 运算符: value}

   | 含义             | 标志符               | 示例                              |
   | ---------------- | -------------------- | --------------------------------- |
   | 相等=            | 无                   | {age:20}  //age=20                |
   | 大于>   大于等于 | $gt   $gte           | {age:{$gt:20}}  //age>20          |
   | 小于<   小于等于 | $lt   $lte           | {age:{$lt:20}}  //age<20          |
   | 不等于!=         | $ne                  | {age:{$ne:20}}  //age!=20         |
   | in  not in       | $in   $nin           | {age:{$in: [10,20,30] }}          |
   | 前%     后%      | /.../   ***无引号*** | {name:/xy/}    //name like '%xy%' |
   |                  |                      |                                   |

   

2. 条件拼接

   - AND：多条件拼接为对象或 $and 为key的数组

   - OR ：$or 为key的数组

     ```shell
     db.collectionName.find( {$and: [ {key1: value1}, {key2:value2} ] } )  //AND
     db.collectionName.find({key1: value1,  key2:value2}).pretty()
     
     db.collectionName.find( {$or: [ {key1: value1}, {key2:value2} ]  } )   //OR
     ```

     

3. 投影：find的第二个参数，返回指定列，默认所有列。指定某列为1或true，_id需手动设0或false才可取消

4. 其他

   - findOne查看第一条
   - distinct去重：distinct('字段名',{...查询条件criteria})
   - 排序sort:{字段名:1升序/-1降序}
   - count()总数
   - 偏移skip(num)
   - limit(num)                        limit和skip可以互换位置，结果相同

5. 6

   ```shell
   db.collectionName.find({},{_id:0,tit:true})	//投影select title from collectionName
   db.books.distinct("name",{"price":{$lt:30}})  //select distinct name from books
   db.qikegu.find({},{"title":1,_id:0}).skip(1).limit(1)  limit 1,1
   ```

## 统计查询 db.clooectionName.aggregate( [ {符号:{ 表达式} } ] 数组)

| 管道符号       | 含义                       | 示例                                                         |
| -------------- | -------------------------- | ------------------------------------------------------------ |
| $group         | 分组                       | {$group : {_id : "分组字段null不分组", count : {$sum : 每组统计字段}}} |
| $match         | filter过滤数据             | {$match : { field:"value"}}  与find参数一致                  |
| $project       | 投影指定列                 | {"$project":{"field":1,"_id":0}                              |
| $sort          | 排序                       | {$sort:{"field":-1}}                                         |
| $skip          | 跳过                       |                                                              |
| $limit         | 限制返回数量               | {$limit: 100}                                                |
| **表达式符号** | **计算当前聚合管道的文档** |                                                              |
| $sum $avg      |                            | $sum为1时类似select cound(*) from \* group by \*             |
| $min  $max     |                            |                                                              |

1. 管道作用：上个结果作为下个命令的参数，==有顺序，可重复==


#常用函数

| 函数            | 功能       | 示例                                |
| --------------- | ---------- | ----------------------------------- |
| print('...')    | 控制台输出 |                                     |
| tojson(object)  | 转为Json   | tojson(new Object('a'));            |
| forEach(函数名) | 迭代       | db.users.find().forEach(printjson); |
|                 |            |                                     |
|                 |            |                                     |
|                 |            |                                     |
|                 |            |                                     |
|                 |            |                                     |
|                 |            |                                     |

#整合SpringBoot（类似JPA）
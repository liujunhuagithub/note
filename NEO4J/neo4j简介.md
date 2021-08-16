# MATCH  (charlie:Person {name: 'Charlie Sheen'}),  (martin:Person {name: 'Martin Sheen'}),  p = shortestPath((charlie)-[*]-(martin)) WHERE none(r IN relationships(p) WHERE type(r) = 'FATHER') RETURN p图数据库

## NEO4J特点

1. 操作高效，原生图DB引擎，免索引邻接节点遍历
2. 图结构天然自然伸展，邻近查询始终有限局部数据
3. 对非结构化数据友好，模型设计灵活
4. 查询速度不因数据增大而减慢，避免RDBMS复杂连接查询
5. 适合关系繁杂的数据存储，适合迭代敏捷开发
6. 支持嵌入式、服务器、分布式高可用模式和实时备份
7. ==完全支持事务ACID特性==
8. 基于原生图，完全由==节点和有向边==构成
9. 使用场景：社交网络，地图，搜索引擎；不适合基于大数据的任务，二进制和大规模分析任务



# Neo4J结构

## Neo4J关键概念

### Node/label

1. 节点：存储实体信息，一个Node可拥有多个label 。 大驼峰(类名)

2. label表明节点类型，一个Node可拥有多个label。通常不同类型label有对应的properties

3. 返回格式：

   ```json
   {
     "identity": 45,  内部维护，不建议指定
     "labels":  "标签list",
     "properties": 属性Map
   }
   ```

   

### relationship/type

1. 关系：两个node间的联系(==有向边==)，一个relationship只能有一个type。下划线分割全部大写(常量)

2. type：关系的类型，可以有对应properties

3. 返回格式

   ```
   {
     "identity": 45,
     "start": 45,
     "end": 46,
     "type": "FRIEND",
     "properties": 属性Map
   }
   ```

   

### path

==节点和关系的交替序列==

###  property

属性：键值对Map。小驼峰(变量名)



## 事务

1. 完全支持ACID，默认隔离级别：READ_COMMITED 读已提交
2. ==一切操作(包括读)都在事务中进行。事务之外执行返回null==
3. 支持显式锁和死锁检测   set.\_lock=true    ..... remove x.\_lock。常用于for循环更新
4. 支持事务嵌套，顶层事务回滚则全部回滚

## 数据类型

- ### Property types

  1. Number——Integer、Float
  2. String
  3. Boolean
  4. Point空间
  5. DateTime/LocalDateTime/Duration等java新日期

- ### Structural types——不能作为属性Key存储，不能作为\$参数

  1. Node :         id,Label, Map (of properties)
  2. Relationship:      id,Type, Map (of properties),start/end
  3. path:  
  4. node、Relationship和path作为模式匹配的结果返回

- ### Composite types——不能作为属性Key存储

- ### NUll 使用 IS NULL /IS NOT NULL

## 约束

1. 支持设置索引     CREATE/DROP INDEX  :Label(Property)
2. 支持unique约束  CREATE/DROP  CONSTRANT ON (n:Label) ASSERT p.property IS UNIQUE

## 配置

1. Linux修改文件打开数限制，建议65535
2. 支持Blot 7687 http 7474 https 7473 neo4j 7687访问协议
3. 页面缓存：dbms.memory.pagecache.size=库大小(du -hc *stroe.db*)X0.012
4. 堆大小8~16GB，初始化大小和最大相同，避免频繁垃圾回收









# Cypher

MATCH (n) RETURN 

CASE n.eyes  WHEN 'blue'  THEN 1  WHEN 'brown' THEN 2  ELSE 3 END   /  CASE  WHEN predicate THEN result  [WHEN ...]  [ELSE default] END

AS result

## 操作符

where  n.属性名=/n:Type/Label     支持路径匹配

```cypher
MATCH
  (timothy:Person {name: 'Timothy'}),
  (other:Person)
WHERE other.name IN ['Andy', 'Peter'] AND (other)-->(timothy)
RETURN other.name, other.age
```

IN   针对list

UNWIND 扩展集合

DISTINCT

+, -, *, /, %, ^幂指

=, <>, <, >, <=, >=, IS NULL, IS NOT NULL      = 和 < > 运算符测试任何值都是 null    支持链式比较相当于and多个条件

STARTS WITH, ENDS WITH, CONTAINS   操作字符串

AND, OR, XOR, NOT

=~正则    +  字符串/list连接

.静态值    []动态(key是表达式计算结果，不参与预编译)值   =(原来属性被删除)  +=(更新)属性(操作数是map)

```cypher
WITH {name: 'Anne', age: 25} AS a
RETURN a[$myKey] AS result

MATCH (restaurant:Restaurant), (category:Category)
WHERE restaurant["rating_" + category.name] > 6
RETURN DISTINCT restaurant.name
```



## 第三方调用

#### \$Params占位符

以\$开头，占位符名和传入参数名保持一致，参数值在执行时提供。类似于预编译sql，加快查询速度

用于  ==表达式，属性值，Id(n)==

和预编译sql一样，==不能作为label/type/property字段名，只能作为值==

## 匹配模式

### 节点match匹配

```cypher
(a)--(b)      直接连接的点，可能出现重复的逆序匹配
(a)-[r:TYPE1|TYPE2]->(b)   或  
      
(a)-[*n]->(b)   路径长度为n————关系个数
(a)-[*n..m]->(b)   路径长度为[n,m]
(a)-[*n..]->(b)   路径长度为[n,无穷大]
(a)-[*..n]->(b)   路径长度为[1,n]
(a)-[*]->(b)   路径长度任意

p = (a)-[*3..5]->(b)    复制给path变量

```

### LIST

1. IN  用于判断是否在list中，将整个操作数作为整体

2. 支持倒序索引

3. 支持切片

   ```
   list[n..m]    [n,m)
   ```

   

4. range(0, 10) 全为闭区间。

5. 支持list表达式构造list

   ```
   RETURN [x IN range(0,10) WHERE x % 2 = 0 | x^3 ] AS result
   ```

   

### Map——Map < String，Object >

1. key为字符串，value可为任意值

2. projection映射——将处理结果映射为Map   

   ```
   map_var{element,..[..n]}
   
   map_var  引用的变量
   element
   	.key 属性选择器：引用map_var属性值
   	key: <expression>   :新增属性值
   	某变量：变量名为key，值为属性value
   	.*   引用所有map_var变量
   ```

## 语法关键字

### Write

1. UNWIND  .. AS .    展开列表list赋予新变量  支持嵌套、+拼接、去重distinct。可用于CREATE

   ```cypher
   WITH [[1, 2], [3, 4], 5] AS nested
   UNWIND nested AS x
   UNWIND x AS y
   RETURN y                            1   2  3  4  5
   
   WITH
     [1, 2] AS a,
     [3, 4] AS b
   UNWIND (a + b) AS x
   RETURN x                              1  2  3  4
   
   WITH [1, 1, 2, 2] AS coll
   UNWIND coll AS x
   WITH DISTINCT x
   RETURN collect(x) AS setOfVals          1 2
   
   UNWIND $events AS event                 创建节点
   MERGE (y:Year )
   MERGE (y)<-[:IN]-(e:Event)
   RETURN e.id AS x ORDER BY x
   ```

   

2. #### MERGE

   ```cypher
   MERGE (keanu:Person {name: 'Keanu Reeves'})
   ON CREATE
     SET keanu.created = timestamp()          首次创建才设置
   ON MATCH
     SET keanu.lastSeen = timestamp()          匹配到才设置
   RETURN keanu.name, keanu.created, keanu.lastSeen
   ```

   

3. 

### Read

1. RETRUEN  直接返回结果   WITH 作为下一句的输入，必须作为单个变量传入

2. AS 别名

3. 字符串匹配  STARTS WITH     ENDS WITH     CONTAINS

4. MATCH匹配

   ```cypher
   MATCH (user:User {name: 'Adam'})-[r1:FRIEND]-(friend)
   MATCH (friend)-[r2:FRIEND]-(friend_of_a_friend)
   RETURN friend_of_a_friend.name AS fofName     内连接
   
   MATCH
     (user:User {name: 'Adam'})-[r1:FRIEND]-(friend),
     (friend)-[r2:FRIEND]-(friend_of_a_friend)
   RETURN friend_of_a_friend.name AS fofName  一个匹配模式，两个路径，有共同节点，等同于以下
   
   MATCH (user:User {name: 'Adam'})-[r1:FRIEND]-()-[r2:FRIEND]-(friend_of_a_friend)
   RETURN friend_of_a_friend.name AS fofName
   ```

   

5. PROFILE   分析性能

### 子查询

1. exists

   ```cypher
   MATCH (person:Person)
   WHERE EXISTS {
     MATCH (person)-[:HAS_DOG]->(:Dog)
   }
   RETURN person.name AS name
   ```

   

2. 

### import数据

### 常用函数

count()  :统计数量

type(r)  ：关系名

id(n/r)：id值

nodes(path)：路径变量的所有节点       relationships(path )：路径变量的所有关系

shortestPath(path )：最短路径   allShortestPaths(path )：所有最短路径

谓词函数  none()：常用于筛选label/type类型

```cypher
MATCH
  (charlie:Person {name: 'Charlie Sheen'}),
  (martin:Person {name: 'Martin Sheen'}),
  p = shortestPath((charlie)-[*]-(martin))
WHERE none(r IN relationships(p) WHERE type(r) = 'FATHER')
RETURN p
```

collect

### 存储过程  CALL

1. 调用  CALL db.labels()   可不加括号
2. YIELD   筛选需要的类型

### 导入CSV文件

1. 支持本地导入   file:///  ，csv文件在*<NEO4J_HOME>/import/*下；支持https/http/ftp

2. LOAD CSV   自动遍历，每行记录‘，’分割为list。

3. 无头    *LOAD CSV FROM*

   ```cypher
   LOAD CSV FROM 'file:///artists.csv' AS line
   CREATE (:Artist {name: line[1], year: toInteger(line[2])})    索引从0开始
   ```

   

4. 有头。*LOAD CSV WITH HEADERS FROM*     表头名作为property访问

   ```cypher
   LOAD CSV WITH HEADERS FROM 'file:///artists-with-headers.csv' AS line
   CREATE (:Artist {name: line.Name, year: toInteger(line.Year)})
   ```

   

5. 导入大量数据  *USING PERIODIC COMMIT* 批处理，默认1000，减少事务开销

   ```cypher
   USING PERIODIC COMMIT  可省的批大小 LOAD CSV FROM 'file:///artists.csv' AS line
   CREATE (:Artist {name: line[1], year: toInteger(line[2])})
   ```

   

# APOC






# Neo4j导入方式

初次使用Neo4j图数据库时,我们可能需要将在关系数据库或其他存储形式的数据批量导入到Neo4j图数据库中.因此,这里我将介绍怎样使用Neo4j导入工具来批量导入数据.
目前,导入到Neo4j的工具有两种:
1. `load csv`指令.
2. `neo4j-import`工具.

<!-- more -->

然而,这两种工具都是基于CSV文件的,那么我们就需要先有CSV文件了.这里我不详细阐述如何获取CSV文件了.

# 关系数据库导出为CSV

1. 借助GUI数据库管理工具,比如`DataGrip`,`Navicat`等等工具,都有将表数据导出为CSV文件的功能.
2. 借助数据库查询语句导出指令.比如`mysql`的`into outfile <导出的目录和文件名>`.等等之类的了.

# CSV内容格式注意事项

# 使用Cypher create语句,为每一条数据写一个create

# 使用java api

# 使用Load CSV指令导入到Neo4j

以`权力的游戏`的数据作为测试数据
[下载地址](https://www.macalester.edu/~abeverid/data/stormofswords.csv)

```
Source,Target,Weight
Aemon,Grenn,5
Aemon,Samwell,31
Aerys,Jaime,18
Aerys,Robert,6
Aerys,Tyrion,5
Aerys,Tywin,8
Alliser,Mance,5
Amory,Oberyn,5
...
```

## Load CSV读取但不存入数据库

注意:从本地加载文件的时候,相对路径(相对于)
```
bash-4.4# pwd
/var/lib/neo4j/import
bash-4.4# ls
relation.csv       stormofswords.csv
```
查询
```
// 查看CSV文件行数
load csv from "file:///stormofswords.csv" as line return count(*)
```
输出
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/neo4j/import/import_read_csv.png)

```
// 查看CSV文件前5行
load csv from "file:///stormofswords.csv" as line
with line
return line 
limit 5;
```
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/neo4j/import/read_csv_top5.png)

```
// 查看CSV文件,并带有头部数据
load csv with headers from "file:///stormofswords.csv" as line
with line
return line 
limit 5;
```
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/neo4j/import/read_csv_with_headers.png)

这几个例子仅仅是用来读取CSV文件,它并没有将数据存入到neo4j的数据库中.

## LOAD CSV指令用法

`load csv from "file-url" as line` 
1. 这条指令就是将指定路径下的CSV文件读取出来
2. 其中的"file-url"就是文件的地址,可以是本地文件路径也可以是网址,只要能从地址中读取到CSV文件即可.
3. 因此也可以这样写.


```
 LOAD CSV FROM "https://www.macalester.edu/~abeverid/data/stormofswords.csv" as line
 return line
```
这样就可以读取网址指定的movie.csv文件.
或者可以使用本地文件路径.
```
// 由于配置文件
conf/neo4j.conf
这个配置项
dbms.directories.import=import
```
所以读取本地文件的时候,我们需要把文件放到import目录下

## 导入CSV时附带表头

我们将使用一个简单的数据模型`(:Character {name})-[:INTERACTS {weight}]->(:Character {name})`。带有标签的节点Character表示文本中的角色，单个关系类型INTERACTS从该角色连接到另一个文本中交互的角色。我们将把角色名作为node属性name存储，把两个角色之间的交互数作为relationships属性weight存储。

首先，我们必须创建一个约束来断言我们架构的完整性：
```
CREATE CONSTRAINT ON (c:Character) ASSERT c.name IS UNIQUE;
```

一旦创建了约束（该语句也将构建一个索引，它将提高通过角色名查找的性能），我们可以使用Cypher的LOAD CSV语句来导入数据：

头部由`row.Source`,`tow.Target`指定
```
LOAD CSV WITH HEADERS FROM "file:///stormofswords.csv" AS row
MERGE (src:Character {name: row.Source})
MERGE (tgt:Character {name: row.Target})
MERGE (src)-[r:INTERACTS]->(tgt)
ON CREATE SET r.weight = toInt(row.Weight)
return *
```
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/neo4j/import/stormofswords.png)

## 导入CSV大文件
如果要导入包含大量数据的CSV文件,则可以使用`PERIODIC COMMIT字句`.

1. 使用`PERIODIC COMMIT`指示neo4j在执行完一定行数后提交数据再继续,这样减少了内存开销.
2. 使用`PERIODIC COMMIT`默认值为1000行,因此数据将每一千行提交一次.
3. 要使用`PERIODIC COMMIT`,只需要在LOAD CSV 语句之前插入`USING PERIODIC COMMIT`语句

```
USING PERIODIC COMMIT
LOAD CSV WITH HEADERS FROM "file:///stormofswords.csv" AS row
MERGE (src:Character {name: row.Source})
MERGE (tgt:Character {name: row.Target})
MERGE (src)-[r:INTERACTS]->(tgt)
ON CREATE SET r.weight = toInt(row.Weight)
```

可以让其每800行提交一次,如下所示.
```
USING PERIODIC COMMIT 800
LOAD CSV WITH HEADERS FROM "file:///stormofswords.csv" AS row
MERGE (src:Character {name: row.Source})
MERGE (tgt:Character {name: row.Target})
MERGE (src)-[r:INTERACTS]->(tgt)
ON CREATE SET r.weight = toInt(row.Weight)
```

# 使用neo4j-import 工具导入到Neo4j

从neo4j 2.2版本开始,系统就自带了一个大数据量的导入工具`neo4j-import`,可支持并行,可扩展的大规模CSV数据导入.
```
bash-4.4# pwd
/var/lib/neo4j/bin
bash-4.4# ls
cypher-shell  neo4j         neo4j-admin   neo4j-import  tools
```

## neo4j-import用法
然而却提示neo4j-import过时了,推荐用新的neo4j-admin了.

```
bash-4.4# bin/neo4j-import
WARNING: neo4j-import is deprecated and support for it will be removed in a future
version of Neo4j; please use neo4j-admin import instead.
Neo4j Import Tool
	neo4j-import is used to create a new Neo4j database from data in CSV files. See
	the chapter "Import Tool" in the Neo4j Manual for details on the CSV file format
	- a special kind of header is required.
```
看了一下neo4j-admin的帮助文档,其实import功能只是一个子集罢了.admin里面包含了更多的功能.

## 1. phones.csv 记录电话号列表，作为nodes结点

![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/neo4j/import/phone_nums.png)

## 2. phone_header.csv 标题文件只有一行数据
```
phone:ID
```


## 3. call.csv 该文件记录通话记录的信息，作为以后关系的建立和关系属性的添加
![](https://raw.githubusercontent.com/Timehsw/gitnote-images/master/neo4j/import/phone_call.png)

## 4. call_header.csv 通话记录头信息
```
:START_ID,:END_ID,duration,num,average
```

## 5. CSV文件准备好了,开始执行导入
注意:执行之前,要打开conf/neo4j.conf文件,将#dbms.directories.import=import注释.然后文件写绝对路径.不然容易报错
```
neo4j-admin import \
	--database=graph_demo.db \
	--nodes:phone="/var/lib/neo4j/import/phone_header.csv,phones.csv" \
	--ignore-duplicate-nodes=true \
	--ignore-missing-nodes=true \
	--relationships:call="/var/lib/neo4j/import/call_header.csv,call.csv"
```
+ 这里以防我们新建的数据库已经存在，我们选择删除已有库再进行导入
+ 记得要先关闭neo4j

+ --nodes子句开头的CSV文件是节点CSV文件
+ --relationships开头的是关系CSV文件
+ --into子句指明了导入的neo4j数据库名称,其中不能包含现有数据库
+ --id-type子句指明了生成节点,关系的主键类型为string类型

注意:由于neo4j-import工具不能使用Cypher语句创建节点,关系,所以需要为节点和关系分别提供不同的csv文件